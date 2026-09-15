# Section 031: Server Write and Ingestion Model - Research Report

**Topic ID:** IDR-SRV-031<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-031 Server Write and Ingestion Model](../IDR%20Plans/idr-srv-031-server-write-and-ingestion-model.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** Five core questions and all detailed questions concerning write surfaces, entry points, acceptance, processing, validation, source authority, transactions, persistence, errors, operations, verification, and handoff<br>
**Methodology Used:** Authority-ranked extraction from approved OGC API - Connected Systems Parts 1 and 2, their pinned editor artifacts and draft OGC API - Features CRUD dependency, HTTP specifications, accepted IDR-SRV-001 through IDR-SRV-030 findings, and bounded implementation evidence; followed by resource-operation, entry-point, responsibility, transaction, failure, and verification mapping<br>
**Research Time:** Approximately 17 hours of AI-assisted research and synthesis, September 14, 2026<br>
**Primary Source(s):**
- [OGC API - Connected Systems - Part 1: Feature Resources](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2: Dynamic Data](https://docs.ogc.org/is/23-002/23-002.html)
- [Pinned OGC API - Connected Systems 1.0 editor source](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)
- [Pinned OGC API - Features draft CRUD source](https://github.com/opengeospatial/ogcapi-features/tree/4e30324a14b682ff4a26ee43aad1eb6428c846a3/extensions/transactions/create-replace-update-delete)
**Supporting Resources:**
- [IDR-SRV-023 Schema and Encoding Validation Strategy](idr-srv-023-schema-and-encoding-validation-strategy-report.md)
- [IDR-SRV-029 Transaction, Consistency, Idempotency, and Concurrency Strategy](idr-srv-029-transaction-consistency-idempotency-and-concurrency-strategy-report.md)
- [IDR-SRV-030 Data Lifecycle, Retention, Archival, and Deletion Strategy](idr-srv-030-data-lifecycle-retention-archival-and-deletion-strategy-report.md)
**Document Purpose:** Establish the decision-ready authoritative Glaux Server mutation and ingestion boundary before publisher, simulator, dynamic-data, security, DDIL, conformance, and implementation specialization<br>
**Author(s):** OpenAI Codex, for the Glaux Project<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** September 14, 2026<br>
**Date:** September 14, 2026<br>
**Last Updated:** September 14, 2026

---

This report labels evidence as **[N] normative standard**, **[A] accepted project baseline**, **[P] project recommendation/decision proposed for acceptance**, **[I] informative implementation evidence**, and **[X] documented conflict or gap**. A combined label states that the conclusion joins those classes; it does not elevate informative evidence into a requirement.

## Table of Contents

1. Executive Summary
2. Scope, Terminology, and Source-Authority Statement
3. Standards and Profile Write-Obligation Baseline
4. Resource and Operation Inventory
5. Entry-Point Taxonomy
6. Accept/Delegate/Reject Decision Matrix
7. Common Ingestion Processing and State-Transition Model
8. Validation and Normalization Responsibility Matrix
9. Identity, Trust-Input, Provenance, and Source-Authority Findings
10. Transaction, Idempotency, Concurrency, Ordering, and Replay Findings
11. Persistence, Latest-State, Event/Outbox, and Audit Implications
12. Error, Retry, Backpressure, and Operational Behavior
13. Security, Policy, DDIL, and Deployment Implications
14. Verification Scenarios and Conformance Boundary
15. Downstream Handoff Matrix
16. Recommendations and Explicit Project Decisions
17. Risks, Contradictions, Assumptions, and Unresolved Questions
18. Validation Against This Plan's Success Criteria
19. References

---

## 1. Executive Summary

Glaux Server should expose one authoritative mutation boundary with multiple controlled adapters, not multiple write models. Public CSAPI HTTP, a future publisher contract, simulator data, broker consumers, federation, file import, and internal services may have different transport and trust characteristics, but every accepted unit must become the same typed resource command and traverse the same applicable identity, contract, validation, authorization, policy, concurrency, transaction, provenance, persistence, event/outbox, audit, and response obligations. No adapter may write canonical tables, latest-state projections, or broker topics directly. **[A/P]**

The approved CSAPI standards make write support conformance-class-dependent. Their Create/Replace/Delete and Update classes require writes for Systems, Deployments, Procedures, Sampling Features, Properties, DataStreams, Observations, ControlStreams, Commands, CommandStatus, CommandResult, Feasibility, FeasibilityStatus, FeasibilityResult, and SystemEvent resources. Part 1 also defines nested subsystem/subdeployment creation and collection-membership mutation. Part 2 requires parent-schema validation and protected stream-schema behavior. These requirements invoke the draft OGC API - Features CRUD work, whose reviewed snapshot is explicitly not an approved OGC Standard. The current draft supplies useful method, OPTIONS, status, and conditional-request rules but does not resolve every CSAPI PATCH or asynchronous-outcome question. **[N/X]**

The recommended initial Glaux contract is deliberately strict:

- advertise and accept only implemented operation/representation combinations from the contract registry;
- use POST for creation and do not support create-by-PUT initially;
- support JSON Merge Patch only on a registered JSON writable projection, apply it atomically, reject identifier/parent/contract mutations, and validate the complete resulting resource; do not PATCH SWE Text/Binary value streams;
- require strong `If-Match` for mutable PUT/PATCH/DELETE, with `428` when absent and `412` when false;
- return `201` plus `Location` for immediately committed creation, `200` or `204` for immediate replacement/update/deletion, and never return a bare `202` without an authorized, durable status resource;
- treat synchronous Command/Feasibility output as a domain status report and asynchronous progress as durable status resources; transport acceptance is never retroactive proof of execution success;
- reject invalid public writes atomically; quarantine only authorized administrative/import material outside active authority; and permit `pending-dependency` staging only under an explicit trusted-source contract;
- default multi-item ingestion to an independently atomic item contract with complete per-item results; do not claim standard atomic-batch conformance; and
- commit canonical state, revision, provenance, validation evidence, current-state selection inputs, domain event, outbox, audit reference, and idempotency/inbox state together where applicable. External publication occurs after commit and cannot roll back the accepted mutation. **[A/P]**

Publisher mechanics belong to IDR-SRV-032, simulator controls to IDR-SRV-033, dynamic semantics to IDR-SRV-034 through 038, security policy to IDR-SRV-039 through 041, and DDIL/reconciliation to IDR-SRV-042/043. This report fixes the boundary those topics must preserve; it does not design those components or authorize draft Part 3 Publish/Subscribe implementation.

## 2. Scope, Terminology, and Source-Authority Statement

### 2.1 Scope

In scope are standards-facing resource mutations, dynamic ingestion, metadata/document submission, Command and Feasibility status/result writes, batches and replay, administrative import, publisher/simulator/broker/federation mediation, common processing, responsibility allocation, transaction and failure boundaries, provenance, operational responses, verification, and downstream ownership. Read/query behavior is included only where it exposes status, validators, committed results, or capability discovery.

Out of scope are final publisher authentication/delivery design, simulator reset/time/scenario APIs, stream semantics and watermarks, detailed command safety/lifecycle, security label syntax and cross-domain policy, DDIL reconciliation protocol, physical database design, dependency selection, and server implementation.

### 2.2 Precise terms

| Term | Meaning in this report |
|---|---|
| Received | Transport envelope was admitted far enough to assign a correlation/receipt identity; no validity or persistence claim |
| Decoded | Bounded parser produced a representation; not yet semantically valid |
| Validated | All checks selected for the declared unit have passed under recorded contract/policy versions |
| Accepted | Glaux authorized the intended canonical effect and entered its commit path; for asynchronous domain work this does not mean completed |
| Committed | The authoritative local transaction durably recorded its complete declared effect |
| Published | An outbox consumer delivered a committed event/message to an external channel; later than and separate from commit |
| Quarantined | Bytes/evidence are isolated and non-authoritative; they cannot satisfy references, appear in normal results, or execute |
| Pending dependency | Trusted, bounded staging state under an explicit integration contract; not canonical visibility or semantic acceptance |
| Delegated | A caller-side adapter must transform or enrich source-native material before Glaux admission, or another bounded component performs post-commit work |
| Rejected | No requested canonical state effect occurred; safe evidence may still be retained under policy |
| Atomic unit | Smallest advertised group that either commits all canonical effects or none |
| Source authority | Recorded authority for a claim; it is not automatically identity trust, authorization, validation, priority, or truth |

### 2.3 Source hierarchy and reproducibility

Approved standards control conformance; accepted project reports control Glaux design; current editor repositories explain pinned artifacts and gaps; implementations supply non-normative feasibility evidence. The following exact source states were used on 2026-09-14:

| Source | Status/version | Pin or stable anchor | Authority and limitation |
|---|---|---|---|
| CSAPI Part 1 | OGC 23-001, 1.0, approved 2025-07-16 | Official publication; editor tag `v1.0.0` commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2` | Normative text and ATS; published artifacts contain recorded gaps |
| CSAPI Part 2 | OGC 23-002, 1.0, approved 2025-07-16 | Same editor tag/commit | Normative text and ATS; Feasibility/PATCH/OAS inconsistencies remain |
| OGC API - Features CRUD | `20-002r2`, `1.0.0-SNAPSHOT`, Draft | `4e30324a14b682ff4a26ee43aad1eb6428c846a3`, 2026-09-11 | Draft normative dependency where CSAPI invokes it; not independently claimed as an approved Standard[^1] |
| OGC atomic/batch proposal | `26-018`, `1.0.0-draft` | Same repository commit, `proposals/atomic-batch-tx` | Informative only; no Glaux conformance claim |
| HTTP Semantics | RFC 9110 | Published RFC | Normative HTTP semantics[^2] |
| Problem Details | RFC 9457 | Published RFC | Normative problem representation[^3] |
| Accepted IDR-SRV-001–030 | Repository main through IDR-SRV-030 acceptance | Linked reports | Controlling Glaux baseline; later topics retain assigned decisions |

Project-controlled STANAG 4789/AEP-4789 material was considered only through accepted, releasable prior-IDR findings. This report does not reproduce controlled text or claim access beyond the established project evidence. **[A]**

## 3. Standards and Profile Write-Obligation Baseline

CSAPI Parts 1 and 2 each define optional Create/Replace/Delete and Update requirements classes. Glaux is not required to claim those classes merely because it implements read operations. If Glaux claims a class, it must implement every applicable requirement for each enabled dependent resource class and advertise only actual capabilities. OPTIONS, `Allow`, `Accept-Post`, and `Accept-Patch` therefore come from the deployment contract registry, not a static global method list. **[N/P]**

The draft CRUD dependency requires mutable endpoints to support one or more of POST, PUT, PATCH, and DELETE; immediate POST success uses `201` and `Location`, immediate PUT/PATCH/DELETE uses the specified `200`/`204` family, and deferred work may use `202`. The draft permits—but does not provide a universal status-resource contract for—deferred processing. Glaux may use `202` only where its own advertised operation resource provides durable state, result, cancellation where allowed, and retry-safe correlation. **[N/X/P]**

The accepted Glaux profile adds stronger safeguards than the draft's optional locking class: existing mutable resources require strong `If-Match` on PUT/PATCH/DELETE; POST and unsafe/replayed operations use a scoped idempotency identity; and no lossy cross-representation replacement is allowed. **[A]**

### 3.1 Method and response baseline

| Operation | Initial Glaux contract | Successful immediate response | Deferred response | Required controls |
|---|---|---|---|---|
| POST create/append | Enabled only on registered collection/nested endpoints | `201` + canonical `Location`; synchronous Command/Feasibility returns its domain status representation under the registered operation contract | `202` only with durable status URI and replay contract | exact `Content-Type`; authorization; idempotency key/source identity where replay-prone |
| PUT replace | Existing resource only; no create-by-PUT | `200` with selected representation or `204` | Only with durable operation resource | exact full representation; strong `If-Match`; URL identity controls |
| PATCH update | Registered JSON writable projection using `application/merge-patch+json`; no SWE Text/Binary patch | `200` or `204` | Only with durable operation resource | strong `If-Match`; immutable field protection; whole-result validation |
| DELETE | Registered resource endpoint, including specified cascade semantics | `200` or `204` | Only with durable operation resource | strong `If-Match`; authorization/hold/ownership impact graph |
| Membership mutation | Part 1 collection contract, distinct from resource deletion | `201`/`204` as registered | Not initially deferred | exact canonical URI/UID syntax; collection/resource authorization |
| Administrative batch/import | Non-standard job endpoint/tooling | `201`/`202` job plus per-item results | Expected for large work | source contract, manifest, limits, item idempotency, quarantine policy |

JSON Merge Patch is selected because it has stable HTTP media semantics and the reviewed draft defines it for eligible feature projections.[^4] This is a Glaux compatibility profile, not an assertion that CSAPI mandates that media type for every resource. `null` means removal under RFC 7396 and arrays are replaced as values, so required members, identities, parent links, schema bindings, and other protected fields are rejected if removal/change would violate the registered writable projection. **[N/X/P]**

## 4. Resource and Operation Inventory

The table inventories normative write exposure if Glaux claims the relevant CSAPI class. “CRD” means create, replace, delete; “U” means PATCH update. Nested and canonical routes remain those of the controlling standard and generated Glaux contract; this report does not repair route defects by inventing public aliases.

| Resource/data family | Standards operation baseline | Mutation character | Principal constraint or gap |
|---|---|---|---|
| System | CRD + U; nested subsystem POST | Versioned metadata/graph aggregate | Default delete with children/Deployment relation is blocked; authorized cascade follows Part 1 impact rules |
| Deployment | CRD + U; nested subdeployment POST | Versioned association/feature aggregate | Deleting association must not silently delete foreign-owned Systems |
| Procedure | CRD + U | Versioned document/semantic resource | SensorML/GeoJSON projections must preserve exact source and meaning |
| Sampling Feature | CRD + U; nested creation where specified | Versioned feature/relationship resource | Geometry/profile and canonical identity rules apply |
| Property | CRD + U | Versioned semantic resource | Published Part 1 condition/link gap requires traceability overlay |
| Custom collection membership | POST list of same-API canonical URI/UID; delete membership via applicable route | Association mutation, not canonical resource delete | One item per line; collection and target authorization distinct |
| DataStream | CRD + U | Versioned contract/stream definition | Once Observations exist, schema replacement/update is rejected with `409`; nonempty delete defaults `409`, cascade explicit |
| Observation | CRD + U | Usually append plus correction/version, not arrival-order overwrite | CREATE/REPLACE violating exact parent DataStream schema is `400` |
| ControlStream | CRD + U | Versioned command contract | Once Commands exist, schema replacement/update is `409`; nonempty delete defaults `409`, cascade explicit |
| Command | CRD + U | Durable domain action request and revision/status history | Parent schema failure `400`; cancellation is CANCELED CommandStatus, not DELETE |
| CommandStatus | CRD + U | Append-oriented domain evidence; correction is new version | State-machine/source authority owned by IDR-SRV-036 |
| CommandResult | CRD + U | Append/version result evidence | Inline result validates against exact ControlStream result schema |
| Feasibility | CRD + U | Domain analysis request, synchronous or asynchronous | Negative result is valid domain output, not HTTP validation failure |
| FeasibilityStatus | CRD + U | Append-oriented analysis status evidence | Published wording contains CommandStatus copy errors; typed Glaux overlay required |
| FeasibilityResult | CRD + U | Append/version analysis evidence | Does not reserve capacity or authorize a Command |
| SystemEvent | CRD + U | Append-oriented event evidence | Event identity, cause/source, revision and disclosure policy required |

Part 2 JSON, SWE JSON, SWE Text, and SWE Binary requirements define representation support by applicable resource and conformance class. Initial Glaux media support follows the accepted IDR-SRV-012 registry: canonical CSAPI `application/swe+json`, `application/swe+text`, and `application/swe+binary` tokens with documented vendor aliases over the same codec where supported; exact `Content-Type`; no sniffing; and no claim for a codec that has not passed capability and corpus tests. **[N/A]**

## 5. Entry-Point Taxonomy

| Entry point | Intended caller | Public/conformance status | Permitted input | Boundary rule |
|---|---|---|---|---|
| Public CSAPI HTTP | External authorized clients | Standards-facing | Registered CSAPI/SensorML/SWE representations only | Full strict pipeline; no quarantine-as-success or hidden enrichment |
| Publisher integration | Glaux Publisher or approved source gateway | Private integration contract; IDR-SRV-032 | Prefer the same CSAPI operations; optional typed envelope only for source/idempotency/ordering metadata | Envelope maps one-to-one to canonical commands; never bypasses validation/policy/transaction |
| Broker consumer | Approved broker adapter | Disabled until explicit contract/profile | Registered message envelope carrying exact representation and immutable message identity | Broker authentication is not end-user authorization; no generic topic-to-table bridge |
| Internal adapter interface | In-process/out-of-process source adapter | Private | Typed canonical resource command plus transformation manifest | No persistence port exposed; adapter cannot mint trusted provenance about itself |
| File/batch import | Authorized operator/service | Administrative | Manifested bundle of supported exact representations | Stage, inventory, scan, then promote each declared atomic unit through same pipeline |
| Federation pull/sync | Trusted peer adapter | Private/DDIL contract | Source revisions, tombstones, contracts, evidence, causal metadata | Never overwrite by arrival order; preserve peer/source authority and conflicts |
| Administrative tool | Privileged operator/service | Administrative | Policy/configured resource commands and lifecycle operations | Privilege does not waive structural/semantic/policy validation; stronger audit required |
| Simulator data | Glaux Simulator | Test/development integration | Ordinary standards-aligned data through normal write path | Synthetic provenance mandatory; cannot exercise reset/time/scenario controls through public CSAPI |
| Simulator control | Test harness/operator | Test-only admin namespace; IDR-SRV-033 | Scenario/reset/clock/fault commands | Separate deployment capability; cannot be advertised as CSAPI or enabled accidentally in production |
| Internal service call | Glaux application component | Private | Same typed application command | Carries established principal/service and correlation; no “trusted internal” bypass |

A private high-throughput publisher endpoint is justified only if IDR-SRV-032 benchmarks demonstrate that the standards-facing route cannot meet a stated objective and the endpoint still invokes the same application command and evidence transaction. Its existence must not appear in the CSAPI conformance declaration unless a controlling standard defines it. **[P]**

## 6. Accept/Delegate/Reject Decision Matrix

This is the plan-required write-surface matrix. `A` = accept through the common pipeline, `D` = delegate transformation/enrichment before admission or post-commit delivery after it, `R` = reject at that entry point, `Q` = authorized administrative quarantine/staging only. Compact verification IDs resolve in Section 14.

| Resource/data class | Operation | Caller/entry point | Controlling clause/project source | Normative status | Request representation | Identity/source prerequisite | Validation class | Transaction unit | Result/response | Accept/delegate/reject disposition | Provenance requirement | Downstream side effects | Error behavior | Verification scenario | Unresolved issue |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| System/Deployment/Procedure/SamplingFeature/Property | POST create; nested create where defined | Public CSAPI; publisher/simulator may reuse | P1 CRD Reqs 60–71; IDR-012/023/029 | N/A | Registered GeoJSON or SensorML JSON projection | Authenticated principal; source scope; server ID or accepted source ID policy | Syntax, structure, profile, semantic, relationship, policy | One aggregate + memberships/evidence | 201 + Location after commit | A canonical; D legacy/source-native; R unsupported media | principal, source, original/digest, transform, contract/policy | revision, graph, event/outbox, audit | 400/401/403/404/409/415/422; no partial graph | V1, V3, V7 | Final client-ID allowlist belongs implementation profile |
| Same descriptive resources | PUT/PATCH/DELETE | Public CSAPI/private callers | P1 CRD 60–70, Update 72–76; IDR-029/030 | N/A/P | Full registered JSON for PUT; merge patch for PATCH | Target identity; strong If-Match; ownership/hold authority | Full post-operation validation; concurrency; cascade impact | One aggregate/impact graph | 200/204; new ETag if representation returned | A registered combinations; R PUT-create, stale, lossy, unauthorized cascade | before/after revision, actor, impact, transform | revision/current pointer, invalidation, event/outbox/audit | 404/409/412/415/422/428 | V2, V4, V8 | Part 1 Property condition and draft PATCH gaps |
| Collection membership | POST add / delete membership | Public CSAPI | P1 CRD Reqs 68–71 | N | `text/uri-list` with one canonical URL/UID per line | Collection and target visible/authorized; same API | Syntax, identity, relationship, policy | One membership set per request | Registered 201/204 and links | A valid same-API references; R foreign/hidden/malformed | actor, source list/digest, collection/target revisions | membership event/outbox/audit | 400/403/404/409/415 | V3 | Exact remove route/status must match generated contract |
| DataStream/ControlStream definition | POST/PUT/PATCH/DELETE | Public CSAPI; admin | P2 CRD 63–65, 68–70; Update 79–80, 83–84 | N/A | Registered JSON/GeoJSON contract representation | Parent System; exact schema artifacts; If-Match for mutation | Contract closure, semantic binding, relationship, state, policy | Stream aggregate + immutable contract revision | 201+Location or 200/204 | A valid; R populated schema change/nonempty delete absent cascade | schema/source fingerprints, compiler/validator versions | contract cache activation, event/outbox/audit | 400/404/409/412/415/422/428 | V4, V5 | Part 2 PATCH prose omits `409`; ATS supplies it |
| Observation | POST append; PUT/PATCH correction; DELETE | Public CSAPI; publisher/simulator/broker/federation adapter | P2 CRD 66–67; Update 81–82; IDR-018/023/029 | N/A | JSON/SWE JSON/Text/Binary only as registered for operation | Parent DataStream exact revision; source/message identity | Bounded codec, parent schema, time, unit/property, relationship, policy | One advertised record/request unit; item-atomic batch default | 201+Location or 200/204; replay returns original result | A canonical; D source protocol/semantic conversion; Q trusted invalid import only; R public invalid | source, ingest path, exact contract, original/digests, simulation flag | history append, deterministic latest candidate, event/outbox/audit | Explicit P2 schema failure 400; otherwise 409/412/413/415/422/428/429/503 | V5, V6, V9, V10 | Watermarks/current selector owned 034–035 |
| Command | POST submit; PUT/PATCH revision; DELETE resource only where allowed | Public CSAPI; approved task adapter | P2 CRD 71–72; Update 85–86; IDR-013/023/029 | N/A | JSON or registered SWE representation against ControlStream | Principal and source; exact ControlStream; idempotency; safety context | Full command schema, temporal/state, authorization, releasability, safety gate | Command + initial status/evidence/outbox atomically; dispatch post-commit | Synchronous domain status or durable async status link; never claim execution at receipt | A only after durable acceptance; D device/source adaptation; R invalid/unsafe/unauthorized | actor/source, policy/safety decisions, contract, dispatch correlation | status history, command outbox, audit; adapter dispatch | Parent-schema failure 400; domain REJECTED/FAILED after durable acceptance is status, not retroactive HTTP error | V11, V12 | Detailed lifecycle/safety 036–038 |
| CommandStatus/CommandResult | POST append; PUT/PATCH/DELETE only under registered authority | Device/command adapter; tightly authorized admin | P2 CRD 73–74; Update 87–88 | N/A | JSON; inline result per exact result schema | Command identity; reporting-source authority; monotonic evidence identity | Structure, state transition, result contract, time, source/policy | One status/result evidence record + current derivation inputs | 201+Location/200/204 | A authorized reporters; Q malformed trusted device evidence; R ordinary public/untrusted writes | device/adapter/source, command revision, evidence time, original/digest | append status/result, derive current, event/outbox/audit | 400/403/404/409/412/415/422/428 | V12, V13 | Reporter policy and transitions 036–038/039 |
| Feasibility | POST submit; registered mutation | Public CSAPI/task adapter | P2 CRD 75; Update 89 | N/X/A | Same parameter encoding class as corresponding command | Principal/source; exact ControlStream feasibility contract; idempotency | Input validity distinct from evaluation outcome | Request + initial status/evidence atomically | Synchronous feasibility status/result or durable async status link | A valid analysis request; R invalid/unauthorized; negative/indeterminate result remains valid domain output | input fingerprint, source, evaluator/version, contract/policy | status/result work outbox and audit | Validation HTTP errors before acceptance; later REJECTED/FAILED as domain status | V11, V14 | Published route/wording/OAS defects require overlay |
| FeasibilityStatus/Result | POST append; guarded correction/delete | Feasibility evaluator/admin | P2 CRD 76–77; Update 90–91 | N/X/A | JSON and registered result representation | Feasibility identity; evaluator authority | Transition, result schema, freshness/expiry, policy | One evidence record | 201+Location/200/204 | A authorized evaluator; Q malformed evidence if policy; R external arbitrary write | evaluator, algorithm/version, source inputs, request fingerprint | history/current view, event/outbox/audit | 400/403/404/409/412/415/422/428 | V13, V14 | Detailed async/expiry semantics 037 |
| SystemEvent | POST append; guarded correction/delete | Authorized server service, source adapter, admin | P2 CRD 78; Update 92; IDR-019/023 | N/A | Registered JSON event | Event/source identity; affected resource/revision visible to policy engine | Vocabulary, causal/reference, time, authority, disclosure | One event record | 201+Location/200/204 | A authorized generators/importers; D source event mapping; R fabricated/unattributed | source/actor, cause, affected revision, generating rule | event history/outbox/audit; never recursive unbounded generation | 400/403/404/409/412/415/422/428 | V7, V13 | Event-generation matrix remains implementation artifact |
| Metadata/document bundle | Administrative import | Import tool/admin | IDR-021–024, 030 | A/P | Manifest + exact supported artifacts | Registered source; package/digest/signature where applicable | Inventory, malware/limits, closure, schema/profile/semantic/policy | Staging job; promotion item/aggregate is explicit | 201/202 job + per-item status | Q first; A only by explicit revalidation/promotion; D legacy transform; R transport threat | exact bytes/digests, manifest, custody, transformation/promotion | artifact registry/cache after promotion, event/outbox/audit | 400/403/409/413/415/422; safe job diagnostics | V7, V15 | Package signing/trust policy 039–044 |
| Multi-record file/batch/replay | Stage, append, promote, or replay declared items | Publisher/import/federation/simulator | IDR-029/030; OAF batch proposal informative | A/P/I | Manifested list/stream of registered item representations | Batch/source/message IDs, declared ordering and atomicity | Per-item full pipeline plus manifest/sequence/replay controls | Item atomic by default; bounded all-or-none job only when explicitly advertised | Durable job/per-item committed, duplicate, failed, pending results | A valid items under contract; Q authorized invalid items; R undeclared partial semantics | batch/source IDs, offsets, item fingerprints, original/digest | per-item events/outbox/audit; job summary | Never return undifferentiated success; deterministic ambiguous-commit recovery | V6, V9, V15 | No current standard batch conformance claim |
| Source-native/legacy payload | Transform, then submit a canonical operation | Publisher/adapter/broker | IDR-014A–C/023 | I/P | Source-specific | Registered adapter/source and transformation version | Pre-admission source checks; canonical checks occur after transformation | None until transformed canonical command enters server | Adapter result, then ordinary server response | D; R if sent directly to public CSAPI | source exact bytes/digest, transformation manifest, adapter version | Only canonical acceptance creates state/events | Unsupported media/route 415/404; adapter errors outside CSAPI claim | V7 | Publisher contract 032 |
| Simulator reset/scenario/clock/fault | Reset, seed, clock, scenario, or fault-control action | Test operator/harness | IDR-033 handoff | P | Test-control command | Explicit test deployment and admin authorization | Control schema, deployment guard, policy, audit | Scenario transaction defined by 033 | Durable admin operation status | D to simulator/admin control plane; R at public CSAPI and production-disabled deployment | actor, scenario/build/seed, affected scope | controlled reset/reseed/fault effects | 404 when capability absent; 403 when forbidden; never masquerade as data write | V16 | Exact semantics 033 |

## 7. Common Ingestion Processing and State-Transition Model

### 7.1 Mandatory logical pipeline

Every accepted mutation follows these logical stages; implementations may fuse stages only if order, evidence, failure behavior, and policy seams remain testable.

1. **Admission and receipt:** bound request/header/body/decompression work; establish correlation, channel, remote/service context, and received time.
2. **Authentication context:** validate transport/service/end-user identity as applicable; do not infer source authority from channel alone.
3. **Route, method, media, and capability selection:** select a single registered operation, request codec, profile, and response contract.
4. **Bounded decoding:** parse exact `Content-Type`/content coding; reject ambiguity, trailing data, limit violations, or unavailable codecs.
5. **Identity and source resolution:** resolve server/client/source IDs, parent/resource scope, message/idempotency identity, source registration, and transformation chain.
6. **Structural/profile validation:** validate the declared representation and applicable CSAPI/AEP/Glaux writable projection.
7. **Reference, semantic, temporal, geospatial, and contract validation:** use exact parent/contract fingerprints; never fetch uncontrolled remote references.
8. **Authorization, policy, releasability, lifecycle, and safety decisions:** preserve non-oracular ordering and internal evidence.
9. **Deterministic normalization:** apply only registered semantics-preserving transforms and record before/after evidence.
10. **Idempotency, precondition, concurrency, and ordering checks:** compare operation fingerprint, strong ETag, source sequence/offset, causal context, and current state inside the transaction.
11. **Atomic persistence:** commit canonical revision/record, relationships, source/provenance, validation evidence, latest-selector inputs, event/outbox, audit reference, and idempotency/inbox state.
12. **Response projection:** construct the authorized response/status, `Location`, ETag, correlation and retry metadata; validate invariants before emission.
13. **Post-commit delivery:** publish outbox work, dispatch accepted Commands, refresh derived caches/search, and retry independently of the committed client mutation.

### 7.2 State model

`received → decoded → contract-selected → validated → policy-authorized → accepted-for-commit → committed → publication-pending → published`

Alternative terminal/side paths are `rejected`, `quarantined`, and `pending-dependency`. Only `committed` and later states may affect canonical reads. `accepted-for-commit` is not externally reported as success if the local transaction outcome is unknown. A timeout with ambiguous outcome is recovered through idempotency/status lookup; it is never converted into a second uncorrelated write. **[A/P]**

For Commands and Feasibility, canonical request commit and domain execution/analysis are separate state machines. A synchronous response may contain terminal domain status. An asynchronous response must identify durable status history. `REJECTED` or `FAILED` after request commit is domain evidence, not a rollback of the committed request. **[N/A]**

## 8. Validation and Normalization Responsibility Matrix

| Responsibility | Caller/publisher/adapter may do | Glaux Server must do | Delegation limit/evidence |
|---|---|---|---|
| Transport/source protocol | Decode proprietary framing; authenticate source-facing link | Bound server-facing envelope and authenticate presented identity | Adapter records source channel/security context; server does not inherit unverifiable claims |
| Representation conversion | Convert legacy/proprietary formats to registered CSAPI/SensorML/SWE | Decode and validate submitted registered representation | Transformation manifest identifies source bytes/digest, code/config version, rules and losses; lossy conversion rejected unless explicit authority/profile |
| Identity mapping | Propose source-scoped IDs and mappings | Assign/resolve canonical ID; enforce uniqueness, ownership and parent scope | Mapping is versioned provenance, never silent string substitution |
| Schema/encoding | Prevalidate for feedback | Select exact immutable contract and revalidate | Caller success cannot replace server evidence; remote schema fetch disabled |
| Semantic/unit/property | Enrich from authorized source mapping | Verify selected bindings and policy; reject invented/unproven equivalence | Unit conversion or semantic rebinding is a transformation, not normalization |
| Time/order | Supply event/result time, sequence, offset and clock evidence | Record ingest/transaction clocks; validate ranges; derive current deterministically | Arrival time never silently overwrites domain time or source sequence |
| Geometry | Supply declared CRS/coordinates; optionally transform with evidence | Validate declared profile/CRS and bounds; preserve exact/source precision per policy | No silent reprojection, snapping, repair, or coordinate loss |
| Nil/quality | Supply explicit nil/quality meaning | Validate against exact SWE contract and preserve distinction | Missing, null, nil, unknown, stale and unavailable are not synonyms |
| Authorization/policy | Collect identity/labels and request decisions | Make and enforce final server authorization/releasability/safety decisions | Never delegated to broker ACL, adapter trust, schema validity, or source priority |
| Preconditions/idempotency | Provide If-Match/key/message ID/sequence | Atomically evaluate, persist, and replay original result or reject conflict | Adapter-side dedup is helpful but never the canonical guarantee |
| Persistence/latest/event | None | Own authoritative transaction, history, current selector, event/outbox and audit reference | No direct table/cache/topic writes |
| Response/error | Translate source-side diagnostics outside public API | Produce canonical HTTP/domain status and safe Problem Details | Adapter cannot report canonical success before server commit |

Allowed normalization is deterministic, semantics-preserving, idempotent, registered, and observable: parsing media parameters, constructing a canonical generated serialization, resolving an accepted source-ID mapping, or normalizing geometry orientation where the accepted profile permits and exact source/transformation evidence remains. It must not invent values/units/times/relationships, change Command parameters, discard extensions/precision, rebind old data to a new contract, repair invalid geometry silently, or promote quarantined content. **[A]**

## 9. Identity, Trust-Input, Provenance, and Source-Authority Findings

Canonical identity, source identity, message identity, request identity, revision identity, and correlation identity are separate fields. POST normally produces a server-assigned local identifier; an accepted client/source identifier remains an alternate/source-scoped identifier unless the operation profile explicitly allows it as canonical. PUT-create is initially unsupported. For PUT/PATCH, the URL target is authoritative and a conflicting body identifier is rejected rather than ignored. **[N/P]**

Every committed write records, as applicable:

- authenticated actor/service and delegation chain;
- source system, publisher/adapter, entry point, tenant/mission scope, and simulation/test context;
- request, message, batch, sequence/offset, correlation, resource, parent, and revision identities;
- received, source/event, result, ingest, transaction, and publication clocks without collapsing them;
- exact representation/media/content coding and retained original bytes or policy-permitted digests;
- transformation/normalization manifest and implementation/configuration versions;
- selected schema/profile/semantic package and immutable fingerprints;
- validation, authorization, policy, releasability, safety, and conflict decisions with restricted internal detail;
- resulting canonical resource/revision and related event/outbox/audit identities.

Source registration is input to policy and conflict handling, not proof of truth or permission. Conflicting source claims remain separately attributable; a versioned resolution activity may select a current view but does not rewrite or silently merge evidence. Cryptographic digests provide integrity/correlation evidence, not authorship, authorization, classification, or safe retention. **[A/P]**

## 10. Transaction, Idempotency, Concurrency, Ordering, and Replay Findings

| Submission class | Atomic unit | Replay/concurrency rule | Ordering/current-state effect |
|---|---|---|---|
| Descriptive resource create | Resource aggregate plus required relationships/evidence | Scoped idempotency key for replay-prone POST; same key/same fingerprint returns original result; different fingerprint `409` | New revision/current pointer after commit only |
| PUT/PATCH/DELETE | One aggregate or declared cascade impact graph | Strong `If-Match`; `428` absent, `412` false; optional idempotency key for PATCH/retry | Revision CAS; no lost update or partial cascade |
| Observation | One advertised record/request unit; item-atomic batch default | Source message ID/key and exact contract fingerprint; duplicate effect suppressed atomically | Append history; current view selected by defined domain rule, never arrival alone |
| Command/Feasibility | Request + initial status/evidence/outbox | Mandatory scoped idempotency for unsafe submission; no double dispatch | Domain status ordered by accepted state machine, source and event time |
| Status/result/event | One evidence record plus derived-current inputs | Immutable evidence ID; corrections are attributable revisions/events | Late valid evidence retained; current selector may or may not change |
| Import/batch | Explicit manifest job; each item atomic by default | Batch ID + item identities/fingerprints; resumable durable ledger | Deterministic per-item outcome; no silent omissions |
| Federation/DDIL replay | One source revision/event/tombstone unit unless later protocol declares group | Inbox identity, peer/source revision and causal context | Conflicts coexist pending deterministic policy; no last-arrival-wins |

The local database transaction is the strongest boundary. It includes the idempotency/inbox record so a crash cannot leave a canonical effect without replay recognition or mark a request complete without its effect. A timed-out caller repeats with the same identity or queries the status URI. Glaux provides effectively-once local effects for recognized identities and at-least-once external delivery from the outbox; it does not claim end-to-end exactly-once processing. **[A]**

The OGC atomic/batch proposal distinguishes atomic all-or-rollback execution from independent batch actions. Its reviewed draft is useful vocabulary but not approved authority. Glaux initially uses item atomicity and explicit per-item outcomes; a bounded all-or-none administrative job may be added only with resource limits, prevalidation, one documented transaction/compensation model, and no false CSAPI conformance claim. **[I/P]**

## 11. Persistence, Latest-State, Event/Outbox, and Audit Implications

For each accepted unit, one local transaction writes the applicable subset of:

1. canonical resource or immutable evidence revision;
2. typed relationships and exact parent/contract binding;
3. source identity and provenance activity/entities;
4. validation/normalization/policy decision evidence;
5. deterministic latest/current selection inputs and pointer/update;
6. domain/lifecycle/SystemEvent record when the accepted event-generation matrix requires one;
7. transactional outbox records for publication, cache/search refresh, command dispatch, or downstream work;
8. audit reference/material permitted by policy; and
9. idempotency result, inbox identity, batch ledger, and source offset as applicable.

Raw admission bytes are staged or retained only under the accepted lifecycle policy. Rejected or suspicious material remains isolated from production search, export, analytics, replication, and ordinary backup paths. Quarantine cannot satisfy relationships, advertise capability, create latest state, emit ordinary domain events, or execute Commands. Promotion is a new authorized action that re-runs the complete pipeline under an explicitly selected current/pinned contract and records new evidence without mutating the original. **[A]**

Latest state is a derived projection, not a synonym for most recently received. Late/backfilled/corrected data remains in history and affects current state only under the resource-specific deterministic selector owned by IDR-SRV-034 through 038. Publication failure changes outbox/operational status and lag; it does not roll back a valid committed write. Poison messages enter a bounded, observable delivery-failure workflow while canonical state and the outbox intent remain explainable. **[A/P]**

## 12. Error, Retry, Backpressure, and Operational Behavior

Body-capable application errors use RFC 9457 `application/problem+json` with stable problem type, safe Glaux code, correlation ID, bounded field/component locations, and retry guidance only when justified. Internal evidence may be richer but must not expose hidden resources, policy predicates, controlled vocabularies, source secrets, stack/SQL paths, or exhaustive alternatives. **[A/N]**

| Condition | Public behavior | Retry rule |
|---|---|---|
| Malformed representation or CSAPI-required parent-schema failure | `400` | Correct request; do not blind retry |
| Unauthenticated / unauthorized / concealed | `401`, `403`, or policy-selected `404` | Reauthenticate or obtain authority; do not infer hidden existence |
| Missing resource/dependency | `404` when safe; explicit integration staging only for authorized contract | Public caller correct dependency; trusted job may await bounded resolution |
| State/schema/idempotency conflict | `409` | Resolve state/key intent; same-key different-intent never retried as same operation |
| Stale / missing required precondition | `412` / `428` | GET authorized current representation/ETag, reconcile, then retry intentionally |
| Unsupported request media/coding | `415` | Select advertised representation; no sniffing |
| Semantically unprocessable absent controlling `400/409` | `422` | Correct meaning; do not blind retry |
| Payload/work limit | `413` | Reduce declared unit; no automatic same-payload retry |
| Source/client rate limit | `429` with `Retry-After` where known | Retry after delay with same idempotency identity and jitter |
| Temporary admission/storage dependency unavailable | `503`, optionally `Retry-After` | Retry only with same recognized identity; status-check first after ambiguous timeout |
| Internal response/commit defect | `500`; no false success | Same-key status/retry contract; alert operator |
| Post-commit publication failure | Original committed result remains valid; status/metrics show lag | Outbox worker retries; caller must not recreate mutation |

Backpressure is end-to-end and bounded. HTTP rejects before expensive parsing when budgets are exhausted; import jobs stop admission or remain queued with visible state; broker consumers pause/limit consumption rather than buffer indefinitely; adapters honor server capacity; outbox consumers use bounded retries, circuit breaking, and operator-visible poison state. Initial numeric thresholds are deployment/performance inputs, not invented here. **[P]**

Required observability includes accepted/rejected/quarantined/pending counts by safe class; bytes/items/rate; validation stage, rule and cost; duplicate/conflict/precondition rate; transaction and commit latency; batch/job progress; current-state lag; outbox depth/age/retries; broker/source offset lag; command dispatch/status latency; dependency staging age; and correlation across receipt, transaction, event and publication without logging sensitive payloads by default.

## 13. Security, Policy, DDIL, and Deployment Implications

- Authentication should occur before costly work where protocol permits, but transport or broker identity never substitutes for authorization, source authority, validation, command safety, or releasability. **[A/P]**
- Every entry point applies tenant/mission isolation, object/action authorization, classification/releasability, source constraints, URI/reference safety, rate/work budgets, and non-oracular errors. Internal calls are not exempt. **[A/P]**
- Commands require a durable policy/safety decision before dispatch; status/result reporters require explicit authority; simulator-originated content is marked synthetic and cannot cross a production boundary merely because it is structurally valid. **[P]**
- Offline validation uses a closed, immutable contract package. Missing schema/profile dependencies fail closed or enter explicit non-active staging; the server never fetches public network schemas to make a write pass. **[A]**
- DDIL replay carries immutable source/message identities, causal/source revisions, contract fingerprints, tombstones, policy context, and conflict evidence. Reconnection does not grant older writes arrival priority or permit authorization/policy bypass. **[A/P]**
- Deployments independently enable public write classes, publisher/broker/import adapters, simulator controls, binary codecs, quarantine, and async operation resources. Capability discovery must be generated from that same registry so unavailable methods/media are neither routed nor advertised. **[A/P]**
- URI resolution is allowlisted, bounded, cached by content identity, and non-recursive across security boundaries. Payloads, compressed data, geometries, schemas, regexes, archives and batches receive explicit size/depth/work limits. **[A/P]**

IDR-SRV-039 through 041 own the final identity, authorization, classification/releasability, command-policy, diagnostic, and audit schemas. IDR-SRV-042/043 own offline availability, synchronization, reconciliation and conflict protocols. This report's invariant is that none of those controls may be implemented solely outside the authoritative server transaction.

## 14. Verification Scenarios and Conformance Boundary

| ID | Scenario and expected result | Test lane |
|---|---|---|
| V1 | Create each enabled Part 1 resource in each registered media type; verify 201, Location, canonical identity, exact provenance and no hidden partial relation | CSAPI class + Glaux contract |
| V2 | PUT/PATCH/DELETE without If-Match, with stale tag, then current tag; expect 428, 412, then one atomic revision/effect | Glaux concurrency contract; draft dependency trace |
| V3 | Add valid/invalid/foreign/hidden collection members; distinguish membership from canonical delete and prevent existence leakage | CSAPI Part 1 + security |
| V4 | Attempt stream schema replace/PATCH after child records and noncascade/cascade delete; verify 409/impact behavior and complete rollback | CSAPI Part 2/ATS + lifecycle |
| V5 | Submit JSON/SWE JSON/Text/Binary Observation fixtures against exact, wrong, missing and changed DataStream contracts; explicit schema failure is 400 | CSAPI encoding/CRD + Glaux codec corpus |
| V6 | Send duplicate, changed-intent duplicate, partial batch, timeout and crash-at-commit-boundary cases; prove original-result replay or 409 and no duplicate effects | Glaux idempotency/failure injection |
| V7 | Submit legacy, transformed, simulated and untrusted content; verify delegation, transformation lineage, synthetic marker, rejection/quarantine isolation | Integration/security contract |
| V8 | Apply merge patches with null, arrays, body ID, parent/contract and required-field changes; verify atomic whole-result validation and immutable-field rejection | Glaux PATCH profile + RFC 7396 |
| V9 | Replay late/out-of-order/backfilled records; retain history and deterministically update or preserve latest without arrival overwrite | Glaux dynamic/DDIL contract |
| V10 | Saturate validation, request, broker and outbox capacities; verify bounded 413/429/503, Retry-After rules, paused consumption and no unbounded queue | Performance/operations |
| V11 | Submit synchronous and asynchronous Command/Feasibility requests; distinguish HTTP admission errors from terminal/continuing domain statuses | CSAPI Part 2 + tasking contract |
| V12 | Crash between command commit/dispatch and during retry; prove durable command/audit/outbox before dispatch and no double canonical effect | Failure injection/safety |
| V13 | Post authorized/unauthorized, valid/invalid, late and conflicting status/result/event evidence; prove append provenance, state rules and isolation | CSAPI classes + domain/security |
| V14 | Return negative/indeterminate feasibility; prove it is valid domain output and neither authorization nor command reservation | CSAPI Part 2/domain |
| V15 | Import a manifested mixed batch; prove declared item atomicity, complete per-item ledger, bounded quarantine, resumability and promotion revalidation | Administrative integration |
| V16 | Probe simulator control endpoints in production/disabled and test-enabled deployments; expect absence/denial outside explicit test capability and no CSAPI claim | Deployment/security |

Official CSAPI ATS/ETS tests prove only applicable claimed normative requirements. Glaux contract tests prove stronger preconditions, media profiles, private entry points, provenance, idempotency, failure recovery, policy, DDIL and operational behavior. Implementation interoperability tests show compatibility but cannot redefine a failed or ambiguous normative requirement. Every test records requirement/project-decision ID, enabled capability/profile, exact fixture, source version, expected status/domain outcome, atomic effects, evidence identities, and cleanup.

## 15. Downstream Handoff Matrix

| Topic | Fixed input from IDR-SRV-031 | Decision still owned downstream |
|---|---|---|
| IDR-SRV-032 Publisher model | One common server command/pipeline; prefer standard routes; explicit source/message/order/transform evidence; no direct persistence | Publisher transport, authentication mechanics, buffering, delivery, batching, high-rate private endpoint justification |
| IDR-SRV-033 Simulator model | Ordinary synthetic data uses normal ingestion with synthetic provenance; controls are separate test-only admin capability | Scenario/reset/clock/seed/fault contract and safe cleanup |
| IDR-SRV-034 Observation ingestion | Exact parent contract, item atomicity, append history, deterministic current selection, late/replay provenance | Observation identity, correction, watermark, current-value and semantic rules |
| IDR-SRV-035 Streaming/backpressure | Commit/outbox separation, bounded queues, source identity and no silent drop | Streaming protocols, subscriptions, flow control thresholds and delivery semantics |
| IDR-SRV-036 Command lifecycle | Durable command + initial evidence before dispatch; HTTP vs domain failure separation | State machine, cancellation/update, reporter authority and execution transitions |
| IDR-SRV-037 Feasibility | Input validation separate from negative/indeterminate result; durable async status | Evaluation lifecycle, expiration, result/alternative semantics |
| IDR-SRV-038 Results/events/status | Append-oriented attributed evidence and deterministic current projection | Exact result linkage, status/current derivation, event-generation rules |
| IDR-SRV-039–041 Security/audit | Final authority remains server-side; non-oracular diagnostics; evidence in transaction | AuthN/Z mechanisms, classification/releasability, command policy, audit schema/retention |
| IDR-SRV-042/043 DDIL | Closed offline contracts, stable identities, inbox/idempotency, causal conflicts, no arrival overwrite | Degraded API behavior, sync protocol, tombstones, reconciliation and operator resolution |
| IDR-SRV-044–049 Implementation/deployment | Registry-driven capability and common application service; durable transaction/outbox/inbox | Rust crates, module/API/storage topology, configuration, deployment and performance sizing |
| IDR-SRV-050–056 Verification | V1–V16 and normative-vs-project lane split | Harnesses, exact case catalog, CI gates, interoperability partners and execution evidence |
| Part 3 research track | Broker/pub-sub cannot bypass server authority; no draft implementation authorized here | Exact Part 3 snapshot, requirements/ATS, topic/envelope/security/QoS mapping and adoption decision |

## 16. Recommendations and Explicit Project Decisions

Acceptance of this report approves the following planning decisions:

1. **Adopt one authoritative write application service.** Every transport and internal caller maps to typed canonical resource commands and the common pipeline; no direct canonical-table/latest/topic bypass.
2. **Make capabilities registry-driven.** Routes, OPTIONS, `Allow`, `Accept-Post`, `Accept-Patch`, OAD and conformance declarations derive from the same implemented operation/representation registry.
3. **Use strict initial HTTP mutation semantics.** POST creates; PUT replaces existing resources only; JSON Merge Patch is the initial registered PATCH format for JSON writable projections; identifiers/parents/contracts are protected; the entire result is validated.
4. **Preserve accepted concurrency/idempotency rules.** Strong If-Match is mandatory for existing PUT/PATCH/DELETE; unsafe/replay-prone writes use scoped idempotency or immutable source-message identity stored with the effect.
5. **Separate receipt, acceptance, commit and publication.** `202` is allowed only with a durable status resource; ambiguous commit is recovered by identity/status; publication and Command dispatch occur post-commit from the outbox.
6. **Use strict public rejection and privileged isolation.** Invalid public writes have no active effect. Quarantine and pending-dependency are bounded, explicit administrative/integration states and require complete revalidation to promote.
7. **Default batches to item atomicity with complete results.** An all-or-none job is a separate bounded capability; Glaux makes no atomic-batch standards claim from the current proposal.
8. **Retain server authority.** Authentication context, authorization, canonical identity, conformance-critical validation, policy/releasability, safety, provenance, transaction, persistence and canonical error decisions remain in Glaux Server.
9. **Preserve exact source and transformations according to policy.** Normalization cannot repair meaning; semantic conversion is attributable adapter work with a transformation manifest and server revalidation.
10. **Treat history and current state separately.** Late, out-of-order, duplicate, replayed and conflicting evidence remains attributable; deterministic domain rules—not arrival order—select current state.
11. **Keep simulator controls and source-native transports outside public CSAPI.** Simulator data may reuse normal writes, while control operations and legacy/proprietary formats require separate bounded adapters/capabilities.
12. **Track the draft dependency without overclaiming.** Pin OGC API - Features CRUD, record deltas, and reassess PATCH/status/OPTIONS behavior when approved; do not claim its draft or the batch proposal as an approved standalone standard.

Priority for implementation planning is: common typed command/registry and problem contract; transaction/idempotency/outbox foundation; descriptive and stream contract writes; Observation ingestion; Command/Feasibility durability; private adapters/batches; then optimized streaming and DDIL reconciliation. Publisher and simulator topic reports may refine their side of the boundary but must raise an explicit contradiction before weakening these invariants.

## 17. Risks, Contradictions, Assumptions, and Unresolved Questions

| Item | Type/impact | Resolution or owner |
|---|---|---|
| OGC API - Features CRUD is draft and mutable | Normative-dependency risk; method/status/PATCH rules can change | Pin `4e30324...`; automated/deliberate delta review at approval; conformance trace distinguishes CSAPI invocation from Glaux profile |
| CSAPI update classes do not settle a universal PATCH document | Interoperability risk | Glaux selects merge patch on registered JSON writable projections; advertise exact media; retest against approved dependency; 044/050 |
| Current merge-patch draft feature rule depends on Features Part 5 schemas | Coverage gap for non-feature CSAPI resources | Glaux-owned writable projections/immutable-field rules are project profile, not claimed inherited obligation |
| Part 1 Property condition link, Part 2 Feasibility wording/routes, PATCH status, and tagged OAS gaps | Artifact/ATS conflict | Maintain normative overlay and requirement-level fixtures; 050–051 |
| Draft permits bare asynchronous CRUD responses but lacks universal outcome contract | Ambiguous success/retry risk | Glaux forbids bare 202 and requires durable operation status; operation design 044–049 |
| Atomic/batch proposal is not approved | False conformance/partial-result risk | Item-atomic Glaux job contract only; revisit after standard maturity |
| Exact client-selected canonical ID policy is not finalized | Collision/authority risk | Server IDs default; resource-specific allowlist and source mapping in implementation/profile ADR |
| Numeric work, queue, retry and quarantine thresholds are absent | Capacity/security risk | Performance/deployment and security policy must supply measured values; fail bounded meanwhile |
| Pending dependencies can become an unbounded shadow store | Operational/security risk | Only explicit trusted contract, TTL/size/owner/status, no active visibility; 032/042–044 |
| Domain latest selectors and lifecycle transitions are not fixed here | Incorrect current state/action risk | 034–038 must define deterministic rules before enabling corresponding production writes |
| Source/broker trust can be mistaken for authorization | Cross-tenant/policy risk | Server-side policy decision on every effect; 039–041 |
| Retaining raw/digests can violate minimization or leak correlation | Evidence/privacy risk | Apply accepted 030 authority/retention model and later audit/security policy |
| Existing implementations omit PATCH/Feasibility or have weak validation | Feasibility evidence is incomplete | Use OSH/CS-Go/pygeoapi only for architectural lessons, never as conformance oracle **[I]** |

Assumptions are limited to accepted project baselines: PostgreSQL-centered local transactions, immutable revisions/evidence, registry-first generated contracts, RFC 9457 problems, strong validators, transactional outbox/inbox, and policy-aware lifecycle. No assumption is made that every deployment claims every write class, supports every media type, enables brokers/simulators/import, or has continuous connectivity.

Unresolved questions are intentionally assigned: exact route-level writable projections and client-ID allowlists (044–050); publisher/broker envelope and throughput (032/035); simulator controls (033); observation/current/watermark semantics (034–035); Command/Feasibility/status/result state machines (036–038); identity/policy/audit exposure (039–041); DDIL conflict/replay (042/043); and Part 3 adoption through its separate research plan. None prevents acceptance of this boundary.

## 18. Validation Against This Plan's Success Criteria

| Success criterion | Status | Evidence |
|---|---|---|
| Every in-scope CSAPI resource family and mutation identified in the controlling standards has a clause-anchored write-surface entry | Met | Sections 3–4 and the requirement-anchored Section 6 matrix |
| Each candidate entry point and operation is classified as directly accepted, contract-constrained, delegated, internal-only, rejected, or unresolved | Met | Sections 5–6; matrix dispositions and unresolved-issue column |
| Standards obligations, AEP profile choices, Glaux decisions, implementation observations, and researcher inferences are visibly distinguished | Met | Section 2 authority hierarchy and report-wide evidence labels |
| Common ingestion stages, state transitions, transaction points, persistence artifacts, and failure boundaries are documented | Met | Sections 7, 10–12 |
| Validation, normalization, identity, provenance, policy, idempotency, concurrency, ordering, replay, and error responsibilities are allocated | Met | Sections 8–12 |
| Normative prose, ATS, schema, OpenAPI, and implementation discrepancies are recorded rather than silently reconciled | Met | Sections 2–4, 6, and 17; upstream-history register Version 1.11 |
| Positive and negative verification scenarios trace to every recommended boundary behavior | Met | Section 14 V1–V16 and references from Section 6 |
| Publisher, simulator, dynamic-data, security, DDIL, architecture, and testing handoffs are explicit and non-overlapping | Met | Section 15 |
| Unknown, unavailable, draft-dependent, or policy-dependent conclusions are labeled with their impact | Met | Sections 2, 13, and 17 |
| Recommendations are bounded to Glaux Server and references are reproducible | Met | Sections 1, 15–16, and 19 |
| Report is polished, recommendation-first, independently readable, and self-contained for the project lead, implementers, and later AI agents | Met | Entire report, especially Sections 1 and 16 |

### Report completion checklist

- [x] Objective and every core/detailed question covered or explicitly handed off
- [x] Normative, accepted-project, recommendation, implementation, and gap evidence distinguished
- [x] Mutable standards/editor evidence pinned by commit/status/date
- [x] Draft OGC API - Features CRUD dependency and atomic/batch proposal status recorded
- [x] Required resource/operation inventory, entry taxonomy, 16-field decision matrix, pipeline, responsibility and state models present
- [x] Failure, recovery, source/provenance, transaction, publication and verification boundaries defined
- [x] Contradictions and implementation evidence limitations retained
- [x] No publisher/simulator/dynamic/security/DDIL topic preempted
- [x] No draft Part 3 or server implementation authorized
- [x] Report independently readable and ready for project-lead review

## 19. References

### 19.1 Controlling project sources

- [IDR-SRV-031 research plan](../IDR%20Plans/idr-srv-031-server-write-and-ingestion-model.md)
- [Overall IDR research plan](../IDR%20Plans/overall-idr-research-plan.md)
- [OGC API - Connected Systems upstream-history evidence register, Version 1.11](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md)
- [IDR-SRV-012 Content Negotiation, Media Types, and Encoding Selection](idr-srv-012-content-negotiation-media-types-and-encoding-selection-report.md)
- [IDR-SRV-013 Error Model, HTTP Status Codes, and Failure Semantics](idr-srv-013-error-model-http-status-codes-and-failure-semantics-report.md)
- [IDR-SRV-014A OpenSensorHub implementation study](idr-srv-014a-osh-csapi-server-implementation-study-report.md)
- [IDR-SRV-014B Connected Systems Go implementation study](idr-srv-014b-connected-systems-go-csapi-server-implementation-study-report.md)
- [IDR-SRV-014C 52North pygeoapi proof-of-concept study](idr-srv-014c-pygeoapi-csapi-server-implementation-study-report.md)
- [IDR-SRV-023 Schema and Encoding Validation Strategy](idr-srv-023-schema-and-encoding-validation-strategy-report.md)
- [IDR-SRV-029 Transaction, Consistency, Idempotency, and Concurrency Strategy](idr-srv-029-transaction-consistency-idempotency-and-concurrency-strategy-report.md)
- [IDR-SRV-030 Data Lifecycle, Retention, Archival, and Deletion Strategy](idr-srv-030-data-lifecycle-retention-archival-and-deletion-strategy-report.md)

### 19.2 Standards and pinned artifacts

- [OGC API - Connected Systems - Part 1: Feature Resources, OGC 23-001, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2: Dynamic Data, OGC 23-002, Version 1.0](https://docs.ogc.org/is/23-002/23-002.html)
- [OGC API - Connected Systems editor source, tag v1.0.0 at `8e03b236...`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)
- [OGC API - Features Create/Replace/Update/Delete draft at `4e30324a...`](https://github.com/opengeospatial/ogcapi-features/tree/4e30324a14b682ff4a26ee43aad1eb6428c846a3/extensions/transactions/create-replace-update-delete)
- [OGC atomic/batch proposal at the reviewed commit](https://github.com/opengeospatial/ogcapi-features/blob/4e30324a14b682ff4a26ee43aad1eb6428c846a3/proposals/atomic-batch-tx/standard/26-018.adoc)
- [OGC SensorML Encoding Standard 3.0, OGC 23-000](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC SWE Common Data Model Encoding Standard 3.0, OGC 24-014](https://docs.ogc.org/is/24-014/24-014.html)
- [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)
- [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html)
- [RFC 6585: Additional HTTP Status Codes](https://www.rfc-editor.org/rfc/rfc6585.html)
- [RFC 5789: PATCH Method for HTTP](https://www.rfc-editor.org/rfc/rfc5789.html)
- [RFC 7396: JSON Merge Patch](https://www.rfc-editor.org/rfc/rfc7396.html)

### 19.3 Informative implementation evidence

- [OpenSensorHub](https://github.com/opensensorhub/osh-core) — adapter/hub and typed-handler patterns; incomplete write-class coverage and not a conformance oracle
- [Connected Systems Go](https://github.com/OS4CSAPI/connected-systems-go) — typed repositories, PostgreSQL and optional MQTT evidence; incomplete PATCH/Feasibility and delivery/security proof
- [52North Connected Systems pygeoapi proof of concept](https://github.com/52North/connected-systems-pygeoapi) — provider/schema-boundary evidence; narrow resource/operation coverage

### 19.4 Numbered source notes

[^1]: OGC API - Features CRUD source identifies document `20-002r2`, version `1.0.0-SNAPSHOT`, as a Draft and warns that it is not an OGC Standard. The repository README states that the draft passed public comment and was being prepared for approval at the reviewed snapshot. Accessed 2026-09-14.
[^2]: RFC 9110 defines POST, PUT, DELETE, conditional request evaluation, status semantics, validators, and retry-relevant HTTP behavior used by this report. Accessed 2026-09-14.
[^3]: RFC 9457 defines `application/problem+json`, problem type/status semantics, extensions, and security considerations for problem details. Accessed 2026-09-14.
[^4]: RFC 7396 defines JSON Merge Patch, including object-member replacement/removal through `null` and replacement of non-object values. The reviewed OGC draft adds a conditional feature-profile requirement for `application/merge-patch+json`; Glaux's broader registered JSON writable-projection use is explicitly a project profile. Accessed 2026-09-14.

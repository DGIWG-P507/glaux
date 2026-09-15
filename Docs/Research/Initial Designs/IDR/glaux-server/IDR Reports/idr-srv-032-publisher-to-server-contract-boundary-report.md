# Section 032: Publisher-to-Server Contract Boundary - Research Report

**Topic ID:** IDR-SRV-032<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-032 Publisher-to-Server Contract Boundary](../IDR%20Plans/idr-srv-032-publisher-to-server-contract-boundary.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** Five core questions and all detailed questions concerning publisher/source roles, identity, registration, authority, contract surfaces, payloads, delivery, validation, errors, security inputs, DDIL, evolution, fixtures, and handoffs<br>
**Methodology Used:** Authority-ranked extraction from approved OGC API - Connected Systems Parts 1 and 2, HTTP and messaging specifications, accepted IDR-SRV-001 through IDR-SRV-031 findings, the bounded upstream-history register, and pinned implementation evidence; followed by role, responsibility, contract-surface, envelope, delivery-state, failure, and verification mapping<br>
**Research Time:** Approximately 17 hours of AI-assisted research and synthesis, September 14, 2026<br>
**Primary Source(s):**
- [OGC API - Connected Systems - Part 1: Feature Resources](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2: Dynamic Data](https://docs.ogc.org/is/23-002/23-002.html)
- [Pinned OGC API - Connected Systems 1.0 editor source](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)
- [MQTT Version 5.0 OASIS Standard](https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html)
**Supporting Resources:**
- [IDR-SRV-019 Provenance, Lineage, Quality, and Trust Metadata Model](idr-srv-019-provenance-lineage-quality-and-trust-metadata-model-report.md)
- [IDR-SRV-023 Schema and Encoding Validation Strategy](idr-srv-023-schema-and-encoding-validation-strategy-report.md)
- [IDR-SRV-029 Transaction, Consistency, Idempotency, and Concurrency Strategy](idr-srv-029-transaction-consistency-idempotency-and-concurrency-strategy-report.md)
- [IDR-SRV-031 Server Write and Ingestion Model](idr-srv-031-server-write-and-ingestion-model-report.md)
**Document Purpose:** Establish the implementable external publisher contract that specializes the accepted common server write boundary without creating a second canonical ingestion model<br>
**Author(s):** OpenAI Codex, for the Glaux Project<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** September 14, 2026<br>
**Date:** September 14, 2026<br>
**Last Updated:** September 14, 2026

---

This report labels evidence as **[N] normative standard**, **[A] accepted project baseline**, **[P] project recommendation/decision proposed for acceptance**, **[I] informative implementation evidence**, and **[X] documented conflict, draft, or gap**. A combined label does not elevate informative or draft material into an approved requirement.

## Table of Contents

1. Executive Summary
2. Scope, Publisher Terminology, and Authority Model
3. Standards and AEP Obligation Baseline
4. Publisher, Adapter, Gateway, Broker, Importer, and Source Taxonomy
5. Publisher/Source/Principal Identity and Registration Lifecycle
6. Publisher-versus-Server Responsibility Matrix
7. Contract-Surface and Transport Analysis
8. Payload, Envelope, Media-Type, and Mapping Rules
9. Validation, Acceptance, Quarantine, and Provenance Behavior
10. Delivery-State, Batching, Idempotency, Ordering, Retry, Replay, and Acknowledgement Behavior
11. Error, Correlation, Diagnostics, and Source-Health Behavior
12. Backpressure, DDIL, Security, Policy, Audit, and Evolution Implications
13. Fixture and Interoperability Matrix
14. Downstream Handoff Matrix
15. Recommendations and Explicit Project Decisions
16. Risks, Contradictions, Assumptions, and Unresolved Questions
17. Validation Against This Plan's Success Criteria
18. References

---

## 1. Executive Summary

Glaux Server should use **standards-facing CSAPI HTTP writes as the default publisher interface and narrow, versioned adapters for broker, federation, and file/batch delivery**. It should not introduce a general-purpose private ingestion API. Every path must resolve to the same typed application command and traverse the common identity, contract, validation, authority, policy, transaction, provenance, persistence, outbox, audit, and response pipeline accepted in IDR-SRV-031. An adapter may change transport or map source-native bytes, but it may not define a second domain model or write canonical storage directly. **[A/P]**

The server-visible actor model has four independent elements: the authenticated principal observed by Glaux; the registered publisher software instance; the represented source; and a time-bounded authority grant describing what that publisher may assert for that source. Authentication proves none of authorization, source authority, semantic validity, policy acceptance, priority, or truth. One principal may operate several publisher instances; one publisher may represent several sources; one source may have several publishers; and authority can vary by resource family, operation, identifier namespace, mission/tenant, transport, profile, and effective time. **[A/P]**

The report proposes **Glaux Publisher Contract v1 (GPC-v1)** as a supplemental, transport-neutral contract profile—not an OGC conformance class. For direct HTTP, the canonical CSAPI payload remains the message body at its registered CSAPI path. A small Glaux metadata binding carries contract version, represented source, source message identity, and optional source epoch/sequence/batch information. The authenticated publisher is derived from credential-to-registration binding rather than trusted from a caller header. `Idempotency-Key` is required for command-like, replay-prone, brokered, imported, and DDIL work and may be required for all publisher POSTs by deployment profile. The spelling follows a current-but-expired IETF draft and is therefore explicitly a Glaux field contract, not a claim of an RFC standard.[^10] **[P/X]**

Three private, versioned operational surfaces are justified, none of which replaces canonical CSAPI mutation:

- `GET /_glaux/publisher/v1/submissions/{submissionId}` resolves asynchronous and ambiguous outcomes;
- `POST /_glaux/publisher/v1/import-jobs` plus `GET /_glaux/publisher/v1/import-jobs/{jobId}` admits bounded manifests and exposes complete item results; and
- `POST /_glaux/publisher/v1/health-reports` accepts optional publisher-observed connectivity/backlog evidence without turning it into sensor/System status.

The server repeats authoritative validation even when the publisher prevalidates. Normal invalid public traffic is rejected atomically. Quarantine is available only to explicitly authorized raw/import or security workflows, is non-canonical and isolated, and cannot satisfy relationships or leak other sources. Missing dependencies are rejected on ordinary direct traffic; a trusted contract may use bounded `pending-dependency` staging with quotas, expiry, custody, and full revalidation. Batches are independently item-atomic with a complete ledger; “partial” is a job aggregate, never a hidden item state. **[A/P]**

Transport receipt, MQTT acknowledgement, validation, durable commit, and later publication are distinct. MQTT QoS and PUBACK demonstrate broker-protocol progress, not Glaux canonical commit.[^8] A broker bridge acknowledges upstream only after the configured durable inbox or commit threshold and deduplicates redelivery through the common admission ledger. Glaux promises no global arrival order. `(represented source, source epoch, source sequence)` is continuity evidence, while domain time and accepted source policy control current-state projections. **[A/P]**

Final security mechanisms, broker/topic design, dynamic-data watermarks, command safety, and DDIL reconciliation remain assigned to IDR-SRV-034 through 043. Draft OGC Connected Systems Part 3 remains evidence for IDR-SRV-035 and is not implemented, selected, or advertised by this report.

## 2. Scope, Publisher Terminology, and Authority Model

### 2.1 Scope and exclusions

In scope are the external contract visible to publishers: onboarding inputs, role/source classification, paths and adapters, payload and envelope rules, responsibility allocation, validation and acceptance, acknowledgement, delivery state, batching, idempotency, ordering, retry/replay, backpressure, safe diagnostics, health evidence, DDIL behavior, contract evolution, fixtures, and downstream requirements.

Out of scope are Publisher product internals; device/source protocols; simulator reset, synthetic-clock, and scenario controls; the final broker and Part 3 binding; precise Observation/status watermarks; Command safety authorization; credential/token design; cross-domain policy syntax; server component/database selection; numeric quotas and retention periods; and implementation. Those decisions remain with their indexed owners.

### 2.2 Precise terms

| Term | Meaning in this report | Not equivalent to |
|---|---|---|
| Authenticated principal | Identity established by the server's authentication mechanism for this interaction | Publisher instance, represented source, authority, or payload author |
| Publisher | External software role that submits through an approved Glaux publication contract | Device/source, human principal, broker, or canonical resource |
| Publisher instance | Registered deployment of publisher software with stable instance identity and version/build evidence | A reusable OAuth client alone or the organization owning it |
| Represented source | Registered origin whose observations, descriptions, events, status, or task results are being conveyed | Transport peer or publisher process |
| Source authority | Versioned grant permitting a publisher/principal to make specified claims for a represented source | Authentication, validity, trust, or priority |
| Source message ID | Source- or adapter-assigned identity for one immutable conveyed message | Canonical ResourceId or HTTP idempotency scope by itself |
| Submission | One attempted contract operation and its durable receipt/correlation identity | A canonical resource or successful mutation |
| Transport acknowledgement | Evidence that a transport participant accepted protocol custody | Validation, commit, publication, or domain success |
| Accepted | The server authorized an intended effect and entered the applicable commit path | Merely received or already published |
| Committed | The complete authoritative local transaction durably recorded its declared effect | Delivered to an external subscriber |
| Published | A post-commit outbox consumer delivered an external event/message | Committed, consumed, or acted upon |
| Quarantined | Isolated non-authoritative bytes/evidence under explicit custody policy | Partially accepted data |
| Pending dependency | Bounded non-canonical staging for an authorized trusted source awaiting a declared prerequisite | A successful reference or general out-of-order store |

### 2.3 Authority model

The authorization tuple is conceptually:

`principal × publisherInstance × representedSource × operation × resourceFamily × identifier/path scope × tenant/mission × contract/profile × transport × effectiveTime`.

The server derives the principal from the authenticated channel, resolves the credential to one or more active publisher registrations, and then evaluates an explicit representation grant for the asserted source and operation. The source identifier and policy markings arriving in a message are claims until verified against that grant and the governing policy. Server validation determines admissibility; source-priority/current-state policy determines selection; neither retroactively proves truth. **[A/P]**

For a gateway or broker bridge, provenance retains both hops: end source and its source message identity, plus gateway/publisher software and authenticated principal. The gateway must not erase the source by becoming the apparent origin of every record. Conversely, an asserted source cannot erase the gateway that transformed or conveyed the content. This preserves the actor distinctions accepted in IDR-SRV-019.[^4]

## 3. Standards and AEP Obligation Baseline

### 3.1 Authority-ranked evidence

| Source | Status/version/pin | Relevant boundary | Authority and limitation |
|---|---|---|---|
| OGC API - Connected Systems Part 1 | OGC 23-001, Version 1.0; tagged source `8e03b236` | Feature-resource writes, links, identifiers, encodings, nested/canonical paths | Approved normative baseline; write classes are conditional |
| OGC API - Connected Systems Part 2 | OGC 23-002, Version 1.0; same tag | Stream/data/task resource writes and parent-schema behavior | Approved normative baseline; published artifacts contain recorded route/schema gaps |
| OGC API - Features CRUD | `20-002r2`, `1.0.0-SNAPSHOT`, Draft; `4e30324a` | HTTP create/replace/update/delete dependency | Draft dependency, not an approved standard; used only where CSAPI invokes it and with gaps disclosed |
| HTTP Semantics / Problem Details | RFC 9110 / RFC 9457 | Methods, status, conditional requests, retry safety, problem documents | Published Internet standards[^1][^2] |
| HTTP Digest Fields | RFC 9530 | `Content-Digest` integrity evidence | Published Standards Track; digest is not authenticity, authority, or truth[^6] |
| MQTT 5.0 | OASIS Standard, 2019 | Sessions, QoS acknowledgements, Receive Maximum, message expiry | Normative for a future MQTT binding only; not application commit semantics[^8] |
| CloudEvents | v1.0.2, commit `fc1f6f31` | Producer/source distinction and event identity precedent | Informative candidate mapping; not selected as the canonical publisher envelope[^9] |
| OAuth Security BCP / mTLS | RFC 9700 / RFC 8705 | Sender-constrained credentials and client/certificate distinctions | Security requirements/input; mechanism choice remains Category G[^12][^13] |
| W3C Trace Context | Recommendation, 2021 | Optional `traceparent`/`tracestate` propagation | Observability only, never business identity or trust[^7] |
| Accepted IDR-SRV-001–031 | Repository main through 031 acceptance | Controlling Glaux identity, validation, provenance, transaction, lifecycle, and write pipeline | Project authority for this specialization |

Project-controlled STANAG 4789/AEP-4789 material is used only through accepted, releasable prior-IDR findings. The controlled AEP baseline recorded by IDR-SRV-019 is `AC/224(JCGISR)D(2026)0005`, dated 27 April 2026, SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`. It supports preserving discoverable, understandable, linked, trustworthy, and securely handled operational context; it does not create publisher headers, transports, trust algorithms, or new public endpoints. **[A]**

### 3.2 Resulting obligations

CSAPI controls the canonical resource representation and standards-facing write path where Glaux claims the relevant transaction class. It does not standardize publisher registration, credential binding, idempotency metadata, broker custody, private import jobs, or source-health reports. Those are Glaux capabilities and must remain separately named/versioned. **[N/P]**

The server may accept only the method/media/profile combinations it advertises. Parent-contract validation for Observations, Commands, statuses, and results remains server-authoritative. Client/server content negotiation, canonical links, generated fields, and response semantics continue to follow the accepted CSAPI/HTTP profile in IDR-SRV-031. Supplemental publisher metadata cannot change the meaning of the canonical body or make a nonconforming payload conforming. **[N/A]**

The shared upstream-history register was date-checked against its September 14 CSAPI `master` snapshot. Relevant open/post-publication issues document unresolved write, encoding, association, schema, and batch details; they do not establish a private ingestion requirement. Part 3 and legacy AsyncAPI evidence remain draft/informative and owned by IDR-SRV-035. **[X]**

## 4. Publisher, Adapter, Gateway, Broker, Importer, and Source Taxonomy

| Publisher class | Represented source class | Typical submitted material | Registration/authority treatment | Boundary consequence |
|---|---|---|---|---|
| Glaux Publisher | One or many registered device, platform, mission, or enterprise sources | Canonical CSAPI resources or source-native material through a registered mapping profile | Instance registration plus source-specific grants; production and test instances separate | Default direct CSAPI HTTP; replay metadata required when buffering/offline |
| Standards-native producer | CSAPI/SensorML/SWE-aware system | Canonical descriptions and dynamic data | Prior source/namespace grant; may omit adapter mapping | Use canonical CSAPI writes; server still repeats validation |
| OSH/Connected Systems publisher | OSH or CSAPI server/device network | Metadata, Observations, status/events, task results | Federated or publisher grant; exact implementation/version recorded | Implementation compatibility is informative, not conformance proof |
| Legacy-feed adapter | Legacy protocol endpoint/file/source database | Mapped canonical payload plus preserved raw artifact reference/digest | Mapping-profile and source-credential custody recorded; higher validation/quarantine controls | Adapter parses/maps; server remains canonical authority |
| Tactical/edge gateway | Intermittent group of downstream sources | Buffered/replayed records from multiple sources | Per-source grants or constrained namespace grant; gateway cannot self-invent sources | DDIL epoch/sequence/message identity and bounded replay are mandatory |
| Broker bridge | Broker subject(s) carrying one or many sources | Message body plus broker/source metadata | Topic/subject binding and source extraction rules registered; final protocol binding deferred | Durable inbox and commit-aware application acknowledgement; broker ACK is insufficient |
| File/batch importer | Curated package, removable media, controlled export, administrative upload | Manifest plus canonical or source-native items | Explicit import permission, package profile, limits, custody, and quarantine rights | Uses private import-job surface, then the common item pipeline |
| Federated-source adapter | Remote CSAPI or approved service | Pulled resources/deltas with remote identity/evidence | Remote endpoint, source namespaces, cursor/authority and conflict policy registered | Pull is external adapter behavior; each local effect uses common commands |
| Command/status gateway | Executor/controller and command lifecycle source | CommandStatus, CommandResult, Feasibility status/result, possibly SystemEvent | Narrow command-stream/source grant; final safety authorization remains IDR-SRV-038 | No authority merely because it knows a Command ID; strict state/concurrency checks |
| Fixture publisher | Test corpus or conformance harness | Golden, invalid, delayed, duplicate, and adversarial inputs | Test-only environment/tenant and credentials; never production authority | Deterministic evidence, expected outcomes, no hidden bypasses |

“Raw,” “mapped,” “derived,” and “authoritative” describe assertions and processing history, not universal trust levels. A legacy adapter may be authoritative for capture but not for a canonical identifier; a standards-native producer may still be unauthorized or wrong. Derived data records its input entities, transform software/profile, activity, and responsible agents. **[A/P]**

## 5. Publisher/Source/Principal Identity and Registration Lifecycle

### 5.1 Registration records

Publisher and source registrations are administrative security/configuration records, not CSAPI resources and not advertised conformance. At minimum, the server must retain:

- stable publisher instance ID; software/product, build/version, deployment/environment, owner and operational contact;
- credential/client/certificate/key references and validity—not secrets—bound to the instance and permitted authentication methods;
- represented source IDs, source class, namespace/UID ownership or mapping, and predecessor/successor relationships;
- authority grants by tenant/mission, environment, operation, resource/data family, path/parent scope, command capability, identifier namespace, media/profile, and transport binding;
- GPC and mapping-profile versions, accepted schemas/encodings, broker subject or import-package rules where applicable;
- idempotency scope/retention and replay horizon, sequence/epoch policy, batch/size/rate/backpressure/DDIL constraints;
- allowed policy-claim issuers/labels and whether raw, pending-dependency, or quarantine custody is permitted;
- activation/effective/expiry times, status/reason, approval evidence, revision, actor, and complete audit history.

One publisher can have many representation grants, and a source can have concurrent publishers with disjoint or overlapping scopes. Overlap requires an explicit source conflict/priority rule; arrival time is not a rule. A source transfer changes grants at an effective boundary and preserves historical provenance—it does not rename the source or rewrite old actor evidence. **[P]**

### 5.2 Lifecycle

The publisher registration state machine is:

`pending-validation → active ↔ constrained → suspended → revoked/expired → retired`.

Administrative correction may return `pending-validation` to `active`; renewal may replace an expiring revision. `revoked`, `expired`, and `retired` are terminal revisions, although a successor registration can be linked. “Quarantine-only” is a capability/handling restriction, not a lifecycle state; quarantined submissions remain separate custody records. **[P]**

Credential rotation creates a new credential binding and may allow a bounded overlap before retiring the old binding. Software-instance replacement creates a new instance ID linked as successor, even when it uses the same logical publisher product. Emergency revocation prevents new admission immediately after policy distribution reaches the enforcement point, but never erases prior provenance or audit. In-flight/ambiguous submissions are reconciled by submission ID and idempotency record rather than blindly replayed under a new identity. **[P]**

Trust is purpose- and time-specific. Static registration is eligibility evidence; authentication is observed identity; authorization is a request decision; payload validation is content evidence; source-health observations are operational evidence; and policy acceptance is a separate decision under a recorded policy version. None should collapse into a single trusted/untrusted flag. **[A/P]**

## 6. Publisher-versus-Server Responsibility Matrix

| Responsibility | Publisher/adapter | Glaux Server | Conflict rule |
|---|---|---|---|
| Source protocol and credentials | Acquire source traffic; hold source credentials when safely possible | Does not require source secrets merely to admit mapped content | Source authentication evidence is retained; publisher credential does not replace it |
| Bounded parsing/decompression | Decode source-native framing; reject locally obvious corruption | Reparse/verify the submitted media/envelope within server limits | Server parser result controls admission |
| Source-specific mapping | Select registered mapping profile; preserve source fields/raw digest | Verify profile/version and canonical mapping; prohibit silent lossy coercion | Mismatch rejects or authorized raw material quarantines |
| Identifier capture | Preserve source IDs/namespaces and stable message IDs | Assign canonical ResourceId; enforce UID/namespace/parent/alias invariants | Publisher cannot assign server-owned identity fields |
| Schema/profile selection | Declare exact profile/media/schema and obtain dependencies | Resolve only authorized registry entries; repeat syntactic/semantic validation | Unknown or incompatible profile rejects |
| Units/CRS/time transforms | May transform only under a named mapping with original evidence | Validate semantics, transformation inputs, uncertainty, clocks, and canonical storage | Server retains both source and normalized evidence; no undocumented repair |
| Relationship resolution | Supply canonical/UID references it is authorized to assert | Resolve under visibility/authority and enforce type/cardinality/cycle rules | Missing dependency rejects unless trusted staging grant applies |
| Authentication context | Protect publisher credentials and channel | Authenticate and bind principal to active instance registration | Payload/header cannot override observed principal |
| Authorization/source authority | Request only granted operations and sources | Evaluate per request/item and hide protected existence | Denial creates no canonical effect |
| Policy markings | Convey source claims and issuer evidence without downgrading | Validate issuer/syntax, apply server policy/releasability, record decision | Caller cannot lower or self-approve handling requirements |
| Prevalidation | Strongly recommended for efficiency and source diagnostics | Always perform authoritative validation applicable to accepted effect | Publisher success never bypasses server checks |
| Idempotency/order evidence | Supply stable key/message ID and source epoch/sequence where required | Scope/deduplicate atomically; detect conflict/gap/replay; choose current by policy | Same key/different intent is `409` and audit signal |
| Buffering/retry | Bound local spool, preserve immutable content/IDs/version, obey backoff | Publish quotas/status, retain dedupe through authorized replay horizon | Retry never mints new identity to evade a prior outcome |
| Canonical persistence/current state | No direct access | Own transaction, revision, lifecycle/current projection, provenance, inbox/outbox, audit | No adapter direct database or broker-side canonical writes |
| Error mapping | Preserve server status/type/correlation and classify local failures | Return canonical safe problem/per-item result and retry hints | Publisher must not reinterpret rejection as success |
| Health reporting | Optionally report local connectivity/backlog/transform/credential observations | Record as attributed operational evidence; derive server-observed health | Health report never mutates System operational status |
| Audit/provenance | Supply end-source and transform evidence | Record observed principal, publisher, source, activity, inputs, decisions, outputs | Submitted attribution stays an assertion, not observed fact |

Safe publisher normalization is limited to registered, deterministic, reversible-or-evidenced transformations. The server may normalize representation into canonical storage but records exact received content/digest, mapping profile, warnings, defaulting, and output revision as required by IDR-SRV-019/023/031. **[A]**

## 7. Contract-Surface and Transport Analysis

### 7.1 Decision analysis

| Option | Benefits | Costs/risks | Standards/compatibility impact | Decision |
|---|---|---|---|---|
| CSAPI-only with no supplemental contract | Small public surface | Cannot bind represented source, replay identity, async ambiguity, import custody, or operational limits consistently | Clean CSAPI but incomplete publisher operations | Reject |
| General private ingestion API with canonical envelope | Uniform-looking transport and possible throughput | Duplicates CSAPI write semantics, creates mapping drift and a second domain API | Weakens conformance clarity and client reuse | Reject unless later benchmark evidence reopens a narrowly scoped need |
| CSAPI writes plus GPC-v1 metadata and narrow adapters | Reuses standards paths while adding source/replay/operational evidence | Requires project profile, registry, and adapter conformance tests | Preserves canonical CSAPI body/paths; extensions clearly separated | **Adopt** |
| Broker as canonical ingestion model | Natural fan-in and buffering | Broker semantics leak into domain, QoS mistaken for commit, harder errors/authorization | Part 3 remains draft; no approved binding selected | Reject as canonical model; evaluate adapter in IDR-SRV-035 |

### 7.2 Adopted surfaces

Direct metadata and dynamic-resource publishers POST/PUT/PATCH/DELETE the exact registered CSAPI paths: `/systems`, `/deployments`, `/procedures`, `/samplingFeatures`, `/properties`, `/datastreams`, `/observations`, `/controlstreams`, `/commands`, `/feasibility`, `/systemEvents`, their canonical items, and the applicable nested child/status/result paths established by IDR-SRV-010/031. The contract registry determines which operations and representations exist; this report does not enable every method globally. **[N/A/P]**

The only publisher-private v1 paths adopted here are:

| Path | Operation | Purpose | Canonical effect |
|---|---|---|---|
| `/_glaux/publisher/v1/submissions/{submissionId}` | GET | Resolve `202`, timeout, duplicate, quarantine/pending state, or ambiguous commit | Read-only status; reveals only the authenticated publisher's authorized scope |
| `/_glaux/publisher/v1/import-jobs` | POST | Submit a bounded manifest/package already in approved custody; never arbitrary server-side URL fetch | Each item maps to the same typed command and independently atomic pipeline |
| `/_glaux/publisher/v1/import-jobs/{jobId}` | GET | Complete per-item state/result ledger and aggregate progress | Read-only; `partial` may describe aggregate only |
| `/_glaux/publisher/v1/health-reports` | POST | Optional attributed publisher-local health evidence | No CSAPI resource or System-status mutation |

Publisher registration management and administrative quarantine review are not publisher self-service APIs in this baseline. Their administrative interface belongs to security/configuration/operations topics. **[P]**

Broker subjects/topics are deliberately **not adopted** here. The contract matrix records them as “registry-supplied subject; unresolved in IDR-SRV-035.” File/import packages use the private job endpoint and a registered manifest; federation is a pull-side adapter that submits ordinary commands internally. A future high-rate private mutation endpoint is allowed only if reproducible performance work proves registered CSAPI operations cannot satisfy an accepted requirement. It must then preserve the same typed command, validation, atomic evidence, errors, and exact canonical mapping and must not be advertised as CSAPI conformance. **[P]**

### 7.3 Required 21-field contract matrix

| Publisher class | Represented source class | Exact endpoint/path or broker subject | HTTP method or message operation | Accepted media type/schema/profile | Operation/data class | Canonical payload mapping | Required envelope fields | Authenticated principal | Authorization scope | Validation responsibilities | Idempotency/ordering fields | Response status and headers or acknowledgement semantics | Retry classification | Error behavior | Provenance/audit requirement | Rate/backpressure constraint | DDIL behavior | Contract-version negotiation | Verification fixture | Unresolved issue |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Standards-native / Glaux Publisher | Registered feature/metadata source | `/systems`, `/deployments`, `/procedures`, `/samplingFeatures`, `/properties` and registered nested/item paths | Registered POST/PUT/PATCH/DELETE | Exact CSAPI JSON/GeoJSON/SensorML media/profile; merge-patch only where registered | Metadata create/replace/update/delete | Body is canonical CSAPI representation | GPC version, source ID, message ID; idempotency key per operation profile; digest/trace optional | Credential-bound principal and publisher instance | Source, family, operation, path/parent, namespace, tenant/mission | Publisher prevalidates; server repeats media/schema/semantic/reference/authority/policy/concurrency validation | Scoped idempotency key; source epoch/sequence when replayed; ETag preconditions on item mutation | Immediate `201` + `Location`, `200`/`204`; `202` + status `Location` only when registered | Network/timeout ambiguous; 429/503 transient; validation permanent; conflict conditional | Received artifact/digest, principal, publisher, source, mapping, validation/policy, revision/effect, correlation | Per principal/publisher/source/family fairness; 429 or 503 + `Retry-After` | Immutable spool/replay IDs and version; no arrival-order overwrite | GPC-v1 header plus advertised media/profile registry | Golden metadata create/update, duplicate, stale ETag, cross-source denial | Exact numeric limits owned by configuration/performance topics |
| Standards-native / Glaux Publisher | Datastream/controlstream source | `/datastreams`, `/controlstreams` and registered nested/item paths | Registered HTTP mutation | CSAPI JSON/SensorML and registered schema profile | Stream/control contract descriptions | Canonical CSAPI resource | Same as metadata; source message identity mandatory for replay | Credential-bound | Source plus stream/control namespace and operation | Server validates schema binding, protected contract changes, children/relationship effects | Key + ETag; source sequence is evidence, not revision | Same immediate/deferred HTTP contract | Contract conflicts conditional; malformed/unsupported permanent | Preserve exact contract/profile and source transform activity | Lower change rate; expensive validation bounded | Replay cannot silently change a stream contract with existing children | GPC-v1 + contract/media profile | Stream create, schema conflict with children, repeated PUT | Dynamic contract evolution details in IDR-SRV-034/036 |
| Observation/status publisher | Sensor/status source | `/observations`, `/datastreams/{id}/observations` | POST/registered batch behavior | Parent-selected SWE/JSON/Text/Binary observation encoding/profile | Observation or status sample | Decode under parent DataStream; store canonical fact/evidence | GPC/source/message; epoch/sequence strongly required; batch item ID if grouped; content digest by high-assurance profile | Credential-bound | Source + parent stream + append operation + policy | Publisher checks frame/schema; server repeats frame/schema/unit/time/FOI/reference/authority/policy and current-selection inputs | Key or registered source-message mapping; epoch/sequence; no global order | `201` + `Location`; async `202` + status URI; batch per-item ledger | Late valid data accepted as history; missing parent rejects or trusted staging; overload transient | Exact bytes/digest, clocks, stream contract revision, source/publisher transform and accepted fact | Per-source stream quotas, bounded parse/decompress, 429/503 | Required immutable spool, original IDs/epoch/sequence/source time; gap explicit | GPC-v1 + parent data profile | Duplicate, late, gap, epoch reset, invalid frame, DDIL replay | Watermarks/current semantics in IDR-SRV-034; wire streaming in 035 |
| Command/status gateway | Executor/controller | `/commands/{id}/status`, `/commands/{id}/result`, `/feasibility/{id}/status`, `/feasibility/{id}/result`, registered item routes | POST/PUT as registered | Parent command/control schema/profile | Task lifecycle status/result | Canonical subordinate task resource | GPC/source/message/key required; epoch/sequence where source ordered | Credential-bound gateway | Exact command/control source, state transition, result operation, tenant/mission | Server checks sender/grant, parent state, transition, schema, concurrency, safety-policy inputs | Mandatory key and transition identity; parent revision/precondition | Immediate committed status/result or `202` status URI; never equate with physical execution | Ambiguous must query/retry same key; stale/invalid transition conditional/permanent | Principal, gateway, represented executor, command/revision, decision, output, audit | Strict command-specific quotas/deadlines; no priority self-assertion | Replay same transition identity; never generate new key after uncertainty | GPC-v1 plus task profile | Duplicate terminal state, cancel/result race, unauthorized executor | Final lifecycle/safety in IDR-SRV-036/038 |
| Legacy adapter | Legacy endpoint/device/file | Applicable canonical CSAPI paths after mapping | HTTP mutation | Registered mapping profile plus canonical media; raw only through import job | Mapped metadata/data/event/task result | Adapter output maps exactly to canonical body; original retained by digest/reference | All GPC identity fields; mapping version; digest required when raw evidence retained | Credential-bound adapter | Source, mapping, namespace, family/operation; optional quarantine right | Adapter parses/maps; server repeats all canonical checks and verifies mapping/profile | Key/message mandatory; source sequence when available; explicit absence/gap | Canonical HTTP response; quarantine/status only if granted | Parse/map permanent locally; server conflict conditional; transport transient | Original digest/custody, adapter build/profile, source evidence, server transform | Source/profile quotas and decompression limits | Buffered replay within grant; missing sequence recorded, not invented | GPC-v1 + mapping profile revision | Legacy golden mapping, unknown unit, lossy mapping, source collision | Raw retention period and exact mappings remain deployment/profile data |
| Tactical gateway | Multiple intermittent edge sources | Applicable canonical paths | HTTP mutation or future registered broker operation | Canonical media/mapping profiles | Mixed source publications, each item source-bound | Each item independently maps to one canonical command | Per-item source/message/epoch/sequence; key; batch ID; immutable profile; digest | Credential-bound gateway | Explicit per-source grant or constrained namespace; mixed batch only if manifest permits | Server evaluates every item independently; gateway-level auth never authorizes an undeclared source | Mandatory item keys/message IDs and source epoch/sequence | Per-item results; async job status; receipt never commit | Backoff and same-key replay; unauthorized source permanent; missing dependency conditional only if grant | Both gateway hop and end-source evidence, spool/replay activity, gaps | Fairness per gateway and source; bounded batch/spool ingress | Core case: continuity/gaps, reconnect, conflict, revocation during spool | GPC-v1; pinned mapping/profile per spooled item | Multi-source DDIL replay, revoked source, duplicate and gap | Detailed reconciliation/authority conflict in IDR-SRV-042/043 |
| Broker bridge | Registered broker sources | **Not adopted here:** registry-supplied subject owned by IDR-SRV-035 | Future consume/publish operation | Future binding; body must retain canonical or exact registered mapping | Same canonical resource/data commands | Bridge maps message to identical typed command; no direct DB | Source/message/contract/digest plus broker topic/partition/offset/QoS as evidence | Bridge principal/connection plus registered publisher | Subject/source/family/operation binding | Bridge framing plus full server validation | Source message + broker offset/epoch; inbox uniqueness; no QoS-as-idempotency | Broker ACK only after configured durable threshold; application outcome via future binding | Redelivery expected; poison/permanent isolated; capacity retry via broker controls | Broker receipt/offset, bridge build, end source, validation/commit and ACK timing | Receive Maximum/session/consumer flow controls plus Glaux quotas | Session expiry/message expiry cannot define canonical retention; replay IDs preserved | Future GPC transport binding/version in IDR-SRV-035 | Commit-before-ACK crash, ACK-before-commit prohibited, redelivery, poison | Protocol, exact subject, application ACK and Part 3 adoption in IDR-SRV-035 |
| File/batch importer | Controlled package/export/removable media | `/_glaux/publisher/v1/import-jobs`; `.../{jobId}` | POST manifest; GET status | `application/json` GPC-v1 manifest referencing approved-custody objects; per-item declared media/profile | Import job containing independently atomic items | Every item maps to ordinary canonical command; manifest is not domain data | Job/source/custody/package digest; item source/message/key/profile/digest; no arbitrary URL | Credential-bound importer/operator | Import, source, package profile, custody, quarantine and family/operation grants | Manifest/package limits first; then full per-item checks | Job key plus per-item keys/message IDs; item sequence if meaningful | `201`/`202` + `Location`; GET complete item ledger; aggregate `partial` allowed | Item-specific; job resumes pending items with same identity; package corruption permanent | Package/custody/digests, operator/importer, each item activity/result | Strict size/count/time/concurrency and storage quotas | Offline package accepted only within authority/replay windows; no silent stale overwrite | URI path fixes v1; manifest declares GPC/mapping versions | Mixed valid/invalid items, resume after crash, package digest mismatch | Object-store mechanism and numeric limits in architecture/deployment topics |
| Federated-source adapter | Remote CSAPI/approved service | Remote GET externally; applicable local canonical command internally | Poll/delta locally translated to typed mutation | Remote media/profile pinned; local canonical profile | Metadata/data synchronization | Preserve remote canonical/UID/revision evidence; assign local identity per IDR-016 | Remote endpoint/source/revision/cursor/digest plus local GPC-equivalent admission context | Local federation service principal | Endpoint/source/namespace/family and conflict policy | Validate remote response then full local admission; do not trust remote conformance claim alone | Remote revision/cursor + local key; epoch/reset explicit | Local command result; remote transport response is not local commit | Remote transient retry; cursor reset/reconciliation conditional | Remote URI/revision/artifact, adapter, principal, mapping, local effect | Per-remote/source fairness and circuit/backoff policy | Persist cursor only with/after local effect as specified; detect rollback/truncation | Federation profile + GPC internal contract | Remote rollback, missing revision, source collision, replay after outage | Full synchronization/conflict model in IDR-SRV-043 |
| Fixture publisher | Synthetic/test source | Same enabled production paths in isolated environment | Same operation under test credentials | Exact positive/negative corpus profile | All selected classes | No bypass; same pipeline and validators | Deterministic GPC fields and expected correlation | Test principal | Test-only tenant/environment/source | Same server validation; fault injection outside canonical contract | Fixed keys/epochs/sequences and replay scripts | Expected exact status/problem/ledger | Test oracle defines classification | Complete evidence assertion for verification | Configured deterministic limits | Delayed/replay cases never cross into production | Pinned fixture contract and server capability document | Golden/invalid/adversarial fixture suite | Simulator-only controls belong to IDR-SRV-033 |

## 8. Payload, Envelope, Media-Type, and Mapping Rules

### 8.1 GPC-v1 logical envelope

GPC-v1 defines logical metadata independent of transport. The initial HTTP binding uses these case-insensitive field names:

| Logical field | HTTP binding | Requirement and trust treatment |
|---|---|---|
| Contract version | `Glaux-Contract-Version: 1` | Required for publisher-profile traffic; project field, not CSAPI |
| Represented source | `Glaux-Source-Id` | Required unless a registration fixes exactly one unambiguous source for the endpoint; always grant-validated |
| Source message ID | `Glaux-Message-Id` | Required for broker/import/DDIL and replay-prone publishers; recommended otherwise; opaque immutable source identity |
| Source epoch | `Glaux-Source-Epoch` | Required when sequences can reset or source sessions matter |
| Source sequence | `Glaux-Source-Sequence` | Required where the source exposes order/offset; decimal nonnegative value in the registered sequence domain |
| Batch identity | `Glaux-Batch-Id` | Required for an item in a publisher batch/import; aggregate correlation only |
| Operation idempotency | `Idempotency-Key` | Required by operation profile; opaque key scoped by registration/source/operation and request fingerprint |
| Content integrity | `Content-Digest` | Required for import/broker/high-assurance profiles; optional otherwise; server verifies and always computes retained evidence digest |
| Trace propagation | `traceparent`, optional `tracestate` | Optional observability context; never used for authorization, idempotency, source identity, or trust |

The principal and publisher instance are deliberately absent: Glaux derives them from the authenticated credential/channel and active registration. Source timestamp belongs in the canonical payload where the resource/data schema defines it; an envelope receipt timestamp is assigned by the server. Policy markings belong in an approved payload/profile or verifiable policy context. Arbitrary headers are untrusted claims and cannot downgrade server policy. **[P]**

HTTP method, canonical target, `Content-Type`, selected profile, content encoding, and conditional headers remain normal HTTP semantics rather than duplicated envelope members. The server stores their normalized values as admission evidence. A body is either canonical CSAPI content under the path's registered representation or an import manifest at the private job path. Raw/source-native bodies are not accepted at arbitrary CSAPI paths. **[N/A/P]**

### 8.2 Mapping and server-owned fields

Canonical mapping is registry-controlled by `(publisher contract version, mapping profile version, source class, operation/data class, media type/schema/profile)`. Mapping profiles state input grammar, output resource family, identifier/namespace mapping, unit/CRS/time treatment, missing/nil behavior, allowable defaults, loss/evidence rules, and verification fixtures. Unknown or incompatible versions reject; the server does not guess from payload shape. **[P]**

ResourceId, revision/ETag, receipt/transaction times, validation/policy outcome, canonical links, derived extents/current selection, provenance activity IDs, audit correlation, and submission/job state are server-assigned. A submitted value in a server-owned field is rejected or ignored only where the registered canonical projection explicitly specifies ignore behavior; silent acceptance must not create an authorship ambiguity. UID, source identity, phenomenon/result time, and asserted actor may be publisher-supplied only under their resource schema and authority grant. **[A/P]**

CloudEvents demonstrates that producer, source, and intermediary can be separate and that `source` plus `id` can identify duplicate events.[^9] Glaux does not adopt CloudEvents as its canonical envelope here because many submissions are resources rather than events and CSAPI bodies already carry domain semantics. A future Part 3/event adapter may map CloudEvents attributes to GPC evidence without changing canonical payloads. **[I/P]**

## 9. Validation, Acceptance, Quarantine, and Provenance Behavior

### 9.1 Ordered admission and outcomes

Every path applies the accepted IDR-SRV-031 stages as relevant: bounded transport admission; authentication; registration/contract resolution; coarse authorization; safe decode; identity/idempotency admission; schema/profile validation; semantic/unit/time/geospatial validation; relationship/dependency resolution; source-authority and policy checks; canonical normalization; concurrency/invariant checks; atomic persistence/provenance/outbox/audit; then response and post-commit publication. Authorization and safe redaction can precede detailed errors so protected resource existence is not disclosed. **[A]**

| Outcome | When permitted | Canonical visibility | Publisher evidence |
|---|---|---|---|
| Reject | Malformed, unsupported, unauthenticated, unauthorized, invalid, policy-denied, conflict, or ungranted missing dependency | None | Safe problem/per-item result and correlation, subject to anti-enumeration policy |
| Accept with warnings | Only when warnings do not violate a required invariant and the profile names the warning class | Committed result | Warnings attached to validation/provenance and safe response |
| Quarantine | Explicit raw/import/security custody grant; suspicious or unmappable material worth retaining | None; cannot satisfy references, queries, current state, commands, or publication | Opaque status/correlation; detailed evidence admin-only where sensitive |
| Pending dependency | Explicit trusted contract, bounded eligible dependency class, TTL/size/owner/status and later revalidation | None until full validation and commit | Durable status; expiry/rejection visible within publisher scope |
| Duplicate | Same scoped key/message and same normalized intent | Original effect only | Original safe response/state; no second effect |
| Conflict/security signal | Same key/message but different fingerprint, source collision, invalid authority overlap | None for attempted effect | `409` or safe policy response; audit escalation |

Public direct publishers do not receive a general “store now, validate later” mode. A batch item is never accepted merely because neighboring items are valid. Quarantine and pending dependency have independent quotas and cannot become covert unbounded queues. **[A/P]**

### 9.2 Provenance and audit minimum

For each attempted submission, retain according to policy: server receipt/submission ID; observed principal and credential binding reference; publisher instance and build/profile; represented source and authority grant revision; route/method/media/contract; message/idempotency/epoch/sequence/batch identities; exact-byte/content digest and raw artifact reference where retained; source, receipt, validation, transaction, and publication clocks; mapping/normalization inputs; validation findings; authorization/policy decision references; resulting ResourceId/revision or non-effect outcome; inbox/idempotency state; and safe correlation among problem, audit, provenance, job, and outbox records. **[A/P]**

Provenance describes what influenced the accepted entity and who was observed/asserted. Audit records attempted security-relevant actions, including denied, throttled, conflict, quarantine, credential, trust-state, and administrative operations. Neither should embed unrestricted payloads, credentials, sensitive topology, or policy reasons. A caller-supplied author/source remains an assertion distinct from the observed authenticated principal. **[A/P]**

## 10. Delivery-State, Batching, Idempotency, Ordering, Retry, Replay, and Acknowledgement Behavior

### 10.1 Delivery-state model

The externally meaningful progression is:

`received → authenticated → contract-resolved → validated → accepted-for-commit → committed → publication-pending → published`.

Possible side/terminal outcomes are `rejected`, `quarantined`, `pending-dependency`, `duplicate`, `retryable-failed`, and `unknown/reconcile`. States may be collapsed in a synchronous response, but their meanings may not be conflated. `published` does not mean every subscriber consumed the message, and task-resource commit does not mean physical command execution succeeded. **[A/P]**

### 10.2 Idempotency and ambiguity

The durable uniqueness scope is `(tenant/security domain, publisher registration, represented source, operation family and canonical target scope, idempotency key)`. The normalized fingerprint includes method/message operation, canonical target, media/profile/contract versions, exact-byte digest where material, and canonical semantic intent. Same scope/key/fingerprint returns the stored safe result or bounded in-progress state; same scope/key with different intent returns `409` with a stable problem type and audit signal. Authentication/authorization is reevaluated before revealing an old result. **[A/P]**

`Glaux-Message-Id` is evidence of source identity; `Idempotency-Key` identifies retry intent. A registered one-message-to-one-operation profile may map the former into the latter, but this equivalence must be explicit and immutable. Hash equality alone is never duplicate identity. Retention must cover the longest authorized publisher replay horizon, ambiguity-recovery period, command safety/lifetime where applicable, and any still-pending job; numeric periods remain deployment policy and must be externally documented. **[A/P]**

After a timeout or connection loss, the publisher queries the submission URI when known and otherwise retries the identical operation with the same key. It must not generate a new key to escape uncertainty. The server's `unknown/reconcile` state blocks a second effect until storage outcome is reconciled or a documented safe terminal decision exists. **[A]**

### 10.3 Ordering, replay, and batches

Glaux promises no global order and does not use arrival order as current-state truth. Source continuity is evaluated within a registered `(source, epoch, sequence-domain)`; gaps, duplicates, resets, reused sequence values, and out-of-order delivery are explicit evidence. Domain phenomenon/result/effective times and versioned source-priority/current-selection rules control history/current projections. Multiple publishers for one source need shared upstream message/epoch semantics or are treated as separate streams until reconciled. **[A/P]**

A replay/backfill preserves original source/message identity, epoch/sequence, source time, content, contract and mapping version, and adds a new ingestion/replay activity. A correction is a new authorized revision/relation, not a duplicate disguised with a new key. Local publisher spool behavior is outside the server, but the contract requires bounded custody, integrity, immutable replay metadata, expiry behavior, and backoff. **[P]**

Default batches are independently item-atomic. Each item has its own authority decision, validation, idempotency, transaction, result, and evidence; the job/response ledger includes every item. Mixed sources, profiles, or authority domains are forbidden unless a manifest identifies each item and the registration explicitly permits the mixture. Aggregate `partial` means some items reached terminal success and others did not; no item itself is partially committed. All-or-none batch semantics require a separately bounded operation and are not part of GPC-v1. **[A/P]**

### 10.4 Acknowledgements

Immediate create commit returns `201` and canonical `Location`; eligible update/delete returns `200` or `204`; deferred work returns `202`, status `Location`, and `Retry-After` where meaningful. The status resource exposes safe state, outcome, canonical result link, item ledger where applicable, retry disposition, and correlation—not protected internals. Duplicate replay returns the original safe outcome and identifiers rather than creating a new effect. **[A/P]**

For MQTT, QoS 1 PUBACK and QoS 2 handshakes concern transfer between MQTT Client and Server; Session Expiry, Receive Maximum, reason codes, and Message Expiry govern that protocol session.[^8] They do not prove Glaux validation or database commit. A bridge may acknowledge only after its documented durability threshold, and the eventual application outcome must be separately correlated. The exact Part 3/broker acknowledgement binding remains IDR-SRV-035. **[N/P]**

## 11. Error, Correlation, Diagnostics, and Source-Health Behavior

### 11.1 Canonical classification

HTTP failures use RFC 9457 `application/problem+json` with stable Glaux problem type, safe title/detail, status, instance or correlation, retry classification, and field/item pointers where authorized. Batch/job results use the same per-item problem model. Broker bindings must preserve equivalent stable codes and correlation; they may not collapse all rejection into transport failure. **[N/P]**

| Condition | Typical HTTP outcome | Retry class | Disclosure rule |
|---|---|---|---|
| Malformed framing/JSON/envelope | `400` | Permanent until corrected/new key as applicable | Safe syntax location, bounded excerpt never required |
| Missing/invalid authentication | `401` | Conditional after credential correction | No source/resource existence |
| Authenticated but ungranted source/operation/policy | `403` or concealment-compatible `404` | Permanent until grant/policy changes | No protected topology or rule internals |
| Missing canonical dependency | `404` on authorized direct relation; trusted staging status where contracted | Conditional after dependency or staging | Do not reveal inaccessible dependency |
| Unsupported media/profile/encoding | `415` or registered capability error | Permanent until representation changes | Return only allowed values visible to caller |
| Schema/semantic/parent-contract invalid | CSAPI-required `400` or registered `422` project profile | Permanent until corrected | Safe pointers/codes; sensitive expected values redacted |
| Identity/state/authority/idempotency conflict | `409` | Conditional resolution; same-key/different-intent permanent | Opaque conflicting identity where needed |
| Missing required precondition | `428` | Add current required precondition | Capability/ETag only if authorized |
| False precondition | `412` | Refresh/reconcile | Current ETag/link only if authorized |
| Throttled publisher/source | `429` + `Retry-After` | Transient | Reveal own limit state only |
| Global overload/maintenance | `503` + `Retry-After` | Transient | No tenant/source internals |
| Timeout/disconnect/ambiguous commit | No response or safe `202`/status | Query then same-key retry | Never advise a new key |
| Quarantined | Opaque accepted-for-custody/status response under explicit grant; otherwise normal reject | Operator/profile dependent | No other-source or detection-rule detail |

RFC 6585 permits `429` with `Retry-After`, and RFC 9110 defines `Retry-After` for temporary unavailability.[^3][^1] The current IETF RateLimit fields remain an active draft; GPC-v1 therefore requires the stable status/`Retry-After` baseline and leaves draft rate fields to a future versioned optional profile.[^11]

### 11.2 Correlation and diagnostics

The server assigns one opaque `submissionId` to every admitted attempt and a separate `jobId` for aggregate import work. Problems, provenance, audit, idempotency, canonical result, validation evidence, and outbox records cross-reference these identifiers internally. Responses expose only the identifiers and links needed by the authorized publisher. W3C trace context may correlate distributed telemetry, but sampling flags and caller-provided trace state are untrusted operational inputs and cannot become business identity or authorization.[^7]

Publisher diagnostics may include stable failure code, retry class, safe pointer/item ID, supported contract/media/profile where disclosure is allowed, `Retry-After`, canonical result/status link, and warnings. Full stack traces, SQL/broker names, credentials, signing material, policy rationale, other-source identifiers, restricted payload fragments, internal topology, and hidden resource existence are administrator-only or prohibited. **[P]**

### 11.3 Source and publisher health

Glaux separately records:

- **server-observed publication health:** last contact/success/rejection, latency, backlog at the server, throttling, gaps, credential/grant state, mapping errors, and commit/publication lag;
- **publisher-reported operational evidence:** its source connectivity, local backlog age/count bands, last source capture/publication, transform failures, credential-expiry warning, software/build, and report time; and
- **domain operational status:** CSAPI DataStream/Observation status data about a represented system.

Only the first two inform publisher operations. `POST /_glaux/publisher/v1/health-reports` is optional and grant-controlled; reports are attributed claims, cannot set the server's computed health, cannot self-approve credentials or authority, and never create a System status Observation. Sensitive diagnostics are bucketed/redacted and scoped to the reporting publisher/source. **[P]**

## 12. Backpressure, DDIL, Security, Policy, Audit, and Evolution Implications

### 12.1 Backpressure and DDIL

Glaux enforces bounded body, decompression ratio, parse complexity, item count, batch bytes, concurrent submissions, pending dependencies, quarantine storage, idempotency records, job retention, per-source rate, per-publisher rate, and global work queues. `429` represents caller/scope fairness; `503` represents global unavailability/load shedding. `Retry-After` is advisory minimum delay, not a reservation. Priority is server policy and cannot be raised by a publisher claim. Broker Receive Maximum/consumer credit can complement, but not replace, server admission quotas. **[P]**

During DDIL, the server makes no promise while unreachable. On reconnect, the publisher resumes under its active registration with immutable source/message/contract metadata, obeys backpressure, and reports gaps rather than fabricating continuity. An expired/revoked grant blocks new effects even for older spooled content unless a separately authorized historical-import policy applies. Valid late data may enter history; current-state selection uses domain/source policy. Deduplication evidence remains live for the authorized replay horizon. Detailed freshness, reconciliation, divergent authority, and disconnected credential behavior remain IDR-SRV-042/043. **[P]**

### 12.2 Security, policy, and audit requirements

Category G must select authentication and credential mechanisms that support publisher-instance binding, least privilege, sender constraint where appropriate, rotation, emergency revocation, audience/environment separation, and replay resistance. OAuth Security BCP recommends asymmetric client authentication and sender-constrained access tokens; mTLS defines certificate-bound tokens while preserving client identity as a separate concept.[^12][^13] These are candidate requirements, not a mechanism selection here.

Authorization must reach every item after source extraction and before protected existence is disclosed. Grants constrain source, family, operation, parent/path/namespace, tenant/mission/environment, policy labels/issuers, command capability, transport, contract/profile, raw/quarantine/import rights, and time. Policy markings are validated claims; server policy controls acceptance, storage, redaction, publication, and audit. HTTP Message Signatures may support a later high-assurance profile only after the application defines covered components, key/authority semantics, freshness, and replay behavior; a signature alone does not establish content truth.[^14]

Audit events include registration/grant/trust changes; authentication and authorization failures; source/identity/idempotency conflicts; validation and policy denials; acceptance/commit/quarantine/pending/expiry; retries and ambiguous reconciliation; batch/job administration; throttling/abuse; credential rotation/revocation; health reports; and publication/acknowledgement transitions. Sensitive content follows the accepted lifecycle/retention policy rather than being copied into logs. **[A/P]**

### 12.3 Evolution and compatibility

Four version domains remain separate:

1. approved CSAPI/SensorML/SWE resource and encoding versions;
2. GPC transport-neutral and HTTP binding version (`Glaux-Contract-Version` and `/_glaux/publisher/v1/...`);
3. source-to-canonical mapping/schema/profile versions; and
4. future transport binding versions, including any broker/Part 3 profile.

Capability discovery advertises exact operation/media/profile/GPC combinations; registration intersects that capability with a publisher's grants. A submission selects one compatible set and the server records it. Unknown major GPC versions reject without guessing. Additive optional fields require declared semantics and ignore/forward rules; semantic breaking changes use a new major version/path. Deprecation publishes dates and successor capability, supports measured overlap, and preserves replay of already-admitted work under its recorded contract. Rolling upgrades must accept old/new versions for the declared overlap and keep idempotency fingerprint interpretation stable. **[P]**

Mutable standards/drafts and implementation pins are rechecked at implementation and conformance freeze. An eventual approved Part 3 can add a binding only after a delta review proves how its event identity, content negotiation, batch, acknowledgement, replay, error, and security semantics map to this boundary. **[P]**

## 13. Fixture and Interoperability Matrix

| Fixture/suite | Publisher/source | Stimulus | Required evidence/assertion | Downstream owner |
|---|---|---|---|---|
| GPC metadata golden set | Standards-native single source | Valid POST, conditional update/delete, media variants | Canonical result, Location/ETag, actor/source separation, evidence transaction | IDR-050/051/054 |
| Principal/publisher/source permutations | Multi-instance/multi-source | Same principal different instances; same source different publishers; transfer/revocation | Correct grant decision and immutable historical attribution | IDR-039/039A/041 |
| Cross-source denial corpus | Gateway/adapter | Authorized publisher asserts ungranted source/namespace/parent | No canonical effect or existence leak; safe audit | IDR-039/040/053 |
| Mapping corpus | Legacy adapter | Golden, lossy, unknown unit/CRS/time, malformed raw | Exact mapping/profile/digest; server repeats validation; quarantine only by grant | IDR-022/023/054 |
| Idempotency/ambiguity suite | All replay-prone classes | Same key/same bytes, same key/different intent, concurrency, timeout after commit | Single effect, original result, `409` conflict, reconcile path | IDR-029/052/054 |
| Ordering/DDIL suite | Tactical gateway | Late/out-of-order, gaps, epoch reset, duplicate, replay after outage/revocation | History retained, no arrival overwrite, gap/conflict evidence, bounded retry | IDR-034/042/043 |
| Batch/import suite | File importer/mixed sources | Valid+invalid items, package digest failure, crash/resume, forbidden mixed authority | Complete item ledger, independent atomicity, same-key resume, aggregate-only partial | IDR-044/052/054 |
| Missing-dependency suite | Direct and trusted staging | Unknown parent before/after child, inaccessible dependency, expiry | Direct safe reject; bounded non-canonical staging only by grant; full revalidation | IDR-023/031/034 |
| Broker crash matrix | Broker bridge | Redelivery; crash before commit; commit before ACK; poison; backlog | No ACK-before-durability, single local effect, stable identity, safe poison handling | IDR-035/044/052 |
| Command gateway race suite | Executor gateway | Duplicate terminal update, stale status, cancel/result race, unauthorized source | Serialized valid transition, no duplicate physical implication, complete audit | IDR-036/038/053 |
| Health separation suite | Publisher health reporter | Backlog/connectivity report resembling System status | Operational evidence only; no CSAPI status mutation or trust self-escalation | IDR-039/046/047 |
| OSH v2.0.2 comparison | OSH publisher | REST/WebSocket/batch behaviors at commit `235c0e...` | Compatibility deltas classified; thin errors/missing proven idempotency not copied | IDR-054/056 |
| Connected Systems Go v1.0.4 comparison | CS-Go publisher/broker | REST and optional MQTT behaviors at commit `244f4dd...` | Useful mapping/broker precedent, but auth/QoS/replay gaps remain informative | IDR-035/054/056 |
| 52North pygeoapi PoC comparison | Narrow standards producer | Supported mutation/auth surface | Basic-auth/narrow behavior treated only as PoC evidence | IDR-054/056 |
| OS4CSAPI client fixtures | Client/fixture publisher | Standard route/media/error corpus | No publisher extension silently changes CSAPI conformance | IDR-050/051/054/056 |
| Resource-exhaustion/adversarial set | Untrusted publisher | Oversize/decompression/batch flood, retry storm, sensitive invalid references | Bounded resource use, 429/503, redaction, audit, recovery | IDR-039/048/052/053 |

Implementation studies demonstrate feasibility but not an interoperable contract. OSH shows typed store/handler seams and batch behavior but no proven end-to-end publisher identity/idempotency/broker contract. Connected Systems Go demonstrates optional MQTT and configuration-derived AsyncAPI, but its reviewed release does not prove REST authentication, broker ACL/TLS, replay, idempotency, or application commit acknowledgement. The pygeoapi work is narrower and uses Basic authentication. Glaux uses these as negative and positive fixtures, never as normative authority. **[I]**

## 14. Downstream Handoff Matrix

| Topic/owner | Fixed input from IDR-SRV-032 | Decision still owned downstream |
|---|---|---|
| IDR-SRV-033 Simulator boundary | Fixture publishers use the same GPC/common write pipeline and test-only grants | Reset/scenario/synthetic clock/deterministic replay controls |
| IDR-SRV-034 Dynamic semantics | Source/message/epoch/sequence, late/history rules, item atomicity, parent profile binding | Observation/status watermarks, current/latest, correction, dynamic batch semantics |
| IDR-SRV-035 Streaming/events | Broker is an adapter; QoS ACK is not commit; exact source identity and application outcome required | Part 3 adoption, protocol/broker, subjects, subscription/publication ACK, replay/backpressure binding |
| IDR-SRV-036/037 Task lifecycle | Gateway identity/grant, mandatory idempotency, parent state/precondition, commit not execution | Command/Feasibility lifecycle, status/result and asynchronous task rules |
| IDR-SRV-038 Command safety | No authority from possession of ID; source/principal/publisher separate; complete audit inputs | Command authorization, safety interlocks, emergency/cancellation policy |
| IDR-SRV-039/039A Security/zero trust | Required identity bindings, grant dimensions, sender constraint/rotation/revocation inputs, anti-enumeration | Authentication/token/certificate architecture and threat controls |
| IDR-SRV-040 Policy | Publisher markings are claims; server owns policy acceptance and redaction | Label syntax, issuer trust, releasability and cross-boundary policy |
| IDR-SRV-041 Audit | Required publisher/source/attempt/state/conflict events and correlations | Audit schema, protection, retention, review/export |
| IDR-SRV-042/043 DDIL/sync | Immutable replay identity/version, gaps/epochs, no arrival overwrite, grant-at-admission, cursor/effect coupling | Freshness, reconciliation, divergent authority and disconnected security details |
| IDR-SRV-044–049 Architecture/deployment | One pipeline, narrow versioned surfaces, registries, durable status/inbox/job, health and quota needs | Components, stores, queues, configuration, observability, deployment limits |
| IDR-SRV-050–056 Verification | Exact fixtures in §13 and contract matrix in §7.3 | Test harness, conformance, performance, security, interoperability execution |

## 15. Recommendations and Explicit Project Decisions

1. **Adopt standards-facing CSAPI mutations as the default publisher surface; reject a general private canonical ingestion API. [P]** A later private high-rate mutation path requires reproducible benchmark evidence and exact mapping to the same typed command/evidence transaction.
2. **Adopt GPC-v1 as a supplemental Glaux contract, not an OGC conformance class. [P]** Its initial HTTP binding is the field/path contract in §§7–8; generated documentation and capability discovery must label all `Glaux-*` fields as project extensions.
3. **Keep principal, publisher instance, represented source, source authority, and asserted payload actor separate. [A/P]** Derive principal/publisher from credential binding; validate represented-source claims against versioned grants.
4. **Use the accepted single write pipeline for direct HTTP, broker, import, federation, gateway, and fixtures. [A]** No adapter may directly write canonical tables/current projections or declare broker delivery to be canonical commit.
5. **Retain server authority for canonical identity, validation, policy, provenance, persistence, audit, errors, and conformance. [A/P]** Publisher preprocessing is efficiency/evidence, not a validation bypass.
6. **Require stable retry identity for effectful and replay-prone operations. [A/P]** Scope keys with publisher/source/operation and fingerprint intent; same-key/different-intent is a conflict. Treat the current Idempotency-Key draft spelling as a versioned project contract. 
7. **Adopt independently item-atomic batches with complete per-item ledgers. [A/P]** Mixed authority/profile/source batches require explicit manifest and grant; `partial` describes only aggregate job outcome.
8. **Separate transport receipt, validation, commit, publication, and domain execution. [A/P]** `202` needs a durable status URI; MQTT/broker acknowledgement alone never proves Glaux commit.
9. **Reject ordinary missing dependencies and invalid public traffic atomically. [A/P]** Permit bounded pending-dependency or quarantine only through explicit trusted/import rights, quotas, expiry, isolation, and full later validation.
10. **Treat source epoch/sequence as continuity evidence, not global/domain order. [A/P]** Preserve late/replayed history and select current state through domain/source policy.
11. **Use `429`/`503` plus `Retry-After` as the stable backpressure baseline. [N/P]** Numeric quotas and the draft RateLimit fields remain configuration/future-profile decisions.
12. **Adopt opaque submission/job correlation and safe RFC 9457 problems. [N/P]** Do not expose other-source existence, topology, protected content, credentials, internal components, or detailed policy logic.
13. **Keep publisher health evidence distinct from system operational status. [P]** Optional health reports are attributed claims and cannot self-change trust, registration, or CSAPI status.
14. **Version standards, GPC, mapping profiles, and transport bindings independently. [P]** Preserve replay under the recorded version and require explicit compatible overlap for rolling upgrades.
15. **Do not select or implement draft Connected Systems Part 3 here. [P]** IDR-SRV-035 must delta-review the accepted Part 3 evidence against this contract before choosing any publish/subscribe binding.

## 16. Risks, Contradictions, Assumptions, and Unresolved Questions

| Item | Treatment now | Required follow-up |
|---|---|---|
| CSAPI relies on a draft CRUD dependency and contains write/OAS inconsistencies | Use approved CSAPI text plus accepted Glaux strict profile and generated registry; disclose gaps | Recheck at implementation/conformance freeze; IDR-050/051 |
| `Idempotency-Key` latest reviewed draft is expired and not an RFC | Use its spelling only as a GPC-v1 project field; define Glaux scope/fingerprint behavior fully | Monitor IETF outcome and delta-review before implementation freeze |
| Current RateLimit header work is a draft | Do not require it; use stable 429/503 and `Retry-After` | Optional future profile after publication/stability review |
| Part 3 working draft and implementation MQTT behaviors may change/diverge | No broker subject/protocol selected; preserve transport-neutral contract | IDR-SRV-035 owns current delta and adoption decision |
| Exact numeric rate, size, batch, quarantine, staging, replay, and retention limits are deployment-dependent | Require every limit category and bounded behavior, but invent no numbers | IDR-SRV-044–049/052 configuration and performance evidence |
| Multiple publishers may legitimately represent one source | Require explicit overlapping grants and source conflict/priority rule; never arrival-wins | IDR-SRV-040/043 policy and reconciliation |
| Source lacks stable sequence/message identity | Record absence, use server receipt plus registered adapter identity, do not invent loss-free continuity | Source mapping profile and IDR-SRV-042/043 |
| Health reports can leak topology or be mistaken for domain status | Optional narrow path, scoped/redacted attributed evidence, no CSAPI mutation | IDR-SRV-039/040/046/047 |
| `202` status retention may expire before a delayed publisher reconciles | Require retention across authorized ambiguity/replay horizon and document expiry | Lifecycle/configuration/performance topics |
| Private path prefix/name may conflict with later API governance | Treat `/_glaux/publisher/v1` as proposed GPC-v1 project namespace | Confirm in architecture/API-definition work without changing semantics |
| Controlled AEP material cannot be redistributed in this report | Rely only on accepted releasable findings and record exact controlled baseline | Project lead controls access and later evidence handling |

Assumptions are limited to accepted IDR-SRV-031 availability, deployment-supplied bounded policies, and the ability of Category G/H work to realize the required identity/configuration/status mechanisms. None authorizes server implementation. No success criterion depends on Part 3 becoming approved or on a particular broker.

## 17. Validation Against This Plan's Success Criteria

| Topic plan success criterion | Status | Evidence |
|---|---|---|
| Every publisher/source class has role, identity, registration, authority treatment | Met | §§4–5 and matrix §7.3 |
| Principal, publisher instance, represented source, source authority explicit | Met | §2.3, §5, recommendations 3 |
| Decide standards operations versus private surface | Met | §7.1–7.2 and recommendation 1–2 |
| Allocate parsing, mapping, validation, normalization, policy, provenance, persistence, errors, audit | Met | §6 and §9 |
| Specify payload/envelope, batching, acknowledgement, delivery, idempotency, ordering, retry, replay, backpressure | Met | §§7.3, 8, 10, 12.1 |
| Do not conflate authentication, authorization, source trust, validation, policy | Met | §§2.3, 5.2, 6, 12.2 |
| Cover record, batch, transport, ambiguous errors without leaks | Met | §§9–11 |
| Address evolution and capability/version compatibility | Met | §12.3 |
| Trace or label recommendations | Met | Labels throughout; §15; numbered sources §18 |
| Make fixtures and simulator/dynamic/security/DDIL/architecture/deployment/testing handoffs explicit | Met | §§13–14 |
| Polished, recommendation-first, independently readable, self-contained | Met for review | §1 plus complete 18-section report and exact 21-field matrix |

Research phases 1–6, report drafting, and author review are complete. Plan-owner acceptance remains deliberately unchecked while this report is **In Review**. The next two workflow actions are: (1) the Glaux Project Lead accepts IDR-SRV-032 after review; and (2) in the same instruction, authorizes execution of exactly IDR-SRV-033. The combined response pattern is: **`accept IDR-SRV-032 and proceed`**. Under the established workflow, a later bare **`proceed`** may express that combined action. This report neither records its own acceptance nor starts IDR-SRV-033.

## 18. References

### Standards and controlled/project sources

1. [OGC API - Connected Systems - Part 1: Feature Resources, OGC 23-001, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html).
2. [OGC API - Connected Systems - Part 2: Dynamic Data, OGC 23-002, Version 1.0](https://docs.ogc.org/is/23-002/23-002.html).
3. [OGC API - Connected Systems `v1.0.0`, commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2).
4. [OGC API - Features CRUD draft, document `20-002r2`, snapshot commit `4e30324a14b682ff4a26ee43aad1eb6428c846a3`](https://github.com/opengeospatial/ogcapi-features/tree/4e30324a14b682ff4a26ee43aad1eb6428c846a3/extensions/transactions/create-replace-update-delete).
5. [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html).
6. [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html).
7. [RFC 9530: Digest Fields](https://www.rfc-editor.org/rfc/rfc9530.html).
8. [RFC 6585: Additional HTTP Status Codes](https://www.rfc-editor.org/rfc/rfc6585.html).
9. [MQTT Version 5.0, OASIS Standard](https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html).
10. [CloudEvents Specification v1.0.2, commit `fc1f6f31f5f011a72183f1bcea20c987cb683ade`](https://github.com/cloudevents/spec/tree/fc1f6f31f5f011a72183f1bcea20c987cb683ade).
11. [W3C Trace Context, Recommendation 23 November 2021](https://www.w3.org/TR/trace-context/).
12. [Idempotency-Key HTTP Header Field, IETF HTTPAPI Working Group Internet-Draft 07, 15 October 2025, expired 18 April 2026](https://datatracker.ietf.org/doc/draft-ietf-httpapi-idempotency-key/07/).
13. [RateLimit header fields for HTTP, IETF HTTPAPI Working Group Internet-Draft 11, 23 May 2026](https://datatracker.ietf.org/doc/draft-ietf-httpapi-ratelimit-headers/11/).
14. [RFC 9700: Best Current Practice for OAuth 2.0 Security](https://www.rfc-editor.org/rfc/rfc9700.html).
15. [RFC 8705: OAuth 2.0 Mutual-TLS Client Authentication and Certificate-Bound Access Tokens](https://www.rfc-editor.org/rfc/rfc8705.html).
16. [RFC 9421: HTTP Message Signatures](https://www.rfc-editor.org/rfc/rfc9421.html).
17. Project-controlled AEP baseline `AC/224(JCGISR)D(2026)0005`, 27 April 2026, SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`; not redistributed; used through accepted project findings.
18. [OGC API - Connected Systems Upstream Standards-History Evidence Register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md).
19. [IDR-SRV-019 Provenance, Lineage, Quality, and Trust Metadata Model](idr-srv-019-provenance-lineage-quality-and-trust-metadata-model-report.md).
20. [IDR-SRV-023 Schema and Encoding Validation Strategy](idr-srv-023-schema-and-encoding-validation-strategy-report.md).
21. [IDR-SRV-029 Transaction, Consistency, Idempotency, and Concurrency Strategy](idr-srv-029-transaction-consistency-idempotency-and-concurrency-strategy-report.md).
22. [IDR-SRV-031 Server Write and Ingestion Model](idr-srv-031-server-write-and-ingestion-model-report.md).

### Informative implementation sources

23. [OpenSensorHub `v2.0.2`, commit `235c0e`](https://github.com/opensensorhub/osh-core/tree/v2.0.2).
24. [Connected Systems Go `v1.0.4`, commit `244f4dd586da685d4d9b75e43f73001028b5bd0e`](https://github.com/OS4CSAPI/connected-systems-go/tree/v1.0.4).
25. [52North Connected Systems pygeoapi proof of concept](https://github.com/52North/connected-systems-pygeoapi).
26. [OS4CSAPI client and testing research, `phase-9`](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9).

[^1]: RFC 9110 controls the HTTP method, status, conditional-request, validator, and `Retry-After` semantics used here. Accessed 2026-09-14.
[^2]: RFC 9457 defines the reusable problem-details representation and warns that problem details can expose sensitive information. Accessed 2026-09-14.
[^3]: RFC 6585 defines `429 Too Many Requests` and permits a `Retry-After` field. Accessed 2026-09-14.
[^4]: Accepted IDR-SRV-019 requires separate observed principal, asserted actor, publisher/source, software agent, and provenance activity evidence; authentication or signatures do not prove semantic truth.
[^6]: RFC 9530 distinguishes digest of actual message content from representation digest and does not make a digest proof of authorship, authorization, or truth. Accessed 2026-09-14.
[^7]: W3C Trace Context standardizes `traceparent` and `tracestate` propagation and identifies trust/abuse considerations; Glaux limits it to observability. Accessed 2026-09-14.
[^8]: MQTT 5.0 defines QoS acknowledgement flows, Session Expiry, Receive Maximum, reason codes, and Message Expiry within MQTT protocol behavior. The conclusion that these do not prove a separate Glaux database commit is an architectural inference. Accessed 2026-09-14.
[^9]: CloudEvents v1.0.2 requires `id`, `source`, `specversion`, and `type`, distinguishes source from producer/intermediary, and treats equal `source`+`id` as duplicate-identifying context. It is used only as informative precedent. Accessed 2026-09-14.
[^10]: The latest reviewed Idempotency-Key document is Working Group draft 07, dated 15 October 2025 and expired 18 April 2026. Glaux therefore adopts a locally complete, versioned field contract rather than claiming standardized RFC semantics. Accessed 2026-09-14.
[^11]: The reviewed RateLimit fields document is Working Group draft 11, dated 23 May 2026 and expiring 24 November 2026. It remains work in progress; GPC-v1 does not depend on it. Accessed 2026-09-14.
[^12]: RFC 9700 recommends asymmetric client authentication and sender-constrained access tokens where feasible; final applicability and mechanisms belong to Category G. Accessed 2026-09-14.
[^13]: RFC 8705 defines mutual-TLS client authentication and certificate-bound tokens. A certificate binding does not erase the distinct client, publisher-instance, source, and authority records required here. Accessed 2026-09-14.
[^14]: RFC 9421 requires an application to define the signature profile and covered components; verification establishes only the configured cryptographic claim, not semantic truth or authorization. Accessed 2026-09-14.

# Section 016: Identifier, URI, and Resource Lifecycle Strategy - Research Report

**Topic ID:** IDR-SRV-016<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-016 Identifier, URI, and Resource Lifecycle Strategy](../IDR%20Plans/idr-srv-016-identifier-uri-and-resource-lifecycle-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core questions, all detailed-question groups, all 6 methodology phases, and all success criteria<br>
**Methodology Used:** Authority-ranked synthesis of accepted IDR-SRV-001 through IDR-SRV-015; direct inspection of approved CSAPI 1.0 requirements, conceptual tables, schemas, tagged artifacts, SensorML 3.0, SWE Common 3.0, URI, HTTP, Web-linking, UUID, and IANA authorities; implementation and interoperability comparison; resource-family lifecycle, collision, security, persistence, and test analysis<br>
**Research Time:** Approximately 5 hours of AI-assisted execution on September 3, 2026<br>
**Official Standards Source Pin:** [`opengeospatial/ogcapi-connected-systems` `v1.0.0` at `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)<br>
**Shared Register Baseline:** OGC API - Connected Systems upstream-history register version 1.9; stable master remained `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f` at the September 3 refresh<br>
**Document Purpose:** Establish the canonical identifier classes, URI construction rules, alias and collision policy, entity identity guarantees, lifecycle vocabulary, deletion/tombstone behavior, and downstream implementation handoffs for the Rust Glaux reference server<br>
**Author:** OpenAI Codex<br>
**Accepted By:** TBD until Glaux Project Lead acceptance<br>
**Acceptance Date:** TBD until accepted<br>
**Date:** September 3, 2026<br>
**Last Updated:** September 3, 2026

---

## Reading Guide and Evidence Labels

| Label | Meaning |
|---|---|
| **N** | Normative or standards-derived finding from an approved applicable source |
| **A** | Project-controlling AEP/STANAG context carried from an accepted report |
| **P** | Proposed Glaux decision awaiting acceptance of this report |
| **I** | Informative implementation, test, interoperability, or community evidence |
| **D** | Official draft evidence that constrains seams but is not an approved requirement |
| **X** | Published inconsistency, ambiguity, or evidence limitation retained explicitly |

`ID`, `UID`, `canonical URL`, `external identifier`, `alias`, `revision`, `relationship ID`, `domain-event ID`, `transport-message ID`, and `database key` are different concepts even when two happen to contain the same UUID-shaped value.

---

## 1. Executive Summary

### 1.1 Decision

Glaux should mint an immutable, opaque, lowercase UUIDv7 **service-local resource ID** for every canonical resource and every independently addressable subordinate record. It should do this even when a publisher supplies a globally oriented UID. The local ID identifies the hosted Glaux resource; the UID identifies the represented real-world or conceptual object across representations and services. Treating either as a database key, human name, route, source-system key, or proof that two federated records are the same would collapse distinct authorities and create unsafe merges.

For every Part 1 resource that requires a UID, Glaux must preserve an authoritative valid-URI UID. If Glaux is the object-identifier authority and no UID is supplied, it should mint a separate `urn:uuid:` UUIDv7 UID. A supplied UID is accepted only with issuer/source and authority context; equality is evidence for reconciliation, not automatic federation merge. Part 2 dynamic resources retain local resource IDs and canonical URLs but do not acquire invented CSAPI UIDs.

Canonical resource URLs are composed from one configured public API root plus the approved plural route and local ID. They contain no routine Glaux release version. A resource has the same local ID and canonical URL across updates, encodings, collection memberships, nested views, deployment changes, cache revalidation, and ordinary software upgrades. Collection and nested paths are views. Every resource retrieved through one of those paths links to the canonical URL.

### 1.2 Principal Findings

1. CSAPI requires resource IDs to be unique across all resources of a given type and says they are normally server-generated. Glaux deliberately strengthens this to one service-wide local-ID namespace. This makes polymorphic keys, logs, caches, and references safer without changing the wire contract [N,P].
2. CSAPI requires UIDs to be valid URIs and unique across all collections; it recommends global uniqueness and UUID or registered-namespace URNs. The UID is independent of resource URL and carries real-world-object identity across services [N].
3. RFC 9562 supersedes RFC 4122. UUIDv7 supplies distributed generation and time-ordered database locality. Glaux uses its canonical lowercase hex-and-dash text form on the wire and 128-bit storage where supported, but applications treat it as opaque. UUIDv7's creation-time leakage is documented; identifiers are never authorization capabilities [N,P].
4. CSAPI fixes the canonical paths. The approved Part 2 overview uses `/controlstreams`, `/commands`, and `/systemEvents`; Glaux retains the accepted plural-route reconciliation where individual published requirements contain singular defects [N,X,P].
5. A new revision does not receive a new resource ID. A new ID is required for a different real-world/conceptual object, a distinct observation/event/task occurrence, or a replacement stream whose populated schema is incompatible. Representation validators and revision numbers change while canonical identity remains stable [P].
6. Valid time, operational status, command status, API deprecation, retention/archival, access denial, and existence are independent axes. A resource can be historically valid, retired, archived, and still retrievable; a failed Command remains a successfully retrievable Command resource [N,P].
7. Deletion is a domain transition plus an HTTP/persistence policy, not immediate ID reuse. Local IDs, canonical URLs, UIDs, and aliases are never reassigned. Referentially significant deletion creates a durable tombstone. Authorized disclosure may return `410 Gone`; concealment or unknown permanence returns `404` [N,P].
8. Permanent redirects are limited to proved identity-preserving moves. `308` preserves the method but remains unsafe by default for Commands and other mutations. Known CSAPI spelling defects use direct GET/HEAD compatibility handlers, not alternate resources or automatic write redirects [N,P].
9. The identifier registry, canonical URL builder, alias/tombstone registry, revision ledger, and lifecycle transition engine must be shared infrastructure rather than handler-specific strings [I,P].
10. No evidence gap blocks IDR-SRV-017. Federation equivalence, exact temporal vocabulary, CRUD transaction semantics, retention periods, command transitions, and security policy remain assigned to later topics.

### 1.3 Recommended Baseline

Adopt the following compact contract:

- `ResourceId`: immutable service-minted UUIDv7, unique service-wide, exposed as lowercase text, never reused.
- `Uid`: valid absolute URI for required Part 1 families; source/issuer-qualified; independent of URL; immutable except through an explicit identity-correction workflow.
- `CanonicalUrl`: configured public HTTPS API root + normative plural route + `ResourceId`; stable through ordinary releases.
- `ExternalIdentifier`: typed `(scheme, value, issuer/source)` assertion; preserved, indexed, and never promoted silently to canonical identity.
- `Alias`: immutable mapping from an old/noncanonical locator or identifier to exactly one canonical resource; never retargeted.
- `Revision`: monotonically increasing opaque entity revision, distinct from identity and HTTP representation-specific ETag.
- `Tombstone`: retained identity/resolution record after disclosed removal; contains safe lifecycle and replacement facts, not the deleted representation.
- `IdempotencyKey`, `RelationshipId`, `DomainEventId`, and `MessageId`: separate namespaces with separate retention and replay rules.

---

## 2. Scope and Plan Alignment

### 2.1 Completed Scope

- Identifier classes, scopes, authorities, syntax, stability, and collision handling.
- Canonical route matrix for service, collection, Part 1, Part 2, subordinate, nested, and queryable views.
- External UID, SensorML identifier, SWE semantic reference, import, simulator, federation, alias, redirect, and DDIL implications.
- Entity revision, replacement, deprecation, retirement, archival, deletion, tombstone, and ID-reuse rules.
- Resource-family lifecycle classifications and transition constraints.
- Persistence, validation, security, fixtures, conformance, and interoperability handoffs.

### 2.2 Deliberately Deferred

- Stored relationship shapes and complete link vocabulary: IDR-SRV-017.
- Exact public temporal/freshness model: IDR-SRV-018 and IDR-SRV-020.
- Provenance assertion and trust model: IDR-SRV-019.
- Database engine/schema/index selection: IDR-SRV-025 through IDR-SRV-030.
- PATCH document, concurrency, atomicity, idempotency, and batch semantics: IDR-SRV-029 and IDR-SRV-031.
- Detailed Command and Feasibility transition machines: IDR-SRV-036 through IDR-SRV-038.
- Authentication, authorization, releasability, concealment, and federation policy: Category G.
- Part 3 transport/profile implementation: IDR-SRV-035 and later authorization; not started here.

### 2.3 Core Question Coverage

| Question | Answer | Evidence location |
|---|---|---|
| CQ-1 Identifier classes | Ten distinct public/internal identity classes with explicit authorities | §§5–6 |
| CQ-2 URI patterns | Canonical, nested, collection, schema, service, and compatibility matrix | §7 |
| CQ-3 Stable identity | Stable-ID invariants, collision/import/DDIL rules, revision/replacement policy | §§8–11 |
| CQ-4 Lifecycle states | Cross-cutting and family-specific state/transition matrices | §§10–11 |
| CQ-5 Downstream implications | Persistence, validation, security, test, and topic handoffs | §§12–14 |

---

## 3. Evidence Base and Authority Classification

### 3.1 Primary Sources

| Source | Version / pin | Authority and anchors | Access / limitation |
|---|---|---|---|
| [CSAPI Part 1](https://docs.ogc.org/is/23-001/23-001.html) | OGC 23-001, Version 1.0; tagged source `8e03b236` | Requirements 1–2; canonical URL requirements; resource attributes; create/replace/delete/update | 2026-09-03; approved source contains known link and transaction defects |
| [CSAPI Part 2](https://docs.ogc.org/is/23-002/23-002.html) | OGC 23-002, Version 1.0; tagged source `8e03b236` | §§7.4, 9–12, 14–16; canonical, schema, command, lifecycle, cascade rules | 2026-09-03; singular/plural and ATS/OAS conflicts retained |
| [OGC API - Features Part 1](https://docs.ogc.org/is/17-069r4/17-069r4.html) | OGC 17-069r4, Version 1.0.1 | Collection/item identity and inherited retrieval/error behavior | 2026-09-03 |
| [SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html) | OGC 23-000, Version 3.0 | `uniqueId`, classified identifiers, references, valid time, history | 2026-09-03; encoding semantics do not allocate Glaux IDs |
| [SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html) | OGC 24-014, Version 3.0 | semantic `definition`, component names, units, reference/local frames | 2026-09-03; semantic and structural identifiers are not resource identity |
| [RFC 3986](https://www.rfc-editor.org/rfc/rfc3986.html) and [RFC 3987](https://www.rfc-editor.org/rfc/rfc3987.html) | Standards Track | URI syntax, resolution, normalization, IRI conversion | 2026-09-03 |
| [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html) | Standards Track, June 2022 | Location, ETag/preconditions, redirects, 404, 409, 410 | 2026-09-03 |
| [RFC 8288](https://www.rfc-editor.org/rfc/rfc8288.html) and [IANA relations](https://www.iana.org/assignments/link-relations) | Standards Track / registry | Link context, target, relation semantics; `canonical` | 2026-09-03 |
| [RFC 9562](https://www.rfc-editor.org/rfc/rfc9562.html) | Standards Track, May 2024; obsoletes RFC 4122 | UUID text, v7, generation, opacity, storage, security | 2026-09-03 |

The plan lists RFC 4122 because CSAPI 1.0 cites it. RFC 9562 now controls UUID generation and URN guidance while preserving the UUID URN namespace. This is a standards maintenance update, not a change to CSAPI's UID requirement.

### 3.2 Project and Comparative Sources

| Source | Authority | Use |
|---|---|---|
| Accepted IDR-SRV-001 through IDR-SRV-015 | Project-controlling | AEP package boundary, routes, links, errors, versioning, implementation lessons, canonical graph |
| IDR-SRV-014A OSH study | Informative pinned implementation evidence | Separate stores and indexes; temporal versions; alias and conflict hazards |
| IDR-SRV-014B CS-Go study | Informative pinned implementation evidence | Central URL construction need; durable IDs/status/results/events; transaction and schema lessons |
| IDR-SRV-014C pygeoapi study | Informative pinned implementation evidence | Provider-native identifier and route coupling hazards |
| IDR-SRV-014D/014F SECD studies | Informative interoperability evidence | Discovery, lifecycle, identity, and client-composition failure modes |
| IDR-SRV-014G discussions study | Proposal/experience evidence | Source-qualified federation identity; UID equality is not proof of object equality |
| IDR-SRV-014H draft Part 3 study | Official draft plus informative implementation evidence | Keep resource, event, message, parent, and transport identity separate |

### 3.3 Evidence Quality and Conflicts

- Approved CSAPI requirements outrank tagged OAS, examples, ATS defects, implementation behavior, and discussion proposals.
- The official stable tag remains `8e03b236`; stable master remained `3fd86c73` at refresh. No shared upstream-history register change was required.
- Part 2's overview, OAS, ATS, and most requirements establish plural canonical forms. Singular `/controlstream` and `/command` statements are published inconsistencies. Accepted IDR-SRV-010 controls Glaux output.
- CSAPI's write classes depend on draft OGC API - Features Part 4. This report identifies identity/lifecycle constraints but does not freeze unresolved PATCH/idempotency semantics.
- The controlled NATO package is not public. Its exact identity and relevant server obligations are carried only from accepted IDR-SRV-001 through IDR-SRV-003.
- Implementation studies corroborate architectural risks; none creates a normative identifier rule.

---

## 4. Identifier and Lifecycle Extraction Methodology

Each concept was classified by:

| Dimension | Values used |
|---|---|
| What is identified | Hosted resource, represented object, external assertion, representation revision, relation, domain event, message, storage row, request |
| Scope | Service-wide, resource-family, parent-local, source-local, globally oriented, transport-local |
| Minting authority | Glaux, authoritative publisher, external registry, derived projection, transport adapter |
| Stability | Immutable, revision-scoped, representation-scoped, request-scoped, ephemeral |
| Addressability | Canonical URL, noncanonical view, subordinate path, opaque internal-only key |
| Lifecycle | Active, retired, archived, deleted/tombstoned, replaced, expired, task-domain state |
| Evidence | N, A, P, I, D, or X |

The analysis applied four invariants:

1. Same hosted entity across views and encodings means same local ID.
2. Same UID does not necessarily mean same hosted representation or authorized equivalence.
3. Changed representation does not necessarily mean changed entity identity.
4. Changed semantic identity must never be disguised as an in-place revision.

---

## 5. Standards-Derived Identifier and URI Behavior Inventory

### 5.1 CSAPI Resource IDs and UIDs

Part 1 Requirement 1 requires each resource ID to be unique across every collection containing resources of the same type. The standard says IDs are typically server-generated and not guaranteed globally unique. Requirement 2 separately requires UIDs to be valid URIs and unique across all server collections; the recommendation prefers globally unique UUID URNs or URNs in registered namespaces. Its rationale explicitly distinguishes the real-world object's cross-service identity from a hosted representation's URL [N].

This produces a non-negotiable separation:

| Concept | Standards meaning | Glaux interpretation |
|---|---|---|
| Local resource ID | Server-local locator token, at least unique by type | Service-minted UUIDv7, strengthened to service-wide uniqueness |
| UID | Persistent URI-form identity of represented object, independent of URL | Required on Part 1 families; preserved with authority; never used alone to merge |
| Canonical URL | Preferred address for this server's resource | Public root + canonical family route + local ID |
| Collection item URL | Membership/view address | Noncanonical view of the same resource; include canonical link |
| Nested URL | Relationship-filtered address | Noncanonical view or subordinate resource; never derives entity identity |

The Part 1 conceptual models make `uniqueIdentifier` required for Systems, Deployments, Procedures, Sampling Features, and Property Definitions. The GeoJSON `uid` and SensorML `uniqueId` mappings preserve that concept. Part 2 inherits local resource-ID behavior but does not define a general UID attribute for its dynamic resource families [N].

### 5.2 SensorML and SWE Common

SensorML's `uniqueId` maps to the CSAPI `uniqueIdentifier`. Its classified `identifiers` can carry serial numbers, model numbers, callsigns, and other assertions. Those are alternate/external identifiers with type and issuer context—not Glaux IDs. SensorML `validTime` limits applicability; it does not version the identifier [N,P].

SWE component `name` values identify positions inside a record or array schema. `definition`, unit links, reference frames, and local frames are semantic/reference URIs. Glaux preserves them with their exact role and must not copy them into `ResourceId`, `Uid`, or database-primary-key fields [N].

### 5.3 URI, HTTP, and Linking

- URI comparison is purpose-sensitive. Glaux stores the submitted lexical form of external identifiers and a separately computed comparison key; it does not globally rewrite arbitrary URIs under an unsafe “normalization” rule [N,P].
- Public links use absolute HTTPS URLs built from an explicitly trusted configured origin. Forwarded headers affect this only under the accepted trusted-proxy policy [P].
- RFC 8288 defines a link as context + relation + target. The IANA `canonical` relation designates the preferred version of a resource and its contents. A canonical link is not itself a redirect [N].
- A successful POST that creates resources should return `201 Created` and a `Location` for the primary created resource. The exact multi-create and asynchronous contract remains IDR-SRV-031/036/037 [N,P].
- Strong ETags and `If-Match` protect against lost updates. ETags identify selected representation state, not domain identity; exact mandatory preconditions remain IDR-SRV-029 [N,P].
- `404` covers absent current representations and intentional non-disclosure. `410` is for known likely-permanent unavailability. Both can be cacheable, so protected/dynamic responses retain IDR-SRV-013's conservative cache policy [N,P].
- `308` describes a permanent URI move and preserves the method, but automatic unsafe-method following can still duplicate or misdirect effects. It is not a general alias mechanism [N,P].

### 5.4 UUID Selection

RFC 9562 UUIDv7 contains a 48-bit Unix-epoch millisecond timestamp plus random data, recommends v7 over v1/v6 when possible, and supports lexicographic/index locality. It also advises treating UUIDs opaquely and storing their 128-bit value in databases where practical [N].

| Option | Strength | Weakness | Decision |
|---|---|---|---|
| Integer/sequence | Small and index-friendly | Central coordination; enumeration; collision on import/merge | Reject for public resource IDs |
| Publisher key | Preserves source convention | Source collisions, mutability, unsafe characters, authority coupling | Preserve only as external identifier |
| UUIDv4 | Mature, random, hides creation time | Poorer ordered-index locality | Allowed for imported existing values, not default mint |
| UUIDv5/name-based | Deterministic | Name changes invalidate key; SHA-1 basis; canonicalization hazards | Reject for canonical IDs |
| ULID/custom sortable ID | Compact/sortable | Additional non-IETF format and parsing rules | Reject without demonstrated need |
| UUIDv7 | Distributed, standard, sortable, 128-bit | Approximate mint time visible; clock/generator quality required | Select as Glaux default |

UUIDv7 is not a security boundary. Authorization occurs on every resolution. Deployments must not infer creation time as authoritative domain time; explicit committed/created/observed times remain separate.

---

## 6. Identifier Class Definitions and Scope Findings

| Class | Scope / authority | Format | Stability / rule |
|---|---|---|---|
| `ResourceId` | Glaux service-wide | Lowercase UUIDv7 text at API; native 128-bit storage preferred | Immutable, opaque, never reused or client-selected on ordinary create |
| `Uid` | Represented object; external authority or Glaux when designated | Absolute URI; new Glaux values `urn:uuid:<uuidv7>` | Immutable unless controlled identity correction; required only where model requires |
| `CanonicalUrl` | One Glaux deployment/API root | Absolute HTTPS URI + canonical path | Stable within contract; old origin/path retained as alias on controlled move |
| `ExternalIdentifier` | Issuer/source-qualified | Original lexical value plus scheme/type/issuer | Preserved; multiple allowed; never independently authoritative without policy |
| `Alias` | Glaux resolution registry | Old URI or typed identifier key | One target forever; may expire from active resolution but never be retargeted |
| `Revision` | Per hosted entity | Monotonic opaque integer/token | Changes on committed domain mutation; not part of canonical URL |
| `ETag` | Representation and selection state | HTTP entity tag | Changes when selected representation changes; strong for mutation preconditions |
| `RelationshipId` | Association fact | Service-local UUIDv7 where independent history needs it | Separate from both endpoint IDs; detailed in IDR-SRV-017 |
| `DomainEventId` | Lifecycle/audit/outbox event | Service-local UUIDv7 | Immutable; separate from affected resource and transport publication |
| `MessageId` | Part 3/transport envelope | Adapter/profile-defined, commonly UUID | Retransmission/dedup namespace; never canonical resource identity |
| `IdempotencyKey` | Principal + operation + target scope | Opaque client request value | Bounded retention; cannot be used as resource ID; IDR-SRV-029 |
| Database key | Persistence adapter | Engine-specific or same binary UUID | Never exposed merely because it exists; migration-safe mapping required |

### 6.1 Authority Ordering

1. Glaux is authoritative for its `ResourceId`, canonical URL, revision, aliases, tombstone, and hosted lifecycle.
2. A recognized registry or authorized publisher can be authoritative for a UID or external identifier, but only within the recorded issuer/scheme scope.
3. An import adapter asserts source URI, source ID, and supplied UID; it does not assert cross-source sameness automatically.
4. Human labels, filenames, database sequences, collection IDs, component names, hashes, and transport topics have no resource-identity authority.
5. Conflicting authoritative assertions enter a quarantined/conflict workflow; “last writer wins” is prohibited for identity.

### 6.2 Namespace Rules

- Resource IDs are unique service-wide even though CSAPI requires only type-wide uniqueness.
- UIDs are unique across every hosted Part 1 resource collection as required. A UID index is therefore cross-family and cross-collection.
- Collection IDs occupy a configured collection namespace, not the resource-ID namespace.
- Parent-local subordinate IDs for CommandStatus/CommandResult are minted service-wide anyway; their URL remains parent-nested.
- Local resource IDs are ASCII lowercase UUID text; clients may not infer family, shard, tenant, date, sequence, or authority from them.
- Nil/max UUIDs, non-RFC variants, brace-wrapped values, uppercase canonical output, malformed forms, and percent-encoded alternate spellings are rejected for new Glaux IDs.

---

## 7. Resource-Family URI and Canonical Addressing Strategy

### 7.1 Service and Collection Routes

| Resource/view | Route under `{api_root}` | Identity rule |
|---|---|---|
| Landing page | `/` | API-root resource; configured root is canonical |
| API definition | `/api` recommended and advertised | Exact route is project convention; discovery link controls |
| Conformance | `/conformance` | One declaration derived from enabled capability registry |
| Collections | `/collections` | Collection inventory |
| Collection detail | `/collections/{collectionId}` | Stable configured collection ID; membership criteria versioned separately |
| Collection items | `/collections/{collectionId}/items` | Selection/view, not independent identity |
| Collection item | `/collections/{collectionId}/items/{resourceId}` | Same entity; emit canonical link to family URL |

`{api_root}` includes any externally configured path prefix. Ordinary Glaux releases never insert `/v1`. Query parameters and representation suffixes select views/representations and are not part of canonical entity identity.

### 7.2 Canonical Resource Matrix

| Family | Canonical collection | Canonical item | UID | Notes |
|---|---|---|---|---|
| System | `/systems` | `/systems/{id}` | Required | Subsystems use same System family and canonical route |
| Deployment | `/deployments` | `/deployments/{id}` | Required | Subdeployments use same Deployment family |
| Procedure | `/procedures` | `/procedures/{id}` | Required | Method/type identity rules apply |
| SamplingFeature | `/samplingFeatures` | `/samplingFeatures/{id}` | Required | Same ID across collection/nested views |
| PropertyDefinition | `/properties` | `/properties/{id}` | Required | Semantic definition must not be confused with UID |
| DataStream | `/datastreams` | `/datastreams/{id}` | Not invented | Populated schema replacement may require new stream |
| Observation | `/observations` | `/observations/{id}` | Not invented | Each distinct observation occurrence gets new ID |
| ControlStream | `/controlstreams` | `/controlstreams/{id}` | Not invented | Plural route reconciles published defect |
| Command | `/commands` | `/commands/{id}` | Not invented | One accepted intent/task resource per ID |
| Feasibility | `/feasibility` | `/feasibility/{id}` | Not invented | Root list is accepted Glaux interoperability baseline |
| SystemEvent | `/systemEvents` | `/systemEvents/{id}` | Not invented | Distinct occurrence; not audit/outbox/message identity |

### 7.3 Nested and Subordinate Routes

| View/resource | Canonical Glaux pattern | Identity consequence |
|---|---|---|
| Subsystems | `/systems/{systemId}/subsystems` | System view; each member canonically `/systems/{id}` |
| Subdeployments | `/deployments/{deploymentId}/subdeployments` | Deployment view; member canonical URL unchanged |
| System sampling features | `/systems/{systemId}/samplingFeatures` | Relationship view |
| System/deployment streams | `/systems/{id}/datastreams`, `/deployments/{id}/datastreams` | Relationship/time-filtered view |
| DataStream observations | `/datastreams/{id}/observations` and `/{observationId}` | Observation keeps top-level canonical URL |
| System/deployment controls | `/systems/{id}/controlstreams`, `/deployments/{id}/controlstreams` | Relationship/time-filtered view |
| ControlStream commands | `/controlstreams/{id}/commands` and `/{commandId}` | Command keeps `/commands/{id}` canonical URL |
| ControlStream feasibility | `/controlstreams/{id}/feasibility` and `/{feasibilityId}` | Feasibility keeps root canonical URL |
| Command status/results | `/commands/{id}/status`, `/commands/{id}/status/{recordId}`, `/commands/{id}/result`, `/commands/{id}/result/{recordId}` | Independently identified nested records; no top-level family |
| Feasibility status/results | `/feasibility/{id}/status`, `/feasibility/{id}/status/{recordId}`, `/feasibility/{id}/result`, `/feasibility/{id}/result/{recordId}` | Same subordinate record model |
| System events | `/systems/{id}/systemEvents` | Relationship view; members canonically `/systemEvents/{id}` |
| DataStream schema | `/datastreams/{id}/schema?obsFormat={mediaType}` | Subordinate negotiated schema representation |
| ControlStream schema | `/controlstreams/{id}/schema?cmdFormat={mediaType}` | Subordinate negotiated schema representation |

Dynamic properties and “current status” are projections over observations/status datastreams and query time. They do not receive fabricated canonical resource IDs. Stream schemas are addressable subordinate resources whose identity is `(parent stream ID, schema role, representation selector, revision)`; they do not become top-level CSAPI resources.

### 7.4 Canonical URL Construction

One typed route registry supplies handlers, payload links, HTTP `Link`, `Location`, OpenAPI paths, alias mapping, and tests. URL construction must:

1. use the configured/trusted external scheme, authority, and base path;
2. use exact case-sensitive canonical route segments;
3. serialize the UUID lowercase without braces or percent-encoding;
4. exclude representation selectors, paging/filter query, fragments, and release numbers;
5. preserve the same result behind direct and trusted-proxy deployments; and
6. fail startup if two route definitions or aliases can resolve ambiguously.

---

## 8. External, Alternate, and Generated Identifier Findings

### 8.1 Ingestion and Import

Every imported object receives a new Glaux `ResourceId`. The adapter records:

- source service/authority identifier;
- source canonical URI, if any;
- source resource type and local ID;
- supplied UID and all alternate identifiers;
- issuer, identifier type/scheme, lexical form, and validation result;
- import/observation time and provenance assertion; and
- reconciliation status (`unreviewed`, `matched`, `distinct`, `conflict`, or later vocabulary).

A trusted restore of the same Glaux service may preserve its original ResourceId through a privileged, audited restore path. Ordinary POST, publisher, simulator, and federation clients cannot choose or overwrite it.

### 8.2 UID Creation and Preservation

- Preserve a valid supplied UID when the publisher is authorized for that identifier authority and it is not already bound inconsistently.
- If a required Part 1 UID is absent and Glaux is designated to allocate object identity, mint an independent `urn:uuid:<uuidv7>`.
- If Glaux is not the identifier authority, reject or quarantine rather than fabricate an apparently authoritative identity.
- Do not rewrite legacy registered/private URNs into UUID URNs.
- Do not use the canonical URL as the UID merely because it is a URI; the standard requires independence.
- Do not reuse a source UID as local ID, even when its terminal component is UUID-shaped.

### 8.3 SensorML and SWE Handling

| Input | Stored role | Public consequence |
|---|---|---|
| SensorML `uniqueId` | UID candidate | Maps to the Part 1 resource UID after authority/uniqueness validation |
| SensorML classified identifier | External/alternate identifier assertion | Round-trips with type/issuer; searchable only under authorized policy |
| SensorML `id`/document-local reference | Encoding-local anchor | Resolved during parse; not promoted to ResourceId |
| SWE component `name` | Parent-schema-local structural key | Stable within a schema revision; may form a pointer, not URL identity |
| SWE `definition` | Semantic concept URI | Preserved/dereferenceable according to validation profile |
| Unit/CRS/frame URI | Semantic/reference identifier | Kept in typed field; never identity alias |

### 8.4 Generated Resources

Server-created observations, events, status records, results, and derived description resources receive normal ResourceIds before durable publication. A deterministic content hash may prove equivalence or detect duplicates, but it does not replace the resource ID. A projection or query result receives no persistent ID unless it is deliberately materialized as one of the canonical resource families.

---

## 9. Collision, Alias, Redirect, Deprecation, and Replacement Findings

### 9.1 Collision Classification

| Collision | Detection | Required behavior |
|---|---|---|
| Generated ResourceId already exists | Service-wide unique constraint | Regenerate before commit; repeated collision is fatal generator/entropy fault |
| Client attempts to supply ResourceId | Request/schema policy | Ignore only where standard encoding explicitly treats it as response-only; otherwise reject safely; never overwrite existing identity |
| Same required UID already bound to same resource | Cross-collection UID index | Treat as update/idempotent identity assertion only after operation, authority, and payload checks |
| Same UID bound to another local resource | UID index + source/issuer | `409` identity conflict or quarantine; never auto-merge/delete |
| Same source `(authority,type,id)` re-imported | Source identity unique index | Reconcile to existing resource under idempotent import rules |
| Same alternate value from different issuers | Qualified identifier index | Allowed as distinct assertions unless scheme defines global authority |
| Same canonical/alias URI maps to two resources | URL/alias unique constraint | Reject configuration or mutation atomically |
| Same object appears with different UIDs | Reconciliation evidence | Preserve both assertions; an explicit governed match may relate them; do not silently replace UID |
| Same UID appears to name different objects | Provenance/content conflict | Quarantine and audit; authority adjudication required |

Identity conflicts are current-state conflicts, not syntax errors. IDR-SRV-013's RFC 9457 problem contract applies, but public detail must not expose hidden existing IDs, tenants, or policy facts.

### 9.2 Alias Invariants

An alias is a resolution statement, not a second canonical identity:

- one normalized lookup key maps to one immutable target ResourceId;
- an alias can be enabled, deprecated, expired from active resolution, or retained as a tombstone, but never retargeted;
- alias lookup applies the same authentication, authorization, concealment, caching, and representation negotiation as canonical lookup;
- alias responses and representations identify the canonical URL;
- aliases are absent from ordinary discovery and OpenAPI unless published as documented deprecated compatibility operations;
- alias creation, use, conflict, expiry, and attempted retargeting are auditable;
- external identifiers are not URL aliases unless an explicit resolver mapping is created.

### 9.3 Direct Handling Versus Redirect

| Case | Policy | Reason |
|---|---|---|
| Resource retrieved through collection/nested path | Serve representation at requested view and include `canonical` link | Required CSAPI multi-view behavior; no redirect needed |
| Known singular route defect, GET/HEAD | Direct compatibility handler that shares canonical state/auth/ETag and emits canonical/deprecation metadata | Accepted IDR-SRV-010A policy; avoids creating alternate identity |
| Same resource moved permanently to a new canonical URL | `308` only after identity, auth, cache, client, and method analysis | Method-preserving permanent move |
| POST/PUT/PATCH/DELETE or tasking alias | No automatic redirect by default | Unsafe retry/follow can duplicate or misdirect effects |
| Different resource supersedes old resource | Do not assert canonical equivalence; retain old resource/tombstone and explicit replacement relation/docs | Replacement is not the same identity |
| Representation alternative | `alternate` link/content negotiation | Not an identity move |

### 9.4 Deprecation and Path Lifetime

The accepted IDR-SRV-010A policy controls API-route deprecation: behavior remains compatible, RFC 9745 metadata is exact, replacement or no-replacement rationale is documented, and the stable support floor is at least two stable minor releases and 12 months, whichever is longer, normally until the next major. Resource lifecycle retirement does not automatically deprecate the API route, and route deprecation does not retire resources [P].

If the public origin or API root must change, the migration record preserves old canonical URLs in an alias registry and operates the previous origin/path for the approved window. Glaux cannot promise that a URL survives loss of DNS/control of the old origin; the stable UID and source-qualified identity remain essential for cross-host continuity.

### 9.5 Replacement Rules

Create a new ResourceId when:

- a different real-world System, Deployment occurrence, Procedure concept/method, Sampling Feature, or Property Definition is represented;
- a new observation, command, feasibility request, status report, result record, or system-event occurrence is created;
- a populated DataStream's observation schema or populated ControlStream's command schema must change incompatibly;
- an import is a distinct hosted representation and has not been explicitly reconciled as the same Glaux entity; or
- the old resource must remain independently citable for audit/reproducibility and the new object has different semantic identity.

Preserve ResourceId and create a new revision when:

- correcting or enriching metadata without changing what the resource represents;
- updating a System's latest location, description, status links, or relationships;
- changing valid-time bounds as an authorized correction to the same entity;
- moving collection membership or retrieving through another view;
- translating among GeoJSON, SensorML, SWE, or other equivalent encodings; or
- changing storage layout, server implementation, cache, or software release.

UID correction is exceptional. It requires authority verification, immutable before/after audit, collision checking, relationship review, and preservation of the prior value as a deprecated identifier assertion where policy permits. It does not change ResourceId merely because the identifier assertion was wrong.

---

## 10. Resource Lifecycle and State-Transition Findings

### 10.1 Orthogonal Lifecycle Axes

Glaux must not implement one overloaded `status` enum. The canonical model requires separate axes:

| Axis | Example values | Meaning / owner |
|---|---|---|
| Hosted existence | active, tombstoned, purged | Whether Glaux retains current representation/identity evidence |
| Administrative lifecycle | draft/internal, active, retired, replaced | Resource governance; exact public model later |
| Temporal validity | not-yet-valid, valid, expired; interval | When description/relationship applies; IDR-SRV-018 |
| Operational availability | available, degraded, stale, unavailable, unknown | Current system/stream condition; IDR-SRV-020 |
| Retention disposition | hot, archived, deletion-pending, purged | Storage policy; IDR-SRV-030 |
| API contract lifecycle | stable, deprecated, sunset-scheduled, retired | Route/field/profile evolution; IDR-SRV-010A |
| Task domain status | PENDING through COMPLETED/FAILED/etc. | Command/Feasibility execution; IDR-SRV-036/037 |
| Publication state | committed, queued, published, acknowledged/expired | Outbox/Part 3 transport; IDR-SRV-029/035 |

`validTime` is not created time. `live=false` is not deletion. `CANCELED` is not HTTP DELETE. `archived` is not `410`. Access denial is not proof of nonexistence.

### 10.2 Generic Administrative State Machine

The common internal state vocabulary is intentionally small:

```text
candidate/internal -> active -> retired -> tombstoned -> purged
                         |          |
                         +------> replaced
```

- `candidate/internal` is a staging state and is not exposed as an ordinary canonical resource until validation, authorization, and durable commit succeed.
- `active` means resolvable under policy; it says nothing about current operational availability or valid time.
- `retired` means no longer current for new operational use, but still identifiable and normally retrievable for history.
- `replaced` is retired plus an explicit link/assertion to a different resource; the replacement never takes the old ID.
- `tombstoned` means the representation has been removed but identity and safe lifecycle facts are retained.
- `purged` means payload and perhaps tombstone detail were removed under approved retention/legal policy; the ID remains reserved forever.

Reactivation from retired may be allowed only as a governed transition for the same entity and must not erase the interval/audit history. Tombstoned or purged IDs cannot be reactivated or reused through ordinary API operations.

### 10.3 Creation Boundary

A canonical URL becomes externally assertable only after one transaction has durably established:

1. ResourceId and required UID uniqueness;
2. parent/source authority and relationship prerequisites;
3. representation/schema/semantic validation;
4. initial entity revision;
5. lifecycle/audit record; and
6. outbox event intent where enabled.

Before this boundary, failures are HTTP request failures. After it, task rejection/failure, asynchronous publication, and operational state are domain lifecycle, not retroactive create failures. Exact transaction and retry behavior belongs to IDR-SRV-029.

### 10.4 Update and Revision

- Every accepted semantic mutation increments the entity revision and records actor/source, commit time, prior revision, reason/operation, and safe change metadata.
- No-op retries may return the existing state without incrementing revision when the idempotency contract proves equivalence.
- Strong ETags must represent the selected current representation. Different media types or authorization views may have different ETags for the same entity revision.
- Historical revision retention is family/policy-specific, but command intent, status, event, observation, and identity transitions require stronger audit preservation than mutable descriptive labels.
- A correction never rewrites previously published event/message IDs; it emits a new domain event when publication is required.

### 10.5 Deletion and Tombstones

Deletion is authorized only after referential, cascade, retention, legal, audit, and policy checks. CSAPI directly requires non-cascading conflict behavior for populated streams and for Systems with nested/associated resources (subject to the accepted Part 1 cascade reconciliation). Glaux adds these invariants [N,P]:

- default to retirement/archival where traceability remains valuable;
- execute cascade atomically and enumerate its authorized scope before commit;
- preserve UIDs, IDs, canonical/alias keys, replacement facts, deletion time, and safe reason in a tombstone when references or clients may persist;
- never expose deleted payload, classified metadata, prior owners, or relationship counts merely because a tombstone exists;
- use authorized `410` when intentional permanent removal may be disclosed, otherwise concealed/ordinary `404`;
- apply explicit cache policy to negative/tombstone responses;
- never recycle any deleted identifier or alias.

The exact retention duration and physical erasure requirements are owned by IDR-SRV-030 and security/policy topics. “Right to delete” may require payload erasure while retaining a non-identifying collision-prevention digest or protected reservation record; this report does not preempt applicable law/policy.

---

## 11. Resource-Family Identity and Lifecycle Matrix

### 11.1 Canonical Families

| Family | Authority / alternate IDs | Stability and lifecycle | Collision / replacement | Key handoffs |
|---|---|---|---|---|
| System | Glaux local ID; required object UID; serial/model/callsign as qualified alternates | Same identity through metadata, latest location, deployment, status, and composition changes; active → retired/replaced/tombstone | New ID for different physical/logical instance; same UID conflict never auto-merges | 017 relationships; 018 validity; 020 status; 025/030 persistence |
| Deployment | Glaux ID; required UID; mission/source IDs qualified | One purposeful deployment occurrence; planned/active/completed concepts refined later; description correction keeps ID | New deployment for a distinct execution/purpose/time; subdeployment remains same family | 017 association fact; 018 interval; 019 provenance |
| Procedure | Glaux ID; required UID; manufacturer/spec IDs | Metadata revision retains ID; retirement preserves reproducibility | Incompatible method/spec semantic change creates new Procedure and replacement relation | 017, 018, 019, 033 |
| SamplingFeature | Glaux ID; required UID; specimen/station IDs qualified | Identity follows the sample/proxy, not merely geometry; validity can expire without deletion | New sample/proxy gets new ID; corrected geometry may retain ID with audit | 017 sampled-feature links; 018 validity; 026 geospatial |
| PropertyDefinition | Glaux ID; required UID; semantic definition URI remains separate | Label/description correction keeps ID; semantic meaning change creates replacement | UID/definition collisions require semantic adjudication | 017 semantic links; 019 authority; 023 validation |
| DataStream | Glaux ID; no invented UID; source stream key alternate | Same series through appended Observations and computed extents; may become inactive/closed/retired | Populated observation-schema incompatibility creates new stream; default DELETE conflict with children | 018/020 time/live; 027 store; 030 retention; 034 behavior |
| Observation | Glaux ID; no invented UID; source observation key alternate | One observation act/result occurrence; append-oriented; correction is governed revision/supersession, not new data occurrence | New act/result gets new ID; parent schema violation is `400`; purge retains reservation as policy requires | 018 times; 019 provenance/quality; 027 storage; 034 semantics |
| ControlStream | Glaux ID; no invented UID; source channel key alternate | Same command contract while compatible; active/inactive/retired separate from System availability | Populated command-schema incompatibility creates new stream; default DELETE conflict with Commands | 020 status; 028/030 store; 036 tasking |
| Command | Glaux ID; no invented UID; external request/correlation and idempotency keys separate | One accepted command intent; status updates do not replace ID; cancellation retains Command | Every distinct intent gets new ID; proven retry returns prior result; replacement/update rules constrained by safety | 029 idempotency; 036 lifecycle; 038 authorization/audit |
| Feasibility | Glaux ID; external request/idempotency separate | One analysis request; status/result records accumulate; never turns into a Command under same ID | Actual Command gets new ID and explicit relation; retry handled separately | 017 relation; 029; 037 lifecycle |
| SystemEvent | Glaux ID; source event key alternate | One reported domain occurrence; append-oriented, historical, separately correctable | New occurrence gets new ID; distinct from lifecycle audit and Part 3 message | 018 time; 019 provenance; 020/034 semantics; 035 publication |

### 11.2 Subordinate Records and Views

| Concept | Identifier/addressing | Lifecycle rule |
|---|---|---|
| CommandStatus / FeasibilityStatus | Service-wide UUIDv7; nested item URL under owning task | Append-oriented status reports; same task ID; terminal rules in 036/037 |
| CommandResult / FeasibilityResult | Service-wide UUIDv7; nested item URL | Result record retained with task; referenced Observation/DataStream IDs remain independent |
| Stream schema | Parent ID + role/format + revision; subordinate URL | Once child records exist, incompatible schema replacement requires new stream; old schema retained for decoding/history |
| Subsystem / Subdeployment | No new type namespace | System/Deployment ResourceId and canonical URL remain top-level; nested membership is a relationship role |
| DeployedSystem | Relationship ID if independently audited | Association lifecycle, not new entity identity; IDR-SRV-017 |
| Current status / latest state | Derived view | No persistent identity unless represented by underlying DataStream/Observation/status record |
| Collection membership | Relationship/config key | Addition/removal does not change member identity |
| Alternate encoding | Same ResourceId and canonical entity | Media representation may have distinct ETag; semantic equivalence tested |
| Part 3 notification | New message/event-envelope ID | References affected ResourceId/canonical URL; retries reuse message identity as profile defines |

### 11.3 Lifecycle Transition Constraints by Family

| Transition | Allowed baseline | Prohibited shortcut |
|---|---|---|
| Description update | Same ID + revision + audit | Minting a new ID for every edit |
| Validity end | Same ID; close interval/retire as applicable | DELETE merely because `validTime` ended |
| System moved/deployed | Same System; new/updated deployment relation | Re-key System by location or deployment |
| Stream schema incompatible after data/tasks | New stream + replacement relation; old retained | In-place schema mutation or reinterpretation of children |
| Observation correction | Preserve occurrence identity and revision/supersession evidence | Silent overwrite that breaks reproducibility |
| Command canceled | Post/record `CANCELED`; retain Command | HTTP DELETE as cancellation |
| Feasibility accepted then Command submitted | New Command with relation | Reusing Feasibility ID for Command |
| Resource replaced | Old retained/retired/tombstoned + explicit replacement | Alias old identity to semantically different resource as if identical |
| Collection removed | Membership/config lifecycle only | Deleting all member resources |
| Retention purge | Protected reservation/tombstone per policy | Identifier reuse |

---

## 12. Versioning, Revision, and Identity Stability Findings

### 12.1 Stable Identity Guarantee

Within a stable Glaux API root, the following remain invariant for the lifetime of the hosted entity:

- ResourceId;
- canonical family and canonical URL;
- original creation/commit identity;
- alias ownership;
- UID history and authority assertions; and
- relationship/event references already committed.

Software SemVer, CSAPI standard version, OpenAPI version, schema package version, contract fingerprint, representation media/profile, database migration, entity revision, and ETag are stored and communicated separately. None is embedded into the canonical resource ID.

### 12.2 Revision Record

The persistence design must support, without committing to tables here:

```text
EntityRevision {
  resource_id,
  revision,
  previous_revision,
  committed_at,
  actor_or_source,
  operation,
  representation_independent_change_digest,
  lifecycle_transition?,
  reason_or_provenance_reference?
}
```

The revision value is opaque publicly. Sequentiality is per entity and cannot be used as a global event clock. A database transaction sequence or outbox offset remains implementation metadata.

### 12.3 Reproducibility

- Observations retain parent DataStream and schema revision needed to decode their result.
- Commands retain parent ControlStream and command-schema revision needed to interpret intent.
- Results/status retain owning task and causal ordering metadata.
- Historical description revisions preserve the identity/relationship/provenance context required by policy; exact snapshot API remains future work.
- Cached clients use validators and canonical links, not inferred UUID time or collection position.

### 12.4 DDIL and Federation

UUIDv7 allows distributed minting without a central sequence, but disconnected writes still require authority, clock-quality, conflict, idempotency, and reconciliation rules. On reconnection:

- local Glaux ResourceIds never change because another source used the same UID;
- source IDs remain source-qualified;
- duplicate or conflicting UIDs enter explicit reconciliation;
- aliases are created only after authorized identity proof;
- tombstones participate in synchronization so deleted identities do not resurrect; and
- ordering uses explicit commit/version vectors or later synchronization mechanisms, never UUIDv7 timestamps alone.

IDR-SRV-042/043 must select the synchronization and conflict algorithm.

---

## 13. Relationship, Temporal, Status, Event, and Command Implications

### 13.1 IDR-SRV-017 Relationship Contract Inputs

Relationships must reference typed ResourceIds/canonical URLs or external source-qualified URIs; store direction, role, validity, provenance, and independent relationship identity where history requires it. They must not embed collection-path identity. A missing/retired/tombstoned target is a typed resolution state, not a null target or permission to delete the relation.

IDR-SRV-017 must define:

- typed internal and external reference forms;
- forward/reverse link generation from one relationship graph;
- relationship IDs and uniqueness;
- source/target family constraints;
- hierarchy cycle and reparenting rules;
- replacement, tombstone, and authorization-filtered traversal; and
- UID hints in link objects without treating them as link authority.

### 13.2 Temporal and Freshness Inputs

IDR-SRV-018 must distinguish created, committed, updated, valid, observed/phenomenon, result, issued, execution, retired, archived, deleted, received, and published times. A UUIDv7 timestamp is not a substitute. Current-state views need an explicit evaluation time and the identities of source records used.

IDR-SRV-020 must define stale, last-known, unavailable, retired, and unknown operational states without changing ResourceId. Status freshness belongs in representation/domain metadata, not URL query identity.

### 13.3 Events and Part 3

One resource mutation may have:

1. affected ResourceId and canonical URL;
2. new entity revision;
3. lifecycle domain-event ID;
4. audit entry ID;
5. outbox record/sequence; and
6. one or more transport message/CloudEvent IDs.

Those values cannot be collapsed. IDR-SRV-014H's draft `subject`, `source`, `parentId`, topic, and UUID-compatible event ID are adapter concerns. The canonical model must provide stable inputs without adopting mutable draft spelling or transport topology.

### 13.4 Commands and Feasibility

The accepted HTTP/domain-state separation controls:

- before durable acceptance, malformed, unauthorized, missing-parent, conflict, and unavailable conditions are HTTP outcomes;
- after durable acceptance, `REJECTED`, `CANCELED`, `FAILED`, progress, and completion are status resources;
- cancellation retains Command identity and adds a `CANCELED` status;
- current status is derived from governed status records, not mutable identity;
- an idempotency key may map a retried request to the existing Command/Feasibility ID, but it never becomes that ID; and
- status/result creation authority, legal transition ordering, retention, and safety are resolved in IDR-SRV-036 through IDR-SRV-038.

---

## 14. Persistence, Validation, Security, Fixture, and Test Implications

### 14.1 Persistence Requirements

Persistence architecture must provide:

- service-wide unique ResourceId index and family discriminator;
- cross-family/cross-collection unique UID comparison index plus lexical value and authority;
- qualified external-identifier index `(issuer, scheme/type, comparison value)`;
- canonical path/origin record and immutable alias-target registry;
- entity revision and lifecycle-transition ledger;
- tombstone/reservation storage separate from active payload;
- parent/subordinate keys for status/result/schema records;
- source/provenance and reconciliation state;
- transactional relationship/membership/cascade support; and
- outbox/event identity separate from entities.

Database-generated sequential IDs may exist internally for performance, but every adapter must resolve them through canonical identity and never leak them accidentally. Partitioning or tenancy must not create two public resources with the same ResourceId.

### 14.2 Validation Rules

| Rule group | Required validation |
|---|---|
| ResourceId | UUIDv7 for newly minted IDs; lowercase canonical output; valid variant/version; nil/max prohibited; service-wide uniqueness |
| UID | Absolute valid URI; no fragment unless the governing scheme/object model permits it; uniqueness across collections; issuer authority; reserved-scheme policy |
| External IDs | Type/scheme/issuer required; length/character limits; original lexical preservation; safe comparison-key derivation |
| URLs | Trusted public root; route registry match; no traversal/dot segment; exact family/case; no ambiguous decoding; canonical link parity |
| Aliases | Unique normalized key; one immutable target; cycle prevention; auth equivalence; no unsafe-method redirect |
| Lifecycle | Legal transition; actor authority; no ID reuse; referenced-child/cascade checks; replacement cannot target self/cycle |
| Revision | Expected current revision/precondition; monotonicity; same transaction as mutation/audit/outbox |
| Cross-encoding | Same ResourceId, UID, relationships, validity, and semantic values after round trip |

URI normalization must be component-aware. Scheme and host case/default-port handling can be normalized for Glaux-owned HTTPS URLs; path segments and arbitrary URN namespace-specific strings are not lowercased or decoded generically. Percent-encoded octets are rejected in Glaux UUID path segments rather than accepted as aliases.

### 14.3 Security Requirements

- UUID unpredictability reduces casual enumeration but never grants access.
- UUIDv7 reveals approximate allocation time; logs/docs must state this, and APIs must use explicit domain timestamps. If a future approved profile prohibits that disclosure, ID format becomes a compatibility decision, not a silent switch.
- Resolve authorization before revealing active/tombstoned/alias/external-identifier matches.
- Equalize concealment across body, status, Location/canonical/replacement links, counts, timing, cache, search, and errors.
- Never accept a publisher's claim to an existing UID/alias without ownership/authority verification.
- Prevent Unicode/confusable, path traversal, double decoding, mixed-case, encoded-slash, host-header, and forwarded-header attacks.
- Sanitize identity-conflict problems; detailed competing identifiers and sources belong in protected audit/operator interfaces.
- Apply retention/releasability to aliases and tombstones because they can reveal prior existence and relationships.
- Task resources require object-level authorization on every canonical, nested, alias, status, and result route.

Detailed controls belong to IDR-SRV-038 through IDR-SRV-040.

### 14.4 Required Fixture Families

1. Same entity through canonical, nested, collection, and alternate-encoding views.
2. Same UID/different sources/different objects; same object/different UIDs; rehosted representation; authorized alias.
3. UUID valid v7/v4 import, uppercase input, nil/max, invalid variant/version, traversal and double-encoding cases.
4. Duplicate create, retry with idempotency key, conflicting external ID, and restore-preserved ID.
5. Revision, ETag, stale `If-Match`, no-op retry, replacement, and history cases.
6. Retired, archived, tombstoned, purged, concealed, and unknown targets with cache behavior.
7. System hierarchy/reparent, deployment completion, stream schema lock, observation correction, command cancellation, feasibility-to-command relation, and event correction.
8. Origin/proxy/base-path move with canonical links, Location, OpenAPI, aliases, and safe GET/HEAD compatibility routes.
9. DDIL independent minting, clock rollback, collision injection, sync conflict, tombstone propagation, and no resurrection.
10. Part 3/outbox cases proving resource ID, revision, event ID, message ID, subject, source, and parent remain separate.

### 14.5 Release and Conformance Tests

- Property-based UUID generation under concurrency/process restart/clock rollback with collision detector.
- Route-registry bijection: every advertised canonical family maps IDs to URLs and parses only its exact form.
- Crawl every representation and assert canonical-link convergence.
- Round-trip identifier/relationship semantics across GeoJSON, SensorML, SWE JSON/Text/Binary where applicable.
- Compare runtime routes, OpenAPI, collection metadata, aliases, lifecycle registry, and conformance declaration.
- Assert ResourceId/UID reservation after deletion and replacement.
- Assert referential/cascade conflicts and atomicity under injected failure.
- Verify 404 versus 410/concealment and cache behavior under principals/policies.
- Run generated clients, CSAPI Explorer, approved ATS, and supplemental defect tests in distinct evidence lanes.
- Measure index locality and lookup/collision behavior at expected scale without turning performance evidence into semantic identity.

---

## 15. Downstream Topic Handoff Matrix

| Topic | Binding starting inputs from IDR-SRV-016 |
|---|---|
| IDR-SRV-017 | Typed ResourceId/canonical/external references; aliases not relations; replacement not identity; relationship IDs independent; missing/tombstoned target states |
| IDR-SRV-018 | All lifecycle timestamps distinct from UUID time; valid time never identity; current/historical evaluation time |
| IDR-SRV-019 | UID/external-ID issuer, source, authority, correction, reconciliation, and trust assertions |
| IDR-SRV-020 | Operational status/freshness orthogonal to existence, validity, retention, and task state |
| IDR-SRV-021–024 | Preserve ID/UID/semantic-reference roles through GeoJSON, SensorML, SWE, validation, and round trips |
| IDR-SRV-025 | Identity registry, revision ledger, aliases/tombstones, family discrimination, tenant/public uniqueness |
| IDR-SRV-026–028 | Stable IDs across geospatial/time-series/document stores; parent schema revision and historical decoding |
| IDR-SRV-029 | Atomic mint/uniqueness/revision/audit/outbox; ETag/If-Match; idempotency key separate from ID; collision retry |
| IDR-SRV-030 | Retire/archive/tombstone/purge policy; no reuse; cascade and legal/security deletion constraints |
| IDR-SRV-031 | POST Location and multi-create identity; client-supplied ID handling; atomic membership/batch outcomes |
| IDR-SRV-032–034 | Stable IDs across encodings/query/status/event projections and corrections |
| IDR-SRV-035 | Resource/revision/domain-event/message identities separate; canonical subject/source inputs; no draft token leakage |
| IDR-SRV-036 | Command ID stable through status; cancellation not deletion; append/govern status; idempotent retry mapping |
| IDR-SRV-037 | Feasibility ID distinct from resulting Command; subordinate status/results; retention |
| IDR-SRV-038 | Identifier authority, task object authorization, replay protection, audit identities |
| IDR-SRV-039/039A/040 | Enumeration, UUID time leakage, concealment, alias/tombstone disclosure, issuer trust, cross-boundary identity |
| IDR-SRV-042/043 | Distributed UUID mint, reconciliation, conflict/tombstone propagation, no resurrection; do not order by UUID time |
| IDR-SRV-046/047 | Trusted public origin/base path; immutable deployment identity; startup route/alias collision checks |
| IDR-SRV-050/051/053/056 | Normative/project test separation; complete identity/lifecycle fixture matrix and external-client traversal |
| IDR-SRV-057 | Carry accepted distinctions and unresolved owner assignments into final synthesis |

---

## 16. Recommendations

| ID | Recommendation | Priority | Authority |
|---|---|---|---|
| R-016-01 | Mint immutable service-wide UUIDv7 ResourceIds for every canonical resource and independently identified subordinate record. | Critical | P based on CSAPI + RFC 9562 |
| R-016-02 | Keep ResourceId, UID, canonical URL, external ID, revision, relation, event, message, request, and database identity as typed concepts. | Critical | N/P |
| R-016-03 | Require valid-URI UIDs for Part 1 families; preserve issuer/source; mint `urn:uuid:` only when Glaux is authorized to allocate object identity. | Critical | N/P |
| R-016-04 | Build every public URL/link/Location/OAD path through one typed canonical route and public-origin registry. | Critical | N/P/I |
| R-016-05 | Preserve IDs/URLs through updates, encodings, memberships, deployments, and releases; use revisions and validators for change. | Critical | N/P |
| R-016-06 | Never auto-merge on UID equality; reconcile with source, authority, semantics, provenance, and explicit evidence. | Critical | P/I |
| R-016-07 | Make aliases immutable one-target mappings and prohibit automatic mutation/tasking redirects until safety/idempotency proof. | High | N/P |
| R-016-08 | Separate administrative, temporal, operational, retention, API, task, and publication lifecycle axes. | Critical | N/P |
| R-016-09 | Prefer retirement/archive over destructive deletion where audit/reproducibility applies; retain protected tombstones/reservations and never reuse IDs. | Critical | P |
| R-016-10 | Require a new stream ID for incompatible populated schema replacement; retain old schema/context for children. | Critical | N/P |
| R-016-11 | Treat Command cancellation as a status transition, not DELETE; keep Feasibility and resulting Command identities distinct. | Critical | N/P |
| R-016-12 | Make identity/lifecycle invariants transactional with revision, audit, relationships, and outbox intent. | Critical | P; mechanism in 029 |
| R-016-13 | Test collision, alias, proxy, deletion, DDIL, cross-encoding, and external-client cases as first-class release gates. | High | P/I |
| R-016-14 | Keep Part 3 message/topic identity in adapters; do not begin implementation from this report. | High | D/P |

### 16.1 Acceptance Effects

Acceptance of this report would:

- select UUIDv7 and the typed identity vocabulary as the planning baseline;
- authorize downstream plans to rely on the URI, alias, stability, replacement, and lifecycle invariants above;
- make IDR-SRV-017 the next single research topic; and
- not authorize server coding, database selection, Part 3 implementation, or any later research topic.

---

## 17. Risks, Constraints, Open Questions, and Plan Validation

### 17.1 Risks and Controls

| Risk | Consequence | Control / owner |
|---|---|---|
| UID treated as federation proof | Wrong assets merged; policy/provenance corruption | Source-qualified assertions and explicit reconciliation; 019/040/043 |
| Local ID exposed as database layout | Migration coupling/enumeration | Opaque UUIDv7 and adapter boundary; 025 |
| UUIDv7 time leakage | Creation activity inference | Document, authorize every lookup, explicit privacy profile decision; 039/040 |
| Public origin misconfiguration | Broken/hostile canonical links | Trusted configured origin and proxy tests; 046/047 |
| Alias retarget/cycle | Reference hijack or loops | Immutable unique mapping/cycle checks/audit |
| In-place schema reinterpretation | Historical data/commands become undecodable | New stream ID and retained schemas; 027/028/034/036 |
| Physical delete removes audit | Broken reproducibility/command safety | Retirement/tombstone default and retention policy; 030/038 |
| Tombstone leaks existence | Classification or relationship disclosure | Concealed 404/policy-filtered safe metadata; 039/040 |
| UUID clock rollback/generator flaw | Collision/order anomaly | CSPRNG/library validation, collision constraint, explicit timestamps; 029/042 |
| One overloaded lifecycle enum | Invalid state combinations and client ambiguity | Orthogonal axes; 018/020/030/036 |
| Redirected task mutation | Duplicate physical action | No automatic unsafe redirect; 029/036/038 |
| Part 3 identity leakage into core | Draft coupling and migration cost | Transport-neutral domain event/outbox seam; 035 |

### 17.2 Open Questions Assigned, Not Blocking IDR-SRV-017

| Question | Owner |
|---|---|
| Exact internal/public relationship record and replacement link relations | 017 |
| Exact lifecycle timestamps, interval algebra, and correction/supersession vocabulary | 018 |
| Identifier issuer registry, assurance level, and UID correction workflow | 019/040 |
| Public administrative/operational status fields | 020 |
| Database engine, schema, partitions, revision retention, tombstone storage | 025–030 |
| Mandatory ETags/preconditions and idempotency-key contract | 029 |
| Retention periods, legal erase, cascade limits, tombstone lifetime | 030/040 |
| Complete Command and Feasibility transition machines | 036/037 |
| Privacy profile response to UUIDv7 timestamp visibility | 039/040 |
| DDIL conflict/version-vector and resurrection prevention algorithm | 042/043 |
| Future incompatible API-root coexistence and permanent-move operation | Triggered major-change plan under 010A |

### 17.3 Success-Criteria Validation

- [x] Identifier classes, authorities, scopes, syntax, uniqueness, stability, and collisions are explicit.
- [x] Canonical URI patterns cover service, collections, all canonical families, nested views, status/results, and schemas.
- [x] External, SensorML, SWE, simulator, import, federation, and generated identifiers are separated.
- [x] Aliases, redirects, deprecation, replacement, revision, deletion, tombstones, and ID reuse are decided.
- [x] Lifecycle states/transitions are defined generally and by resource family without collapsing orthogonal axes.
- [x] Persistence, validation, security, fixtures, conformance, client, proxy, DDIL, and Part 3 implications are included.
- [x] Implementation observations are informative and do not override approved standards.
- [x] Every deferred decision has a named owning topic.
- [x] IDR-SRV-017 is unblocked without starting it.

### 17.4 Required Content Validation

| Plan-required content | Location | Status |
|---|---|---|
| 1. Executive summary | §1 | Complete |
| 2. Scope and plan alignment | §2 | Complete |
| 3. Evidence/authority | §3 | Complete |
| 4. Extraction methodology | §4 | Complete |
| 5. Standards inventory | §5 | Complete |
| 6. Identifier classes | §6 | Complete |
| 7. Resource-family URI strategy | §7 | Complete |
| 8. External/alternate/generated IDs | §8 | Complete |
| 9. Collision/alias/redirect/deprecation/replacement | §9 | Complete |
| 10. Lifecycle/transitions | §§10–11 | Complete |
| 11. Version/revision/stability | §12 | Complete |
| 12. Relationship/time/status/event/command implications | §13 | Complete |
| 13. Persistence/validation/security/fixture/test implications | §14 | Complete |
| 14. Downstream handoffs | §15 | Complete |
| 15. Recommendations | §16 | Complete |
| 16. Risks/constraints/open questions | §17 | Complete |
| 17. Plan validation | §17.3–17.4 | Complete |
| 18. References | §18 | Complete |

### 17.5 Matrix Field Validation

The combined matrices in §§6–7, 9–11, and 14 provide every required field: resource family; identifier class; scope/authority; canonical URI; alternate/external handling; stability; lifecycle states/transitions; collision/alias; revision/replacement; security; persistence; validation; test; handoff; and unresolved notes.

### 17.6 Review Gate and Controlled Transition

**Final status:** Research execution, synthesis, drafting, and self-review complete. Report is **In Review**.

No IDR-SRV-017 research or Part 3 implementation was started. If the Glaux Project Lead accepts this report, the next permitted iteration is exactly `IDR-SRV-017: Relationship and Linkage Model`.

---

## 18. References

### 18.1 Project and Accepted Research

- [IDR-SRV-016 Research Plan](../IDR%20Plans/idr-srv-016-identifier-uri-and-resource-lifecycle-strategy.md)
- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)
- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Research Report Template](../../../../../Governance/research-report-template.md)
- [Accepted IDR-SRV-006 Part 1 baseline](idr-srv-006-csapi-part-1-requirement-baseline-report.md)
- [Accepted IDR-SRV-007 Part 2 baseline](idr-srv-007-csapi-part-2-requirement-baseline-report.md)
- [Accepted IDR-SRV-009 service-root baseline](idr-srv-009-landing-page-api-definition-and-conformance-declaration-behavior-report.md)
- [Accepted IDR-SRV-010 navigation baseline](idr-srv-010-collections-resources-links-and-navigation-behavior-report.md)
- [Accepted IDR-SRV-010A versioning baseline](idr-srv-010a-api-versioning-backward-compatibility-and-deprecation-strategy-report.md)
- [Accepted IDR-SRV-013 error baseline](idr-srv-013-error-model-http-status-codes-and-failure-semantics-report.md)
- [Accepted IDR-SRV-014A through IDR-SRV-014H implementation/evidence studies](.)
- [Accepted IDR-SRV-015 canonical model](idr-srv-015-canonical-glaux-server-resource-model-report.md)

### 18.2 Controlling OGC Sources

- [OGC API - Connected Systems - Part 1: Feature Resources, OGC 23-001](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2: Dynamic Data, OGC 23-002](https://docs.ogc.org/is/23-002/23-002.html)
- [Official CSAPI `v1.0.0` source at `8e03b236`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)
- [OGC API - Features - Part 1: Core Corrigendum](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [OGC SensorML Encoding Standard 3.0](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC SWE Common Data Model Encoding Standard 3.0](https://docs.ogc.org/is/24-014/24-014.html)
- [OGC schema repository](https://schemas.opengis.net/)
- [W3C/OGC Semantic Sensor Network Ontology](https://www.w3.org/TR/vocab-ssn/)

### 18.3 URI, HTTP, Linking, and Identifier Authorities

- [RFC 3986 - Uniform Resource Identifier: Generic Syntax](https://www.rfc-editor.org/rfc/rfc3986.html)
- [RFC 3987 - Internationalized Resource Identifiers](https://www.rfc-editor.org/rfc/rfc3987.html)
- [RFC 9110 - HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)
- [RFC 8288 - Web Linking](https://www.rfc-editor.org/rfc/rfc8288.html)
- [RFC 9562 - Universally Unique IDentifiers](https://www.rfc-editor.org/rfc/rfc9562.html)
- [IANA Link Relation Types](https://www.iana.org/assignments/link-relations)
- [IANA URI Schemes](https://www.iana.org/assignments/uri-schemes/uri-schemes.xhtml)

### 18.4 Implementation and Interoperability Evidence

- [OpenSensorHub](https://github.com/opensensorhub/osh-core)
- [Connected Systems Go](https://github.com/OS4CSAPI/connected-systems-go)
- [pygeoapi](https://github.com/geopython/pygeoapi)
- [SECD interoperability repository](https://github.com/Sam-Bolling/csapi-server-interop-secd)
- [OS4CSAPI client](https://github.com/OS4CSAPI/ogc-client-CSAPI_2)
- [CSAPI Explorer](https://ogc-csapi-explorer.pages.dev/)
- [OS4CSAPI discussions](https://github.com/orgs/OS4CSAPI/discussions)

---

**Completion state:** Research, synthesis, report drafting, and self-review complete; the report awaits Glaux Project Lead acceptance. IDR-SRV-017 and Part 3 implementation remain unstarted.

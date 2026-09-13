# Section 017: Relationship and Linkage Model - Research Report

**Topic ID:** IDR-SRV-017<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-017 Relationship and Linkage Model](../IDR%20Plans/idr-srv-017-relationship-and-linkage-model.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core questions, all detailed-question groups, all 6 methodology phases, and all success criteria<br>
**Methodology Used:** Authority-ranked synthesis of accepted IDR-SRV-001 through IDR-SRV-016; direct extraction from the approved CSAPI 1.0 conceptual tables, requirements, encodings, schemas, and tagged artifacts; review of SensorML 3.0, SWE Common 3.0, SOSA/SSN, OGC API - Features, RFC 8288, and IANA authorities; implementation and interoperability comparison; and resource-family direction, cardinality, lifecycle, time, security, persistence, validation, and test analysis<br>
**Research Time:** Approximately 6 hours of AI-assisted execution on September 13, 2026<br>
**Official Standards Source Pin:** [`opengeospatial/ogcapi-connected-systems` `v1.0.0` at `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)<br>
**Shared Register Baseline:** OGC API - Connected Systems upstream-history register version 1.9; stable master remained `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f` at the September 13 check<br>
**Document Purpose:** Establish the typed, encoding-neutral relationship facts, linkage projections, traversal rules, external-reference states, integrity constraints, and downstream handoffs that the Rust Glaux reference server must implement<br>
**Author:** OpenAI Codex<br>
**Accepted By:** TBD pending Glaux Project Lead review<br>
**Acceptance Date:** TBD<br>
**Date:** September 13, 2026<br>
**Last Updated:** September 13, 2026

---

## Reading Guide and Evidence Labels

| Label | Meaning |
|---|---|
| **N** | Normative or standards-derived finding from an approved applicable source or incorporated artifact |
| **A** | Project-controlling AEP/STANAG adoption or operational-context finding carried from an accepted report |
| **P** | Accepted Glaux project decision or recommendation proposed here for acceptance |
| **I** | Informative implementation, test, interoperability, or community evidence |
| **D** | Official draft evidence useful for seam design but not an approved requirement |
| **X** | Published inconsistency, ambiguity, or evidence limitation that must remain visible |

A **relationship fact** records a typed semantic connection between resource references. A **link** is a representation of navigation from one Web resource to another. A **nested endpoint** is a relationship-filtered view. A **collection membership** is inclusion in a selectable set. None of those concepts is automatically interchangeable with another, and URL nesting does not by itself prove ownership.

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Relationship Extraction Methodology
5. Standards-Derived Relationship Inventory
6. Canonical Relationship Classes
7. Resource-Family Relationship Matrix
8. Directionality, Cardinality, Lifecycle, and Temporal Findings
9. Link Relation and Representation Findings
10. Traversal and Reverse-Link Findings
11. SensorML and SWE Common Relationship Implications
12. External Reference, Federation, DDIL, and Stale-Link Findings
13. Security, Policy, and Releasability Implications
14. Persistence, Validation, Fixture, and Test Implications
15. Downstream Topic Handoff Matrix
16. Recommendations
17. Risks, Constraints, and Open Questions
18. Validation Against Plan Success Criteria
19. References

---

## 1. Executive Summary

Glaux needs a relationship model, not a collection of URL strings. CSAPI Parts 1 and 2 define a connected graph through conceptual associations, canonical resources, nested routes, direct `@link` and `@id` members, collection views, and a small Part 1 relation vocabulary. SensorML adds process inheritance, physical attachment, deployment participation, nested components, and signal connections. SWE Common adds schema-local semantic and structural references. These surfaces overlap, but they do not establish one uniform public relationship-resource API or one complete link-relation vocabulary.

The recommended baseline is:

> **Glaux records the smallest authoritative typed relationship fact and projects every authorized forward link, reverse link, nested view, filter result, collection membership, and encoding-specific reference from that fact. A path, link object, embedded object, semantic URI, and stored relationship are distinct forms with explicit mappings; no representation becomes a second source of truth.** [N,P]

The canonical internal model uses a stable `RelationshipKind`, typed local or external endpoint references, canonical direction, optional independent `RelationshipId`, provenance, lifecycle state, and a temporal-qualification seam. Independently identified relationship facts use the accepted UUIDv7 policy. Simple intrinsic ownership may be persisted as an aggregate foreign key while still participating in the same domain relationship service; temporally qualified, many-to-many, federated, attributed, audited, or independently changing associations require first-class facts. Reverse links are derived from one authoritative direction rather than written twice.

Five distinctions prevent most design failures:

1. **Composition is not deployment.** A permanent subsystem is a System related to a parent System; a temporary aggregate is expressed through Deployment participation and time.
2. **Membership is not identity or ownership.** Canonical, nested, recursive, and custom-collection responses select the same resource identities.
3. **Semantic binding is not necessarily a resource edge.** SWE `definition`, unit, frame, and local component paths must remain typed semantic/schema references unless a later profile establishes a managed Property relationship.
4. **A missing target is not an absent relationship.** Active, retired, tombstoned, unresolved-external, stale, inaccessible, and concealed targets require distinct internal resolution states.
5. **Transport events are not relationships.** Resource and workflow relationships provide stable inputs to audit, outbox, and draft Part 3 adapters, but topic hierarchy, message IDs, and mutable draft fields do not redefine the domain graph.

The Part 1 `ogc-rel:` vocabulary remains controlling for its published applicability. Ordinary output uses the exact Table 3 spelling, including `ogc-rel:controlStreams`; comparison is case-insensitive under RFC 8288. Bare example values remain compatibility fixtures only. Part 2 supplies routes and `@id`/`@link` members but no complete relation vocabulary, so Glaux should define a versioned project relation namespace for missing schema, observation, command, feasibility, result, and event navigation. Those relations must never be mislabeled as approved CSAPI requirements.

Security is graph-wide. Authorization to view a source does not imply permission to reveal a target, edge, reverse count, stale cache, tombstone, or external origin. Optional links may be omitted after policy evaluation. If a standards-required association cannot be exposed safely, the server must apply a profile-defined whole-resource or error policy rather than return a structurally misleading half-truth. Exact policy behavior remains with IDR-SRV-039/040.

Acceptance of this report would establish a decision-usable relationship baseline and unlock IDR-SRV-018. It would not finalize temporal interval algebra, database tables, registration/update APIs, event vocabulary, policy rules, retention, federation protocols, or Part 3 adoption.

---

## 2. Scope and Plan Alignment

### 2.1 In Scope and Completed

- Extracted Part 1 and Part 2 associations, hierarchy, scoped routes, representation references, cardinalities, and recursion rules.
- Reconciled inherited Features/Web-linking behavior with accepted IDR-SRV-010 navigation decisions.
- Defined canonical relationship classes, endpoint types, identity, direction, uniqueness, lifecycle dependency, queryability, and derivation.
- Classified SensorML and SWE Common relationships without collapsing process/schema-local constructs into CSAPI resource edges.
- Defined local, external, federated, DDIL, stale, retired, tombstoned, inaccessible, and concealed target states.
- Identified persistence, indexing, transaction, security, validation, fixture, conformance, and interoperability implications.
- Incorporated implementation and community evidence as non-normative regression input.

### 2.2 Explicitly Out of Scope

- Exact interval algebra, current/as-of/freshness evaluation, and temporal query syntax: IDR-SRV-018.
- Provenance vocabulary, trust calculation, source reconciliation, and quality policy: IDR-SRV-019.
- Status, availability, and event vocabulary: IDR-SRV-020.
- Final SensorML/SWE codecs and semantic binding rules: IDR-SRV-021 through 024.
- Database engine, table/index layout, graph technology, and cache product: IDR-SRV-025 through 030.
- Registration/update and transaction endpoint semantics: IDR-SRV-031 through 034.
- Command/feasibility state-machine authority: IDR-SRV-036 through 038.
- Authentication, authorization, classification, release, federation, and DDIL protocols: IDR-SRV-039 through 043.
- Part 3 transport adoption and final pub/sub architecture: IDR-SRV-035.

### 2.3 Core Research-Question Coverage

| Question | Short form | Status | Evidence |
|---|---|---|---|
| CQ1 | Required relationship classes | Complete | §§5–7 |
| CQ2 | Authority and classification | Complete | §§3–7 |
| CQ3 | External representation and client navigation | Complete | §§9–12 |
| CQ4 | Temporal, lifecycle, policy, provenance, and version context | Complete within topic boundary | §§8, 12–15 |
| CQ5 | Downstream architecture, validation, conformance, fixture, and interoperability implications | Complete | §§14–18 |

### 2.4 Prior-Report Reconciliation

IDR-SRV-015 established one encoding-neutral typed aggregate graph, relationship facts, external references, and projection-only routes/encodings. IDR-SRV-016 fixed ResourceId, UID, canonical URL, alias, replacement, revision, tombstone, and missing-target distinctions. This report preserves those decisions: aliases are not relationships; replacement does not merge identity; collection paths are not relationship-target identity; local targets use typed ResourceIds/canonical URLs; external targets remain source-qualified; and relationship identity is independent where history or mutation requires it.

IDR-SRV-010's navigation registry and exact Table 3 output spelling remain controlling. This report supplies the internal relationship-kind mapping and generation rules that IDR-SRV-010 intentionally deferred. It does not reopen accepted paths, aliases, or relation-output policy.

---

## 3. Evidence Base and Authority Classification

### 3.1 Primary Sources Reviewed

| Source | Version/status | Authority and stable anchors | Access date | Limitations |
|---|---|---|---|---|
| [OGC 23-001, CSAPI Part 1](https://docs.ogc.org/is/23-001/23-001.html) | Version 1.0, approved | §§7.3–7.10, 9–19; conceptual association tables; Requirements 1–91 | 2026-09-13 | Published relation and example inconsistencies remain [X] |
| [OGC 23-002, CSAPI Part 2](https://docs.ogc.org/is/23-002/23-002.html) | Version 1.0, approved | §§7–16; DataStream, Observation, ControlStream, Command, status, result, feasibility, event tables; Requirements 1–104 | 2026-09-13 | Paths, OAS, schemas, and ATS contain known singular/plural and coverage defects [X] |
| [Official CSAPI release source](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2) | `v1.0.0`, commit `8e03b236...` | `api/part1`, `api/part2`, `common/link.json` | 2026-09-13 | Reproduces approved package artifacts; examples are informative unless incorporated |
| [OGC API - Features Part 1](https://docs.ogc.org/is/17-069r4/17-069r4.html) | 1.0.1 corrigendum, approved | §§7.12–7.16, Requirements 12–35 | 2026-09-13 | Collection/item rules do not define domain-edge semantics |
| [SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html) | Version 3.0, approved | §§7–9, especially `typeOf`, `attachedTo`, aggregate components/connections, deployment | 2026-09-13 | Rich description graph is not automatically an API resource graph |
| [SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html) | Version 3.0, approved | §§7–10; components, `definition`, unit, reference-frame, field/element structure | 2026-09-13 | Schema-local paths and semantic identifiers require context |
| [SOSA/SSN](https://www.w3.org/TR/vocab-ssn/) | W3C/OGC Recommendation | System, Deployment, `hasSubSystem`, `deployedSystem`/`hasDeployment`, observation/procedure relations | 2026-09-13 | Ontology inference does not add undocumented CSAPI endpoints |
| [RFC 8288](https://www.rfc-editor.org/rfc/rfc8288) | Standards Track | §§2–3; link context/target, registered and extension relation types | 2026-09-13 | Does not define CSAPI domain relations |
| [IANA Link Relations](https://www.iana.org/assignments/link-relations) | Current registry | `self`, `alternate`, `canonical`, `collection`, `item`, `related`, `describedby`, `status`, paging, service relations | 2026-09-13 | OGC and CSAPI extension values are outside the registry |

### 3.2 Project-Controlling and Accepted Inputs

| Source | Role in this topic |
|---|---|
| IDR-SRV-001–005 | AEP/STANAG operational boundary: preserve linked, trustworthy context without absorbing external systems or adjacent standards |
| IDR-SRV-006–008 | Accepted Part 1/Part 2/conformance requirement and defect baselines |
| IDR-SRV-009–014 | Root, navigation, query, representation, errors, OpenAPI, and deployment-operation constraints |
| IDR-SRV-014A–014G | Pinned implementation, smoke, interoperability, and community regression evidence |
| IDR-SRV-014H | Draft Part 3 authority boundary and transport-neutral relationship/event seam |
| IDR-SRV-015 | Canonical resource families and typed aggregate graph |
| IDR-SRV-016 | Typed identity, canonical addressing, alias/replacement, lifecycle, and target-resolution constraints |

The controlled `AC/224(JCGISR)D(2026)0005` package dated 27 April 2026 remains the project AEP/STANAG baseline with recorded SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`. It was not redistributed or newly quoted here. Accepted IDR-SRV-001 through 003 establish that AEP-4789 Volume II adopts CSAPI Parts 1 and 2, SensorML 3.0, and SWE Common 3.0, while the detailed technical rules remain in those OGC standards. Volume I requires discoverable, understandable, linked, trustworthy, secure context but does not create additional public relationship encodings. [A]

### 3.3 Supporting Implementation and Community Evidence

| Evidence | Relationship lesson | Authority limit |
|---|---|---|
| IDR-SRV-014A, OpenSensorHub | Nested-handler reuse is useful; accepted links can be silently dropped; verify create/read/traverse/restart | Implementation and issue evidence only [I] |
| IDR-SRV-014B, connected-systems-go | Typed repositories, relationship helpers, absolute-link repair, and E2E association tests are useful precedents | Go/PostGIS choices are not Glaux requirements [I] |
| IDR-SRV-014C, pygeoapi | Representation-separated stores and analyzed-text relationship fields can break equivalent nested/filter results | Dated deployment and source behavior only [I] |
| IDR-SRV-014D–014F, SECD/client evidence | Generic `alternate` links, proprietary `parentId`, silent filters, and wrapper differences impair portable traversal | Compatibility corpus, not normative output [I] |
| IDR-SRV-014G, OS4CSAPI discussions | Public-base URL failures, graph-mechanic vocabulary, client loss of nested structures, and absence-state distinctions are useful scenarios | Discussion claims are not standards requirements [I] |

### 3.4 Authority and Conflict Rules

1. Approved standards and incorporated requirements control normative claims.
2. AEP/STANAG material controls project adoption and operational scope, not invented wire syntax.
3. Accepted prior IDR decisions control Glaux planning unless this report explicitly identifies and resolves a conflict.
4. Tagged schemas/OAS/ATS inform exact structures; conflict with normative prose remains visible.
5. Implementations, tests, examples, and discussions supply hypotheses and regression cases, never new obligations by popularity.
6. Project extensions are named, versioned, capability-advertised, and kept separate from conformance claims.

---

## 4. Relationship Extraction Methodology

### 4.1 Extraction Unit

Each candidate relation was recorded using the plan's minimum fields: relationship identifier; source and target family; semantic name/class; source anchor; authority; canonical direction; cardinality; lifecycle dependency; temporal validity; representation/link relation; queryability; security; persistence; validation; test; handoff; and unresolved notes.

An association entered the canonical matrix only when at least one of these held:

- an approved conceptual table, requirement, or incorporated schema defines it;
- an accepted prior report requires it as an implementation-support fact;
- it is a necessary inverse or query projection of an authoritative fact; or
- the report explicitly classifies it as a bounded project extension.

### 4.2 Classification Axes

| Axis | Values used |
|---|---|
| Authority | normative, inherited, representation-specific, AEP-context, derived, project-support, implementation-specific, draft, unresolved |
| Edge form | intrinsic parent, hierarchy, association fact, membership, workflow ownership, semantic binding, schema-local connection, external reference, derived view |
| Storage posture | aggregate-local, first-class fact, append-oriented child, derived/indexed, external cache/reference, undecided |
| Direction | canonical forward, inverse projection, symmetric only if explicitly defined |
| Cardinality | exactly one, zero-or-one, zero-or-many, one-or-many, mutually exclusive variant |
| Time | stable/current revision, validity-qualified, occurrence-time contextual, derived-as-of, freshness-qualified |
| Exposure | direct member, link, nested endpoint, collection/filter, embedded object, ID reference, semantic URI, internal only |

### 4.3 Decision Tests

A relationship requires an independently identified first-class fact when it has its own validity, provenance, role/attributes, lifecycle, mutation authority, external-source assertion, audit value, or many-to-many identity. It may remain aggregate-local when it is an intrinsic exactly-one owner reference with no separate lifecycle. It is derived when another authoritative edge completely determines it. It remains representation-local when its endpoint exists only inside a SensorML/SWE document or schema tree.

No relation is inferred solely from a URL shape, coincident UID, link title, field-name resemblance, RDF inverse, or co-membership in a result page.

### 4.4 Validation of Extraction

The extraction was checked in four passes:

1. conceptual model/table against route requirements;
2. route and table against JSON/GeoJSON/SensorML schemas;
3. standards result against IDR-SRV-010, 015, and 016;
4. model against known OSH, CS-Go, pygeoapi, SECD, client, proxy, and draft Part 3 failure cases.

---

## 5. Standards-Derived Relationship Inventory

### 5.1 Part 1 Associations

| Source | Association | Target | Published cardinality/use | Primary surface |
|---|---|---|---|---|
| System | `systemKind` | Procedure | zero-or-one | direct `systemKind@link` / SensorML `typeOf` |
| System | subsystems | System | zero-or-many; relationship present under class | nested set plus `ogc-rel:subsystems`; inverse parent |
| System | sampling features | Sampling Feature | zero-or-many; relationship present under class | nested set plus `ogc-rel:samplingFeatures` |
| System | deployments | Deployment | zero-or-many, optional association | nested set plus `ogc-rel:deployments` |
| System | implements procedures | Procedure | zero-or-many, optional | `ogc-rel:procedures`; inverse `implementingSystems` |
| System | datastreams / controlstreams | Part 2 streams | zero-or-many | scoped endpoints and Table 3 relations |
| Deployment | platform | System or external Feature | zero-or-one | direct `platform@link` / SensorML deployment field |
| Deployment | deployed systems | System | zero-or-many; relationship present | direct `deployedSystems@link` / SensorML deployed-system objects |
| Deployment | subdeployments | Deployment | zero-or-many; relationship present under class | nested set and parent/child relations |
| Deployment | FOI / sampling features / streams | respective target | zero-or-many, conditional/optional | association links and scoped/recursive views |
| Procedure | implementing systems | System | zero-or-many, optional | `ogc-rel:implementingSystems` inverse view |
| Sampling Feature | parent System | System | exactly one in the resource model | `ogc-rel:parentSystem` |
| Sampling Feature | sampled feature | local/external Feature | exactly one in the resource model | direct `sampledFeature@link`; Table 3 placement conflict [X] |
| Sampling Feature | sample of | Sampling Feature | zero-or-many, optional | `ogc-rel:sampleOf` |

The Part 1 Table 3 vocabulary has thirteen exact published values. Its `ogc-rel:featuresOfInterest` row names System applicability not supported by the System model/mappings, and `ogc-rel:controlStreams` differs lexically from lower-case resource/path spelling. Glaux follows the accepted IDR-SRV-010 conflict disposition rather than inventing a System-to-FOI edge or a second relation identity. [N,X,P]

### 5.2 Composition, Aggregation, and Recursion

Part 1 distinguishes permanent/integral **composition**, created as nested subsystem resources, from temporary **aggregation**, expressed by deployment. A subsystem remains an ordinary System with canonical identity. A subdeployment remains an ordinary Deployment. System and Deployment recursive searches require cycle-safe direct/transitive selection, and their related Sampling Feature, stream, deployed-System, or FOI views aggregate recursively as prescribed. Recommended System `datetime` behavior includes permanently attached and appropriately deployed components for current, instant, or interval evaluation. [N]

### 5.3 Part 2 Associations

| Source | Association | Target | Usage/cardinality | Representation/route consequence |
|---|---|---|---|---|
| DataStream | system | System | exactly one, required | `system@link`; System-scoped reverse view |
| DataStream | observations | Observation | zero-or-many set, required association | nested observations endpoint |
| DataStream | procedure / deployment | Procedure / Deployment | zero-or-one, optional | direct `@link` fields |
| DataStream | sampling features / FOIs | list, optional | zero-or-many | direct link/list members and conditional scoped endpoints |
| Observation | datastream | DataStream | exactly one, required | `datastream@id`; nested membership |
| Observation | sampling feature / procedure | Sampling Feature / Procedure | zero-or-one, optional | local `@id` and/or `@link` as schema defines |
| ControlStream | system | System | exactly one, required | `system@link`; System-scoped reverse view |
| ControlStream | commands | Command | zero-or-many set, required association | nested commands endpoint |
| ControlStream | procedure / deployment | Procedure / Deployment | zero-or-one, optional | direct `@link` fields |
| ControlStream | sampling features / FOIs | list, optional | zero-or-many | direct members and conditional scoped endpoints |
| Command | controlstream | ControlStream | exactly one structurally | `controlstream@id`; nested membership |
| Command | sampling feature / procedure | Sampling Feature / Procedure | zero-or-one | `samplingFeature@id`, `procedure@link` |
| Command | status / result | CommandStatus / CommandResult | ordered zero-or-many | subordinate nested endpoints |
| CommandStatus | command or feasibility owner | Command/Feasibility | exactly one logical owner | owner ID and nested route; polymorphic owner must be explicit internally |
| CommandResult | output target | Observation list, one DataStream, external resource, or inline data | at least one allowed variant | mutually constrained `data` / `observation@link` / set / stream / external form |
| Feasibility | controlstream / target / procedure | same structural context as Command | command-like | separate identity and subordinate status/result |
| SystemEvent | system | System | exactly one | global canonical event plus System-scoped history |

Part 2 maps several associations to SOSA/SSN predicates, but these mappings are semantic annotations, not permission to infer additional API facts or endpoints. The published `sosa:madeByActuator` mapping on a ControlStream-to-System association is directionally awkward because a stream is not itself an Actuation; Glaux preserves it in representation traceability without using ontology inference as the operational relationship key. [N,X]

The CommandResult conceptual text says at least one output property must be present, while the tagged JSON schema uses `oneOf` across inline, single-Observation, Observation-set, DataStream, and external-link branches. In that JSON encoding exactly one branch must validate; the canonical model retains a typed output variant and records the broader conceptual wording as a publication conflict rather than accepting arbitrary multi-branch payloads. [N,X]

### 5.4 Inherited Web and Collection Relationships

Features collection/item links describe Web-resource context: `self`, `alternate`, `collection`, and OGC `items`; paging uses `next` and optionally `prev`; CSAPI adds `canonical` for noncanonical item views. These relations connect representations and views. They do not replace the domain facts that select membership. One resource may appear in canonical, nested, recursive, and multiple custom collections while retaining one identity. [N]

### 5.5 Relationship Classes Not Established by the Standards

The following tempting edges must not be invented as baseline authoritative facts:

- System directly owns every Observation, Command, status, result, or Feasibility resource; these are derived through streams/workflow parents.
- Procedure directly belongs to a Deployment; any connection is derived through participating Systems/streams unless asserted by a later profile.
- Every SWE `definition` URI is a local Property resource.
- Every SensorML component is an independently addressable CSAPI System.
- Every System has a direct FOI relationship because Table 3 contains a stray applicability entry.
- A Part 3 topic segment or `parentId` draft field is a canonical relationship record.

---

## 6. Canonical Relationship Classes

### 6.1 Relationship Fact Contract

The encoding-neutral contract should support this logical shape without fixing Rust structs or database tables:

| Field | Rule |
|---|---|
| `relationship_id` | Typed `RelationshipId`; UUIDv7 when the fact is independently identified; omitted only for aggregate-intrinsic edges whose owner revision supplies identity |
| `kind` | Stable `RelationshipKind` enum independent from wire spelling |
| `source` / `target` | Typed `ResourceRef`; local ResourceId/family or external source-qualified URI/reference |
| `canonical_direction` | Exactly one writable semantic direction; inverse names are projections |
| `role` / attributes | Kind-specific validated values, never arbitrary labels that change semantics |
| `validity` | Optional relationship-validity seam; interval semantics owned by IDR-SRV-018 |
| `provenance` | Assertion/derivation source and authority seam; vocabulary owned by IDR-SRV-019 |
| `lifecycle` | Active, retired/superseded, or protected historical state; exact retention owned downstream |
| `resolution` | Local active, local retired, local tombstoned, external unresolved, external resolved/fresh/stale/unavailable, inaccessible, or concealed |
| `revision` | Relationship revision or containing aggregate revision; never inferred from UUID time |
| `derivation` | Asserted, aggregate-intrinsic, inverse-derived, closure-derived, rule-derived, policy-filtered, or cache-derived |
| `anchors` | Standards requirement/table and encoding-map identifiers for traceability |

### 6.2 Seven Operational Classes

| Class | Examples | Canonical behavior |
|---|---|---|
| Intrinsic ownership/context | DataStream→System, Observation→DataStream, Command→ControlStream | exactly one; aggregate/local FK permissible; parent change is governed and audited |
| Composition hierarchy | System→child System, Deployment→child Deployment | at most one direct parent; cycle-prohibited; closure derived |
| Qualified association fact | Deployment→deployed System, System→Procedure, stream→Deployment/SF/FOI | first-class when temporal, many-to-many, attributed, or independently mutable |
| Workflow ownership/history | Command/Feasibility→status/result | append-oriented ordered children; subordinate identity; parent not inferred from URL alone |
| Collection membership | resource→custom collection | independent selection fact/rule; removal does not delete resource |
| Semantic/schema reference | Property base, SWE definition/unit/frame, SensorML connection path | typed URI/local-path/reference with context; not automatically a resource edge |
| External/federated reference | platform/FOI/result/schema/service | source-qualified, non-owning, freshness/provenance/policy aware |

### 6.3 Authoritative, Derived, and Materialized

Only the authoritative direction is writable. Reverse views, nested memberships, recursive closures, and direct/indirect query projections are derived. They may be materialized or indexed for performance if transactionally tied to the same fact and provably rebuildable. A materialized view never becomes a second source of truth.

### 6.4 Uniqueness and Duplicate Suppression

Default uniqueness is `(kind, source, target, role, validity identity, authority scope)`, refined per kind. Intrinsic edges usually allow one active target per source. Hierarchies allow one active permanent parent but may coexist with multiple deployment aggregations. Repeating the same target in recursive or multi-hop selection is deduplicated by canonical ResourceId, while distinct relationship facts remain available for audit or qualified queries.

Aliases, equivalent UIDs, replacement links, and matching external identifiers do not collapse two endpoint identities or two relationship facts. Reconciliation must be explicit and authorized.

---

## 7. Resource-Family Relationship Matrix

**Abbreviations:** `1` exactly one; `0..1`; `*` zero-or-many; `own` lifecycle-dependent child; `assoc` non-owning association; `T` temporal seam; `D` derived; `E` external-capable; `P1/P2` approved CSAPI part; `Proj` project extension/projection.

| ID | Source → target | Kind / class | Anchor / authority | Direction / cardinality | Lifecycle / time | Representation / query | Security, persistence, validation, test | Handoff / notes |
|---|---|---|---|---|---|---|---|---|
| REL-001 | System → System | permanent subsystem / hierarchy | P1 §§10.2–10.7; SSN `hasSubSystem` [N] | parent→child; parent `0..1`, children `*` | composition; revision-valid; cycle-free | `ogc-rel:subsystems`; inverse `parentSystem`; nested/recursive/datetime | fact or aggregate FK + closure; target type, no self/cycle; depth/reparent/restart/policy tests | 018,025,030,032,040,053 |
| REL-002 | Deployment → Deployment | subdeployment / hierarchy | P1 §12 [N] | parent→child; parent `0..1`, children `*` | composition-like hierarchy; T seam | parent/subdeployment relations; nested/recursive | same graph invariants as REL-001; separate kind prevents cross-family cycles | 018,025,030,053 |
| REL-003 | Deployment → System | deployed-system participation / qualified association | P1 Deployment table; SSN `deployedSystem` [N] | deployment→system; many-to-many | assoc, not ownership; T/provenance/role | direct `deployedSystems@link` or SensorML objects; inverse System deployments; filters | first-class fact; overlap/authority/dedup tests; do not cascade-delete System | 018,019,025,031,040,042 |
| REL-004 | Deployment → System/external Feature | platform / external-capable assoc | P1 Deployment table [N] | deployment→platform; `0..1` | non-owning; T with deployment | direct `platform@link`; typed local/external ref | polymorphic allow-list; no external cascade; broken/stale/policy cases | 018,019,021,040,043 |
| REL-005 | System → Procedure | system kind / type link | P1 System table; SensorML `typeOf` [N] | system→procedure; `0..1` | stable per description revision | direct `systemKind@link` / `typeOf` | resolvable Process-compatible target; cross-encoding equality | 019,021,023 |
| REL-006 | System → Procedure | implements / many-to-many assoc | P1 tables; SSN `implements` [N] | system→procedure; `*`; inverse implementing systems | non-owning; validity/provenance seam | `ogc-rel:procedures`; inverse `implementingSystems`; filter | first-class or join fact; avoid duplicating REL-005 semantics | 018,019,025,053 |
| REL-007 | System → Sampling Feature | sampling-feature context / intrinsic association | P1 System/SF tables [N] | system→SF; System `*`, SF parent `1` | SF context dependency; validity seam | nested set / `samplingFeatures`; inverse `parentSystem` | exactly one parent, local target; create/read/reverse/restart | 018,025,030,032,053 |
| REL-008 | Sampling Feature → local/external Feature | sampled feature / semantic domain assoc | P1 SF table [N,X] | SF→Feature; `1` | non-owning external-capable; validity/provenance | direct `sampledFeature@link`; Table 3 generic-link conflict retained | typed local/external; must not delete target; placement golden tests | 018,019,021,040,053 |
| REL-009 | Sampling Feature → Sampling Feature | sample-of / directed chain | P1 SF table [N] | sample→sampled SF; `*` | non-owning; T seam | `ogc-rel:sampleOf`; filter/link set | cycle policy required; local/external resolution as allowed by encoding | 018,024,025,040 |
| REL-010 | Property → Property/semantic URI | base property / semantic derivation | P1 Property/SensorML derived property [N] | derived→base; `0..1` | revision-valid | direct semantic reference, not generic relation link by default | local cycles prohibited; external URI source-qualified | 019,021,024,025 |
| REL-011 | DataStream → System | produced by / intrinsic context | P2 DataStream table [N] | stream→system; `1`; inverse System streams | stream lifecycle depends on context; no automatic System ownership decision | `system@link`; System-scoped endpoint; `ogc-rel:datastreams` on System | aggregate FK acceptable; target exists/type/authorization; route/filter equality | 025,027,030,032,034,040 |
| REL-012 | DataStream → Observation | stream membership / ownership | P2 DataStream/Observation tables [N] | stream→observation `*`; inverse observation→stream `1` | own; append-oriented; occurrence times separate | nested observations; observation `datastream@id`; canonical item | child key/index; no cross-stream duplicate; canonical/nested equality and cascade/retention tests | 018,027,030,034,053 |
| REL-013 | DataStream → Procedure/Deployment | common context / optional assoc | P2 DataStream table [N] | stream→each; `0..1` | deployment edge T; procedure revision-valid | direct `procedure@link`, `deployment@link`; filters as defined | typed target; deployment interval intersection for scoped view | 018,019,025,034 |
| REL-014 | DataStream → Sampling Feature/FOI | subject context / assoc | P2 DataStream table [N] | stream→target; `*` | T/contextual; FOI E | direct members and conditional nested endpoints | local/external distinction; nested route only local hosted targets; membership/filter equality | 018,019,024,026,034,040 |
| REL-015 | Observation → Sampling Feature/Procedure | record-specific context | P2 Observation table [N] | observation→each; `0..1` | occurrence-context; explicit may refine stream common context | `samplingFeature@id`, `procedure@link` as schema defines | distinguish absent from inherited/effective; schema/target compatibility | 018,019,023,024,034 |
| REL-016 | ControlStream → System | receiving system / intrinsic context | P2 ControlStream table [N,X] | stream→system; `1`; inverse System streams | stream context; no inferred Actuation identity | `system@link`; System-scoped endpoint; `ogc-rel:controlStreams` | same integrity as REL-011; preserve predicate mapping only as annotation | 025,030,032,036,040 |
| REL-017 | ControlStream → Command/Feasibility | workflow membership / ownership | P2 §§10–11 [N,X] | stream→child `*`; child→stream `1` | own; command-like resources retain identity/history | nested commands/feasibility; local IDs; canonical items | parent-scoped authorization, schema binding, alias/path defect fixtures | 030,033,036,037,040,053 |
| REL-018 | ControlStream → Procedure/Deployment/SF/FOI | common control context / assoc | P2 ControlStream table [N] | same cardinalities as DataStream | T/contextual; FOI E | direct fields and conditional scoped endpoints | verify targets compatible with controlled properties and authorization | 018,019,024,025,036,040 |
| REL-019 | Command/Feasibility → Sampling Feature/Procedure | task-specific context | P2 Command model/Feasibility reuse [N] | `0..1` each | workflow contextual; retained with request | `samplingFeature@id`, `procedure@link` | effective context rules and command authority tested | 019,024,036,037,040 |
| REL-020 | Command/Feasibility → status/result | workflow child history | P2 §§10.9–10.14, 11 [N,X] | parent→children `*`; each child→one parent | own; append/history; ordering uses explicit times/sequences | subordinate `/status` and `/result`; project relations | polymorphic owner type, legal lifecycle, no orphan, transactional append | 018,020,025,029,036–038,040 |
| REL-021 | CommandResult → Observation(s)/DataStream/external | output-reference variant | P2 CommandResult table/schema [N,X] | result→target; list/single/external/inline constraints | retained audit; external non-owning | `observation@link`, observation-set, `datastream@link`, `external@link`, or inline | enforce allowed combinations; external stale/concealed; task-to-data trace tests | 019,023,027,036,038,040,043 |
| REL-022 | SystemEvent → System | concerns / intrinsic event context | P2 SystemEvent table [N] | event→system `1`; inverse System events `*` | event append/history; occurrence time separate | event System ref; `/systems/{id}/events`; global canonical event | parent exists/policy; nested/global identity and correction tests | 018,019,020,025,034,040 |
| REL-023 | Resource → custom collection | membership / selection | Features + P1/P2 collection rules [N,P] | many-to-many | no ownership; explicit/rule validity seam | collection/items views and `collection`/`canonical` links | fact/rule/index; removal ≠ delete; duplicate/canonical parity tests | 018,025,030,032,040 |
| REL-024 | Stream → schema revision | schema binding / lifecycle dependency | P2 schema endpoints [N,P] | stream→active schema `1` plus history | schema revision governed; populated incompatible change creates new stream per IDR-016 | `/schema`; `describedby`/project schema relation | immutable historical compatibility; child round-trip/golden tests | 018,023,025,027,034,036 |
| REL-025 | SensorML aggregate → component/connection | representation-local process graph | SensorML §§8.4, 9.3 [N] | nested components/connections | document revision-local | embedded objects and local path endpoints | validate path resolution/acyclic rules as SensorML requires; do not mint CSAPI resources automatically | 021,023,025 |
| REL-026 | SWE component → semantic definition/unit/frame/child | schema-local semantic/structural binding | SWE Common §§7–10 [N] | directed tree/reference | schema revision-local | URI, `uom`, `referenceFrame`, field/element path | validate in schema context; never equate URI with local Property without mapping | 022–024,025 |
| REL-027 | Local resource → external resource/service | federated reference | P1 cross-API linking; P2 external results; AEP context [N,A,P] | local→external; `*` by kind | non-owning; freshness/resolution seam | Link object with absolute URI, media/title/UID hints | source-qualified cache/reference; SSRF-safe validation; stale/unavailable/concealed tests | 019,023,039–043,055–056 |
| REL-028 | Resource mutation → audit/domain event/outbox message | implementation-support causality | IDR-015/016; draft P3 seam [P,D] | resource/revision→records; one-to-many | transactional/audit retention | internal and transport adapters; not CSAPI domain link by default | distinct IDs, atomic outbox, authorization, replay/idempotency | 019,020,029,033,035,038,042 |

The matrix intentionally omits direct System→Observation/Command/status/result edges and Procedure→Deployment edges. Those are derived traversal paths, not independent baseline facts. It also keeps status DataStreams/Observations separate from CommandStatus workflow records.

---
## 8. Directionality, Cardinality, Lifecycle, and Temporal Findings

### 8.1 Canonical Direction and Inverses

Every `RelationshipKind` has one canonical semantic orientation even if the API exposes both directions. The recommended canonical orientations are parent→child for hierarchies, owner/container→child for membership histories, describing resource→referenced context for direct associations, and local resource→external target for federation. A reverse name such as `parentSystem`, `implementingSystems`, or a System-scoped stream view is generated from the same fact.

This rule prevents two writable rows from disagreeing. It also permits different wire projections: REL-007 is stored conceptually as System→Sampling Feature even though a Sampling Feature emits a `parentSystem` link; REL-003 is Deployment→System even though System exposes deployments.

### 8.2 Cardinality and Family Constraints

Cardinality is enforced at the active-fact boundary, not inferred from JSON array shape:

- a Sampling Feature has one parent System and one sampled feature under the Part 1 conceptual model;
- a DataStream and ControlStream each have one System;
- an Observation has one DataStream;
- a Command or Feasibility record has one ControlStream;
- a status/result record has one typed workflow owner even where the reused wire schema uses a command-named member;
- a System or Deployment has at most one active permanent parent in its own hierarchy;
- deployment participation, implemented Procedures, collection membership, sample-of, and subject/FOI links can be many-to-many; and
- CommandResult variants follow their schema's allowed combinations rather than a universal one-target rule.

Wrong-family references fail before persistence. A UUID that resolves to a resource of the wrong family is not a valid edge. External targets use a kind-specific target allow-list and do not bypass semantic validation merely because they are URLs.

### 8.3 Lifecycle Dependency Classes

| Dependency | Meaning | Examples | Delete/retire implication |
|---|---|---|---|
| Strong child ownership | Child cannot be interpreted without retained parent context | Observation/DataStream; Command/ControlStream; status/result/workflow owner; schema revision/stream | Retain or cascade under IDR-030; never leave an untyped orphan |
| Composition context | Child is an independently addressable resource in a permanent hierarchy | subsystem, subdeployment | Parent deletion must reject, cascade, or reparent transactionally under explicit policy |
| Non-owning association | Target lifecycle is independent | Deployment/System, System/Procedure, stream/Deployment/SF/FOI | Removing edge does not delete target |
| Membership | Inclusion in a view/collection | custom collection membership | Removing membership never deletes canonical resource |
| External reference | Glaux does not own target | external FOI/platform/result/schema/service | Local deletion removes/refires local fact only; never invokes remote deletion |
| Derived view | Rebuildable projection | reverse links, recursive closure, indirect query result | No independent lifecycle; invalidated/rebuilt with authoritative fact |

IDR-SRV-030 owns retention and cascade mechanics. This report fixes only the dependency class and the rule that cascading across non-owning or external edges is prohibited.

### 8.4 Temporal Relationship Seams

Relationship validity is distinct from resource validity, observation phenomenon time, result time, command issue/execution time, event time, database commit time, cache freshness, and publication time. A relationship fact therefore has an optional validity seam without selecting interval syntax here.

Relationships that require explicit IDR-SRV-018 treatment include:

- Deployment→System participation and Deployment→platform;
- System membership obtained through deployment aggregation rather than permanent composition;
- stream→Deployment and derived Deployment-scoped stream membership;
- System/Deployment recursive association views evaluated at current, instant, or interval time;
- sensor replacement/reparenting and configuration-dependent associations;
- collection or publisher assertions with limited validity;
- effective stream/record context when an Observation or Command supplies an override; and
- external cache resolution and relationship freshness, which must not be confused with validity.

Permanent composition still changes across revisions and must be auditable, but it need not be modeled as a business-time interval unless IDR-SRV-018 selects that uniformity. Status and result chronology belongs to workflow/event time rather than turning every parent-child edge into a temporal association.

### 8.5 Reparenting, Cycles, and Replacement

System and Deployment hierarchies reject self-links and cycles before commit. Reparenting is a governed transaction that removes or retires the old active fact, creates the new fact, updates closures and nested views, increments affected revisions, records audit/outbox intent, and checks policy on both old and new graph neighborhoods. A target's replacement or tombstone never silently rewrites existing relationship history. New facts may point to a replacement only through an explicit authorized decision.

Sampling `sampleOf` and local Property derivation also require cycle detection. SensorML process signal connections follow their own document-local validity rules and must not be fed into the CSAPI hierarchy closure engine.

---

## 9. Link Relation and Representation Findings

### 9.1 Link Contract

The tagged shared Link schema requires `href` and permits `rel`, `type`, `hreflang`, `title`, `uid`, `rt`, and `if`. Endpoint requirements can make `rel` and `type` mandatory even though the common schema is permissive. `uid` is a target hint, never the authority for the link or a substitute for resolving `href`/ResourceId. Glaux must not emit undeclared JSON members merely because an open schema accepts them. [N,P]

All Glaux-generated local links use the trusted public origin/base path and route registry accepted in IDR-SRV-009/010/016. They never expose an internal reverse-proxy host or derive canonical identity from the request Host header without the approved proxy trust policy.

### 9.2 Registered and OGC Navigation Relations

| Relation | Use in Glaux relationship projection |
|---|---|
| `self` | exact response or representation URL |
| `alternate` | another representation of the same Web resource, never a generic domain relationship |
| `canonical` | preferred canonical item URL when current retrieval is noncanonical |
| `collection` | containing collection context for an item representation |
| `item` | IANA relation available when a generic collection-to-item link is appropriate; do not replace OGC `items` |
| `items` | OGC-defined items endpoint from a collection descriptor |
| `next` / `prev` | page navigation within the same authorized selection |
| `related` | bounded fallback when no precise relation exists; not a substitute for known CSAPI relations |
| `describedby` | applicable schema or semantic description; not the same as owning a schema revision |
| `status` | suitable registered project navigation from a command-like context to status; not prescribed by Part 2 |
| `service-desc` / `service-doc` | API definition and human documentation |
| `data` / `conformance` | OGC root discovery relations |

### 9.3 CSAPI `ogc-rel:` Vocabulary

Ordinary output uses these exact Part 1 Table 3 values: `ogc-rel:parentSystem`, `ogc-rel:subsystems`, `ogc-rel:samplingFeatures`, `ogc-rel:deployments`, `ogc-rel:procedures`, `ogc-rel:parentDeployment`, `ogc-rel:subdeployments`, `ogc-rel:featuresOfInterest`, `ogc-rel:implementingSystems`, `ogc-rel:sampledFeature`, `ogc-rel:sampleOf`, `ogc-rel:datastreams`, and `ogc-rel:controlStreams`.

They are unregistered extension relation URIs, not IANA tokens. RFC 8288 makes extension relation comparison case-insensitive, so lower-case `ogc-rel:controlstreams` is the same relation identity, not an alias. HTTP `Link` headers quote the value because of the colon. Known bare forms seen in tagged examples may be accepted only by a named compatibility adapter with telemetry and fixtures; Glaux does not emit duplicate bare and prefixed links. [N,X,P]

### 9.4 Direct Members, IDs, Embedding, and Links

The relationship projector must honor encoding-specific placement:

- Part 1 uses direct `systemKind@link`, `platform@link`, `deployedSystems@link`, and `sampledFeature@link` members alongside generic link arrays.
- Part 2 uses `system@link`, `procedure@link`, `deployment@link`, `featureOfInterest@link`, result link variants, and API-local `datastream@id`, `controlstream@id`, `command@id`, and `samplingFeature@id` members.
- SensorML may embed component/process/deployment structures and resolvable process links.
- SWE schemas embed hierarchical components and semantic/unit/frame references.

A serializer obtains one typed fact/reference and maps it into the selected encoding. It does not parse its own output back into the domain to establish truth. Client input is decoded into typed commands and validated before relationship mutation.

### 9.5 Project Relation Namespace for Part 2 Gaps

Part 2 does not define precise relations for every schema, observations, commands, feasibility, status, result, or event endpoint. Glaux should create one documented, versioned HTTPS relation namespace under project control, for example conceptually `https://dgiwg-p507.github.io/glaux/rel/{name}`, with at least:

- `schema`, `observations`, `commands`, `feasibility`, `results`, `events`, and any direction-specific relation not adequately expressed by registered `status` or `describedby`;
- stable definitions, source/target applicability, cardinality, direction, and deprecation rules;
- capability/OpenAPI/documentation registration and strict golden tests; and
- no claim that the relation is an approved OGC relation.

The exact URI base and final list must be frozen with the public-origin/documentation and API-version topics before implementation. Reusing bare example words or overloading `alternate` is rejected.

### 9.6 Content Negotiation, Versioning, and Lifecycle

`self` identifies the exact negotiated view; `canonical` points to the preferred resource; `alternate` changes representation, not identity; domain relations point to the correct related item or set and supply the target media type when known. Link generation follows the selected API-version/profile registry. Retired resources retain authorized canonical identity and relationship history. Aliases may redirect safe retrievals under IDR-SRV-016 but are never emitted as the relationship target when the canonical target is known.

Links to tombstoned, inaccessible, or stale targets are policy-filtered and resolution-aware. The server must not manufacture a `404`-prone local link merely because an external or concealed relationship exists.

---

## 10. Traversal and Reverse-Link Findings

### 10.1 Traversal Principle

Glaux should be link-followable without embedding unbounded graphs. Item responses include bounded direct association links; set-valued and potentially large reverse relationships point to paged nested/query endpoints. Recursive closure is explicit through applicable query parameters rather than recursive embedding.

### 10.2 Required Consistency Invariants

- A System's DataStream/ControlStream selection equals the authorized inverse of each stream's required System fact.
- A nested Observation/Command/status/result/event is the same canonical identity and version as its canonical item under equivalent authorization.
- A System's deployments equal the inverse of Deployment→System participation under the selected time scope.
- Procedure implementing-System results equal the inverse of System→Procedure implementation facts.
- parent and child hierarchy views come from one fact and one direct-versus-transitive rule.
- direct association filters, nested routes, and link targets return the same authorized set for the same scope.
- collection and recursive selection deduplicate by canonical identity without erasing distinct qualified relationship facts.

### 10.3 Direct, Indirect, and Closure Queries

The query layer must distinguish:

1. direct dereference of a local or external target;
2. direct relationship selection by kind/source/target;
3. inverse selection;
4. hierarchical closure with explicit direct/transitive semantics and bounded depth;
5. multi-hop derived traversal such as System→DataStream→Observation;
6. temporal/as-of relationship selection; and
7. semantic selection through Property/SWE bindings.

Only routes and filters established by accepted API/query work are publicly promised. An internal graph service can support more composition than the public API exposes. Unknown or unsupported relationship filters must not silently return an unfiltered success.

### 10.4 Expensive and External Reverse Relationships

Reverse links are required where CSAPI defines them or where Glaux explicitly advertises a project relation. Exhaustive global reverse links are not implied for external resources because Glaux cannot know all remote referrers. Local reverse counts and links are emitted only after policy filtering. Expensive reverse queries use paged endpoints and capability metadata; they are not embedded arrays.

### 10.5 Client Profiles

| Client | Traversal posture |
|---|---|
| CSAPI/OGC hypermedia client | Follow typed links and canonical targets; no need to guess routes |
| Generated OpenAPI client | Construct documented paths while receiving the same relation semantics |
| CSAPI Explorer | Use exact relation names/types and canonical/nested equality; tolerate controlled external variants in an adapter |
| Generic Features client | Receive valid collection/GeoJSON spine; may ignore CSAPI-specific relations without corrupting resources |
| Mobile/tactical-edge client | Prefer shallow bounded links, paged sets, cached descriptors, and explicit freshness/completeness metadata |

---

## 11. SensorML and SWE Common Relationship Implications

### 11.1 SensorML Mapping Boundary

SensorML describes process definitions/instances, physical attachment, deployment, components, connections, interfaces, inputs, outputs, parameters, positions, contacts, capabilities, characteristics, lineage, and constraints. Glaux must preserve those structures losslessly when the corresponding representation classes are selected, but it must map only validated CSAPI resource-level relations into the canonical graph.

| SensorML construct | Canonical treatment |
|---|---|
| `typeOf` | maps to System `systemKind`/Procedure where used by CSAPI; otherwise a resolvable Process reference |
| `attachedTo` | maps to permanent parent System when the CSAPI representation mapping establishes that context |
| Deployment `deployedSystems` / platform | maps to REL-003/004 with names/roles retained as qualifiers where applicable |
| `components` | document-local aggregate-process composition unless component is explicitly registered as a CSAPI System |
| `connections` | document-local directed signal/path links; validate source/destination paths, do not expose as resource hierarchy |
| inputs/outputs/parameters | schema/interface bindings to SWE components, not automatic DataStream/ControlStream resources |
| contacts/documents/capabilities/characteristics | owned descriptive structures or external links; separate from graph relations unless explicitly profiled |

One canonical entity and relationship set must project equivalently into GeoJSON and SensorML. Format-specific omission is allowed only where the encoding does not carry that concept; it cannot select a different population or silently drop a writable association.

### 11.2 SWE Common Boundary

SWE Common defines a hierarchical data-component tree and uses properties such as `definition`, `uom`, `referenceFrame`, `axisID`, fields, coordinates, elements, constraints, nil values, quality, and encodings. These references provide semantic and structural context for Datastream observation schemas, ControlStream command/result/feasibility schemas, SensorML inputs/outputs/parameters, and values.

Glaux must preserve:

- component identity within a schema by stable path/name context;
- semantic definition URI/CURIE separately from a local Property ResourceId;
- unit and reference-frame references separately from observed/controlled Property identity;
- tree ordering where it affects encoded data order;
- schema revision identity and the stream binding that interprets historical data; and
- local references/connections only within the owning document/schema scope.

IDR-SRV-022 through 024 must decide when a semantic URI resolves to a managed Property, how equivalent terms are reconciled, and how schema-component paths are exposed. IDR-SRV-017 establishes only that these are typed references rather than generic Web links or unqualified strings.

---

## 12. External Reference, Federation, DDIL, and Stale-Link Findings

### 12.1 External Reference Contract

An external reference retains the absolute URI, expected family/resource type where known, media type, optional UID hint, source/issuer, assertion provenance, policy markings/reference, last validation information, and resolution state. Public encoding maps only allowed fields; richer state remains internal or appears through an explicitly documented extension.

An external target is non-owning. Linking to another OGC/NATO service or dataset does not mean Glaux implements, validates, accredits, or controls that service or target standard. This preserves the IDR-SRV-005 boundary.

### 12.2 Resolution States

| State | Meaning | Response implication |
|---|---|---|
| Local active | Authoritative target exists and is visible | emit canonical local link |
| Local retired/archived | Identity retained, ordinary active selection differs | emit only under lifecycle/policy rules |
| Local tombstoned | Payload unavailable but protected identity remains | safe metadata/link only if authorized; otherwise conceal |
| External unresolved | URI/assertion stored; never successfully checked or checking not attempted | relationship remains; do not represent as local null |
| External fresh | Last check satisfied configured freshness | link may be emitted if policy allows |
| External stale | Last validation is older than policy threshold | link plus freshness indication only through supported profile; never silently claim current |
| External unavailable | Recent resolution failed | retain fact and failure metadata; do not delete automatically |
| Inaccessible | Caller may know relationship but cannot retrieve target under applicable policy | exact disclosure controlled by 039/040 |
| Concealed | Existence itself cannot be revealed | omit/deny without counts, timing, or reverse leakage |

### 12.3 Request-Time Link Safety

Glaux should not crawl arbitrary external links synchronously while serving ordinary resource reads. External validation occurs through governed background/import workflows with scheme/host policy, address resolution controls, redirect limits, size/time limits, media validation, audit, and credential isolation. This is necessary to avoid SSRF, availability coupling, and classified-origin leakage.

### 12.4 DDIL and Reconciliation

Disconnected nodes operate from a policy-authorized local graph snapshot with explicit as-of, freshness, and completeness context. UUIDv7 supports distributed identity minting but does not solve relationship authority or ordering. On synchronization:

- relationship facts carry source/provenance and idempotency identity;
- conflicting exclusive parents or cardinalities enter deterministic reconciliation rather than last-write-wins by UUID time;
- tombstones and retired relationship facts propagate so edges do not resurrect;
- external references remain source-qualified;
- derived closures and reverse indexes are rebuilt from reconciled facts; and
- partial replication is never advertised as a globally complete reverse graph.

IDR-SRV-042/043 select clocks, version vectors/conflict rules, replication scope, and operational protocols.

---

## 13. Security, Policy, and Releasability Implications

### 13.1 Edge-Level Authorization

Authorization is evaluated over the source, relationship kind, target, direction, requested operation, time, tenant/security domain, and releasability context. Permission to read a source is insufficient to reveal:

- the existence or identifier of a target;
- the fact that two otherwise visible resources are related;
- relationship role, validity, provenance, or prior history;
- reverse counts, empty/nonempty distinction, recursive depth, or hidden descendants;
- external origin, query string, title, UID hint, or media metadata; or
- alias, replacement, retirement, tombstone, cache, or synchronization state.

### 13.2 Response Shaping

Optional relationship links and members are omitted after policy evaluation. Paged counts and next links are computed over the authorized selection, not filtered after pagination. Recursive traversal prunes unauthorized branches without leaking their depth or count. Caches vary on every security-relevant context and must not reuse a more privileged graph projection.

Where a conceptual or schema-required relationship cannot be exposed, silently substituting null or an empty array can be false. The final policy profile must choose among concealing the whole source, returning an authorized error, or using an explicitly defined redacted representation that remains schema/conformance-correct. IDR-SRV-039/040 own that choice.

### 13.3 Mutation and Audit

Creating, deleting, reparenting, time-qualifying, resolving, or traversing sensitive edges is separately authorized and audited. Command/control, deployment, external gateway, cross-domain, and hierarchy mutations receive elevated object-level checks. A user able to create a source resource cannot assert arbitrary relationships to an existing target or claim another publisher's UID/source authority.

### 13.4 Inference Risks

High-risk signals include hidden child counts, changing page totals, error/timing differences, stable external origins, deployment membership, command targets, status/result existence, event frequency, and tombstone/replacement links. Tests must exercise pairwise and multi-hop inference, not only direct 403/404 responses.

---

## 14. Persistence, Validation, Fixture, and Test Implications

### 14.1 Persistence Requirements Without Selecting a Database

The later persistence architecture must support:

- typed local/external endpoint references and family discrimination;
- independently identified relationship facts plus aggregate-local intrinsic references;
- uniqueness/cardinality constraints scoped by active validity and authority;
- hierarchy adjacency and efficient direct/transitive traversal;
- append-oriented workflow children and ordered history;
- relationship revisions, retirement/tombstones, provenance, and audit/outbox coupling;
- reverse indexes and derived/materialized views that can be rebuilt;
- collection memberships distinct from resources;
- external resolution/freshness cache state distinct from relationship truth; and
- tenant/policy partitioning without changing public ResourceId semantics.

A relational model with typed joins, foreign keys, recursive queries/closure support, and document storage for representation-local SensorML/SWE content is plausible, but IDR-SRV-025 owns that comparison. A graph database is not selected merely because the domain is a graph.

### 14.2 Transaction Boundaries

A successful relationship mutation atomically commits the authoritative fact or aggregate change, affected resource revisions, uniqueness/cardinality checks, derived-index invalidation/materialization intent, audit, and outbox intent. Cascades, reparenting, and workflow child creation cannot acknowledge success while dropping the relationship. External link validation is normally outside that transaction and updates separate resolution state.

### 14.3 Validation Layers

| Layer | Minimum validation |
|---|---|
| Decode/schema | allowed member/link/ID shape for selected encoding and version |
| Identity | valid typed ResourceId; canonical target; no alias/UID substitution as authority |
| Type | source/target family allow-list and local/external capability |
| Cardinality | exactly-one, optional, set, and variant constraints over active facts |
| Graph | no prohibited self/cycle; parent uniqueness; bounded recursion; duplicate fact rules |
| Temporal | well-formed validity seam; later interval/overlap rules from 018 |
| Lifecycle | no unauthorized link to purged/replaced/tombstoned target; dependency/cascade checks |
| Semantic | stream/schema/property/target compatibility; SensorML/SWE local-path validity |
| Authorization | assert/traverse/reverse/disclose checks before existence leakage |
| Projection | forward/reverse/nested/filter/collection/link and cross-encoding equivalence |
| Persistence | commit/restart/rebuild/sync retains the same fact and projections |

### 14.4 Required Fixture Families

1. Three-level System and Deployment hierarchies, direct/transitive queries, reparent, self-link, cycle, duplicate descendant, and hidden middle node.
2. Permanent subsystem plus time-qualified deployment aggregation for current, instant, interval, overlap, and completed deployment cases.
3. System/Procedure kind versus implements relations, many-to-many inverse lookup, and wrong-family target.
4. Sampling Feature required parent/sampled target, local/external sampled feature, sample chains, cycle, and direct-property/generic-link conflict fixtures.
5. DataStream/ControlStream common context, record-specific override, local/external FOI, nested/filter/inverse equality, and empty relationship sets.
6. Observation/Command/Feasibility/status/result ownership, canonical/nested equality, orphan attempts, ordered histories, and every CommandResult variant/invalid combination.
7. Multiple custom collection memberships, membership removal without delete, canonical update visibility, and policy-filtered counts.
8. GeoJSON/SensorML/SWE/JSON round trips preserving identity and relationship semantics without requiring identical syntax.
9. Bare/prefixed and case-variant `ogc-rel` input fixtures, exact deterministic output, Part 2 project relation links, proxy/public-origin construction, and alternate/canonical misuse negatives.
10. Active, retired, archived, tombstoned, replaced, unknown, inaccessible, concealed, external unresolved/fresh/stale/unavailable targets.
11. DDIL concurrent creation, exclusive-parent conflict, relationship tombstone propagation, no resurrection, partial-replica completeness, and closure rebuild.
12. Create/read/traverse/reverse/restart and failed-transaction cases inspired by OSH, CS-Go, pygeoapi, SECD, CSAPI Explorer, and OS4CSAPI evidence.

### 14.5 Test Lanes and Release Gates

| Lane | Assertions |
|---|---|
| Unit/property | kind/family/cardinality tables; relation comparison; cycles; dedup; projection determinism |
| Codec/golden | exact JSON/GeoJSON/SensorML/SWE placement, links, IDs, media, and round-trip semantics |
| Repository/integration | transactions, uniqueness, restart, closure/reverse rebuild, cascade/retention seam |
| API/metamorphic | nested = direct filter = inverse; canonical identity; recursive/direct/time composition; unsupported-filter rejection |
| Security | edge-level disclosure matrix, counts/paging/cache/timing, multi-hop inference, mutation authority |
| Conformance | normative requirements and ATS kept separate from Glaux extension tests and known-defect adapters |
| Interoperability | CSAPI Explorer, generated client, generic Features client, mobile/DDIL snapshot, external-link behavior |
| Fault/recovery | mid-transaction failure, stale external dependency, cache corruption, sync conflict, outbox replay |

A 201/204 response is not evidence of relationship correctness. Release requires read-back, forward and reverse traversal, canonical/nested equality, restart persistence, and projection equality where applicable.

---

## 15. Downstream Topic Handoff Matrix

| Topic | Controlling handoff from IDR-SRV-017 |
|---|---|
| IDR-SRV-018 | Define relationship validity/evaluation time, interval overlap, current/as-of semantics, deployment participation, reparenting/configuration history, and validity-versus-freshness |
| IDR-SRV-019 | Define assertion source, provenance, lineage, trust, authority, correction, external resolution evidence, and reconciliation metadata for relationship facts |
| IDR-SRV-020 | Define status/availability/event vocabulary and effective-state derivation without turning statuses or messages into relationship identity |
| IDR-SRV-021 | Map the canonical graph to SensorML `typeOf`, `attachedTo`, deployment, components, connections, documents, and links; preserve representation-local scope |
| IDR-SRV-022 | Model SWE schema-component identity, paths, ordering, definitions, units, frames, constraints, and stream/schema bindings |
| IDR-SRV-023 | Validate cross-encoding relationship equivalence, allowed direct-link/ID placement, project relations, and historical schema binding |
| IDR-SRV-024 | Decide Property/SWE semantic resolution, observed/controlled/property-role relationships, equivalence, units, and vocabulary governance |
| IDR-SRV-025 | Compare persistence options against typed facts, intrinsic FKs, many-to-many/temporal edges, closures, reverse indexes, external refs, tenancy, and rebuildability |
| IDR-SRV-026/027 | Implement relationship-aware spatial and time-series query paths, deployment/FOI intersections, partition/index locality, and parent-child integrity |
| IDR-SRV-028 | Store rich SensorML/SWE documents and external reference/cache metadata without creating representation-specific truth |
| IDR-SRV-029 | Atomically commit fact/aggregate mutations, revisions, audit, materialization invalidation, and outbox intent |
| IDR-SRV-030 | Select cascade, reject, reparent, retire, archive, tombstone, purge, and relationship-history retention policy by dependency class |
| IDR-SRV-031/032 | Define registration and write forms for typed relationship assertions, cardinality conflicts, nested creation, membership, and read-after-write guarantees |
| IDR-SRV-033/034 | Define stream/schema/context update and observation/status relationship semantics, including override/inheritance and idempotency |
| IDR-SRV-035 | Consume canonical affected-resource/relationship inputs; keep draft Part 3 topic, `parentId`, event, and message identities in adapters |
| IDR-SRV-036–038 | Define command/feasibility owner, target, status/result authority, legal transitions, cancellation, output references, safety, and audit retention |
| IDR-SRV-039/039A | Threat-model graph enumeration, SSRF, relationship mutation, multi-hop inference, timing/count/cache leaks, and tasking target disclosure |
| IDR-SRV-040 | Define edge-level classification/releasability, required-association redaction policy, cross-boundary traversal, reverse completeness, and external-origin handling |
| IDR-SRV-041–043 | Define observability, federation, DDIL replication, conflict resolution, partial-graph completeness, tombstone propagation, and no-resurrection rules |
| IDR-SRV-044–049 | Configure trusted public origin, route/relation registry, deployment identity, external validation workers, cache isolation, and operational health |
| IDR-SRV-050/051 | Trace each normative and project relationship invariant separately to conformance and regression tests |
| IDR-SRV-052/053 | Build unit/property/integration fixtures for every matrix row, state, direction, encoding, failure, and known implementation defect |
| IDR-SRV-054–056 | Verify performance, security, and external-client traversal over direct/inverse/recursive/temporal/policy-filtered relationships |
| IDR-SRV-057 | Synthesize the accepted relationship contract without promoting open temporal, persistence, policy, or Part 3 choices |

### 15.1 Immediate Controlled Transition

If accepted, this report authorizes IDR-SRV-018 as the next single-topic iteration. It does not authorize implementation, database selection, relation-namespace publication, or Part 3 adoption. Until acceptance, downstream work may inspect this report but must not treat its project recommendations as settled baseline.

---

## 16. Recommendations

| ID | Recommendation | Priority | Basis |
|---|---|---|---|
| R-017-01 | Adopt one typed canonical relationship graph whose facts project all links, nested views, reverse queries, filters, memberships, and encodings. | Critical | N/P |
| R-017-02 | Store only one writable canonical direction; derive inverses and closures, permitting rebuildable transactional materialization. | Critical | P |
| R-017-03 | Use stable `RelationshipKind` and typed endpoint references independent from wire names, URL hierarchy, UID, alias, or database keys. | Critical | N/P |
| R-017-04 | Assign UUIDv7 `RelationshipId` to independently changing, temporal, attributed, external, audited, or many-to-many facts; allow aggregate identity for intrinsic owner edges. | High | IDR-016/P |
| R-017-05 | Enforce the resource-family, cardinality, dependency, uniqueness, and cycle rules in §7 before commit. | Critical | N/P |
| R-017-06 | Keep permanent subsystem composition distinct from time-qualified Deployment aggregation. | Critical | N |
| R-017-07 | Keep collection membership, nested selection, canonical identity, lifecycle ownership, semantic reference, and external relationship as separate concepts. | Critical | N/P |
| R-017-08 | Emit exact Part 1 Table 3 `ogc-rel:` spellings, compare extension relation URIs case-insensitively, and confine bare values to a named compatibility adapter. | High | N/X/IDR-010 |
| R-017-09 | Define a versioned HTTPS Glaux relation namespace for Part 2 navigation gaps; advertise and test it as a project extension, never an OGC claim. | High | P |
| R-017-10 | Preserve encoding-specific direct links, local IDs, embedded objects, and SensorML/SWE references through explicit codecs over the same graph. | Critical | N/P |
| R-017-11 | Use bounded links and paged query endpoints for large/reverse relationships; never embed unbounded recursive graphs. | High | P |
| R-017-12 | Represent missing, retired, tombstoned, external unresolved/fresh/stale/unavailable, inaccessible, and concealed target states distinctly inside the domain. | Critical | IDR-016/P |
| R-017-13 | Apply edge-level authorization before link, count, page, reverse, cache, error, or timing disclosure; hand required-association redaction to 039/040. | Critical | A/P |
| R-017-14 | Do external-link resolution asynchronously through SSRF-safe governed workers; do not couple ordinary reads to remote availability. | Critical | P |
| R-017-15 | Make relationship mutations transactional with revisions, constraints, audit, derived-index invalidation, and outbox intent. | Critical | P |
| R-017-16 | Gate releases on create/read/traverse/reverse/restart, nested/filter equality, cross-encoding equivalence, security, and DDIL reconciliation tests. | High | I/P |
| R-017-17 | Keep draft Part 3 topic/message fields outside relationship identity; supply stable resource and edge facts to the later adapter decision. | High | D/P |

### 16.1 Decision Analysis

| Option | Benefits | Costs/risks | Decision |
|---|---|---|---|
| Serialize links directly from stored representation documents | Initially simple | Format drift, dropped associations, broken reverse queries, duplicate truth | Reject |
| Make every relationship a public first-class resource | Uniform IDs/history | Nonstandard API surface, verbosity, needless identity for intrinsic edges | Reject as universal rule; retain internal first-class facts where needed |
| Store both directions as writable rows | Fast direct lookup | Drift, conflict resolution, transaction complexity | Reject |
| Store smallest authoritative direction and derive/materialize views | Integrity, rebuildability, encoding neutrality | Requires registry and disciplined projection | Adopt |
| Use generic `alternate`/`related` for all domain links | Easy for generic tooling | Loses semantics and violates accepted relation use | Reject |
| Use exact CSAPI relations plus registered and named project relations | Precise, traceable, interoperable | Extension governance required | Adopt |
| Embed full graph | Fewer client calls | Unbounded payloads, cycles, staleness, policy leakage | Reject |
| Bounded direct links plus paged relationship endpoints | Predictable and secure traversal | More requests | Adopt |

### 16.2 Implementation Shape and Relative Estimate

These estimates are planning ranges, not commitments and exclude database/product selection:

| Work item | Complexity | Relative effort | Assumptions |
|---|---|---|---|
| Relationship kind/constraint registry and typed references | High | 2–3 implementation weeks | Rust domain foundation exists |
| Projection/link/nested/reverse service | High | 3–5 weeks | Route and capability registries available |
| Hierarchy/closure and temporal seam | High | 2–4 weeks | Final interval semantics deferred to 018 |
| External reference/resolution-state seam | Medium-high | 2–3 weeks | Background validation and security details deferred |
| Codec integration across GeoJSON/SensorML/Part 2 JSON/SWE | High | 4–7 weeks | Can proceed incrementally by resource family |
| Relationship validation and transaction integration | High | 3–5 weeks | Persistence/transaction architecture selected later |
| Complete fixtures, security, interoperability, and fault tests | High | 4–7 weeks | Parallel with implementation; includes external clients |

The most effective implementation order is one vertical relationship slice—System hierarchy plus DataStream/Observation ownership—through domain, persistence seam, API projections, codecs, policy hook, and tests, followed by Deployment, Sampling Feature/Procedure, control/tasking, collections, and federation.

---

## 17. Risks, Constraints, and Open Questions

### 17.1 Risks and Controls

| Risk | Consequence | Control / owner |
|---|---|---|
| Treat URL nesting as ownership | Wrong cascade and identity behavior | §6 classes; 030/032 |
| Writable forward and reverse edges | Silent graph drift | one authoritative direction; 025/029 |
| Conflating UID/alias with target identity | Cross-source link corruption | typed ResourceId/external ref; IDR-016/019 |
| Overloading `alternate` or bare relation words | Nonportable navigation | exact registry and project namespace; 010/014/017 |
| Representation-specific truth | Different populations/relationships by format | one graph, round-trip gates; 021–023 |
| Over-embedding recursive/reverse graph | payload growth, cycles, stale/policy leakage | bounded links and paged queries |
| External dereference on read | SSRF and availability coupling | governed asynchronous resolver; 039/043 |
| Post-filter counts/pages/cache | existence and classification leakage | policy-first query projection; 039/040 |
| Last-write-wins DDIL hierarchy | invalid parents and resurrection | source-aware reconciliation/tombstones; 042/043 |
| RDF inference expands API facts | invented endpoints or false semantics | explicit relationship registry only |
| Generic JSON storage bypasses constraints | wrong family/cardinality/target drift | typed validation and repository constraints |
| Draft Part 3 fields leak into core | migration coupling to unstable draft | adapter boundary; 014H/035 |

### 17.2 Open Questions Assigned, Not Blocking Acceptance

| Question | Owner |
|---|---|
| Exact interval model, overlap, current/as-of evaluation, and reparent history | IDR-SRV-018 |
| Provenance, assertion authority, trust, and relationship reconciliation vocabulary | IDR-SRV-019 |
| Effective status/event relationships and availability derivation | IDR-SRV-020 |
| Exact SensorML/SWE mapping and semantic Property resolution | IDR-SRV-021–024 |
| Database tables, indexes, closures, partitions, cache/materialization technology | IDR-SRV-025–029 |
| Retention/cascade/reparent/delete/tombstone periods and legal erasure | IDR-SRV-030 |
| Client-write form and update conflict/error contract | IDR-SRV-031–034 |
| Final Glaux relation namespace URI and published vocabulary lifecycle | IDR-SRV-014/046/047 before implementation |
| Required-association policy response when target cannot be disclosed | IDR-SRV-039/040 |
| Federation/DDIL completeness, sync, conflict, and remote-link protocol | IDR-SRV-042/043 |
| Whether later standards maintenance resolves Table 3 and Part 2 relation gaps | upstream register / IDR-SRV-057 |

### 17.3 Constraints Fixed by This Report

- No relationship derives public identity from a collection/nested URL, UID coincidence, link title, or database key.
- No reverse view is independently writable.
- No non-owning/external association cascades deletion into its target.
- No hidden relationship may leak through link, count, page, cache, error, or timing behavior.
- No SensorML/SWE local graph construct becomes a CSAPI resource edge without explicit mapping.
- No draft Part 3 message/topic field becomes canonical relationship identity.

---

## 18. Validation Against Plan Success Criteria

### 18.1 Success-Criteria Validation

| Topic-plan criterion | Status | Evidence |
|---|---|---|
| Relationship classes identified with source anchors | Met | §§5–7 |
| Source and target families mapped | Met | §7 |
| Normative, inherited, derived, support, implementation, optional, unresolved distinguished | Met | §§3–6; labels throughout |
| Direction, cardinality, dependency, time, queryability documented | Met | §§7–8, 10 |
| Link representation and traversal guidance | Met | §§9–10 |
| SensorML, SWE Common, Web linking, OGC API implications | Met | §§5, 9, 11 |
| Persistence, validation, security, conformance, fixtures, interoperability | Met | §§12–14 |
| Implementation/community lessons incorporated non-normatively | Met | §3.3; §14.4–14.5 |
| Recommendations decision-usable and server-bounded | Met | §§16–17 |
| Downstream handoffs explicit | Met | §15 |
| References explicit and reproducible | Met | §19 |

### 18.2 Required Content Validation

| Required report content | Location | Status |
|---|---|---|
| 1. Executive summary | §1 | Complete |
| 2. Scope and plan alignment | §2 | Complete |
| 3. Evidence/authority | §3 | Complete |
| 4. Extraction methodology | §4 | Complete |
| 5. Standards inventory | §5 | Complete |
| 6. Canonical classes | §6 | Complete |
| 7. Resource-family matrix | §7 | Complete |
| 8. Direction/cardinality/lifecycle/time | §8 | Complete |
| 9. Link relation/representation | §9 | Complete |
| 10. Traversal/reverse | §10 | Complete |
| 11. SensorML/SWE | §11 | Complete |
| 12. External/federation/DDIL/stale | §12 | Complete |
| 13. Security/policy | §13 | Complete |
| 14. Persistence/validation/fixture/test | §14 | Complete |
| 15. Handoffs | §15 | Complete |
| 16. Recommendations | §16 | Complete |
| 17. Risks/open questions | §17 | Complete |
| 18. Validation | §18 | Complete |
| 19. References | §19 | Complete |

### 18.3 Matrix-Field Validation

The combined tables in §§6–7 include every required field: relationship ID; source/target; name/class; source anchor and authority; direction; cardinality; lifecycle dependency; temporal validity; representation/link relation; queryability; security; persistence; validation; tests; handoff; and notes. Detail that would make the 18-column table unreadable is expanded in §§8–14 and referenced by stable REL identifiers.

### 18.4 Research Status and Review Gate

- [x] Phase 1 source/framework complete
- [x] Phase 2 standards extraction complete
- [x] Phase 3 canonical model complete
- [x] Phase 4 link/traversal analysis complete
- [x] Phase 5 downstream implications complete
- [x] Phase 6 synthesis complete
- [x] Deliverable draft complete
- [x] Deliverable self-reviewed against plan
- [ ] Deliverable accepted by Glaux Project Lead

The report is complete and in review. No next topic or implementation work begins until the project lead accepts it.

---

## 19. References

### 19.1 Controlling Standards and Registries

- [OGC API - Connected Systems - Part 1: Feature Resources, OGC 23-001, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2: Dynamic Data, OGC 23-002, Version 1.0](https://docs.ogc.org/is/23-002/23-002.html)
- [Official CSAPI `v1.0.0` source at commit `8e03b236...`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)
- [OGC API - Features - Part 1: Core corrigendum, OGC 17-069r4](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [OGC SensorML Encoding Standard 3.0, OGC 23-000](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC SWE Common Data Model Encoding Standard 3.0, OGC 24-014](https://docs.ogc.org/is/24-014/24-014.html)
- [W3C/OGC Semantic Sensor Network Ontology](https://www.w3.org/TR/vocab-ssn/)
- [RFC 8288, Web Linking](https://www.rfc-editor.org/rfc/rfc8288)
- [RFC 3986, URI Generic Syntax](https://www.rfc-editor.org/rfc/rfc3986)
- [RFC 9110, HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)
- [IANA Link Relation Types](https://www.iana.org/assignments/link-relations)
- [IANA Media Types](https://www.iana.org/assignments/media-types/media-types.xhtml)

### 19.2 Project and Governance Sources

- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)
- [IDR-SRV-017 Research Plan](../IDR%20Plans/idr-srv-017-relationship-and-linkage-model.md)
- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Research Report Template](../../../../../Governance/research-report-template.md)
- Project-controlled `AC/224(JCGISR)D(2026)0005`, 27 April 2026, including STANAG 4789 and AEP-4789 Volumes I/II; SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`; available only to authorized project holders.

### 19.3 Accepted Research Inputs

- [IDR-SRV-001 server obligation baseline](idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md)
- [IDR-SRV-002 functional mapping](idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities-report.md)
- [IDR-SRV-005 related-standards boundary](idr-srv-005-related-nato-standards-boundary-review-report.md)
- [IDR-SRV-006 Part 1 baseline](idr-srv-006-csapi-part-1-requirement-baseline-report.md)
- [IDR-SRV-007 Part 2 baseline](idr-srv-007-csapi-part-2-requirement-baseline-report.md)
- [IDR-SRV-010 navigation baseline](idr-srv-010-collections-resources-links-and-navigation-behavior-report.md)
- [IDR-SRV-014A OpenSensorHub study](idr-srv-014a-osh-csapi-server-implementation-study-report.md)
- [IDR-SRV-014B connected-systems-go study](idr-srv-014b-connected-systems-go-csapi-server-implementation-study-report.md)
- [IDR-SRV-014C pygeoapi study](idr-srv-014c-pygeoapi-csapi-server-implementation-study-report.md)
- [IDR-SRV-014D SECD study](idr-srv-014d-secd-csapi-server-implementation-study-report.md)
- [IDR-SRV-014G OS4CSAPI lessons](idr-srv-014g-os4csapi-discussions-lessons-learned-study-report.md)
- [IDR-SRV-014H draft Part 3 study](idr-srv-014h-draft-csapi-part-3-publish-subscribe-and-implementation-study-report.md)
- [IDR-SRV-015 canonical resource model](idr-srv-015-canonical-glaux-server-resource-model-report.md)
- [IDR-SRV-016 identifier/URI/lifecycle strategy](idr-srv-016-identifier-uri-and-resource-lifecycle-strategy-report.md)

### 19.4 Official Maintenance and Supporting Evidence

- [CSAPI upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md), version 1.9
- [CSAPI issue #173, Part 1 relationship-link inconsistency](https://github.com/opengeospatial/ogcapi-connected-systems/issues/173)
- [CSAPI PR #176, partial example relation fixes](https://github.com/opengeospatial/ogcapi-connected-systems/pull/176)
- [OpenSensorHub issue #337, Deployment association round trip](https://github.com/opensensorhub/osh-core/issues/337)
- [OpenSensorHub issue #339, Observation sampling-feature association](https://github.com/opensensorhub/osh-core/issues/339)
- [connected-systems-go pinned evidence from IDR-SRV-014B](https://github.com/OS4CSAPI/connected-systems-go/tree/e900da88738cca92872038b703c4ad537fc0c8fd)
- [CSAPI Explorer pinned evidence from IDR-SRV-010](https://github.com/OS4CSAPI/ogc-csapi-explorer/tree/00f1c188e05738ee03390fd95f09d351e073a9c3)

---

## Report Completion Checklist

- [x] Topic ID matches overall research plan index
- [x] Topic plan is linked and aligned
- [x] Core and detailed research questions are covered
- [x] Findings have explicit reproducible evidence
- [x] Normative, draft, project, and implementation evidence are distinguished
- [x] Mutable sources identify versions, commits, or access dates
- [x] Controlled and ambiguous evidence limitations are explicit
- [x] Accepted prior findings are reconciled
- [x] Executive summary is independently readable
- [x] Recommendations are bounded and actionable
- [x] Risks and open questions are assigned
- [x] Success criteria and required content are validated
- [ ] Plan-owner acceptance and acceptance date are recorded
- [x] Next step is bounded to review/acceptance, with no later topic started

# Section 015: Canonical Glaux Server Resource Model - Research Report

**Topic ID:** IDR-SRV-015<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-015 Canonical Glaux Server Resource Model](../IDR%20Plans/idr-srv-015-canonical-glaux-server-resource-model.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core questions, all 8 detailed-question groups, all 6 methodology phases, and all 9 success criteria<br>
**Methodology Used:** Authority-ranked synthesis of the accepted IDR-SRV-001 through IDR-SRV-014H corpus; direct inspection of the approved CSAPI 1.0 source, conceptual tables, requirements, schemas, and tagged artifacts; SensorML 3.0 and SWE Common 3.0 conceptual-model review; boundary, lifecycle, relationship, temporal, representation, persistence, validation, and test analysis<br>
**Research Time:** Approximately 4.5 hours of AI-assisted execution on August 31, 2026<br>
**Official Standards Source Pin:** [`opengeospatial/ogcapi-connected-systems` `v1.0.0` at `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)<br>
**Shared Register Baseline:** OGC API - Connected Systems upstream-history register version 1.9<br>
**Document Purpose:** Establish the Rust-independent, encoding-neutral canonical Glaux Server resource/domain model that downstream identity, relationship, temporal, provenance, status/event, representation, persistence, validation, and test research can refine without rediscovering or collapsing standards concepts<br>
**Author:** OpenAI Codex<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** September 3, 2026<br>
**Date:** August 31, 2026<br>
**Last Updated:** September 3, 2026

---

## Reading Guide and Evidence Labels

| Label | Meaning |
|---|---|
| **N** | Normative or standards-derived finding from an approved, applicable source or an artifact incorporated by a requirement |
| **A** | Project-controlling AEP/STANAG adoption or operational-context finding carried from an accepted report |
| **P** | Accepted Glaux project decision or project recommendation |
| **I** | Informative implementation, test, interoperability, or community evidence |
| **D** | Official draft evidence; useful for seam design but not an approved requirement |
| **X** | Published inconsistency, ambiguity, or evidence limitation that must remain visible |

An internal **domain entity** is a concept with identity, invariants, relationships, and lifecycle behavior inside Glaux. An **API resource** is an addressable HTTP contract. A **representation** is an encoding-specific projection of an API resource or record. A **persistence record** is an implementation mechanism. These terms are not interchangeable.

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority
4. Resource-Model Extraction Method
5. Standards-Derived Resource Inventory
6. Canonical Glaux Resource Families
7. API Resource and Domain Entity Boundaries
8. Lifecycle Boundaries
9. Relationship and Linkage Implications
10. Temporal, Freshness, Status, and Event Implications
11. SensorML and SWE Common Implications
12. Implementation and Interoperability Lessons
13. Persistence, Validation, Fixture, and Test Implications
14. Downstream Topic Handoffs
15. Decision Analysis
16. Recommendations and Implementation Implications
17. Risks, Constraints, and Open Questions
18. Validation Against Plan Success Criteria
19. Next Steps and Handoff
20. References
21. Appendices

---

## 1. Executive Summary

Glaux Server needs one **encoding-neutral canonical graph**, not one data model per media type, route, collection, or implementation. The graph has four standards-facing layers:

1. **service and discovery resources** describe the API and the collections it exposes;
2. **descriptive feature/resource entities** identify Systems, Procedures, Deployments, Sampling Features, Property Definitions, and external Features of Interest;
3. **dynamic stream and record entities** identify DataStreams, Observations, ControlStreams, Commands, Feasibility requests, Command Status records, Command Results, and System Events; and
4. **supporting schema and implementation entities** preserve stream schemas, representation mappings, description revisions, provenance, collection membership, external references, audit facts, and transport-neutral domain events.

The central design decision is:

> **A canonical entity is the representation-independent semantic record. Every canonical, nested, collection, SensorML, GeoJSON, JSON, HTML, and future event/data view projects the same authorized identity, version, relationships, and lifecycle state. Routes and encodings never create a second truth.** [N,P,I]

The approved standards lead to several non-obvious boundaries:

- `Subsystem` is a System role in a hierarchy, not a second entity type; `Subdeployment` is likewise a Deployment role. [N]
- A deployed-system object is an association fact/view relating a Deployment, a System, and optionally a Procedure; it is not another copy of the System. [N,P]
- Collection membership and nested-route membership are views over canonical entities, not identity or ownership. [N]
- A Property Definition is an addressable semantic resource; a SWE Common component's `definition` is a semantic reference and must not silently become a duplicate Property entity. [N]
- A DataStream or ControlStream is a stable channel description with a format-specific schema. Its aggregate times, properties, result type, and availability can be server-derived from records and operational state. [N]
- An Observation is an activity/record and its result is a value or external reference, not a second Observation. [N]
- Status is primarily represented by `status` DataStreams and Observations, while command status is an append-oriented sequence of `CommandStatus` records. A generic mutable “status blob” is not the canonical model. [N,P]
- Command Status and Command Result have independent local IDs and nested API collections, but Version 1.0 does not make them ordinary global canonical families. They remain first-class internal workflow records scoped to a Command or Feasibility request. [N,P]
- Feasibility is a Command specialization submitted through a feasibility channel, with shared parameter-schema, status, and result concepts; it is not a disconnected workflow model. [N]
- System Event is a historical event record. It is distinct from an internal resource-lifecycle domain event and from the draft Part 3 CloudEvent envelope. [N,D,P]
- The removed/nonconforming System History material must not be claimed as a CSAPI 1.0 resource class. Glaux should nevertheless preserve versioned System-description state internally so a future profile can expose history without remodeling the entity. [X,P]

The recommended internal organization is a **typed aggregate graph plus append-oriented facts**. Stable descriptive aggregates own their immediate invariants; streams own schemas and child-record constraints; commands own status/result histories; relationships use typed references; representations are explicit codecs; and publication uses transport-neutral domain events written through an outbox-capable transaction seam. This report does not select database tables, final URI syntax, relationship link tokens, time/freshness policy, event vocabulary, or Part 3 transport binding. Those decisions remain with their assigned downstream topics.

No evidence gap blocks IDR-SRV-016 after this report is accepted. The controlled NATO package is not present in the public repository, so this topic uses its exact fixed identity, status, and relevant findings as established by accepted IDR-SRV-001 through IDR-SRV-003 rather than reasserting inaccessible text.

---

## 2. Scope and Plan Alignment

### 2.1 In Scope

- Canonical resource families and internal entities.
- Resource, subresource, collection, association, schema, state, event, and workflow boundaries.
- Stable-description versus dynamic-record distinctions.
- Lifecycle responsibility classification.
- Relationship, time, status, representation, persistence, validation, security, and test implications.
- Accepted implementation, smoke-test, interoperability, discussion, and draft Part 3 lessons.
- Explicit downstream handoffs.

### 2.2 Out of Scope

- Final local-ID, UID, URI, alias, tombstone, or deletion policy (`IDR-SRV-016`).
- Final link relations, cardinality enforcement, graph traversal, or write forms (`IDR-SRV-017`).
- Final clock, interval, validity, freshness, stale, or last-known rules (`IDR-SRV-018`).
- Final provenance, quality, lineage, trust, or releasability schema (`IDR-SRV-019`).
- Final status/readiness/event vocabulary and transition rules (`IDR-SRV-020`).
- SensorML/SWE codec design or schema-validation pipeline (`IDR-SRV-021` through `024`).
- Database, index, cache, transaction, or retention architecture (`IDR-SRV-025` through `030`).
- Registration/update API mechanics and command/feasibility state machines (`IDR-SRV-031`, `034`, `036`, `037`).
- Part 3 adoption, MQTT topics, broker behavior, or implementation (`IDR-SRV-035`).

### 2.3 Core Research Question Coverage

| ID | Core question | Status | Primary answer |
|---|---|---|---|
| CQ-1 | Canonical resource families and internal entities | Complete | §§5-7; Appendix A |
| CQ-2 | Authority and concept source | Complete | §§3, 5, 11 |
| CQ-3 | Boundaries among resources, collections, records, events, commands, and status | Complete | §§6-10 |
| CQ-4 | Lifecycle responsibilities | Complete | §8 |
| CQ-5 | Downstream implications | Complete | §§13-14, 19 |

### 2.4 Phase Completion

| Plan phase | Work performed | Output |
|---|---|---|
| 1. Collection/framework | Pinned approved source; reviewed governance, plan, goal, all accepted reports, schemas, and conceptual sources; defined evidence and boundary classes | §§3-4 |
| 2. Standards inventory | Extracted Part 1/2 families, inherited API resources, SensorML concepts, SWE components, and AEP role | §5; Appendix A |
| 3. Boundary analysis | Classified API resources, domain entities, association facts, value objects, projections, and support entities | §§6-7 |
| 4. Relationship/lifecycle/time/status | Built lifecycle and typed-edge implications without deciding later-topic policy | §§8-10 |
| 5. Implementation/persistence/validation/tests | Reconciled 014A-014H and accepted behavior findings | §§12-13 |
| 6. Synthesis | Established canonical model, decision analysis, recommendations, handoffs, and validation | §§14-21 |

---

## 3. Evidence Base and Authority

### 3.1 Primary Sources

| Source | Version/status | Authority | Stable anchor | Access | Limitation |
|---|---|---|---|---|---|
| [OGC API - Connected Systems Part 1](https://docs.ogc.org/is/23-001/23-001.html) | OGC 23-001, Version 1.0; approved publication | Normative when applicable | Clauses 6-21 and Annex A | 2026-08-31 | Modular classes; publication defects retained in IDR-006 |
| [OGC API - Connected Systems Part 2](https://docs.ogc.org/is/23-002/23-002.html) | OGC 23-002, Version 1.0; approved publication | Normative when applicable | Clauses 6-23 and Annex A | 2026-08-31 | Published model/path/schema defects retained in IDR-007 |
| [Approved source/artifact tag](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2) | `v1.0.0`; commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`; commit 2025-07-16 | Exact approved-source and artifact pin | `api/part1`, `api/part2`, `sensorml`, `swecommon` | 2026-08-31 | Artifacts support; requirement text controls conflicts |
| [OGC API - Features Part 1](https://docs.ogc.org/is/17-069r4/17-069r4.html) | OGC 17-069r4, Version 1.0 | Normative where CSAPI inherits it | Landing, API definition, conformance, collections, items, queries | 2026-08-31 | Feature language needs the explicit Part 2 resource adaptation |
| [SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html) | OGC 23-000, Version 3.0 | Normative package source | Core; AbstractProcess; Simple/Aggregate Process; Physical Component/System; Deployment; JSON | 2026-08-31 | It is a representation/concept source, not a parallel CSAPI identity service |
| [SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html) | OGC 24-014, Version 3.0 | Normative package source | Core; simple, record, choice, block, geometry components; encodings | 2026-08-31 | Component/schema semantics are not automatically top-level API resources |
| [SOSA/SSN 2017 Recommendation](https://www.w3.org/TR/vocab-ssn/) | W3C/OGC Recommendation, 19 October 2017 | Stable semantic authority where applicable | Systems, Procedures, Deployments, Samples, Observations, Results | 2026-08-31 | CSAPI 2025 intentionally diverges in places; see IDR-004 |
| `AC/224(JCGISR)D(2026)0005`, 27 April 2026 | Fixed project-controlled ratification package; SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C` | Project-controlling NATO adoption context | STANAG enclosure; AEP-4789 Volumes I-II | Via accepted IDR-001-003 | Controlled copy not in this public repository; visibly pre-promulgation |

### 3.2 Accepted Project Evidence

| Source group | Evidence used | Authority boundary |
|---|---|---|
| IDR-SRV-001-005 | NATO/AEP obligations, coherent standards package, terminology, external-integration boundary | Accepted project baseline; does not replace OGC technical requirements |
| IDR-SRV-006-008 | Complete Part 1/2 requirement, conformance, schema, and interpretation inventories | Controlling accepted extraction of approved standards |
| IDR-SRV-009-014 | Discovery, navigation, query, negotiation, error, version, and OpenAPI behavior | Accepted Glaux behavior baseline |
| IDR-SRV-014A-014D | OSH, CS-Go, pygeoapi, and SECD implementation observations | Informative implementation evidence only |
| IDR-SRV-014E-014G | Client smoke, interoperability, and community lessons | Informative/test evidence; no new standards obligations |
| IDR-SRV-014H | Draft Part 3 authority and implementation study | Accepted design input; Part 3 remains an experimental candidate |

### 3.3 Authority Rules

1. Applicable approved OGC requirements and explicitly incorporated schemas/tables control wire behavior. [N]
2. AEP-4789 Volume II controls the four-standard package/adoption context; it does not restate every technical requirement. [A]
3. Accepted IDR reports control Glaux interpretations and handoffs unless this report identifies and resolves a material conflict. [P]
4. Official examples, OpenAPI, implementations, tests, and discussions inform design and risk; they do not override requirements. [I]
5. Draft Part 3 may shape transport-neutral seams only. It creates no current endpoint, topic, message, or conformance obligation. [D]

### 3.4 Evidence Quality and Limitations

- The approved tag was independently fetched and inspected at the same commit recorded by accepted IDR-006/007/010. This is a reproducible source baseline.
- The controlled NATO PDF was not duplicated into the public repository and was not rediscovered in this workspace. Exact package identity, handling, status, and findings are carried only from the accepted, hash-verified reports.
- Classification/handling boundary: accepted IDR-001 through IDR-003 treat the package as a project-controlled local source and intentionally do not store or link it publicly. Because the source itself was unavailable in this execution environment, this report does not infer or restate any additional classification marking.
- Part 1 and Part 2 have known normative-text, model-table, schema, ATS, and OpenAPI disagreements. This report preserves the accepted interpretation registers rather than choosing silently.
- CSAPI's use of newer SOSA names does not establish full conformance to the 2017 Recommendation. Public Glaux vocabulary follows the approved CSAPI contract while semantic-version mismatches remain visible.

---

## 4. Resource-Model Extraction Method

### 4.1 Classification Axes

Every concept was classified across five independent axes:

| Axis | Values |
|---|---|
| Authority | normative, AEP context, project decision, informative implementation, draft candidate, unresolved |
| Semantic role | entity, association fact, value object, event/fact, workflow record, derived projection, service metadata |
| API role | canonical top-level resource, collection, nested resource, subordinate resource, related external resource, representation-only |
| Lifecycle | registered description, mutable configuration, append-oriented fact, derived state, workflow state, generated document |
| Responsibility | server authoritative, externally authoritative/cached, server-derived, policy-filtered, transport adapter |

### 4.2 Boundary Tests

A concept warrants a distinct internal entity when several of these apply:

- it has stable identity independent of encoding or route;
- standards assign it a canonical or independently addressable resource;
- it has its own lifecycle, authorization, provenance, or audit boundary;
- other resources refer to it from multiple contexts;
- it must survive representation changes;
- its invariants or transactions differ from its parent.

A concept should remain a value object or projection when it is defined only by its owner, has no independent lifecycle, exists solely to encode a relationship/value, or is generated from authoritative entities.

### 4.3 Canonical-Model Invariants

The synthesis applied these project invariants:

1. One logical entity has one canonical identity even when it appears through many routes, collections, or encodings. [N,P]
2. Wire spelling is confined to codecs/adapters; internal semantic names do not embed `@id`, `@link`, draft `parentId`, or media-specific nesting. [P]
3. Required relations are typed references, not arbitrary writable link arrays. [N,P]
4. Server-derived summaries and reverse edges are projections, not competing writable facts. [N,P]
5. Schema-governed payloads retain a schema identity/version and are not accepted as untyped JSON blobs. [N,P]
6. Authorization and releasability can filter a representation or graph view but must not fork identity. [A,P]
7. Resource lifecycle events, operational System Events, Observation data, and transport envelopes remain distinct. [N,D,P]

---

## 5. Standards-Derived Resource Inventory

### 5.1 Inherited Service and Collection Resources

| Resource/concept | Source | Canonical classification | Model implication |
|---|---|---|---|
| Landing page | Features/CSAPI Common | Generated API resource | Projection from capability/navigation registry; no business entity |
| API definition | Features/CSAPI Common | Generated API document | Built from the same route/schema/profile registry as runtime behavior |
| Conformance declaration | Features/CSAPI Common | Generated claim resource | Versioned set of evidence-backed conformance URIs |
| Collection aggregate/detail | Features Part 1; CSAPI adaptation | API resource plus collection definition | Collection descriptor and membership/view definition are distinct from member identity |
| Collection items | Features/CSAPI | Query/view | Returns canonical entities in an authorized membership context |
| Schema/profile/documentation target | CSAPI Parts 1-2; project | Generated or registered support resource | Schema descriptor/document identity must be explicit and reproducible |

### 5.2 Part 1 Descriptive Families

| Family | Standards concept | Canonical conclusion |
|---|---|---|
| System | Sensor, actuator, sampler, platform, process, human, simulation, group, or other connected System | First-class entity/API resource. Primary role/type does not split identity or restrict capabilities. |
| Subsystem | A System nested beneath a parent System | Same System entity with a typed, possibly time-qualified parent relationship; not a subtype table or duplicate resource. |
| Procedure | Reusable method, specification, datasheet, or methodology implemented by Systems | First-class non-spatial feature entity/API resource; representation class varies by procedure/asset kind. |
| Deployment | Purpose-, time-, and often place-specific deployment of Systems | First-class entity/API resource. Deployment location is not the FOI/Sampling Feature geometry. |
| Subdeployment | A Deployment nested within another Deployment | Same Deployment entity with typed parentage; not a second entity family. |
| Deployed System | Deployment-to-System association, optionally qualified by Procedure | Association fact/value object and nested projection, never a copied System entity. |
| Sampling Feature | Sample/proxy used in an observation or control sampling chain | First-class feature entity/API resource, linked to a parent System and sampled Feature. |
| Feature of Interest | Ultimate domain feature being observed or affected | Typed external-or-local feature reference. Glaux need not own the domain feature's lifecycle. |
| Property Definition | Addressable semantic definition derived from a base property, optional object type/statistic | First-class non-feature entity/API resource; distinct from JSON fields and SWE component descriptors. |

### 5.3 Part 2 Dynamic and Tasking Families

| Family | Standards concept | Canonical conclusion |
|---|---|---|
| DataStream | Stable channel description for one System output and its live and/or archived Observations | First-class entity/API resource that owns observation-schema binding and derived stream summaries. |
| Observation | Time-qualified observation record with parameters and inline or linked result | First-class fact/API resource scoped to a DataStream; result is a value/reference, not another Observation. |
| Status stream | DataStream with `type=status` whose Observations describe the parent System/subsystem | Ordinary DataStream specialization; current status is a temporal projection over observations. |
| ControlStream | Stable channel description for commands affecting a System or external feature | First-class entity/API resource that owns command and result schema bindings. |
| Command | Request/record with parameters, sender, issue/execution time, current-status projection, status history, and results | First-class workflow entity/API resource scoped to a ControlStream. |
| Command Status | Time-stamped status/progress report with local ID | First-class internal workflow fact and nested API resource; not a global canonical family. |
| Command Result | Inline result or reference to Observation(s), DataStream, or external dataset, with local ID | First-class internal workflow record and nested API resource; variant value is schema-governed. |
| Feasibility | Command specialization sent on the feasibility channel with shared parameters/status/result model | First-class command-like workflow entity/API resource; separate role and endpoint, shared domain core. |
| System Event | Time-qualified operational/maintenance/configuration event related to one System | First-class historical event/API resource; separate from resource mutation events. |
| Dynamic feature property | Time-varying feature property and snapshot projection | Attribute/history concern tied to a resource and time series; not a free-standing generic resource family. |
| System description history | Older System representations at different valid times | Internal description revisions/future view. Removed from the Version 1.0 conformance surface; do not claim as core. |

### 5.4 Schema and Encoding Concepts

| Concept | Source role | Canonical conclusion |
|---|---|---|
| Observation schema | Part 2 per-DataStream and per-format operation | First-class subordinate schema descriptor/document, version-bound to a DataStream |
| Command schema | Part 2 per-ControlStream and per-format operation | First-class subordinate schema descriptor/document, version-bound to a ControlStream |
| Feasibility result schema | Part 2 ControlStream feasibility behavior | Separate schema role even when shared with command parameter structure |
| SensorML description | Part 1 representation of System, Procedure, Deployment, Property | Encoding-specific projection over canonical entities; may contain richer structured metadata |
| SWE component tree | Data/parameter/result structure and semantics | Typed schema/value model embedded or referenced by streams/descriptions; not automatically an API entity |
| SWE encoding | JSON, text, or binary mapping for one component tree | Codec/configuration object separate from semantic data structure |
| Representation envelope | GeoJSON Feature/FeatureCollection, SensorML JSON, Part 2 JSON collections | Wire projection only; envelope names do not define storage/domain types |

### 5.5 AEP-4789 Context

The accepted AEP baseline adds cross-cutting requirements for persistent identity, context, provenance, validity, freshness, quality, policy, handling, degraded connectivity, staged enrichment, last-known state, and asynchronous tasking. These are **model obligations**, not proof that each concern needs a top-level API resource. The canonical entities therefore expose extension seams for metadata and policy without prematurely designing their exact schema. [A,P]

---

## 6. Canonical Glaux Resource Families

### 6.1 Family A — Service Capability and Discovery

This family contains the landing page, API definition, conformance declaration, collection catalog, collection descriptors, schema/profile documents, and documentation targets. The canonical internal source is a **Capability Registry** describing enabled requirements, routes, families, encodings, schemas, link rules, and evidence. HTTP documents are generated projections.

These resources are policy-filterable but not ordinary business aggregates. A deployment may suppress a capability from an unauthorized caller, yet runtime, OpenAPI, conformance, and links must remain mutually truthful for that caller.

### 6.2 Family B — Descriptive Connected-System Graph

First-class aggregates:

- `System`
- `Procedure`
- `Deployment`
- `SamplingFeature`
- `PropertyDefinition`

Supporting typed references/facts:

- System parentage and deployment-qualified attachment;
- Deployment parentage and deployed-system participation;
- System-to-Procedure implementation/system-kind references;
- Sampling Feature-to-System and sampled-feature/sample-chain references;
- local or external Feature-of-Interest references;
- property derivation and semantic binding.

Descriptions share a common semantic core—local identity, global/source identifier where applicable, name, description, type, validity, provenance/ownership seams, and typed links—but they do not share one undifferentiated “resource” payload. Family-specific invariants remain explicit.

### 6.3 Family C — Observation and Status Data

First-class aggregates/facts:

- `DataStream`
- `Observation`
- `StreamSchema` in the observation role

A DataStream binds a producer System, output name where present, optional Procedure/Deployment/Sampling Feature/FOI context, observed-property semantics, supported formats, validity, live/archived capability, and one schema per supported observation format. Observations bind the actual time, result, and per-observation context.

A `status` DataStream uses the same model. Status components are described by SWE Common and Property Definitions; status Observations carry time-qualified values. A current System status view is derived from the applicable authorized records plus later freshness policy.

### 6.4 Family D — Control, Feasibility, and Results

First-class aggregates/facts:

- `ControlStream`
- `Command`
- `FeasibilityRequest` as a Command specialization/role
- `CommandStatusRecord`
- `CommandResultRecord`
- `StreamSchema` in parameter/result/feasibility roles

A ControlStream binds the receiving System, input, controlled properties, targets, supported formats, asynchronous behavior, live acceptance state, and schemas. A Command is accepted by the API only after schema and policy checks; this does not prove execution, acceptance by the device, or physical effect. Current status is a projection over its ordered status records. Results remain typed variants and may refer to data outside Glaux.

### 6.5 Family E — Operational and Resource Events

- `SystemEvent` records operational facts such as calibration, configuration change, relocation, or decommissioning. The published placeholder type URIs are not a safe closed production vocabulary; final policy belongs to IDR-SRV-020/024.
- `ResourceLifecycleEvent` is an internal transport-neutral fact that a canonical resource was created, updated, or deleted. It has event identity, resource identity, resource family, operation, commit/visibility times, optional parent/collection context, and version references.
- A draft Part 3 `ResourceEvent`, `ResourceData`, or aggregate message is an adapter projection of these facts/entities. It is not stored as the canonical entity and does not define MQTT topics internally.

### 6.6 Family F — Implementation-Support Entities

The following are necessary internal concepts but are not automatically public CSAPI resource families:

| Internal concept | Purpose | Public exposure |
|---|---|---|
| Entity revision | Optimistic concurrency, history, provenance, representation reproducibility | Via headers/history/profile only when later defined |
| Collection definition/membership | Arbitrary collection views and membership lifecycle | Collection API projections |
| Typed relationship fact | Integrity, time qualification, reverse/recursive derivation | Links, nested endpoints, filters, embedded refs |
| External resource reference | Source-qualified link to externally owned Feature, schema, result, or service | Typed links/references |
| Ingestion receipt/source assertion | Separate API acceptance from source truth and downstream completion | Audit/provenance interfaces later |
| Policy/ownership/tenant context | Authorization, releasability, isolation, accountable mutation | Filtered output; security APIs later if any |
| Outbox/domain event | Atomic handoff from committed entity mutation to optional publishers | Internal; adapter to later Part 3 |
| Codec/representation mapping | Deterministic projections and loss accounting | Media negotiation and schema/docs |
| Tombstone/alias | Preserve lifecycle and reference resolution after change | Final form belongs to IDR-SRV-016 |

---

## 7. API Resource and Domain Entity Boundaries

### 7.1 Boundary Matrix

| Concept | API resource role | Internal role | Boundary decision |
|---|---|---|---|
| System/Procedure/Deployment/Sampling Feature/Property | Canonical Part 1 family | Aggregate entity | Direct semantic counterpart; representation-neutral core |
| Subsystem/Subdeployment | Nested and canonical view of ordinary member | Relationship role | Never duplicate the child entity |
| Deployed System | Embedded/nested association representation | Association fact/value object | Reference System identity plus deployment-specific qualifiers |
| Collection member | Item view | Membership fact/query result | Collection URL does not own or identify the entity |
| DataStream/ControlStream | Canonical Part 2 resource | Channel aggregate | Own schema bindings and stable channel context, not all child payload fields |
| Observation | Canonical and nested resource | Append-oriented fact | Stable local identity; result is inline value or external reference |
| Command/Feasibility | Canonical/nested workflow resource | Workflow aggregate | Shared command core; separate kind and policy path |
| Command Status/Result | Nested resource collection with local IDs | Child fact/record | First-class internally but not promoted to unsupported global family |
| System Event | Canonical/nested resource | Historical operational event | Separate from entity revision and publication event |
| Stream schema | Subordinate format-selected resource | Schema descriptor/version | Addressable and versioned; bytes/document may be generated or stored |
| SensorML/SWE document | Representation/payload schema | Codec projection/value graph | Not an independent duplicate entity unless the standard gives it identity |

### 7.2 Views Over Multiple Entities

The following API views may combine several internal facts:

- a System representation combines the System aggregate, selected current description revision, authorized direct relationships, derived reverse links, and a current/latest location projection;
- a Deployment representation combines the Deployment aggregate, participation facts, hierarchy, related stream references, and authorized Features of Interest;
- a DataStream representation combines channel configuration, schema registrations, observation-derived time/property/result summaries, and live availability;
- a Command representation combines request data, accountable sender, current-status projection, and links to status/results;
- a collection or recursive nested response combines member entities with membership, hierarchy, authorization, time, and query projections.

This composition is why storing a serialized response as the canonical record is unsafe.

### 7.3 Internal Concepts That Must Not Leak as Standards Claims

Database row IDs, table names, tenant partitions, source-adapter IDs, queue offsets, outbox sequence numbers, cache keys, hash values, lock versions, actor/session IDs, and delivery acknowledgements are implementation metadata. They may support identifiers, audit, or events but must not be mislabeled as CSAPI resource attributes or link relations.

### 7.4 Extension Boundary

Glaux may expose useful extensions such as richer typed links, history, provenance, policy metadata, or compatibility routes only when they are:

- namespaced/profiled and documented;
- optional or separately gated;
- absent from formal conformance claims unless covered;
- derived from the same canonical identity/state;
- represented in OpenAPI/schema/test evidence; and
- safe under authorization and content negotiation.

---

## 8. Lifecycle Boundaries

### 8.1 Lifecycle Classes

| Class | Resource families | Canonical lifecycle responsibility |
|---|---|---|
| Registered descriptions | System, Procedure, Deployment, Sampling Feature, Property | Register, validate, revise, relate, archive/delete under selected transaction profile; preserve revision/provenance seams |
| Hierarchy/association facts | subsystem, subdeployment, deployment participation, stream context, collection membership | Create/remove relation without changing member identity; enforce type and graph invariants |
| Channel definitions | DataStream, ControlStream | Register configuration/schema; allow compatible description updates; prevent child/schema inconsistency |
| Append-oriented facts | Observation, Command Status, Command Result, System Event | Validate, identify, append, query, retain; correction/deletion policy remains later |
| Workflow aggregates | Command, Feasibility | Accept/update/cancel only under schema, authority, and legal transitions; derive current state from history |
| Generated state | stream summaries, current status, counts, reverse links, landing/OAS/conformance | Recompute/project from authoritative facts; never accept as unconstrained client truth |
| External references/caches | Features of Interest, external result/schema/resource | Preserve source-qualified identity, provenance, freshness, and authorization; do not claim source ownership |

### 8.2 Authoritative Versus Derived Fields

Examples of authoritative or accepted inputs include System identity/description, Procedure details, Deployment purpose/time, Sampling Feature geometry/chain, Property semantics, stream configuration, Observation results, Command parameters, and externally issued status/event assertions after validation.

Examples of server-derived fields include:

- canonical URL and response navigation links;
- collection and nested memberships that follow stored relationships;
- DataStream `observedProperties`, `phenomenonTime`, `resultTime`, and `resultType` summaries;
- ControlStream issue/execution extents and live/availability projections where configured;
- Command `currentStatus`;
- current status/last-known projections;
- recursive association closures and reverse links;
- counts, paging links, API definition, and conformance declarations.

Derived values need lineage to their input set or revision even when that lineage is not public.

### 8.3 Deletion and Archival Boundary

This report does not choose hard delete, tombstone, retention, or cascade mechanics. It establishes required model facts:

- deleting a custom collection membership is not deleting the member;
- deleting a canonical resource must address every membership, required child/relation, reference, history, and event consequence coherently;
- observation/tasking histories may have policy, audit, or safety retention needs independent of parent visibility;
- external targets cannot be deleted by deleting a local reference;
- publication events must refer to the same committed lifecycle outcome, not an earlier request attempt.

### 8.4 Lifecycle State Ownership

Do not place a universal `active/inactive/deleted` enum on every entity. Different concepts have different state authorities:

- description validity and archival are resource-lifecycle concerns;
- deployment activity follows valid time and later operational policy;
- stream `live` describes data/command availability;
- System health/readiness comes through status data and events;
- Command/Feasibility states follow their own transition vocabulary;
- freshness/staleness is an evaluation of time and policy, not necessarily source-supplied state.

---

## 9. Relationship and Linkage Implications

### 9.1 Canonical Typed Graph

| Source | Relationship | Target | Nature |
|---|---|---|---|
| System | parent/subsystem | System | Hierarchical; potentially time-qualified; cycle-prohibited |
| System | system kind / implements | Procedure | Direct typed reference; many-to-many for implementation |
| System | participates in | Deployment | Many-to-many association, possibly qualified by Procedure/time |
| Deployment | parent/subdeployment | Deployment | Hierarchical; cycle-prohibited |
| Deployment | platform | System or external Feature | Polymorphic typed reference |
| Deployment | deployed systems | System | Association facts; recursive aggregate view |
| Sampling Feature | parent System | System | Required ownership/context relationship |
| Sampling Feature | sampled feature / sample of | external/local Feature or Sampling Feature | Sampling chain; directed and cycle-aware |
| Property | base property | Property or semantic URI | Derivation hierarchy; cycle-prohibited for local graph |
| DataStream | producer | System | Required parent/context |
| DataStream | Procedure/Deployment/SF/FOI | respective entity/reference | Optional stream-level common context |
| Observation | member of | DataStream | Required parent |
| Observation | Procedure/SF | respective entity | Optional record-specific override/context |
| ControlStream | receiving System | System | Required parent/context |
| ControlStream | Procedure/Deployment/SF/FOI | respective entity/reference | Optional common control context |
| Command | member of | ControlStream | Required parent |
| Command | target/Procedure | Sampling Feature/Procedure | Optional record-specific context |
| Command | status/result | child records | Ordered/typed workflow history |
| Result | output reference | Observation(s), DataStream, external dataset | Variant typed reference with optional time selection |
| System Event | concerns | System | Required parent |

### 9.2 Stored, Derived, and External Edges

- Store the smallest authoritative direction needed to preserve truth and validate mutation.
- Derive reverse links and recursive closures where possible; do not let two independently writable directions drift.
- Carry time qualifiers on association facts whose truth changes over time.
- Preserve external targets as source-qualified references with media/type/provenance context; a dangling or unauthorized target is not the same as a nonexistent relationship.
- Treat link arrays as representations generated from the typed graph. Client-write forms, relation vocabulary, cardinality, and integrity errors belong to IDR-SRV-017.

### 9.3 Collection and Nested Views

Collection membership can be explicit, rule-derived, policy-derived, or relationship-derived. A nested path expresses context, not new identity. Every nested or arbitrary-collection item must resolve to the same canonical entity and version visible under equivalent authorization.

### 9.4 Graph Safety Requirements for Later Design

Downstream work must define:

- cycle detection for System, Deployment, Sampling Feature, and Property derivation graphs;
- direct versus transitive semantics and depth limits;
- temporal membership evaluation;
- deduplication by canonical identity;
- authorization-aware traversal without relationship-existence leakage;
- local/external target resolution and broken-reference policy;
- atomic consistency among forward facts and generated reverse/nested views.

---

## 10. Temporal, Freshness, Status, and Event Implications

### 10.1 Distinct Time Dimensions

| Time | Applies to | Model meaning |
|---|---|---|
| Description valid time | System, Procedure, stream descriptions, Sampling Feature | When a description/configuration is valid |
| Deployment valid time | Deployment/participation | When systems are deployed for the purpose/context |
| Phenomenon time | Observation/result and DataStream summary | Time the estimated property value applies |
| Result time | Observation and DataStream summary | Time the result was obtained/generated |
| Issue time | Command/Feasibility and ControlStream summary | Time request was issued/received under final profile |
| Execution time | Command/status and ControlStream summary | Planned or actual effect/execution interval |
| Report time | Command Status | Time the status report was generated |
| Event time | System Event | Time the operational event occurred |
| Request/ingest time | Internal receipt | Time Glaux received an assertion/request |
| Commit/visibility time | Entity revision/domain event | Time state committed/became externally visible |
| Publication/delivery time | Optional Part 3 adapter | Time message was emitted/delivered |
| Expiration/freshness evaluation time | Derived policy | Time at which data becomes stale/invalid for a use |

These times must not be substituted for one another. Final precision, interval closure, defaulting, clock, freshness, and stale rules belong to IDR-SRV-018.

### 10.2 Current Versus Historical State

- Current System location is a projection of the latest applicable location fact, subject to time and authorization.
- Current status is a projection over status observations, events, availability assertions, and freshness policy.
- Current Command status is a projection over ordered status records; terminality and transition legality belong to IDR-SRV-036/037.
- Description history is an internal revision sequence. Exposing it as System History is not a current CSAPI 1.0 conformance claim.
- Stream aggregate extents summarize the records available in that authorized view and may change with retention or policy.

### 10.3 Status Model

Use three separate constructs:

1. **Operational measurements/state values** — `status` DataStream plus schema-governed Observations.
2. **Operational events** — System Event history for discrete facts/change explanations.
3. **Service-derived availability/freshness view** — a projection that evaluates recent records, stream liveness, validity, and policy.

Do not use Command Status for System health, do not use System Events as a time-series replacement, and do not treat SensorML capabilities/operating ranges as current readiness.

### 10.4 Event Model Boundary

| Event class | Subject | Payload purpose | Identity |
|---|---|---|---|
| System Event | System | Operational/maintenance/configuration fact | Canonical event resource ID |
| Resource lifecycle domain event | Any canonical resource | Internal create/update/delete fact | Event ID distinct from resource ID |
| Draft Part 3 Resource Event | Resource/collection URL | External notification envelope | Adapter-projected CloudEvent ID |
| Observation/data message | DataStream/records | Complete native data record(s) | Observation or batch/message identity as applicable |
| Audit event | Actor/action/decision | Accountability/security evidence | Audit-record identity |

---

## 11. SensorML and SWE Common Implications

### 11.1 SensorML Boundary

SensorML 3.0 describes physical and non-physical processes. Its core includes identity and descriptive metadata, classifiers, contacts, documentation, valid time, history, capabilities, characteristics, inputs, outputs, parameters, modes/configuration, interfaces, positions, components/connections, inheritance through `typeOf`, and deployments.

Canonical Glaux treatment:

- System and Procedure entities retain the semantic facts needed to produce valid SensorML classes without storing class-specific documents as separate truth.
- `System.systemKind`/`Procedure` and SensorML `typeOf` are related but must be mapped explicitly, not equated by field name.
- Physical equipment/human observers can map to Physical Component/System; simulation/process Systems can map to Simple/Aggregate Process according to Part 1 rules.
- Procedure has no location/position in the CSAPI resource model even when SensorML classes can normally carry one.
- SensorML Deployment content is a rich representation of the same Deployment identity, not a second deployment aggregate.
- SensorML history/events, capabilities, and characteristics do not automatically become System Event or current-status resources.

Exact class selection, loss policy, stored-versus-generated strategy, and round trips belong to IDR-SRV-021.

### 11.2 SWE Common Boundary

SWE Common defines typed component graphs:

- simple components: Boolean, Text, Category, Count, Quantity, Time and ranges;
- aggregate/choice/block components: DataRecord, Vector, DataChoice, DataArray, Matrix, and SWE DataStream;
- optional geometry components;
- semantic `definition`, label/description, reference frame/axis, units, constraints, nil values, and quality;
- JSON, text, and binary encoding descriptors separate from the component structure.

Canonical Glaux treatment:

- A SWE component tree is a typed schema/value object with stable schema identity/version, not an arbitrary map.
- Component `definition` references a semantic Property concept; it does not necessarily identify a locally hosted Property resource.
- DataStream/ControlStream schema roles use SWE components for observation results, status values, command parameters, and results.
- Values are validated against the applicable component tree and encoding; the same semantic structure may have several encodings.
- SWE's `DataStream` class is an encoded data-block construct and must not be confused with the CSAPI `DataStream` API resource.

Exact supported components, units, semantic resolution, code generation, and codecs belong to IDR-SRV-022/024.

### 11.3 Representation Equivalence Contract

For any two supported representations of one logical resource:

- canonical identity and local identity agree;
- authorization selects the same logical resource population;
- relationships have equivalent meaning even if encoded differently;
- format-applicable fields map deterministically;
- known information loss is explicit, tested, and never changes identity;
- updating through one accepted encoding updates the same entity;
- round-trip tests distinguish semantic equivalence from byte equality.

Part 1's intentional asymmetry remains: Sampling Feature has GeoJSON but no Part 1 SensorML representation; Property has SensorML but no Part 1 GeoJSON representation. An extension must not be advertised as core conformance.

---

## 12. Implementation and Interoperability Lessons

### 12.1 Patterns to Adopt

| Evidence | Useful pattern | Canonical-model use |
|---|---|---|
| OpenSensorHub | Typed domain-store/filter interfaces and broad relationship traversal | Keep handler/domain/repository boundaries and typed queries |
| Connected Systems Go | Typed domain/wire separation, focused formatters, broad route coverage | Separate entity, repository, codec, and route registries |
| pygeoapi study | Provider extensibility and explicit encoding selection | Use adapters but require one encoding-neutral truth |
| SECD | End-to-end DataStream/Observation and ControlStream/Command usability | Preserve task-to-result graph and schema discoverability |
| Part 3 study | Transport-neutral lifecycle/data/event concepts and outbox seam | Add internal domain-event boundary without binding MQTT |

### 12.2 Patterns to Avoid

- separate `smljson` and `geojson` documents acting as independent truth;
- nested routes or custom collections creating separate entity copies;
- untyped arbitrary JSON for Observation results, Command parameters, status, or results;
- conflating `id`, `uid`, canonical URL, version, relationship ID, event ID, and transport correlation;
- persisting response-generated links as client-controlled relationships;
- using collection envelopes such as `systems` or `datastreams` instead of the normative `items`/`features` contracts;
- flattening Feature type, asset type, semantic Property, and JSON field into one generic `type`/property mechanism;
- implementing only a compelling demo path while omitting Procedure, Property, Feasibility, System Event, lifecycle, validation, and authorization boundaries.

### 12.3 Client and Interoperability Findings

Accepted smoke/interoperability evidence shows that clients fail or silently return empty results when servers disagree on collection wrappers, canonical/nested identity, feature/type fields, link placement, path plurality, media populations, required semantics, or schema routing. The canonical model therefore requires:

- one field and relationship vocabulary independent of client library shape;
- complete semantic objects rather than client-specific flattened DTOs;
- explicit wire adapters for published inconsistencies;
- equivalent population/count/identity across alternate encodings;
- fixtures covering root, nested, collection, and external-reference resolution.

### 12.4 Draft Part 3 Constraint

The internal model must preserve resource family, resource identity, canonical URL, immediate structural parent, collection context, lifecycle operation, resource version, event identity, and separate request/commit/visibility/publication time. It must not embed the draft `parentId` spelling, `consys`/`csapi` tokens, MQTT topic hierarchy, QoS, retain, or implementation-specific enable flags. Those belong to a later versioned adapter/profile.

---

## 13. Persistence, Validation, Fixture, and Test Implications

### 13.1 Persistence Implications, Not Schema Design

The future persistence architecture must be capable of representing:

- typed entity identity, revisions, aliases, and tombstones;
- System and Deployment hierarchies with cycle-safe traversal;
- many-to-many and time-qualified relationship facts;
- externally owned targets without copying ownership;
- stream schema identity/version and supported format bindings;
- high-volume time-ordered Observations;
- Commands with ordered status/result histories and accountable actors;
- System Events and resource-lifecycle domain events;
- collection definitions/memberships and authorization-aware projections;
- derived summaries that can be rebuilt;
- transactionally coherent outbox entries.

This does not imply one relational schema, graph database, document store, or event store. IDR-SRV-025 through 030 own that choice.

### 13.2 Validation Layers

| Layer | Required checks |
|---|---|
| Syntax/media | JSON/SWE/GeoJSON/SensorML parse, media type, charset, basic envelope |
| Incorporated schema | Pinned official schema graph plus documented resolver overlays |
| Semantic domain | Family type, required attributes, enum/open-vocabulary policy, Procedure no-geometry, time rules |
| Relationship | Target family, cardinality, parentage, cycle, local/external validity, authorization |
| Stream payload | Observation/Command/result validates against the exact parent schema and selected format |
| Workflow | Command/Feasibility transition, status/result ownership, terminal behavior |
| Projection parity | canonical/nested/collection/encoding views share identity, version, membership, and meaning |
| Policy/security | actor authority, releasability, tenant/mission scope, information leakage |
| Event/outbox | event/resource IDs, operation, version, parent/collection context, atomic commit linkage |

### 13.3 Fixture Corpus

Minimum canonical scenarios:

1. three-level System hierarchy with permanent and deployment-qualified relationships;
2. three-level Deployment hierarchy with overlapping Systems and Procedures;
3. Systems spanning equipment, human, simulation, process, group, and platform roles;
4. Procedure datasheet/methodology examples with enforced no-location rule;
5. Sampling Feature chain to local and external Features of Interest;
6. direct and transitive Property derivation plus cycle negative;
7. DataStreams for scalar, vector, record, coverage, complex, status, live-only, archived-only, and mixed use;
8. Observations with inline and external results, per-record Procedure/Sampling Feature, and time edge cases;
9. synchronous/asynchronous ControlStreams with schema versions and self/external targets;
10. Commands covering every status, cancellation/update case, partial/final result variant, and authority denial;
11. Feasibility accepted/rejected/completed/failed with inline and absent results;
12. System Events with extension type, placeholder-URI negative, and mapping-gap cases;
13. equivalent GeoJSON/SensorML/JSON/HTML projections and intentionally unsupported family/media pairs;
14. canonical/nested/custom-collection identity equality and authorization-filtered graph views;
15. resource lifecycle/outbox fixtures plus draft Part 3 adapter golden messages without broker implementation.

### 13.4 Traceability

Each implemented family should be traceable through:

`standard clause/requirement/table/schema → Glaux profile rule → canonical entity/field/relation → codec/route/query → persistence contract → validation rule → positive/negative/conformance/interoperability test`

Model fields introduced solely for implementation must use project IDs, not counterfeit OGC requirement URIs.

---

## 14. Downstream Topic Handoffs

| Topic | Required handoff from IDR-SRV-015 |
|---|---|
| IDR-SRV-016 | Distinguish local ID, UID, canonical URL, entity revision, alias, external reference, relationship fact, event ID, and tombstone; preserve identity across views |
| IDR-SRV-017 | Finalize typed edges, cardinalities, direct/derived/external status, parentage, time qualification, link tokens, write forms, cycle/integrity policy |
| IDR-SRV-018 | Define all times in §10, interval/default/clock rules, current projections, freshness, stale, last-known, and association-time evaluation |
| IDR-SRV-019 | Add source, actor, provenance, lineage, quality, trust, releasability, handling, and derivation metadata without forking entity identity |
| IDR-SRV-020 | Finalize status-stream/current-view split, availability/readiness semantics, System Event mapping/vocabulary, resource-event boundary, and retention |
| IDR-SRV-021 | Map canonical System/Procedure/Deployment/Property entities to SensorML classes; define rich-metadata storage, generation, loss, inheritance, and round trips |
| IDR-SRV-022 | Define supported SWE component/value/encoding graph for observations, status, commands, feasibility, and results |
| IDR-SRV-023 | Vendor/resolve official schemas; layer structural, semantic, relationship, stream, and projection validation; document repairs |
| IDR-SRV-024 | Govern Property Definitions, component `definition`, units, code spaces, object/statistic semantics, and external vocabularies |
| IDR-SRV-025-030 | Select persistence/index/transaction/retention architectures capable of all §13.1 facts without storing encodings as separate truth |
| IDR-SRV-031/034 | Define mutation/ingestion and observation/status update semantics over these aggregates and append-oriented records |
| IDR-SRV-035 | Decide Part 3 adoption/profile and project canonical entities/events through a versioned transport adapter |
| IDR-SRV-036/037/038 | Define Command/Feasibility transition, update/cancel, status/result, safety, authority, and audit behavior |
| IDR-SRV-039-043 | Apply authorization, zero trust, policy, audit, DDIL, synchronization, and conflict rules to entity/relationship boundaries |
| IDR-SRV-050/051/053/056 | Build conformance/traceability/fixture/interoperability evidence around §13 |

---

## 15. Decision Analysis

| Option | Benefits | Costs/risks | Standards/compatibility impact | Decision |
|---|---|---|---|---|
| A. Store each wire representation as its own resource truth | Simple initial CRUD per media type | Drift, empty populations, broken round trips, duplicate identity | Conflicts with canonical/nested/alternate equivalence and observed interoperability | Reject |
| B. Generic document/resource model with arbitrary JSON and links | Fast route expansion | Loses typed invariants, schema ownership, safety, graph integrity, useful Rust types | Cannot reliably enforce Part 1/2 model tables or stream schemas | Reject |
| C. One giant object graph mirroring every SensorML/SWE class | Rich and lossless in theory | Excessive coupling, persistence complexity, API/domain collapse | Makes encoding model dictate core identity and lifecycle | Reject |
| D. Typed encoding-neutral aggregates, facts, value objects, projections, and codecs | Standards-aligned, testable, safe evolution, supports all encodings | More deliberate boundaries and mapping work | Best fit for CSAPI modular resources and four-standard package | **Adopt** |
| E. Event-sourced model as mandatory architecture | Natural history/outbox | Premature storage/transaction choice; query/retention complexity | Not required by standards | Defer as persistence option |

### 15.1 Adopted Canonical Boundary

Adopt Option D. The model should be expressed in Rust-independent terms first and later realized with:

- typed aggregate IDs and references;
- family-specific domain types;
- explicit value objects for time, geometry, property semantics, schemas, and results;
- append-oriented fact types;
- codec and route registries;
- repository/query interfaces independent of storage engine;
- domain services for graph, validation, projection, policy, and workflow;
- outbox-capable mutation boundaries.

This is a design baseline, not authorization to implement the server yet.

---

## 16. Recommendations and Implementation Implications

1. **R-015-01 — Adopt the typed, encoding-neutral canonical graph.** Priority: High. Every API and representation view must project one entity identity/version/relationship state.
2. **R-015-02 — Preserve family-specific aggregates.** Priority: High. Do not collapse System, Procedure, Deployment, Sampling Feature, Property, streams, records, workflows, and events into generic JSON resources.
3. **R-015-03 — Treat hierarchy roles as relationships.** Priority: High. Subsystem and Subdeployment remain ordinary entities plus typed parentage.
4. **R-015-04 — Separate association facts from entity copies.** Priority: High. Deployed-System, membership, nested-route, and reverse-link views reference canonical entities.
5. **R-015-05 — Make schemas first-class subordinate concepts.** Priority: High. Bind every Observation, Command, feasibility result, and inline result to a pinned parent schema/format.
6. **R-015-06 — Model status through typed temporal records and projections.** Priority: High. Keep System status, Command Status, stream liveness, and description validity distinct.
7. **R-015-07 — Keep four event classes separate.** Priority: High. System Event, resource lifecycle event, audit event, and transport message must not share one ambiguous type.
8. **R-015-08 — Generate wire links and envelopes.** Priority: High. Typed relationships and a navigation registry control outputs; arbitrary response links are not persistence input.
9. **R-015-09 — Preserve external ownership.** Priority: High. Features of Interest, schemas, results, procedures, and other resources may be referenced across servers without local duplication or false authority.
10. **R-015-10 — Design codecs and parity tests before endpoint proliferation.** Priority: High. GeoJSON, SensorML, Part 2 JSON/SWE, HTML, and future Part 3 adapters must share semantic golden cases.
11. **R-015-11 — Retain version/history and outbox seams without choosing storage.** Priority: High. Later persistence work must support reproducible revisions and atomic domain-event handoff.
12. **R-015-12 — Keep published conflicts in an interpretation registry.** Priority: Medium. Model/schema names, required-empty associations, System Event mappings, route plurality, history removal, and placeholder URIs require explicit overlays/tests.

### 16.1 Relative Implementation Complexity

| Work item | Complexity | Why | Owning later topics |
|---|---|---|---|
| Shared typed identity/reference/value primitives | Medium | Cross-family invariants and external refs | 016-019 |
| Part 1 descriptive aggregates and hierarchy | High | Rich representation and relationship mappings | 017, 021, 025 |
| DataStream/Observation/status model | High | Schema binding, scale, temporal summaries | 018, 022, 027, 034 |
| Control/Command/Feasibility model | Very high | Safety, policy, schemas, async workflow/results | 036-041 |
| Codec/projection layer | Very high | Multi-standard round-trip and loss policy | 021-024 |
| Graph validation/query/projection | High | Cycles, recursion, time, authorization | 011, 017, 018, 023, 026 |
| Revision/outbox/event seam | High | Atomicity, history, future Part 3 | 020, 025, 029, 035 |

No calendar estimate is defensible until the owning architecture/profile topics choose mechanisms. These relative values are planning inputs only.

---

## 17. Risks, Constraints, and Open Questions

### 17.1 Risks and Controls

| Risk | Consequence | Control/handoff |
|---|---|---|
| Generic-resource over-collapse | Lost standards semantics and unsafe tasking | Family-specific domain types and invariants |
| Encoding-driven storage | Cross-format drift and empty populations | One canonical entity plus codecs/parity tests |
| Over-modeling every nested structure as entity | Complexity and duplicated lifecycle | Apply boundary tests in §4.2 |
| Under-modeling relationship facts | Wrong recursion, temporal membership, and authorization | Typed graph in IDR-SRV-017 |
| Current-state blob overwrites history | Lost freshness, provenance, audit, and reconstruction | Append-oriented facts plus projections |
| Published schema/text mismatch | False rejection or accidental divergence | Pinned interpretation/overlay registry |
| External references treated as local | False authority and deletion hazards | Source-qualified external-reference type |
| Draft Part 3 terms leak internally | Migration lock-in and false conformance | Transport adapter boundary |
| Authorization filters fork identity | Cache/link/interoperability failures | Same entity identity, authorized projection |
| Dynamic payloads remain untyped | Unsafe ingestion/control and unusable data | Parent-schema binding and layered validation |

### 17.2 Open Questions Assigned Downstream

| Question | Why unresolved here | Owner |
|---|---|---|
| Exact canonical ID/UID/URL/version/alias/tombstone types | Dedicated identity/lifecycle policy | 016 |
| Which relationships are stored, derived, time-qualified, writable, or external? | Requires graph and API mutation design | 017 |
| How are current, stale, last-known, and snapshot values computed? | Requires temporal/freshness policy | 018/020 |
| Which provenance/quality/trust fields apply to each entity/fact? | Requires metadata authority model | 019 |
| Stable System Event vocabulary and JSON `message` mapping | Published placeholders/mapping gap | 020/021/024 |
| Store canonical rich semantic graph, SensorML documents, or both? | Representation/persistence tradeoff | 021/025/028 |
| Supported SWE component subset and code-generation strategy | Requires full component study | 022 |
| Schema version changes after child records exist | Transaction/migration policy | 023/029/034/036 |
| Whether event sourcing helps selected aggregates | Persistence architecture choice | 025/027/029 |
| Final Part 3 profile and replay/delivery contract | Draft binding incomplete | 035 |

### 17.3 Non-Blocking Evidence Limitations

- The controlled NATO source is cited through accepted reports and its fixed hash; public reproduction is intentionally unavailable.
- Exact link-relation vocabulary, several Part 2 paths, System Event fields/URIs, and “required if any” associations remain published inconsistencies. The canonical semantic model can proceed while adapters and later policies decide their external forms.
- System History is useful conceptual material but is not an approved Version 1.0 conformance class. Internal revision capability is a project requirement, not a claim that this removed route is normative.

---

## 18. Validation Against Plan Success Criteria

| Success criterion | Status | Evidence |
|---|---|---|
| Canonical families identified with anchors | Met | §§3, 5-6; Appendix A |
| API/domain/representation/persistence/support concepts distinguished | Met | §§4, 7 |
| Part 1, Part 2, Features, SensorML, SWE, and AEP mapped | Met | §§3, 5, 11 |
| Resource lifecycle boundaries identified | Met | §8 |
| Relationship, identity, time, status, event, representation, persistence, validation, and test implications documented | Met | §§9-14 |
| Implementation/community findings incorporated as non-normative evidence | Met | §12 |
| Recommendations decision-usable and server-bounded | Met | §§15-17 |
| Downstream handoffs explicit | Met | §14 and §19 |
| References explicit and reproducible | Met | §§3, 20; immutable source pin |

### 18.1 Report Completion Checklist

- [x] Topic ID and research plan match the overall index.
- [x] Every core and detailed question is answered or assigned explicitly.
- [x] Approved, AEP, project, implementation, and draft evidence are distinguished.
- [x] Mutable evidence is version/commit pinned.
- [x] Controlled-source limitations are explicit.
- [x] Known conflicts with accepted reports were checked; none was introduced.
- [x] Recommendations are actionable and bounded.
- [x] Risks/open questions and owners are recorded.
- [x] Acceptance remains pending and is not implied by report completion.

---

## 19. Next Steps and Handoff

### 19.1 Review Boundary

This report is complete and in review. It defines the canonical resource-model baseline but does not authorize implementation or any later topic.

The next two project-governance actions are:

1. Glaux Project Lead accepts or returns IDR-SRV-015.
2. Upon acceptance, Glaux Project Lead authorizes exactly one next topic: `IDR-SRV-016: Identifier, URI, and Resource Lifecycle Strategy`.

Per the established workflow, the project lead may perform both actions with **`proceed`**.

### 19.2 IDR-SRV-016 Starting Inputs

IDR-SRV-016 must begin with:

- the entity/fact/value/projection distinctions in §§5-8;
- the “one identity across views” invariant;
- distinct local ID, UID, canonical URL, entity revision, external reference, relation/event/message identity needs;
- canonical versus nested/collection identity behavior;
- lifecycle classes and deletion boundaries;
- source-qualified external references;
- Part 3 prohibition against embedding draft `parentId` or topic structure.

---

## 20. References

### 20.1 Approved Standards

- [OGC 23-001, OGC API - Connected Systems - Part 1: Feature Resources, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC 23-002, OGC API - Connected Systems - Part 2: Dynamic Data, Version 1.0](https://docs.ogc.org/is/23-002/23-002.html)
- [OGC API - Connected Systems `v1.0.0` source and artifact pin](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)
- [OGC 17-069r4, OGC API - Features - Part 1: Core](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [OGC 23-000, SensorML Encoding Standard, Version 3.0](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC 24-014, SWE Common Data Model Encoding Standard, Version 3.0](https://docs.ogc.org/is/24-014/24-014.html)
- [W3C/OGC Semantic Sensor Network Ontology, 2017 Recommendation](https://www.w3.org/TR/vocab-ssn/)
- [OGC schema repository](https://schemas.opengis.net/)

### 20.2 NATO and Project Sources

- NATO `AC/224(JCGISR)D(2026)0005`, *NATO Standard - STANAG 4789*, 27 April 2026. Project-controlled source; SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`.
  - Enclosure 1: *STANAG 4789, Sensor Integration Standard for NATO JISR Operations*.
  - Enclosure 2: *AEP-4789 Volume I, Sensor Integration Standard for NATO JISR Operations - Reference View*, Edition A, Version 1.
  - Enclosure 3: *AEP-4789 Volume II, Sensor Integration Standard for NATO JISR Operations - Core APIs and Encodings*, Edition A, Version 1.
- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)
- [IDR-SRV-015 Research Plan](../IDR%20Plans/idr-srv-015-canonical-glaux-server-resource-model.md)
- [Research Report Template](../../../../../Governance/research-report-template.md)
- Accepted IDR-SRV-001 through IDR-SRV-014H reports in this directory.
- [OGC API - Connected Systems upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md)

---

## 21. Appendices

### Appendix A. Canonical Resource-Model Matrix

| Resource/concept | Family | Authority/source | Model role | API exposure | Lifecycle | Key relationships | Time/status | Representation/SWE | Persistence/validation/test | Primary handoff |
|---|---|---|---|---|---|---|---|---|---|---|
| Landing/API/conformance | Service | Features/CSAPI Common [N] | Generated metadata | Root resources | Generate/version with capabilities | collections, schemas, docs | deployment/profile version | JSON/HTML/OpenAPI | registry parity and link walk | 014, 050 |
| Collection definition | Service | Features/CSAPI [N] | View definition | `/collections` | register/configure/generate | member family, memberships | optional extent | feature/resource descriptor | descriptor/items parity | 017, 025 |
| System | Description | Part 1 §9 [N] | Aggregate entity | `/systems/{id}` | register/revise/archive/delete | parent, Procedures, Deployments, SF, streams | valid time, latest location, derived status | GeoJSON/SensorML | hierarchy/type/representation parity | 016-021 |
| System parentage | Description | Part 1 §10 [N] | Relationship fact | nested subsystem view | attach/detach/time-qualify | parent/child Systems | optional deployment time | links/nested route | cycle/recursion/dedup | 017-018 |
| Procedure | Description | Part 1 §13 [N] | Aggregate entity | `/procedures/{id}` | register/revise/archive/delete | implementing Systems, streams | description valid time | GeoJSON/SensorML; no location | class/no-geometry/round trip | 016-017, 021 |
| Deployment | Description | Part 1 §11 [N] | Aggregate entity | `/deployments/{id}` | register/revise/archive/delete | parent, Systems, platform, FOI/SF, streams | required deployment valid time | GeoJSON/SensorML | hierarchy/participation/geometry distinction | 016-021 |
| Deployed System | Association | Part 1 mappings [N] | Association fact/value | embedded/nested view | create/remove with participation | Deployment, System, Procedure | deployment-qualified | GeoJSON link/SensorML object | no System duplication | 017-018, 021 |
| Sampling Feature | Description | Part 1 §14 [N] | Aggregate entity | `/samplingFeatures/{id}` | register/revise/archive/delete | parent System, sampled Feature, sample chain, streams | optional valid time | GeoJSON core; no Part 1 SensorML | chain/type/external-target tests | 016-018, 024 |
| Feature of Interest | External/domain | Part 1/2/SOSA [N] | External/local typed reference | related resource if hosted | externally governed/cache | SF, streams, results | domain-specific | arbitrary feature representation | source/authority/freshness | 017-019 |
| Property Definition | Semantics | Part 1 §15 [N] | Aggregate entity | `/properties/{id}` | register/revise/archive/delete | base Property, object type, statistic | no universal current state | SensorML; no Part 1 GeoJSON | URI/derivation/cycle tests | 016-017, 024 |
| DataStream | Dynamic | Part 2 §9 [N] | Channel aggregate | `/datastreams/{id}` | register/revise/retire/delete constraints | System, schema, Observations, Procedure, Deployment, SF/FOI | validity, derived phenomenon/result ranges, live | JSON description; SWE payload schemas | schema immutability/summary parity | 018, 022-027, 034 |
| Observation schema | Schema | Part 2 §9 [N] | Subordinate schema descriptor | `/datastreams/{id}/schema` | version/bind per format | DataStream, payload components | schema effective time/version | JSON/SWE/protobuf/extension | pinned resolver and payload validation | 022-023 |
| Observation | Dynamic | Part 2 §9 [N] | Append-oriented fact | `/observations/{id}` and nested | ingest/correct/retain/delete policy later | DataStream, Procedure, SF, result | phenomenon/result/ingest/commit | JSON/SWE value | parent-schema/time/identity tests | 018-019, 027, 034 |
| Status DataStream/Observation | Status | Part 2 §9 [N] | Stream specialization/facts | ordinary DS/Obs routes | ingest/retain/project | System/subsystem, Properties | time/freshness/current projection | SWE component values | stale/last-known golden cases | 018, 020, 022, 034 |
| ControlStream | Control | Part 2 §10 [N] | Channel aggregate | `/controlstreams/{id}` | register/revise/retire | System, schemas, Commands, Procedure, Deployment, targets | validity, issue/execution ranges, live | JSON description; SWE parameter schemas | authority/schema/mutation tests | 018, 022-023, 036-040 |
| Command schema | Schema | Part 2 §10 [N] | Subordinate schema descriptor | `/controlstreams/{id}/schema` | version/bind per format/role | ControlStream, parameters/results | effective version | JSON/SWE/protobuf/extension | exact parent-schema validation | 022-023, 036 |
| Command | Control | Part 2 §10 [N] | Workflow aggregate | `/commands/{id}` and nested | accept/update/cancel/complete/fail | ControlStream, target, Procedure, status/results | issue/execution/current-status | JSON/SWE parameters | state/auth/schema/audit tests | 018-020, 036, 038-041 |
| Command Status | Control | Part 2 §10 [N] | Append-oriented child fact | nested status collection | append/retain; terminal rules later | Command/Feasibility, results | report/execution time | JSON model | ordering/transition/projection tests | 018, 020, 036-037 |
| Command Result | Control | Part 2 §10 [N] | Typed child record | nested result collection | append/retain | Command, Obs, DS, external | optional selected result range | inline schema value or link | exactly-one/at-least-one variant rules | 017-019, 036-037 |
| Feasibility | Control | Part 2 §11 [N] | Command specialization | `/feasibility/{id}` plus nested | accept/analyze/cancel/complete/fail | ControlStream, shared status/results | issue/report/execution | command parameter/result schemas | sync/async and unused-status tests | 036-038 |
| System Event | Event | Part 2 §12 [N,X] | Historical event entity | `/systemEvents/{id}` and nested | append/correct/retain policy later | System | event time | SensorML Event-derived JSON mapping | placeholder URI/mapping-gap tests | 020-024 |
| Entity revision | Support | AEP/project [A,P] | Internal record | not automatically public | create per committed revision | entity, actor/source, previous revision | valid/commit/visibility | representation-neutral | optimistic concurrency/replay | 016, 019, 025, 029 |
| Resource lifecycle event/outbox | Support | accepted 014H [D,P] | Internal event fact | later Part 3 adapter only | append atomically with mutation | resource, parent, collection, revision | request/commit/visibility/publication | transport-neutral → adapter | atomicity/replay/duplicate tests | 020, 025, 029, 035 |
| External reference | Support | CSAPI distributed model/AEP [N,A,P] | Typed reference | link/embedded ref | refresh/reconcile, no target ownership | source, canonical target, media/type | retrieved/freshness later | link or semantic URI | broken/unauthorized/source-change tests | 016-019 |

### Appendix B. Detailed Question Coverage

| Detailed question group | Coverage | Status |
|---|---|---|
| Standards and concept sources | §§3, 5, 11 | Complete |
| Canonical resource families | §§5-6; Appendix A | Complete |
| API resource versus domain entity | §§4, 7 | Complete |
| Lifecycle boundaries | §8 | Complete |
| Relationship/linkage | §9 | Complete; final policy assigned |
| Temporal/status/event | §10 | Complete; final policy assigned |
| SensorML/SWE | §11 | Complete; codec strategy assigned |
| Implementation/community lessons | §12 | Complete |
| Validation/test/interoperability | §13 | Complete |

### Appendix C. Completion State

**Completion state:** Research, synthesis, report drafting, review, and plan-owner acceptance complete; IDR-SRV-016 is authorized as the next single-topic research iteration.

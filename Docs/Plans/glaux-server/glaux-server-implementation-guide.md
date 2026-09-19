# Glaux Server Implementation Guide

**Version:** 1.3<br>
**Date:** 19 September 2026<br>
**Effort:** Glaux Server<br>
**Status:** Baselined — Goal-paired capability explanations added; implementation not started<br>
**Depends On:** [Glaux Server Goal and Definition v1.8](glaux-server-goal-and-definition.md), Approved

**Revision summary:** Expands §1.1 into plain-English explanations paired to Goal §§5–7: what each capability requires, which selected technologies and mechanisms will deliver it, and where the detailed design is specified. Goal v1.8 adds direct links without changing its approved scope; Roadmap v1.18 aligns current references and the implementation handoff. Technical contracts, standards pins, tests, all 286 task definitions and the published issue preparation baselines are unchanged. This documentation update does not implement or verify server software.

## Executive Summary

Build a full-scope Rust reference implementation of OGC API - Connected Systems Parts 1 and 2, with their applicable SensorML 3.0 and SWE Common 3.0 requirements, experimental Part 3 publication, the selected experimental static Part 4 sampling types, and bounded Features Part 3/CQL2 observation filtering. Preserve the approved Goal and Definition's security, status, tasking, ecosystem-integration, and interrupted-connectivity responsibilities without turning the server into an enterprise infrastructure project.

The implementation is one deployable Rust service backed by PostgreSQL/PostGIS. The HTTP interface, background publication, command handling, and controlled synchronization use the same validated resource model and transactional write functions. GeoJSON, SensorML, ordinary JSON, and SWE encodings are representations of those resources, not separate databases. Applications use the published CSAPI interfaces; an additional interface must have a specific purpose and be identified as a Glaux extension.

The selected engineering choices are:

- Axum/Tokio for HTTP and asynchronous work; SQLx for explicit PostgreSQL transactions.
- Relational storage for identity, relationships, timestamps, and lifecycle; JSONB for validated structured content; database storage for bounded original documents.
- Small, typed Rust modules with separate domain and encoding packages; no required microservices, graph database, message broker, or enterprise identity installation to exercise the basic reference server.
- Strict validation of public writes, authorized reads and writes, transactionally recorded publication events, and explicit command outcomes.
- Standards-derived tests plus real requests from independent clients. The implementation's own serializers are not the sole judge of correctness.
- Optional authenticated Server-Sent Events (SSE) as a Glaux interface and outbound MQTT 5 as the experimental Part 3 binding. Neither is represented as an approved Part 3 standard.
- Experimental static sampling Point/Curve/Surface types through existing SamplingFeature routes; no whole-Part-4 or derived-volume promise.
- Discoverable CQL2 JSON filters over direct sampling geometry and per-datastream scalar results, without changing native CSAPI query semantics or reconstructing missing history.

The core completion target remains all 25 direct CSAPI conformance classes and applicable prerequisites. The selected experimental capabilities and six additional filtering classes are also planned implementation deliverables, tracked separately from that count. Incremental releases may implement fewer classes, but must say exactly what works. JSON-only result support, read-only operation, partial command/feasibility APIs, or SSE-only streaming are not substitutes for the complete intended server; JSON-only CQL2 expression encoding does not restrict SWE result formats. A simulated device is valid test equipment for the full tasking interface; selecting operational hardware is not a new server-completion prerequisite.

This guide defines technical design and verification. The [Roadmap](glaux-server-roadmap.md) assigns implementation order and tasks. Governance defines the working rules. The approved Goal and Definition controls scope. No separate requirements document, decision-record system, or new approval process is introduced.

**Start with [§1.1: How the implementation fulfills the Goal](#11-how-the-implementation-fulfills-the-goal)** for a plain-English explanation of what each selected technology contributes to the required capabilities.

## Table of Contents

1. [Purpose and Scope Baseline](#1-purpose-and-scope-baseline)

   - [How the implementation fulfills the Goal](#11-how-the-implementation-fulfills-the-goal)

2. [Architecture Context](#2-architecture-context)
3. [Design Principles and Constraints](#3-design-principles-and-constraints)
4. [Implementation Specifications](#4-implementation-specifications)
5. [Integration Points](#5-integration-points)
6. [Data and API Contracts](#6-data-and-api-contracts)
7. [Conformance and Verification Strategy](#7-conformance-and-verification-strategy)
8. [Testing Strategy](#8-testing-strategy)
9. [Risks and Remaining Implementation Checks](#9-risks-and-remaining-implementation-checks)
10. [Quality Gates and Exit Criteria](#10-quality-gates-and-exit-criteria)
11. [Change Control](#11-change-control)
12. [References and Research Use](#12-references-and-research-use)
13. [Appendix: Standards Interpretations and Project Choices](#13-appendix-standards-interpretations-and-project-choices)

## 1. Purpose and Scope Baseline

<a id="11-scope-and-acceptance-boundary"></a>

### 1.1 How the implementation fulfills the Goal

Read this section alongside [Goal and Definition §§5–7](glaux-server-goal-and-definition.md#5-core-capability-scope). Each pairing summarizes what the Goal requires and explains how the selected implementation will deliver it. The technologies named here are Glaux engineering choices, not requirements imposed by OGC. These are plans for the completed server, not claims that the software already works; exact library versions and remaining implementation proofs are addressed in [§2.4](#24-platform-choices) and [§9](#9-risks-and-remaining-implementation-checks).

The explanations use the Goal's headings so a reader can move directly between a capability and its implementation. The linked detailed sections retain the precise behavior, limits and tests. This overview does not add requirements or replace those contracts.

#### Goal 5.1: Connected-System Discovery and Navigation

[Goal §5.1](glaux-server-goal-and-definition.md#51-connected-system-discovery-and-navigation)

- **The Goal calls for:** Finding the connected systems, information and operations available through the server, with descriptions that applications can understand.

  **To do this, the implementation will use:** An Axum web API with linked entry points for systems, collections, data and other resource families. Shared route definitions describe the supported addresses, operations and formats; the server uses them to assemble navigation links and an OpenAPI description of the enabled interface. A client can start with the server's root address and follow those links. A locally hosted documentation page and downloadable examples help people understand and exercise the same interface.

**Detailed design:** [§4.1 Discovery and API description](#41-discovery-navigation-and-api-description); [§6.2 Endpoints and representations](#62-endpoint-and-representation-baseline).

#### Goal 5.2: Registration and Description

[Goal §5.2](glaux-server-goal-and-definition.md#52-registration-and-description)

- **The Goal calls for:** Registering, describing, updating and retrieving systems, procedures, deployments, sampling features, properties and their associated information.

  **To do this, the implementation will use:** Axum to receive application requests, typed Rust resource models to represent the different kinds of information, and schema and semantic validators to check their content and relationships. Shared application functions check the caller's authority and permitted changes. SQLx then executes PostgreSQL transactions that save the accepted descriptions and related records together. Sensors, platforms, actuators and samplers are described through the applicable standard resources; their names do not create additional invented API families.

- **The Goal calls for:** Persistent identification and descriptions that communicate capabilities, deployment context, provenance, validity and lineage.

  **To do this, the implementation will use:** SensorML's structured descriptions together with database identifiers and explicitly checked relationships. Local resource IDs, persistent UIDs and source-specific identifiers retain their different roles. Relationships connect systems to components, procedures, deployments and sampling features. Retained description revisions distinguish when information applied from when the server received it; supplied method, source and input references preserve provenance and lineage without guessing missing history. PostgreSQL stores searchable relationships separately from validated extensible description content and retained original documents.

- **The Goal calls for:** Different representations of a resource preserving its identity, relationships and descriptive meaning.

  **To do this, the implementation will use:** One authoritative stored resource and explicit standards-specific mappings for its supported representations. Where both apply, a client can request SensorML JSON or GeoJSON describing that same resource. Serde handles JSON serialization; the project's mapping and validation code preserves the required meaning. Original source content is retained separately where needed, so a simpler representation does not erase richer information. Not every resource supports every representation, and a replacement that would lose existing writable content is rejected under the detailed write contract.

- **The Goal calls for:** Genuine support for the selected experimental Part 4 sampling types, not merely acceptance of arbitrary JSON.

  **To do this, the implementation will use:** The existing SamplingFeature API plus type-specific schema and semantic validation. A sampling point requires Point geometry, a sampling curve LineString geometry, and a sampling surface Polygon geometry. Creation and changes must pass the selected type, geometry and association checks before storage. These checks add the approved static specializations without removing generic SamplingFeature behavior or promising the entire Part 4 draft.

**Detailed design:** [§4.2 Descriptions and relationships](#42-descriptions-identity-relationships-and-collections); [§4.2.1 Experimental sampling types](#421-experimental-static-part-4-sampling-types); [§4.3 Validation and mappings](#43-sensorml-swe-common-validation-and-semantic-bindings); [§4.6 Writes](#46-writes-ingestion-concurrency-and-deletion); [§4.7 Storage](#47-persistence-spatial-indexes-and-data-lifecycle); [§4.10 Provenance and access](#410-authentication-authorization-trust-and-audit).

#### Goal 5.3: Access and Exchange

[Goal §5.3](glaux-server-goal-and-definition.md#53-access-and-exchange)

- **The Goal calls for:** Storing, retrieving and exchanging observations with their producing systems, observed properties, subjects, times, units and descriptive context intact.

  **To do this, the implementation will use:** DataStreams that bind a producing System and output to a defined value structure. SWE Common describes that structure and its units, quality and missing-value rules. The server validates submitted observations against the stream's contract, stores accepted values and their references in PostgreSQL, and uses the corresponding JSON, Text and Binary encoders for supported exchanges. Changing a description cannot silently reinterpret previously stored values; the stream-schema restrictions and representation mappings protect that meaning.

- **The Goal calls for:** Historical and current matching information, including the approved enhanced spatial and measured-value observation searches.

  **To do this, the implementation will use:** Explicit SQL queries and database indexes for source, relationship and time selection, with PostGIS for spatial tests. For enhanced observation searches, the server follows the observation's direct SamplingFeature relationship to explicit geometry established as applicable at observation time. It does not substitute a current location or an intersecting ancestor when that evidence is missing. Discoverable CQL2 JSON filters let a client combine the selected spatial conditions and typed scalar value comparisons within a datastream with applicable native filters. Access restrictions are applied before selection; units, missing values and latest-result selection keep their defined meaning.

- **The Goal calls for:** Preserving provenance and quality context rather than losing it during access or conversion.

  **To do this, the implementation will use:** Supplied links to producing systems, actual method/model revisions, exact input observations or source artifacts, creation times and responsible-role assertions. These are stored separately from the audit record of who uploaded the data. Quality components retain the subject, metric, units and supplied uncertainty or confidence information through the supported encodings. Unknown context stays unknown: storing these assertions does not certify their truth or automatically combine them into a universal probability score.

**Detailed design:** [§4.3 Value contracts and encodings](#43-sensorml-swe-common-validation-and-semantic-bindings); [§4.4 Observation access](#44-datastreams-observations-querying-and-spatial-behavior); [§4.4.1 Enhanced filtering](#441-enhanced-observation-filtering); [§4.10 Provenance and quality](#410-authentication-authorization-trust-and-audit); [§6.3 Query rules](#63-query-rules-and-limits).

#### Goal 5.4: Streaming and Dynamic Data

[Goal §5.4](glaux-server-goal-and-definition.md#54-streaming-and-dynamic-data); [Goal §4 experimental Part 3 commitment](glaux-server-goal-and-definition.md#4-standardization-basis)

- **The Goal calls for:** Applications receiving supported updates as information changes, while preserving the connection between saved data and published updates.

  **To do this, the implementation will use:** A database outbox—a record of outgoing work saved in the same transaction as the accepted change—and a retained publication log. Background workers move committed work into that log and deliver it with bounded retries. This keeps publication tied to accepted database changes and supplies the retained history needed for the specified recovery paths. Delivery can repeat after an interruption; it is not an exactly-once processing guarantee.

- **The Goal calls for:** Live exchange, including the explicitly experimental Part 3 support.

  **To do this, the implementation will use:** An optional Server-Sent Events (SSE) interface for authorized resource-change notifications over HTTP, and a separate outbound MQTT 5 adapter for the selected experimental event and native-data messages. The MQTT adapter uses rumqttc, with Mosquitto as the reference test broker; AsyncAPI describes its actual topics and formats. SSE is a Glaux extension, not the Part 3 binding. Native MQTT data is live delivery; a disconnected recipient uses the authorized HTTP/snapshot recovery path rather than assuming that the broker has retained every missed observation.

**Detailed design:** [§4.8 Publication and delivery](#48-publication-live-delivery-and-experimental-part-3); [§4.11 Catch-up and recovery](#411-interrupted-connectivity-replay-synchronization-and-conflicts); [§1.3 Experimental boundary](#13-experimental-part-3-boundary).

#### Goal 5.5: Tasking and Control

[Goal §5.5](glaux-server-goal-and-definition.md#55-tasking-and-control)

- **The Goal calls for:** Authorized applications submitting commands and following their execution status and results.

  **To do this, the implementation will use:** ControlStreams that describe accepted command inputs, the shared validation and access checks, and durable command/work records in PostgreSQL. A background command worker calls a device-adapter interface that translates the request into the connected system's own operations. Standard command-status and result resources expose the supported synchronous and asynchronous outcomes. A deterministic reference adapter makes these workflows runnable and testable without operational hardware; production adapters supply their own device protocols and safety interlocks.

- **The Goal calls for:** Distinguishing feasibility, acceptance, execution and confirmed outcomes, including safe behavior when communication fails.

  **To do this, the implementation will use:** Separate feasibility requests and results, retained command-lifecycle evidence, stable attempt identities and checks before dispatch. A completed feasibility analysis may answer that an action is not possible; it neither authorizes nor performs the action. A timeout or lost response is not proof that a command failed physically. Uncertain work is reconciled through adapter evidence rather than blindly resent, and editing or deleting a public report cannot restart an already completed action.

**Detailed design:** [§4.9 Commands and feasibility](#49-commands-feasibility-status-and-results); [§6.4 Response and recovery contracts](#64-success-error-and-command-response-contracts).

#### Goal 5.6: Status and Availability

[Goal §5.6](glaux-server-goal-and-definition.md#56-status-and-availability)

- **The Goal calls for:** Understanding a system's reported operating condition while distinguishing current evidence from stale, delayed or last-known information.

  **To do this, the implementation will use:** Status DataStreams and observations with defined value structures and meaningful timestamps. Database queries select relevant evidence by when it applied, not simply by which message arrived last. Configured freshness rules can identify old or missing evidence without inventing a failed-device state. System status remains separate from whether the server, broker or command channel is reachable.

- **The Goal calls for:** Access to System Events that explain relevant changes in the connected system.

  **To do this, the implementation will use:** The standard System Event resources and stored links to the affected System, event time, definition and source content. These represent reported occurrences such as calibration or relocation. An HTTP access log or a resource-edit notification is not automatically evidence that such an event occurred.

**Detailed design:** [§4.5 Status and System Events](#45-status-availability-dynamic-properties-and-system-events); [§4.12 Server diagnostics](#412-configuration-deployment-observability-and-developer-use).

#### Goal 5.7: Security, Authorization, and Trust

[Goal §5.7](glaux-server-goal-and-definition.md#57-security-authorization-and-trust)

- **The Goal calls for:** Enforcing configured rules about who can see information, publish or change data, subscribe to updates and command systems.

  **To do this, the implementation will use:** A verified-identity component and an access-policy interface shared by API operations. The selected identity adapter validates externally issued OAuth JWT access tokens, rather than trusting their contents without checking them. Caller/group/source permissions and resource relationships restrict queries, links, writes and tasking. Delivery checks and broker topic permissions protect live updates as well as ordinary reads. TLS protects configured network connections; a missing or unavailable authorization decision is not treated as permission.

- **The Goal calls for:** Accountability and protection of sensitive contextual information, including across organizational boundaries.

  **To do this, the implementation will use:** Durable audit records for important writes and command attempts, plus independent access checks for provenance, quality, schemas, source documents and related links. Supplied labels and binding evidence are preserved without inventing a universal per-field marking policy or claiming that a transformed document retains a verified signature. The server integrates with deployment identity and policy services; it does not provide those organizations' identity administration, release authority, cross-domain guards or accreditation.

**Detailed design:** [§4.10 Authentication, authorization and audit](#410-authentication-authorization-trust-and-audit); [§4.8 Delivery access](#48-publication-live-delivery-and-experimental-part-3); [§4.12 Safe configuration](#412-configuration-deployment-observability-and-developer-use).

#### Goal 5.8: Cross-Environment and DDIL-Informed Operation

[Goal §5.8](glaux-server-goal-and-definition.md#58-cross-environment-and-ddil-informed-operation)

- **The Goal calls for:** Useful, correct behavior when connections are disrupted, intermittent or bandwidth-limited.

  **To do this, the implementation will use:** Locally stored resources, schemas and configuration so permitted historical reads do not depend on a continuously connected publisher or broker. Bounded publication queues and recovery windows support delayed delivery. Timestamps keep last-known information distinguishable from fresh evidence, and disconnected operation does not extend expired credentials or manufacture current sensor availability.

- **The Goal calls for:** Controlled exchange and recovery that preserves source identity, handles repeated or delayed updates and identifies conflicts.

  **To do this, the implementation will use:** An administrative export/import format with resource identities, revisions, dependencies, explicit deletion records and receipts for accepted exchanges. PostgreSQL transactions accept coherent changes together; revision/ancestry checks distinguish known repeats from conflicting or missing changes. Consistent snapshots and retained change history support bounded catch-up, while unresolved conflicts remain explicit. Backup/restore procedures re-establish recovery continuity and hold uncertain commands for reconciliation. Protected omissions are not automatically deletions, and imported command history does not authorize new physical actions. This supplies server recovery behavior, not network connectivity or a general federation service.

**Detailed design:** [§4.11 Exchange and conflicts](#411-interrupted-connectivity-replay-synchronization-and-conflicts); [§4.7 Backup and restore](#47-persistence-spatial-indexes-and-data-lifecycle); [§4.12 Runtime use](#412-configuration-deployment-observability-and-developer-use).

#### Goal 5.9: Validation, Conformance, and Verification

[Goal §5.9](glaux-server-goal-and-definition.md#59-validation-conformance-and-verification)

- **The Goal calls for:** Correct standards behavior, not just successful requests or JSON that happens to parse.

  **To do this, the implementation will use:** Pinned local standard/schema artifacts, operation-specific structural validation, and Rust semantic checks for identities, relationships, times, units and state. Tests link back to the controlling requirements and use independently specified values, resource IDs and bytes as expected answers. Real PostgreSQL/PostGIS and HTTP tests check the actual storage and API behavior; independent clients and real-broker checks cover the boundaries they exercise.

- **The Goal calls for:** Strong verification across normal, invalid, unauthorized and interrupted workflows, with honest claims about what works.

  **To do this, the implementation will use:** Automated build/lint/test checks, negative and fault tests, bounded generated-input/property tests, parser fuzzing and targeted checks that deliberately introduce relevant faults to assess assertion strength. These methods accompany their owning capabilities rather than arriving only at release time. Ordinary issue/PR evidence records what actually ran, including failures and gaps. Conformance declarations are checked against implemented and tested behavior: all 25 direct CSAPI classes and applicable prerequisites remain the core target, with the six selected filtering classes and the bounded experiments tracked separately. Test success is not an OGC certification claim.

**Detailed design:** [§4.3 Validation](#43-sensorml-swe-common-validation-and-semantic-bindings); [§7 Conformance](#7-conformance-and-verification-strategy); [§8 Testing](#8-testing-strategy); [§10 Completion conditions](#10-quality-gates-and-exit-criteria).

#### Goal 6: Role in the Glaux Ecosystem

[Goal §6](glaux-server-goal-and-definition.md#6-role-in-the-glaux-ecosystem)

- **The Goal calls for:** A common server interface usable by Glaux applications, publishers, simulators and independent external clients.

  **To do this, the implementation will use:** The same published CSAPI resources and operations for standard-covered exchanges, with any additional integration interface explicitly identified. OpenAPI, schemas and executable examples describe those contracts; tests with OS4CSAPI and independent HTTP/Python clients check supported workflows. A developer can run the reference examples without first building the other Glaux products. The server supplies APIs, not those products' operational web or mobile user interfaces.

**Detailed design:** [§5 Integration points](#5-integration-points); [§4.1 API documentation](#41-discovery-navigation-and-api-description); [§4.12 Independent setup](#412-configuration-deployment-observability-and-developer-use).

#### Goal 7: Implementation Character

[Goal §7](glaux-server-goal-and-definition.md#7-implementation-character)

- **The Goal calls for:** A maintainable, open-source Rust reference implementation that other implementers can build, understand and test.

  **To do this, the implementation will use:** A small Rust workspace separating resource rules, standards/encoding logic and the running server, with PostgreSQL/PostGIS as its authoritative store. Shared application functions keep HTTP requests, command handling and publication from acquiring contradictory rules. Native build instructions, a Compose deployment example, sample data, executable help and backup/restore instructions make the selected design reproducible on approved prerequisites. A message broker is required only for the MQTT adapter, not for ordinary HTTP use.

- **The Goal calls for:** Incremental delivery without quietly reducing the intended capability or turning research recommendations into new objectives.

  **To do this, the implementation will use:** The existing Roadmap and its dependency-linked issues, with tests and documentation included in each authorized implementation task. Releases describe their actual supported behavior and limitations; full completion still requires all adopted capabilities. The core Parts 1/2 target includes required JSON, Text and Binary value support and complete intended tasking. Experimental Parts 3 and selected static Part 4 retain their labels, while Part 5 implementation remains deferred. The Guide's detailed contracts and recorded interpretations explain chosen mechanisms; the research is supporting evidence, not an additional unchecked backlog.

**Detailed design:** [§2.2 Component boundaries](#22-component-boundaries); [§2.4 Platform choices](#24-platform-choices); [§4.12 Developer use](#412-configuration-deployment-observability-and-developer-use); [§1.5 Part 5 disposition](#15-part-5-and-provenance-research-disposition); [§10 Completion conditions](#10-quality-gates-and-exit-criteria); [Roadmap implementation workflow](glaux-server-roadmap.md#5-implementation-iterations-and-task-size).

#### Design and verification index

The existing capability index below provides the detailed design and verification locations for the same approved scope.

| Approved capability | Design in this guide | Main verification |
|---|---|---|
| Discovery and navigation (§5.1) | §4.1, §6.2 | Follow links from root; check collections, schemas, API description, and declarations |
| Registration and description (§5.2) | §§4.2–4.3, §4.6 | Create, replace, patch, and delete; verify identity and equivalent representations |
| Access and exchange (§5.3) | §§4.3–4.4, §4.7 | Compare exact query results, timestamps, relationships, units, and encoding meaning |
| Streaming and dynamic data (§5.4) | §§4.4–4.5, §4.8 | Live delivery, duplicate delivery, interruption, replay, and experimental MQTT tests |
| Tasking and control (§5.5) | §4.9 | Feasibility, acceptance, execution, status, results, and failure scenarios |
| Status and availability (§5.6) | §4.5 | Stale and delayed evidence must not become false current status |
| Security, authorization, and trust (§5.7) | §4.10 | Positive and negative access tests, including lists, links, streams, and commands |
| DDIL-informed operation (§5.8) | §4.11 | Offline operation, retries, conflict detection, and safe recovery |
| Validation and verification (§5.9) | §4.3, §§7–8 | Standards tests, independent clients, and accurate release claims |
| Ecosystem integration and independent use (§§6–7) | §5, §4.12 | Build/run/test from documented dependencies without other Glaux products |
| Experimental static Part 4 (§4, §5.2) | §1.4, §4.2.1, §13 | Typed Point/Curve/Surface round trips, shape/association validation and query results |
| Enhanced observation filtering (§4, §5.3) | §4.4.1, §6.3.1, §7.4 | Six-class/dependency coverage, discovered queryables and independently expected spatial/value results |

Implementation-complete means the entire target is implemented and verified, documented examples work, known limitations are explicit, and no unresolved issue invalidates a claimed capability. Passing a partial release's tests does not establish full completion. A successful command-adapter test establishes that adapter's behavior, not universal device safety or accreditation.

Goal §8 exclusions remain in force. Identity administration, organizational release authority, cross-domain guards, networking infrastructure, federation agreements, and production operations belong to deployments. The server enforces configured rules and supplies integration contracts; it does not supply those organizations or infrastructure.

### 1.2 Standards baseline and exact dependencies

The controlling published package is [CSAPI Part 1 v1.0][S1], [CSAPI Part 2 v1.0][S2], [SensorML v3.0][SML], and [SWE Common v3.0][SWE]. Use the published requirements and normative tests, with official artifacts pinned to their source revisions. The CSAPI publication source is tag `v1.0.0`, commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`. A moving upstream branch is not a replacement baseline.

The direct inherited dependencies identified in [IDR-008][R008] are:

| Dependency | Version and scope incorporated |
|---|---|
| [OGC API - Common Part 1](https://docs.ogc.org/is/19-072/19-072.html) | OGC 19-072, v1.0.0: Core, Landing Page, JSON |
| [OGC API - Features Part 1](https://docs.ogc.org/is/17-069r4/17-069r4.html) | OGC 17-069r4, v1.0.1 (Core Corrigendum): Core and GeoJSON, with CSAPI's resource adaptations; class URI version remains `/1.0/` |
| OGC API - Features Part 4 | OGC 20-002r1, `1.0.0-draft.2`: Create/Replace/Delete and Update. Unapproved inherited draft; see the pin and reconciliation note below |
| SensorML 3.0 | `json-simple-process`, `json-physical-system`, `json-deployment`, `json-derived-property`, including their applicable dependencies and CSAPI mappings |
| SWE Common 3.0 | `json-record-components`, `json-encoding-rules`, `text-encoding-rules`, `binary-encoding-rules`, including the underlying applicable component and encoding rules |
| Simple Feature Access Part 1 | OGC 06-103r4, v1.2.1: the WKT grammar used by spatial filtering, not a new server API |
| Web formats and protocols | HTTP semantics/caching (RFC 9110/9111), JSON (RFC 8259), GeoJSON (RFC 7946), web linking (RFC 8288), and the date/time and schema rules incorporated by the above standards |

The transaction baseline remains Features Part 4 at `9ca25f56a58ed822ea8a685a7a41afa7181aaa8b`. The comparison with later write research's `4e30324a14b682ff4a26ee43aad1eb6428c846a3` found substantive differences, not just the Common Part 5 rename: body-ID handling, request media errors, discovery headers and optional response/preference classes changed. Follow the earlier `/put-rid` and Update `/rid` rules by ignoring the body resource's local identifier on replacement/patch; the path identifies the target. Do not import the later permission to reject a differing local ID or substitute Common Part 5 conformance URIs. Optional discovery headers can be supplied as documented HTTP behavior without adopting the later classes. Sections 4.6 and 13 distinguish local IDs from protected domain properties. [F4], [F4Later]

Conformance URI bases are exactly `http://www.opengis.net/spec/ogcapi-connectedsystems-1/1.0` and `http://www.opengis.net/spec/ogcapi-connectedsystems-2/1.0`. Requirement identifiers use `/req/`; declarations use `/conf/`. The repository name contains `connected-systems`, but these identifiers use `connectedsystems`. Section 7 lists every direct target class.

References do not import every independent capability of every related standard. XML import, unrestricted CQL2/joins, an arbitrary SensorML process-execution engine, and separate OGC service families are not added by this guide. The bounded filtering addition in §1.4 is an explicit project choice, not an inherited CSAPI obligation. Apply SensorML/SWE requirements where the selected CSAPI classes and mappings invoke them.

### 1.3 Experimental Part 3 boundary

Use the official Part 3 branch snapshot [`6f529a15bfa63259febc3620378d3e5a06305333`][P3] as the research-backed experimental baseline. The checked draft contains incomplete MQTT-binding material. Its resource-event and resource-data concepts inform §4.8, but Glaux must define and label its own concrete topic, security, discovery, and delivery choices where the draft does not. No approved Part 3 conformance claim is authorized. [IDR-014H][R014h], [IDR-035][R035]

### 1.4 Selected Part 4 experiment and additional filtering

Goal §4 retains the static Point/Curve/Surface implementation alternative from [IDR-058][R058], based on official `part4-working-draft` commit [`05a3c62d198ee52d0cf81a734b700967b7d864a1`][P4]. Implement the specialized contracts in §4.2.1, not merely generic GeoJSON storage. No approved Part 4 conformance URI is introduced. Dedicated Solid, Specimen, StatisticalSample, FeaturePart, relative/parametric types, mobile snapshot behavior and derived-volume computation are outside this experiment; their descriptions are not all equally costly, but none is an implicit commitment. Existing generic Part 1 and dynamic-data requirements remain intact.

The separately adopted querying capability uses [Features Part 3 v1.0, OGC 19-079r2][F3] and [CQL2 v1.0.0, OGC 21-065r2][CQL2], with the six complete classes in §7.4 and JSON expression encoding. It supports approved generic SamplingFeatures as well as the selected experimental types. The CSAPI endpoint/queryable mapping is a documented Glaux choice, not a claim that these standards define observation relationships. Broader CQL2 classes, Text expressions, arbitrary joins, unrelated related-property searches and reconstruction of missing geometry are not selected. This does not narrow native CSAPI filtering or SWE value support. [IDR-059][R059]

The scope is approved and this Guide supplies its technical baseline. Optional deployment enablement does not make these deliverables optional for full project completion. Advertise only implemented, enabled, tested behavior.

### 1.5 Part 5 and provenance research disposition

Following the accepted [Part 5 study][R060] and synthesis Addendum C, defer a Part 5 wire implementation. Retain the existing separation of logical values, immutable schema contracts and codecs so a later agreed encoding can be added without reinterpreting stored data. No Protobuf dependency, media type, endpoint, placeholder implementation or additional completion condition is introduced. Required SWE Binary remains in scope. Revisit a concrete pinned binding when justified by new specification/implementation evidence; publication is not an automatic prerequisite or automatic adoption decision.

Apply [the provenance study][R061] and synthesis Addendum D through the existing description, value, revision, security and test design below. Distinguish production history from server processing, preserve supplied exact-input/method/role context, and retain the meaning of quality assertions. No second observation model, public PROV API, graph database, universal confidence score or security-marking regime is adopted. These clarify Goal §§5.2–5.3 and 5.7; the approved capability scope is unchanged.

## 2. Architecture Context

### 2.1 Existing work and repository boundaries

This `glaux` repository is the planning and ecosystem documentation repository. Server implementation belongs in [`DGIWG-P507/glaux-server`](https://github.com/DGIWG-P507/glaux-server). Inspection for the first drafting pass found no Rust implementation in the meta-repository and only a README on the server repository's main branch. The design below is work to build, not an assessment of existing runtime behavior.

- **Build new:** server packages, database schema/migrations, codecs, HTTP handlers, validation, access checks, publication and command workers, tests, examples, and reference deployment files.
- **Extend:** the existing server README and build/run documentation when implementation starts; this guide as design details are resolved.
- **Reuse as inputs:** approved scope, completed research, official standards and permitted schema artifacts, and established Rust/PostgreSQL libraries. Peer server code is comparative evidence, not an assumed reusable Glaux implementation.
- **Remain separate:** Publisher, Simulator, Web App, and Mobile applications. A small server test client or fake device adapter is not an implementation of those products.

### 2.2 Component boundaries

Use a Cargo workspace with three initial production packages: `glaux-domain`, `glaux-standards`, and `glaux-server`. This retains a compile-time boundary around domain types and wire encodings without creating every package suggested in [IDR-045][R045] before it contains useful code. Application, persistence, HTTP, security, publication, and adapter modules initially live inside `glaux-server`; split them later only for a demonstrated dependency, reuse, or build need.

| Component | Owns | Must not own |
|---|---|---|
| Domain | Resource identities, typed relationships, times, schema bindings, invariants, command state rules | HTTP extraction, SQL rows, MQTT topics |
| Standards | CSAPI wire types and mappings, SensorML/SWE parsing and encoding, structural/semantic validation | Database writes, authorization decisions, arbitrary remote fetching |
| Application functions | Read/write use cases, orchestration of access checks, validation, transactions, and effects | Independent copies of transport-specific business rules |
| PostgreSQL module | SQL queries, transaction boundaries, constraints, revisions, audit/outbox records | Public wire schemas or device actuation |
| HTTP module | Routing, query parsing, negotiation, headers, response/error mapping, API description | Bypassing application validation or writing tables directly |
| Security module | Verified caller context, configured access rules, query restrictions, safe denial | Issuing enterprise identities or deciding organizational release authority |
| Publication module | Reading committed events, bounded replay, delivery attempts, SSE/MQTT encoding | Creating authoritative data from a notification alone |
| Command adapters | Device-specific validation, dispatch, cancellation, and status/result translation | Inventing successful physical outcomes or bypassing authorization |
| Runtime | Configuration, dependency wiring, startup/shutdown, supervised workers, telemetry | Domain decisions hidden in global state |

Dependency direction is toward domain types. Standards code may depend on the domain package; the domain package does not depend on standards, Axum, SQLx, or a broker. Repository/adapter interfaces should describe useful operations, such as committing an observation or reporting a command result, rather than a generic abstraction for every SQL operation.

### 2.3 Interaction model

```text
Glaux applications and independent clients
                  |
             CSAPI HTTP
                  |
      authentication / query / codecs
                  |
      application functions + access rules
                  |
       PostgreSQL/PostGIS transaction
       resources + revisions + audit + outbox
                  |
          committed background work
           /                    \
   event publication       command adapter
    SSE / MQTT            device or test double
```

Reads run through authorized queries and the selected encoder. Writes validate before committing; external delivery begins only after commit. Adapter reports return through the application write boundary. A failed broker or device connection must not undo a committed observation or cause a command to be blindly submitted again.

### 2.4 Platform choices

Use Rust stable, Axum with Tokio/Tower, Serde for wire serialization, SQLx with PostgreSQL/PostGIS, and `tracing` for structured telemetry. Use explicit SQL for temporal, spatial, and relationship queries. These are engineering choices supported by [IDR-025][R025] and [IDR-044–045][R044], not OGC requirements.

Pin an exact Rust toolchain, crate lockfile, PostgreSQL/PostGIS image, and enabled library features when the first build is established. Do not copy research-time patch versions into a claim of a tested build. The first implementation must prove the chosen JSON Schema validator against the actual recursive SensorML/SWE corpus, with HTTP/filesystem reference resolution disabled by default. A validator accepting simple JSON examples is insufficient. Primary implementation references include [Axum](https://docs.rs/axum/latest/axum/), [SQLx](https://docs.rs/sqlx/latest/sqlx/), and the [jsonschema reference-resolution controls](https://docs.rs/jsonschema/latest/jsonschema/#external-references); pin version-specific documentation with the eventual dependency lock.

## 3. Design Principles and Constraints

1. **One resource, one identity.** Nested routes, collection membership, and alternate encodings resolve to the same resource; they do not create competing copies.
2. **Standards behavior before convenience.** A library default, upstream example, or peer implementation cannot override a published requirement silently.
3. **Keep meaning intact.** Preserve schema versions, property definitions, units, time meaning, and source evidence. Do not treat all JSON objects or timestamps alike.
4. **One write boundary.** Public clients, test tools, device reports, and synchronization cannot bypass validation, authorization, or transaction rules.
5. **Commit before external effects.** A database change and its queued publication/dispatch work commit together; external acknowledgements describe delivery, not domain truth.
6. **State uncertainty honestly.** Accepted is not executed; last-known is not current; transport success is not proof of a physical effect.
7. **Keep the reference server usable.** Supply small examples, a runnable local deployment, and clear errors. Require additional infrastructure only for the capability using it.
8. **No unearned claims.** Disabled routes, incomplete encodings, drafts, and known deviations remain visible in release documentation.

Each capability below separates required behavior, selected implementation, and verification. Sections 9 and 13 retain implementation checks and standards interpretations in this guide. They are not instructions to create a separate decisions program. Library choices, table layouts, default limits, and custom endpoints remain project design choices unless a cited standard requires them.

## 4. Implementation Specifications

### 4.1 Discovery, navigation, and API description

**Required behavior and source.** Serve a linked landing page, conformance declaration, API description, collections, and each implemented resource family using CSAPI/Common/Features rules. Links must allow a client to discover the supported API from the root. Collections are views over canonical resources. [S1], [S2], [IDR-009–010][R009]

**Implementation.** Keep a small typed definition of each route's methods, resource family, parameters, representations, and conformance dependencies beside its handler. Use it to assemble the router, links, and deployment API description. This is ordinary shared metadata, not a new registry service. Curated descriptions/examples remain versioned code assets. Tests check the metadata against actual requests independently.

Generate one implementation-specific OpenAPI 3.1 description covering enabled Parts 1 and 2 and clearly labeled extensions. Serve JSON plus a locally hosted human-readable documentation page; provide downloadable schemas/examples for offline use. Select and pin the documentation renderer during implementation. Do not deploy the upstream example OpenAPI bundle unchanged, and do not claim OGC's separate OAS 3.0 class merely because a 3.1 document is available. [IDR-014][R014]

Build absolute links from a configured public API root. Trust forwarded origin headers only from configured reverse proxies. Keep canonical URLs stable across routine software releases. An API-root path prefix is deployment configuration, not a version of a resource or SensorML schema. Use canonical links on alternate/nested views and preserve applicable media type and query context in paging links.

**Verification.** Begin with only the root URL; discover and exercise every advertised family, collection, schema, and representation. Test a path-prefixed reverse proxy, forged forwarding headers, empty collections, disabled capabilities, and a missing resource. Compare documented methods/media types with actual responses in both directions.

### 4.2 Descriptions, identity, relationships, and collections

**Required behavior and source.** Implement Systems, Subsystems, Deployments, Subdeployments, Procedures, Sampling Features, and Property Definitions with their specified associations and formats. Preserve persistent identifiers and cross-representation meaning. A Procedure is not a positioned System; a Property Definition is not a GeoJSON Feature. [S1], [IDR-015–017][R015], Goal §§5.2–5.3

**Implementation.** Use typed IDs internally. Mint UUIDv7 local resource IDs, while treating them as opaque locators, not authorization tokens or authoritative occurrence times. Preserve URI-form UIDs and separately store source-specific identifiers with their source authority. Enforce uniqueness and report conflicts rather than automatically merging resources that happen to share a label or source identifier.

Store Systems and Deployments once. Parent/child membership, deployment participation, procedure references, and sampling relationships are typed associations. Enforce endpoint types, required cardinalities, and hierarchy-cycle prevention. Derive reverse links and nested queries from those associations. Inline SensorML components do not automatically become separately addressable Systems.

Store description revisions with valid time and receipt/commit time separately. Internal revisions support concurrency, provenance, and reproducible representations; they do not create a published CSAPI System History class or justify advertising removed draft `/history` routes. Keep external feature/result references without assuming Glaux owns their lifecycle or can fetch them safely.

Collections store metadata and either explicit membership or a documented server-managed view. Collection membership does not change canonical identity. Custom-collection membership operations and deletion semantics must follow the selected transaction requirements, including distinguishing removal from a collection from deletion of the resource itself.

**Verification.** Register a System with a Procedure, subsystem, Deployment, Sampling Feature, and Property; reach the same IDs through direct, nested, and collection routes. Round-trip GeoJSON/SensorML where both apply, test duplicate UIDs, invalid cycles, optional/external references, and authorized deletion effects. A resource must not disappear merely because a different representation was requested.

#### 4.2.1 Experimental static Part 4 sampling types

**Required behavior and source.** Implement static sampling points, curves and surfaces using the existing SamplingFeature identity, associations, GeoJSON routes and applicable transaction operations. The project requirement is Goal §4; draft `/req/sampling-spatial/{type,om,shapes,location}` supplies the selected semantics. The conditional mobile `location-time` behavior is not selected. [P4Types], [IDR-058 §§4.2–4.3, 5–7][R058]

**Implementation.** Use `application/geo+json`, with the type in `properties.featureType` and the conceptual shape in top-level `geometry`, not a second `properties.shape` object. Bind the following specialized types to their explicit geometry. The URI column appends its suffix to `http://www.opengis.net/def/samplingFeatureType/OGC-OM/2.0/`:

| Accepted type alias | Canonical type URI suffix | Required non-null GeoJSON geometry |
|---|---|---|
| `om:SamplingPoint` | `SF_SamplingPoint` | `Point` |
| `om:SamplingCurve` | `SF_SamplingCurve` | `LineString` |
| `om:SamplingSurface` | `SF_SamplingSurface` | `Polygon` |

Accept those exact aliases and full URIs; expand aliases for validation and emit the full URI consistently, retaining the submitted form where original-source preservation applies. This reconciles the draft prose's URI/CURIE permission with its full-URI schema constants as a documented experimental encoding choice. Do not accept arbitrary prefixes as equivalent types. Require explicit geometry of the matching kind; reject missing/null geometry and mismatched shape for these advertised specializations rather than falling through to a generic schema. Preserve valid coordinate precision/height and common sampling associations. The generic Part 1 rules for other/non-spatial features remain unchanged.

Package a versioned local validator for the selected subset, composing the approved generic SamplingFeature contract and the pinned subtype constraints. Repair only references needed by that subset; retain the upstream artifacts and describe adaptations in §13. The draft's incomplete `anySamplingFeature` bundle, generic fallback and pose-only alternatives do not define support. No request-time schema fetching, new resource family, or geometry-derivation engine is needed.

Create, replace and patch must validate the complete resulting specialized resource through the ordinary write boundary; reads and list filters must preserve its type, geometry and relationships. Use explicit geometry for the existing `bbox`/`geom` operations. Keeping a static description does not establish its applicability to every historical observation; §4.4.1 governs that additional query meaning.

**Verification.** Independently test all three types, both accepted identifier forms and canonical output, required associations, wrong/missing/null geometries, coordinate order, permitted height, CRUD/PATCH round trips, ordinary spatial results and access controls. Prove that unsupported specialized fields are not silently lost or advertised as typed support. These are exact selected-draft/project tests, not approved Part 4 certification.

### 4.3 SensorML, SWE Common, validation, and semantic bindings

**Required behavior and source.** Implement the SensorML mappings and SWE component/encoding rules invoked by the selected classes, not merely JSON syntax validation. Observation and command values must match the relevant parent stream's schema. [SML], [SWE], [IDR-021–024][R021]

**Implementation.** Keep three distinct things: the received source document where preservation is needed, the typed meaning used by the server, and the generated wire representation. Store exact original SensorML/schema bytes with media type and digest; do not serve unfiltered original bytes as a shortcut around access checks. Preserve permitted extensions without allowing them to replace reserved identities or validated relationships.

The SWE component model covers scalars, ranges, records, vectors, choices, arrays, matrices, and geometry, together with names/order, definitions, units, constraints, nil values, quality, and reference frames. Compile an immutable component/encoding description into a bounded validation/codec plan. Bind observations and commands to its revision. A stream-schema edit must never reinterpret historical values; enforce the published schema-change restrictions and use a new stream when an incompatible contract cannot be changed legally.

**Quality JSON interpretation.** The published simple-component JSON schema does not declare `quality`; permissive acceptance of an unknown member is not validation. Retain unmodified upstream schemas and apply a separate, explicitly documented semantic check. Interpret optional `quality` as an array whose entries are either inline `Quantity`, `QuantityRange`, `Category` or `Text` components (the conceptual Quality union), or reference objects. Validate each component's units, constraints, nil rules and supplied value. An empty array means no supplied quality and can be omitted from canonical output. An inline value supplies static quality; a description without a value does not imply zero uncertainty. This is a Glaux interpretation supported by the conceptual model and legacy peer examples, not a claim that the published JSON schema defines this form. [SWE §8.2.15][SWE], [R022], [R061], [QualityPeer]

Dynamic quality uses an ordinary field in the encoded record, referenced by a local `href` to its component `id` in the same immutable contract. For example, a measurement's `quality: [{"href":"#PRESS_QC"}]` can identify a Category field with `id: "PRESS_QC"`; each record carries that field's own value. Resolve one eligible component and record occurrence, rejecting duplicate IDs, invalid targets, unresolved local references and cycles. Enforce the existing size/depth limits on nested quality and reference traversal. Missing/nil quality remains unavailable, not a pass. Preserve nonlocal reference metadata without fetching it or pretending to resolve per-record quality. No hidden extra codec values, new quality endpoint or mandatory universal metric is introduced.

Ordinary CSAPI JSON schema resources wrap SWE descriptions such as `resultSchema` or `parametersSchema`. These are not themselves JSON Schema documents. SWE payload formats use `recordSchema` plus an encoding definition. The schema query selectors are `obsFormat` and `cmdFormat`; `commandFormat` is a response member, not the command-schema query parameter.

Validate in this order: request limits and media type; safe parsing; structural schema; typed resource/component semantics; relationships and units/time; authorization and source authority; state/concurrency constraints; transaction. Authenticate and perform inexpensive admission checks before costly parsing, then perform object-specific authorization once the target is known. Return safe paths and reasons without exposing protected schema details.

Select structural validation by operation and request/response direction. JSON Schema `readOnly`/`writeOnly` annotations do not automatically remove generated fields from required-input checks. Maintain explicit request projections for server-generated/read-only members and response projections for write-only members, retaining unchanged upstream schemas and recording each adaptation. Clients need not supply generated IDs, server-generated parent links, format lists or derived summaries. Validate generated responses independently, and preserve §4.6's specific treatment of supplied resource-local IDs. [R023], [DataStreamSchema], [ObservationSchema]

Install required schema references locally and resolve them from an allowlist. Do not fetch arbitrary `$ref`, SensorML links, data URLs, or result URLs during a public request. Bound recursion, array sizes, regex work, binary allocation, decompression, and total request cost. Fail explicitly on unsupported encoding features; do not advertise them as complete.

Implement JSON, Text, and Binary codecs for the full applicable target, with field order, choice discriminators, optional values, nil representations, byte order, sizes, and framing tested. Receiving a BinaryEncoding descriptor is not proof that its values can be decoded. XML/legacy import is not necessary to claim these CSAPI JSON-based classes and is not added as a separate implementation objective.

Preserve unit declarations and property URIs. Validate UCUM codes where supplied against the incorporated UCUM basis; a semantic unit URI is not invalid simply because it is not a UCUM code. No implicit unit conversion is performed on command input. Any supported observation conversion must specify dimensional compatibility, precision, and original values. Labels and matching dimensions alone do not establish property identity.

**Verification.** Use valid and invalid examples for every component family and format, recursive references, nil versus missing, exact numeric boundaries, variable arrays, binary lengths, text delimiters, and geometry. Compare independently defined expected values after round-trip conversion. Test that schema changes cannot reinterpret old observations or queued commands and that malformed input cannot trigger external network/file reads.

### 4.4 Datastreams, observations, querying, and spatial behavior

**Required behavior and source.** Expose DataStreams, Observations, their schemas, links, filters, and prescribed temporal/spatial behavior. Preserve source and feature context, phenomenon time, result time, units, and quality. [S2], [IDR-011][R011], [IDR-018][R018], [IDR-027][R027]

**Implementation.** A datastream belongs to its producing System and binds a particular output and immutable value contract. Each observation stores the stream and schema revision, relevant feature reference, value, phenomenon/result times, and receipt metadata. Store typed query columns separately from a validated extensible payload. Derive stream summaries from supported contracts and authorized data rather than accepting client-written summaries as fact.

Register a DataStream through `POST /systems/{sysId}/datastreams`, supplying the singular write-only `schema` member for its initial encoding. Validate and compile the contract before accepting observations; derive advertised `formats` and equivalent schema representations only from available lossless codecs. The `/schema` route is retrieval, not a separate registration endpoint. Use the corresponding `POST /systems/{sysId}/controlstreams` pattern for ControlStreams. While nested observations or commands exist, reject schema-modifying PUT/PATCH with `409`; calling a change compatible or creating an internal revision does not bypass that restriction. Description-only changes remain possible. [S2 §§14.2, 14.4, 15.2, 15.4][S2], [DataStreamSchema], [ControlStreamSchema]

Parse query parameters into typed predicates. Apply route/collection scope and caller access restrictions before filters, aggregation, paging, and encoding. Parameters combine with AND; alternatives within a list combine with OR according to their specified grammar. Reject malformed or unsupported parameters rather than ignoring them. A well-formed identifier matching nothing produces an empty result, not a syntax error.

Implement both Advanced Filtering classes fully, including hierarchy, relationship, property, spatial, temporal, command, and event filters on their applicable routes. The separately selected Features Part 3/CQL2 capability adds the bounded observation filters below; it does not introduce client sorting, field selection, expansion or arbitrary joins. Section 6.3 summarizes the query families; the exact endpoint applicability comes from each requirement, not the upstream OpenAPI omissions.

Use RFC 3339 instants and slash intervals, including the inherited open-end forms. For `resultTime=latest`, first apply all other predicates within the endpoint scope, then select the greatest visible result time and retain ties. The canonical observation endpoint and a single-stream nested endpoint have different scopes. Do not generalize `latest` or `now` to every temporal parameter based on an example.

Use PostGIS for spatial predicates. Preserve source geometry/reference information and generate the required CRS84 GeoJSON order. Treat `bbox` and WKT `geom` according to their different rules, including geometry-less features. Use fixtures for antimeridian crossing, 3D inputs, empty geometries, and invalid coordinates. Never silently discard vertical or moving-position information while claiming an equivalent rich representation.

Use deterministic ordering with a unique ID tie-breaker: result time then ID for observations; stable ID order for ordinary resource lists unless a family requires a different rule. Return server-generated opaque `next` links, not a promised public offset API. Reauthorize continuations. For ordinary lists, document keyset paging as a changing view rather than claiming a cross-request snapshot; use an explicit snapshot/export mechanism for synchronization (§4.11). Omit optional totals when they cannot be calculated correctly and affordably.

**Verification.** Seed distinguishable records and assert exact selected IDs and order, not just response shape. Exercise filter combinations, latest-time ties, delayed ingestion, shared Systems, recursive hierarchies, empty matches, authorized counts, paging under inserts/deletes, and indexed spatial/temporal queries. Compare stored/retrieved value semantics across all supported formats.

#### 4.4.1 Enhanced observation filtering

**Required behavior and source.** Support the selected Features Part 3/CQL2 classes and Goal §5.3 through the observation-list contract in §6.3.1. Named queryables describe a filtering view; they need not be extra fields in returned observations. Preserve existing recursive `foi` association matching as a different operation from testing direct sampling geometry. [F3], [CQL2], [IDR-059 §§4.1–4.3][R059]

**Implementation.** Resolve `samplingGeometry` from the observation's direct SamplingFeature association and retained, authorized explicit geometry applicable at its phenomenon time. Never substitute an ancestor, System position, measured position result, implicit footprint, latest description or result time. For an instant, require an unambiguous applicable geometry assertion; for a phenomenon-time interval, require one unambiguous explicit geometry applicable throughout that interval. Otherwise the value is unavailable. These are Glaux mapping rules, not Part 4 mobile reconstruction. Missing validity, no movement event or a currently static-looking point is not evidence of past location. Do not interpolate, carry positions forward across unknown gaps or dereference external links during filtering.

On a single datastream, bind selected scalar queryables to immutable SWE contract/component paths, property identity, scalar type, unit and nil rules. Evaluate decoded logical values independently of response encoding. A numeric literal uses the advertised component unit; no implicit cross-unit conversion or text-to-number cast is performed. Do not flatten arrays or infer any/all semantics. The selective query view does not restrict which otherwise valid SWE data can be stored or returned.

Advertise only mappings yielding at most one scalar per observation:

| SWE component | Initial queryable mapping |
|---|---|
| Count | `integer`; exact numeric comparison without floating-point loss |
| Quantity | `number`; finite values in the advertised unit |
| Boolean | `boolean`; false is a value, not absence |
| Text / Category | `string`; retain code-space/property identity, without inferred ranking or numeric conversion |
| Numeric Time | `number` in its declared unit and time frame/origin, without guessed UTC conversion |
| Calendar Time with established temporal meaning | `string` with `date` or `date-time`; compare exact dates/instants only when the source frame supports that interpretation |

Resolve each component through the observation's contract revision and recognize its declared nil sentinel first. Remaining NaN and infinities map to unavailable numeric values (CQL2 NULL), with their distinct source states preserved. This is a finite-number filtering view, not rejection of valid SWE values. A compatible mapping revision must preserve property identity, scalar meaning, unit/code space/time frame and missing/nil rules. An explicitly recorded path change can retain an alias only for the same logical value; changing Cel to Fahrenheit or a scalar to an array is not silently compatible. Queryable compatibility never overrides CSAPI's stream-schema restrictions. [SWE], [R059]

Resolve authorized queryable use and the eligible observation/property/relationship view before evaluating predicates. Legitimately unavailable geometry, missing scalar content and declared nil map to CQL2 NULL for that queryable, with source distinctions retained internally. Protected information must not simply be substituted with NULL: apply §4.10's denial/concealment policy to prevent membership, null tests, errors or counts revealing it. Only TRUE selects a record. Combine native filters and CQL2 with AND, then apply `resultTime=latest` within the resulting scope, ordering, counts and paging. Distinct relationship paths cannot duplicate an observation.

Translate a bounded, typed expression tree to allowlisted, parameterized database operations; neither property names nor literals become raw SQL. Use exact PostGIS predicates after any index prefilter. Bind next-page cursors to normalized native/CQL2 filters, endpoint scope and queryable-mapping version, and reauthorize every continuation. Reject a mismatched/retired mapping; retain §4.4's changing-view paging contract rather than promising cross-request snapshots.

Only accepted authoritative revisions participate. A staged conflicting input does not erase the previous accepted value. An authorized correction identifies superseded evidence and retains revision/audit linkage; update its projections atomically or evaluate authoritative values until the projections are current. A stale index must not exclude a true match. If accepted retained geometry still contains unresolved competing applicable assertions, `samplingGeometry` is unavailable, not last-arrival-wins. Data corrections may change ordinary paged results; they do not create a retrospective snapshot.

**Verification.** Use the independently specified cases in IDR-059 and §8.2: direct versus ancestor geometry, historical versus current position, absent history, scalar thresholds versus nil, mixed types/units, native-filter combinations and exact selected IDs. Check disclosure, errors, class coverage and continuation, not merely expression parsing.

### 4.5 Status, availability, dynamic properties, and System Events

**Required behavior and source.** Support status information and System Events while distinguishing evidence time, last-known state, and operational availability. Dynamic Sampling Feature properties retain observation context. [S1], [S2], [IDR-020][R020], [IDR-034][R034], Goal §§5.4 and 5.6

**Implementation.** A status datastream uses the standard datastream/observation machinery and a typed SWE contract. A current-status view selects relevant evidence by its meaningful time, not merely the last received row. Preserve delayed samples as history without replacing newer evidence accidentally. Where freshness is assessed, document the configured age/source rule and evaluation time; absence of fresh evidence is unknown or stale, not proof of a failed device.

Keep System operational status, stream delivery state, command-channel availability, server health, and authorization separate. Do not invent a universal readiness score or a new mandatory status API. Use standard fields and observations first; expose additional assessment metadata only through a documented extension where needed.

System Events record actual events such as calibration, relocation, or configuration change. They are not aliases for HTTP access logs, resource-update notifications, or every observation arrival. Store event identity, parent System, event time, type/definition, descriptive content, and source. Use `application/json` and the Part 2 JSON item/collection schemas: map conceptual type/name/time to `definition`/`label`/`time` and preserve supported SensorML Event content. Creation uses `/systems/{sysId}/events`; canonical PUT/PATCH resolves the parent from the existing event, without requiring it to be resubmitted. Nested routes and any supplied association must agree. The selected parent mapping is an authorized `links` entry with `rel: "system"` and the canonical parent `href`. Do not accept competing conceptual/JSON names as ambiguous aliases. The unresolved conceptual `message`/parent-field discrepancy remains qualified in §13; the selected mapping must not silently discard submitted content. [S2 §16.1.11][S2], [System Event issue][EventIssue]

**Verification.** Exercise fresh, old, delayed, equal-time, missing, and conflicting status observations; demonstrate that API uptime does not imply a System is available. Verify historical dynamic properties and System Event query results, and distinguish a metadata edit from evidence of a real-world event.

### 4.6 Writes, ingestion, concurrency, and deletion

**Required behavior and source.** Implement the selected Create/Replace/Delete and Update classes, including subordinate resources and collection membership. Validate Observation/Command bodies against their parent schemas. Preserve atomicity and correct error semantics. [S1], [S2], [IDR-029][R029], [IDR-031][R031]

**Implementation.** Public creation uses POST; PUT replaces an existing resource, not an undocumented create-by-PUT operation. The request URI identifies the target of PUT/PATCH; ignore a supplied resource-local `id` under the selected inherited draft. It cannot rename the resource, and mismatch alone is not a conflict. This does not permit changing an immutable domain UID, protected parent association or schema contract. Validate those independently.

PUT replaces client-writable content, not a merge with the old representation. Omitted optional fields are removed where the submitted format can express them. Preserve server-owned identity/generated relationships and restricted revision evidence separately. Accept cross-format replacement only when the mapping can represent existing client-authored content without loss. Otherwise return `409` without mutation, with an authorized explanation and compatible replacement format; neither silently discard richer content nor secretly merge it. This is the selected Glaux treatment of the open cross-encoding question. [Replacement issue][ReplaceIssue]

PATCH uses `application/merge-patch+json` against one writable projection: SensorML JSON for Systems, Procedures, Deployments and Properties; GeoJSON for Sampling Features; ordinary CSAPI JSON for Part 2 resources. `Accept` negotiates the response, not this target. Omitted members retain values, `null` removes a member, and arrays are replaced whole. Reject attempted changes outside the writable projection except the ignored resource-local `id`; preserve untouched out-of-projection content. Validate the entire resulting resource before one atomic commit. If record content has no lossless ordinary-JSON form, do not fabricate one: other metadata remains patchable, while replacing that content uses PUT with an advertised compatible encoding. Document this restriction per operation rather than omitting a required PATCH family. Publish `Accept-Patch` and method support. [RFC 7396](https://www.rfc-editor.org/rfc/rfc7396.html#section-2), [Patch issue][PatchIssue]

Implement required mutation operations for observations, commands, command status/results, feasibility requests and their status/results, and System Events as well as descriptions and streams. Internally retaining revisions does not make the public resource immutable. Authorization, parent-schema invariants, and current command state can restrict a particular change; these checks must not become a blanket removal of an entire required operation.

Each accepted operation commits resource state, required revisions, relationships, audit information, any retry record, and outgoing event/work records in one transaction. Use database constraints, short transactions, and row locks where coordinated changes require them. Do not hold a database transaction open while contacting a device, broker, or identity service.

Emit strong representation-specific ETags where HTTP permits and honor supplied conditional requests accurately. After a PUT that transforms submitted content, omit validators from that success response and expose the current validator on a subsequent GET, as required by [RFC 9110 §9.3.4](https://www.rfc-editor.org/rfc/rfc9110.html#section-9.3.4). The baseline permits ordinary standards writes without a mandatory `If-Match` header. This deliberately does not adopt IDR-029/031's stronger mandatory-header policy. Supplied stale conditions fail with `412`; internal locking protects server invariants, but cannot detect a stale client's unconditional overwrite. Document this tradeoff and encourage conditional updates in examples.

Provide an optional, documented `Idempotency-Key` extension for creation and command submission. Scope the key to verified caller/source, operation, and target; retain its request digest and outcome atomically. The same key and same intent return the same resource/outcome, while different intent under the same key returns a conflict. Reauthenticate and authorize replay disclosure before returning stored outcomes. The response must reflect the documented replay contract, not an accidentally fresh second creation. Record a configurable terminal-outcome retention period and explain that reuse after expiry does not guarantee deduplication; unresolved admitted command work retains its key (§6.4). Do not require this header for all CSAPI clients or deduplicate distinct measurements merely because their values match.

For a supported multi-record request, use a bounded all-or-nothing transaction: validate all items before acceptance and return a specified standard response where one exists. This is a Glaux atomicity choice, not a new OGC batch claim. Larger Publisher jobs send bounded requests and track their outcomes; there is no mandatory private ingestion envelope. Verify resource-specific array/framing rules before enabling each bulk operation.

Apply the specified deletion and dependent-resource behavior with authorization across the affected set. A populated stream's default deletion fails with `409`; authorized `cascade=true` deletes the stream and its prescribed nested resources. Treat `cascade=false` as non-cascading, following the published tests, and reject malformed Boolean values. Outstanding private command admissions do not veto an otherwise required authorized cascade (§6.4). Keep restricted internal revision/tombstone evidence where needed for audit and synchronization, but remove the public resource as required. Never use internal retention to pretend a failed deletion succeeded. Do not reuse a deleted ID. Deleting a Command is not a request to cancel device execution. [S2 requirements 65/70 and tests A.65/A.70][S2]

**Verification.** Test partial-invalid batches, parent/schema mismatch, concurrent updates, stale preconditions, retry-key collision and expiry, transaction rollback, relationship cleanup, duplicate POST delivery, and restart after commit but before response. Assert that a write rejected before durable admission leaves no canonical resource or outgoing event behind. An HTTP wait failure after command admission is not rollback (§6.4).

### 4.7 Persistence, spatial indexes, and data lifecycle

**Required behavior and source.** Preserve coherent resources, context, history needed for meaning, and durable accepted data. The database product and layout are design choices. [IDR-025–030][R025], [IDR-049][R049], Goal §§5.2–5.3 and 5.8

**Implementation.** PostgreSQL/PostGIS is the single authoritative store. Section 6.1 defines the logical tables and invariants. Use typed relational columns and foreign keys for identities/parents, schema references, times, and lifecycle. Use JSONB for validated extensible structures, not as a substitute for relationships or query constraints. Preserve exact bounded document bytes in `bytea` with digests because JSONB does not preserve a source document byte-for-byte.

Begin with ordinary indexed observation tables. Choose native time partitioning only after measuring the representative workloads in §8; do not make TimescaleDB a prerequisite. If partitioning is adopted, preserve global ID uniqueness explicitly because a partitioned unique constraint normally includes the partition key. Time-series scale must not silently change resource identity or URL stability.

Indexes cover canonical/UID lookups, parent and relationship traversal, stream plus result/phenomenon time, and relevant PostGIS geometry. Add indexes from demonstrated query plans, not every field. Apply a precise spatial predicate after bounding-box index selection. Assigning an SRID is not coordinate transformation.

Enhanced filters reuse those associations, retained geometry/resource revisions and selective scalar projections. Retain each projection's observation, contract and component mapping; do not expand every SWE leaf into an index. Geometry lookup uses documented semantic applicability rather than the latest stored row. Recompute affected projections after an authorized correction and invalidate incompatible mapping cursors; never fabricate history removed by retention or never supplied.

Keep timestamp meaning and source precision. Use normalized instants for indexed comparisons and retain the source lexical value/precision where necessary. For precision beyond PostgreSQL timestamp resolution, store a checked remainder or exact normalized numeric value and use it in ordering/comparison; do not silently round a boundary match. The exact Rust/time storage type is a remaining implementation proof (§9).

Do not enable automatic retention/purge by default. An explicitly configured policy may remove old records only while preserving the required visible deletion behavior and dependencies among values, schemas, source documents, pending work, and synchronization tombstones. Derived summaries can be rebuilt; accepted observations and command evidence cannot be reconstructed from sampled logs.

Migrations are immutable SQL files packaged with the server and run by an explicit administrative command. Normal startup checks compatibility rather than applying destructive upgrades silently. Test backup and restore into a separate, isolated database, including artifacts, schema bindings, IDs, tombstones, and pending work. Keep external effects and command admission/dispatch disabled until restore checks and command reconciliation permit activation; reads can be validated without actuating devices.

A rollback or fork establishes a fresh recovery/source epoch before serving continuity tokens. Bind SSE and exchange continuity to that epoch; old tokens require a fresh snapshot, even when their numeric position exceeds the restored log head. A normal restart without rollback retains the epoch. Fence an old serving instance before activating its replacement; a test clone must not publish or command as the original. Restored pending work remains held, and reconciliation also covers commands accepted after the backup whose execution or retry records may now be absent. Do not automatically resubmit them or promise same-key deduplication across a lost recovery interval. Document that limitation and use retained receipts/adapter evidence to settle uncertain work before re-enabling the affected command path. [R049]

**Verification.** Use real PostgreSQL/PostGIS tests for constraints, precise time boundaries, query plans, migrations, rollback, and backup/restore. Confirm that restore produces the same authorized API resources and no unintended command dispatch. Test deletion with retained internal evidence and with subsequent stale synchronization input.

### 4.8 Publication, live delivery, and experimental Part 3

**Required behavior and source.** Support dynamic exchange and the planned experimental Part 3 work, preserving timing, identity, and honest recovery behavior. Particular transports and recovery interfaces below are Glaux choices. [IDR-035][R035], [P3], Goal §5.4

**Implementation.** A database outbox stores committed changes with stable event ID, resource ID/revision, kind, relevant time, and a reference to the immutable payload/revision. A worker copies or projects them into a retained publication log and records that handoff transactionally. Serialize allocation of replay position, log insertion, and outbox handoff through transaction commit so a visible replay position cannot skip an earlier uncommitted entry. Do not use an unconstrained database sequence as proof of commit order.

The worker publishes from committed log records with bounded concurrency, retry/backoff, and a finite retention policy. Delivery is at least once: a crash after send but before recording acknowledgement can repeat a message. Consumers need stable identity and revision information. A resource deletion is an event, not a normal native data record with invented null contents. Public lifecycle notifications, native observations, System Events, command status, and private diagnostics remain distinct categories.

**SSE extension.** Use `GET /extensions/glaux/events?scope=...`, disabled unless configured, with `text/event-stream` output. Require one URL-encoded API-root-relative resource or collection path from the enabled route allowlist, such as `/datastreams/{id}/observations`. Scope means that resource or the direct members of that collection, not arbitrary URL fetching, recursive traversal or CQL2 evaluation. Reject duplicate/unknown selectors and paths containing an authority, query or fragment. Authorize the selected scope and each emitted event, using retained access context for deletions.

Authenticate with ordinary HTTP credentials. Browser examples use authenticated fetch and a conformant SSE parser; native EventSource does not offer an arbitrary Authorization-header option, and tokens must not be put in URLs. Send resource notifications as `event: resource` with the CloudEvents JSON below, and `event: checkpoint` with `{}` at stream start and a configured fixed cadence. Checkpoints do not disclose hidden change counts. A client requesting full state uses §4.11's snapshot/catch-up contract; subscribing alone starts at the current committed log boundary and does not promise historical completeness. [SSE]

SSE `id` is an opaque authenticated-encrypted cursor binding recovery epoch, position, caller, exact scope, policy/mapping version and expiry. Accept it only through `Last-Event-ID`; no unauthenticated plaintext sequence or bearer credential is embedded. Preserve replay order within a subscriber, and advance a cursor only after all eligible earlier events have been written to that stream. This is not proof that the application processed them. Malformed/tampered or mismatched-scope cursors receive `400`; expired/retired recovery context, including a pre-restore epoch, receives `410` before streaming, with fresh-snapshot guidance; revoked authority receives the ordinary denial. Recheck authority during delivery and close on expiry/revocation. Bound subscriber buffers and disconnect slow consumers so they can resume or resnapshot. SSE remains a Glaux interface, not a Part 3 binding.

**MQTT experiment.** Adopt an optional, disabled-by-default outbound MQTT 5 adapter called `glaux-csapi-part3-exp/0.1`, based on the pinned draft. Cover Resource Events and native Observation/status-observation and System Event data first; describe any additional supported resource data precisely. Initial choices are QoS 1, non-retained messages, and these topic families:

```text
{prefix}/glaux-csapi-part3-exp/0.1/events/{canonical-relative-resource-path}
{prefix}/glaux-csapi-part3-exp/0.1/data/{canonical-relative-collection-path}/{format-name}
```

The names, path encoding and delivery rules are Glaux's experimental binding. Configure `prefix` as nonempty slash-separated ASCII letters/digits/`_`/`-` segments; disallow empty segments, wildcards and leading/trailing slash. Canonical relative paths omit the leading slash and use server-generated canonical IDs and route literals, not arbitrary names, UIDs, query text or nested aliases. Never substitute percent-decoding or wildcard matching for resource identity. Format suffixes are `json`, `swe-json`, `swe-text`, `swe-binary` for the corresponding enabled data media types; System Event data initially uses `json`. Do not adopt the draft's illustrative `om-json`/`cmd-json` media names as approved Part 2 encodings. [P3]

Use CloudEvents 1.0.2 structured JSON, `application/cloudevents+json`, for resource events. `source` is the configured API root, `subject` the canonical resource URL, `id` the stable event identity, and `time` the operation time. Emit `org.ogc.api.consys.<token>.<operation>` with `create`, `update` or `delete`. Tokens are `system`, `deployment`, `procedure`, `samplingfeature`, `property`, `datastream`, `observation`, `controlstream`, `command`, `commandstatus`, `commandresult`, `systemevent`; use `system`/`deployment` for canonical subsystem/subdeployment identities. Explicit Glaux additions for the draft's unfinished token mapping are `feasibility`, `feasibilitystatus`, `feasibilityresult`, and `collection`. Include authorized `parentid` where applicable, correcting the draft's uppercase spelling; omit optional `data` summaries in this binding. This avoids accidental payload disclosure but does not make event existence public. [CloudEvents], [P3]

Generate AsyncAPI 3.0 with enabled channels, MQTT 5 server/authentication, publish direction and exact message schemas. For example, the `events/observations/{id}` channel has CloudEvents content while `data/datastreams/{id}/observations/json` has native observation JSON; both use the configured prefix/binding segment. Describe actual path parameters and QoS/retain settings, not an unbounded wildcard as an authorization promise. Link the authenticated `/extensions/glaux/asyncapi` description using extension relation `urn:glaux:rel:experimental-asyncapi`, never a fabricated OGC relation. Select `rumqttc`'s MQTT 5 interface and Eclipse Mosquitto as the reference integration-test broker; pin tested crate/image versions during implementation. These choices have not been compiled or run in this drafting iteration. [MQTTClient], [Broker]

An MQTT topic is a disclosure boundary. Enable it only when broker permissions and topic partitioning can enforce the intended recipient access; do not publish a broadly visible topic and rely on consumers to filter protected data. Wildcards, credential expiry, policy changes, queued delivery, and topic metadata require tests. Broker acceptance never establishes recipient processing. MQTT retained messages or persistent sessions alone do not provide synchronization completeness.

Use separate publisher/subscriber credentials and TLS for non-loopback examples. The reference deployment permits subscribers only where the whole topic has one enforceable authorized audience; otherwise do not enable that topic. Use zero session expiry for subscriber examples to avoid offline protected queues, and test disconnect/revocation of already connected subscribers. Deployments retaining sessions need explicit queue-expiry/revocation enforcement. Native MQTT data is live delivery without an application recovery cursor in this version; consumers recover through authorized HTTP/snapshot exchange, not a claim of gap-free MQTT replay. QoS 1 alone does not establish recovery completeness. [MQTT]

Batch Resource Events, inbound broker resource writes, and inbound command submission are not part of this initial experimental binding. They are not removed from a published Part 3 completion target—there is no approved Part 3 target yet—but require a revised experimental design if adopted. A broker is required only when running this adapter; it is not required for ordinary CSAPI HTTP or SSE.

**Verification.** Interrupt the broker and worker before/after send; verify durable committed events and retry within retained history, allowing duplicates. Disconnected live MQTT recipients use the documented HTTP/snapshot recovery path; broker acknowledgement is not receipt by every subscriber. Test payload/schema consistency, deletion events, ordering within the documented scope, cursor expiry, snapshot/catch-up, slow consumers, authorization changes, and agreement between AsyncAPI and actual topics. Do not claim global device-event order or exactly-once delivery.

### 4.9 Commands, feasibility, status, and results

**Required behavior and source.** Implement ControlStreams and their schemas, Command resources, synchronous/asynchronous processing, status/result resources, and Feasibility requests with their own status/results. Feasibility assesses an action; it does not authorize or execute it. [S2], [IDR-036–038][R036]

**Implementation.** Persist immutable command intent/revisions and an adapter-work record before dispatch. Validate parameters against the selected ControlStream schema, derive the submitting identity from authenticated context, authorize the action and target, and record which contract/configuration applies. A submitter cannot impersonate another sender by writing a field. Keep submission permission separate from permission to report device status/results.

Support exactly these public command status codes: `PENDING`, `ACCEPTED`, `REJECTED`, `SCHEDULED`, `UPDATED`, `CANCELED`, `EXECUTING`, `COMPLETED`, `FAILED`. Apply the standard's status-specific progress/execution-time rules. Keep delivery timeout and uncertain physical outcome as internal delivery evidence or clearly identified metadata; do not invent an additional standard status value.

For synchronous processing, return one terminal status report (`COMPLETED`, `REJECTED`, or `FAILED`) after the adapter finishes within a configured bound. Asynchronous processing exposes durable status reports and results. Section 6.4 specifies the POST interpretation, including recovery when the synchronous waiting bound is exceeded. A lost connection or elapsed timeout does not justify a fabricated `FAILED` physical outcome or an automatic switch of a synchronous contract to asynchronous behavior.

The adapter interface covers capability/availability, command submission, supported cancellation/update, feasibility evaluation, and reporting status/results. Each attempt carries a stable command/attempt identity. Recheck relevant policy, configuration/schema, and deadlines immediately before dispatch. If a crash makes the external effect uncertain, reconcile with the adapter/device; do not resend a non-idempotent action merely to obtain certainty. A deterministic fake device adapter supplies reproducible reference examples and tests; production device adapters implement their own protocols and interlocks behind the same boundary.

Cancellation follows the command-status behavior, not HTTP DELETE. A `CANCELED` report must reflect authorized, effective cancellation under the adapter contract; merely requesting cancellation is insufficient evidence that actuation stopped. Similarly, deleting a status/result resource does not erase an already executed physical effect. Required public CRUD/PATCH operations use internal revision history so corrections remain accountable. Late reports must not blindly overwrite a newer terminal state; apply the adapter's sequence and permitted-transition rules and retain unresolved contradictory evidence privately.

Keep execution state distinct from editable public report records. Authorized correction/deletion of report content does not reopen a terminal execution, regress `currentStatus` to an earlier `EXECUTING` report, or enqueue another dispatch. Derive execution/current-status facts from retained authoritative lifecycle evidence, not simply the last remaining public report. Allow valid report/result edits, but reject an attempted forbidden lifecycle transition and retain contradictory outcome evidence privately. This is Glaux's explicit reconciliation of terminal-status rules with required report CRUD/PATCH, not removal of those operations. [S2 §§10.11, 14.6, 15.6][S2]

Feasibility reuses the command parameter schema and has its own result schema and status/result resources. Do not use `SCHEDULED` or `UPDATED` for feasibility. A `COMPLETED` feasibility analysis can report that an action is infeasible; analysis success and action feasibility are different facts. Any validity window or conditions are represented according to that ControlStream's result contract, not a mandatory invented universal feasibility object.

**Verification.** Exercise synchronous and asynchronous success/rejection/failure, feasibility-negative results, required time fields, forbidden feasibility statuses, unauthorized reporters, duplicate submission, competing changes, late reports, cancellation versus deletion, and all required status/result write operations. Crash at each dispatch boundary and demonstrate that neither HTTP nor MQTT acknowledgement is treated as execution proof.

### 4.10 Authentication, authorization, trust, and audit

**Required behavior and source.** Enforce configured access rules, protect writes and tasking, support identity/policy integration, preserve accountability, and avoid protected-data disclosure. Organizations retain identity administration, policy ownership, release authority, and accreditation. [IDR-039][R039], [IDR-039A][R039a], [IDR-040–041][R040], Goal §5.7

**Implementation.** Define an authenticator producing a verified caller context and an access-policy interface deciding actions on resources. Use externally issued OAuth access tokens from a configured identity provider, including providers supporting OpenID Connect. For the selected JWT access-token adapter, check token type, signature and allowed algorithm, issuer, audience, validity, and applicable scopes; never treat decoding a token as authentication. Cache trusted issuer keys with a bounded refresh policy and fail safely when verification is unavailable. Do not use an ID token as an API access token. Use [RFC 9068](https://www.rfc-editor.org/rfc/rfc9068.html#section-4) for a provider using that access-token profile and [RFC 8725](https://www.rfc-editor.org/rfc/rfc8725.html) for JWT validation practices; other provider/token formats need an explicit adapter contract.

Provide explicit development identities for loopback-only examples and automated tests, clearly labeled and rejected by normal network-facing configuration. A read-only anonymous deployment is an explicit policy option, not a way to enable unauthenticated writes. Support TLS at the service or a documented trusted reverse proxy. Publish only configured browser origins; do not confuse CORS with access control.

Start with configured caller/group/source permissions and resource relationships. Authorize the action and scope before queries, counts, extents, links, schema disclosure, latest selection, or streaming delivery can reveal data. Restrict publishers to assigned Systems/streams, command submitters to permitted targets, and status reporters to the adapter/source authority they represent. Where a richer external policy service is integrated, document its input, result, timeout, and unavailable behavior; no network-facing operation defaults to allow on policy-service failure.

Prefer authorizing complete conformant resources over arbitrary field redaction. A protected field cannot simply be removed if that makes the advertised schema false. If partial disclosure is required, define a valid projection/profile or deny that representation. Schema documents, command capabilities, relationship links, original artifacts, error details, and replay history receive the same protection as ordinary data.

Record durable audit information for meaningful mutations and command attempts: verified actor/source, operation, target/revision, time, outcome, and correlation identifier. Record relevant denials safely without turning every denied request into unbounded database load. Do not log secrets, bearer tokens, raw sensitive payloads, or protected policy reasons. Diagnostic logs may be sampled; committed write/command accountability cannot depend on them. A mandatory tamper-proof ledger, enterprise security platform, or universal trust score is not proposed.

Preserve production context separately from ingestion/audit history. Where supplied, bind an observation or result component to the producing System and Procedure, the actual method/model/calibration/configuration revision, exact antecedent observation revisions or source artifacts, creation time and asserted responsible roles. A present-day Procedure description is not proof of the configuration that actually ran. Keep missing context unknown; do not reconstruct exact inputs from current links or invent human participants. Existing standard fields and permitted extensions carry public content; typed internal references and retained source artifacts preserve context not covered by a standardized public mapping. No new mandatory observation members are imposed. [R061]

Record authenticated uploader, asserted producer/creator, method author and custodian as different roles. Retain the source and verification state of an asserted delegation or responsible organization; authentication alone does not validate that assertion. Never infer a PROV delegation solely because software processed an observation. Human accountability can be represented when known without making an invented human relationship a condition for every sensor measurement.

Preserve each quality assertion's subject (including the particular result component), metric, value, units/scale and supplied method/version, evaluator and coverage conditions. Identification confidence, property-value uncertainty, likelihood and a statistical confidence level are not interchangeable labels. Do not normalize them into a generic truth probability or combine scores merely because they share a 0–1 scale. Antecedent overlap can matter to a downstream analysis; retaining input relationships does not promise automatic uncertainty fusion. Shared batch/compound-result metadata applies only where genuinely common; preserve per-item exceptions.

Authorize provenance relationships, exact inputs, roles and quality context independently of the result they describe. No hidden contributor, relationship count or source artifact may leak through links, schemas, query NULL behavior, errors, exports or event topics. If a valid authorized projection cannot be produced, deny the representation. Preserve supplied labels/binding evidence without claiming that a transformed representation retains the original signature's validity. IC-EDH/ISM or STANAG 4774/4778 integration would require an applicable authoritative policy and concrete binding profile; neither a universal per-JSON-field marking rule nor permission to remove required markings follows from this Guide. No national/NATO labeling adapter is selected here. [R040], [R061]

**Verification.** Use an access matrix over resource families and actions, including indirect links, collections, schemas, errors, status/results, and events. Test expired/wrong-audience tokens, revoked permissions, malicious references, SQL injection, parser exhaustion, cross-source writes, prohibited tasking, and identity/policy outages. Review supported authentication dependencies against current primary security guidance when pinned.

### 4.11 Interrupted connectivity, replay, synchronization, and conflicts

**Required behavior and source.** Preserve source identity and time, keep useful local behavior during interruption, handle repeat/delayed input consistently, detect conflicts, and distinguish last-known evidence. Do not assume continuous network connectivity or supply deployment federation infrastructure. [IDR-042–043][R042], Goal §5.8

**Implementation.** Keep schemas, configuration, permitted vocabulary data, and stored resources locally usable. Local authorized reads remain possible when a remote publisher/broker is disconnected. Credential/policy validity still bounds access; offline operation does not extend expired authority indefinitely. A missing remote source affects freshness/availability, not the truth of already retained historical records.

For controlled exchange, begin with an administrative export/import adapter for configured peers or authorized files, not a universal federation API. Export canonical identities/source IDs, resource revisions, source and semantic times, schema references, relevant relationships, and deletion markers. Include a source-scoped exchange identity and digest so the receiver can distinguish an exact replay from different content under the same identity. This envelope is a Glaux integration format, not a new CSAPI representation.

Use one versioned JSON manifest with bounded `records` for the initial adapter; no archive extraction or arbitrary remote payload fetching. Its contract is:

| Field | Meaning and validation |
|---|---|
| `format`, `exchangeId`, `source`, `sourceEpoch` | Exact `glaux-exchange/1`; unique exchange ID; configured source identity and continuity epoch. Claimed identities must match the authenticated/authorized import context |
| `recipient`, `scope`, `scopeVersion` | Configured recipient and exact peer-owned resource scope/access version; not a client-supplied permission grant |
| `mode`, `snapshotId`, `baseCursor`, `endCursor`, `complete` | `snapshot` or `changes`; snapshot identity where applicable; source-issued scoped continuity tokens; explicit completeness for this authorized scope only |
| `records` | Bounded array of resources, required schema/source artifacts and explicit tombstones; dependencies identify other records or existing accepted revisions |
| Record identity/revision | `kind`, `sourceId`, `revision`, `predecessor`, `dependencies`, `ancestry`; revision is a source-epoch-scoped monotonically increasing integer encoded as a decimal string, with explicit predecessor identity and bounded overlap evidence |
| Record content | `operation` (`upsert`/`delete`), `mediaType`, `payloadBase64`, `sha256`; digest checks exact decoded bytes, not semantic equivalence. Delete records have no payload and retain identity/revision/deletion evidence |

Payloads preserve standard resource values/times and applicable supplied context; manifest relationships retain source parent/schema IDs for explicit local mapping. Cap encoded and decoded sizes before allocation. Base64 is file packaging, not a new public observation encoding. Retain the exact manifest bytes and their digest in the exchange receipt; altered bytes under the same exchange identity conflict rather than being guessed equivalent. No JSON canonicalization/signature scheme or general multi-master protocol is implied. Generate JSON Schema and positive/negative fixtures for this contract in the implementation repository, not a separate planning document. [R043]

Exports include only records authorized for their recipient; imports re-evaluate local access and source authority rather than inheriting the sender's permission decision. A receiver validates the same resource semantics as HTTP, maps remote to local identities explicitly, and commits accepted changes with the exchange receipt in one transaction. Identical accepted input is a no-op returning the recorded outcome. Unknown dependencies stay in a restricted staging area or are rejected with a clear reason; they do not become partially valid public resources. A conflicting revision, UID collision, or attempted resurrection of a deleted resource is retained/reported without silently overwriting accepted state. Resolution is an authorized new operation with a recorded explanation.

Within a source epoch/resource chain, apply a successor only against its known predecessor; a revision number alone is not proof of ancestry. Retained known predecessors are harmless repeats, different content at the same revision is a conflict, and missing ancestry requires dependency recovery or a fresh snapshot. An explicitly admitted snapshot establishes a baseline only within its agreed scope; it cannot override independent local changes. New source epochs require fresh continuity establishment, not numeric comparison with an earlier epoch.

Snapshot `ancestry` supplies the revision/predecessor chain and immutable-content fingerprints needed to recognize every older record that can overlap subsequent replay, including changes still pending outbox handoff. Fingerprints bind operation, identity, relationships and contract metadata as well as payload bytes. An immediate predecessor alone is insufficient. Retain this evidence for the recovery window, enforce its authorization/size limits, and reject an export as incomplete if the required overlap cannot be established. A receiver verifies replayed content against that evidence rather than accepting any numerically smaller revision as harmless. This is bounded source-chain evidence, not a general provenance graph.

Provide administrative `exchange inspect` and `exchange resolve` operations over restricted staged conflicts. Resolution selects keep-local or accept-incoming, requires the expected current local revision and a reason, and records the chosen source candidate, new local revision and audit together. Revalidate authority and dependencies at resolution time. There is no general force-overwrite or arrival-time-wins command. Do not advance a completeness checkpoint past unresolved/rejected required records; retry/resolution or a new scoped snapshot must settle the gap.

For consistent export plus catch-up, read resource state and a committed publication-log position in one repeatable-read database snapshot and materialize a bounded export. Replay subsequent log records after import. Changes already present in the snapshot but still awaiting publication-log insertion may reappear. Include source-scoped revision ordering/ancestry, including deletion revisions, so replayed known predecessors are ignored rather than rolling back the imported state or creating false conflicts. This ordering must be tested so no post-snapshot change can be skipped. Do not pretend independent ordinary paginated HTTP reads form this snapshot. Limit export duration/size and expire exports explicitly.

Administrative exchange tokens are not interchangeable with SSE `Last-Event-ID`. When an authorized export is explicitly for one SSE caller and exact route scope, the exporter may additionally issue `sseResume` at that snapshot's log boundary, bound to the same caller, scope, policy/mapping version and expiry as §4.8. The exported view must match that caller's permitted subscription view. Otherwise no SSE bootstrap token is issued: use administrative catch-up for that exchange. There is no new public snapshot API or promise that ordinary paginated HTTP reads are an atomic SSE bootstrap. [PostgreSQL isolation][PGIsolation]

Retain tombstones and exchange receipts for the configured recovery window; if a peer is older than that window, require a new snapshot rather than guessing missing history. A replacement snapshot must reconcile the agreed peer-owned scope, including absent/deleted members, or initialize a fresh replica; simply adding the currently present rows cannot remove stale replicas. Preserve locally changed contenders as conflicts. Never infer conflict resolution from arrival order, largest UUID, or unsynchronized node clocks. Do not synchronize queued command intent into automatic dispatch. Imported command history is evidence; new physical action requires the local authorization and dispatch process.

Absence implies deletion only for an explicitly complete authoritative replacement snapshot with unchanged, agreed ownership/access scope. A policy-filtered omission or changed scope is not a tombstone. Require fresh scope agreement or a fresh replica when that distinction cannot be established. Configure finite manifest/record/decoded-byte, duration, staging, replay and queue limits, plus an explicit recovery-retention window. Startup rejects missing/unbounded settings for an enabled adapter; the runnable example selects and measures safe values during implementation. A size limit must cause an explicit incomplete/rejected export, never a falsely complete snapshot.

**Verification.** Disconnect sources, send delayed observations, replay identical input, change the input under a reused identity, conflict two revisions, and attempt resurrection after deletion. Interrupt import/export and log handoff at transaction boundaries. Test a peer beyond retention and a command whose external effect is unknown. Demonstrate correct recovery without automatic unsafe actuation.

### 4.12 Configuration, deployment, observability, and developer use

**Required behavior and source.** A developer can build, configure, run, test, and exercise the server without completing other Glaux applications. Supply operational reference behavior, not managed production operations. [IDR-046–049][R046], Goal §§6–8

**Implementation.** Deliver native Rust development instructions with PostgreSQL/PostGIS as a documented dependency and a Compose example for the complete local reference deployment. Use an explicit optional service group for the experimental broker. Do not install software on a user's machine as an implicit part of guide preparation or testing; document prerequisites and respect organizational installation policy.

Provide `serve`, `migrate`, `check-config`, and administrative sample-data/export/import operations. Their final flags belong in executable help and the server README when implemented. Document build, migration, sample loading, serving, tests, backup/restore, and disposal separately; no automatic reset of a persistent database. Sample loads use the validated application path.

Keep any contributor or coding-assistant instructions synchronized with the commands, conventions and architectural boundaries they describe. Update affected instructions in the same implementation issue as the change, and link explanations to the controlling standards and this Guide with current executable examples. This applies to guidance if present; it does not require an additional assistant-specific file, tool or documentation system. [IDR-062][R062]

Use typed configuration with unknown-key rejection and startup validation. Configure public origin, listeners, database, identity/policy integration, limits, optional adapters, and retention explicitly. Read secrets from protected files/environment references or deployment secret providers, not checked-in examples. Redact effective configuration diagnostics. Reject unsafe combinations such as public listeners with development authentication.

Expose liveness and readiness separately. Liveness reports whether the process can respond; readiness reports whether required storage/schema/configuration permits serving its declared capability. Optional broker failure can degrade publication without stopping valid historical reads. Report that condition through protected diagnostics, not false global readiness or a false System status.

Use structured logs and bounded-cardinality metrics for request latency/errors, database work, validation failures, queue depth/age, dropped/disconnected consumers, and command attempts. Avoid resource IDs or arbitrary query strings as metric labels. Supply a simple way to inspect them; an external dashboard or telemetry backend is optional.

Graceful shutdown stops new work, drains bounded in-flight operations, and leaves undelivered/uncertain work in a recoverable durable state. A restarted worker resumes only work whose retry semantics permit it. Pin build dependencies and container images, document licenses, and run dependency/security checks as normal implementation maintenance.

**Verification.** Follow the README from a clean supported environment with only documented prerequisites. Build and run, load a small synthetic dataset, exercise discovery/read/write/stream/tasking examples, restart, migrate, and restore. Test malformed config, missing secrets, unavailable database, broker outage, and clean shutdown without hidden data deletion.

## 5. Integration Points

| Consumer or dependency | Contract and boundary | Integration proof |
|---|---|---|
| Glaux Web App and Mobile | Same discoverable CSAPI HTTP interfaces as external clients; optional documented SSE extension | Root-to-observation/status workflow with ordinary credentials, including interrupted connection |
| Glaux Publisher | Standard description/stream/observation writes; configured source permissions; optional retry keys | Register source resources, publish schema-bound values, retry without duplicate effect when a key is supplied |
| Glaux Simulator | Standard APIs and isolated synthetic data; no privileged direct database mutation | Deterministic fixtures and command adapter tests work without the Simulator product |
| External CSAPI clients | Published routes, links, filters, schemas, formats, errors, and accurate declarations | OS4CSAPI and an independent Python client exercise the same known dataset |
| Identity and policy services | Verified access-token/caller contract and explicit access-decision adapter | Allowed/denied/expired/unavailable cases, including disconnected operation |
| Device/tasking adapters | Typed command, attempt, schema, deadline, cancellation, and report interfaces (§4.9) | Fake adapter plus documented adapter contract; real device acceptance remains adapter-specific |
| MQTT broker | Optional outbound experimental binding and broker access policy (§4.8) | Reconnection, duplicate delivery, topic authorization, and message/schema checks |
| Configured peers/import tools | Restricted Glaux export/import format, source mapping, replay/conflict rules (§4.11) | Snapshot plus replay and explicit conflicts without bypassing the write boundary |
| PostgreSQL/PostGIS | Versioned schema, transactions, migrations, backup/restore | Real-database integration and continuity tests |

No Glaux application receives a hidden bypass around the CSAPI contract for an operation the standard covers. A later specialized integration must state what the standard interface cannot supply and document the additional contract here before relying on it. Deployments select and operate peer networks, brokers, identity providers, and devices; the reference server supplies tested boundaries.

## 6. Data and API Contracts

### 6.1 Logical data model and storage ownership

These are logical storage groups and invariants, not claims that migration SQL has been written. Final columns and indexes will be verified during implementation; table layout may be refined without weakening these design commitments.

| Logical group | Key data and relationships | Integrity and update rules |
|---|---|---|
| Resource identity | Local ID, family, UID where required, lifecycle, current revision | Unique local identity; unique required UID; IDs are not reused |
| Descriptions and revisions | Family-specific validated content, valid time, source artifact, commit/receipt metadata | Changes create accountable revisions; one selected current representation per defined context |
| Associations | Typed System/Deployment hierarchy, procedure, sampling, feature, and membership links | Foreign keys for local targets; checked external references; cardinality/cycle rules |
| Collections | Metadata, resource type, explicit membership or named server-managed query | Same member identity across routes; authorized membership and counts |
| Stream descriptions | Producing/receiving System, output/input binding, schema references, supported formats | Schema capabilities determine value validation; clients cannot invent supported codecs |
| Schema contracts | Immutable source bytes/digest, typed component tree, encoding, version | Values bind a specific contract; no mutation of the meaning of stored values |
| Observations | ID, stream/contract, phenomenon/result times, feature context, typed result, quality/source | Validated values; correction/replacement retains needed internal evidence |
| System Events | ID, one parent System, occurrence time, type, content/source | Distinct from resource changes and transport notifications |
| Commands and feasibility | ID, ControlStream/contract, caller, submitted intent, processing mode | Accepted intent/attempts and public resource revisions are distinguishable |
| Status and results | Parent command/feasibility, local item ID, report/execution times, status or result content | Public CRUD/PATCH with authorized corrections; independent retained execution evidence prevents deletion/correction reopening terminal work |
| Source artifacts | Digest, media type, exact bounded bytes, source metadata | No assumption that JSONB equals original bytes; access and retention follow content |
| Production context | Supplied producer/method revision, exact input/artifact references, responsible-role assertions, quality subject/method | Bind to observation/component/revision; preserve unknowns and per-item distinctions; independently authorize disclosure |
| Server audit | Verified actor/source, operation, target/revision, receipt/commit times, outcome, transform references | Written with meaningful mutations; uploader is not automatically creator; no raw secrets or generic graph requirement |
| Outbox, publication log, work | Stable event/attempt IDs, committed payload/revision, replay position, delivery state | Commit before send; idempotent handoff; no automatic duplicate physical action |
| Retry/exchange records and tombstones | Scoped identity/digest, saved outcome, source/revision, deletion marker | Duplicate versus conflict distinction; bounded retention and explicit recovery expiry |

Do not use a single untyped resource table as the entire model. Shared identity/revision storage may be common, while family-specific tables and Rust types enforce different relationships and mutation rules. Public extension fields belong only in allowed/advertised representations; internal policy, audit, and retry records are not new CSAPI families.

### 6.2 Endpoint and representation baseline

Paths below are relative to the configured API root. They summarize the full intended contract, not a declaration that routes are already implemented. Implement applicable GET/list and transaction operations from the relevant classes; do not mechanically enable every HTTP method on every row. [IDR-010][R010], [IDR-012][R012]

| Surface | Canonical paths and related operations | Representation |
|---|---|---|
| Service documents | `/`, `/conformance`, linked API definition/documentation | JSON service documents; OpenAPI JSON and documentation |
| Collections | `/collections`, `/collections/{id}`, `/collections/{id}/items`, `/collections/{id}/items/{resourceId}` | Family-appropriate collection/item encoding |
| Systems | `/systems`, `/systems/{id}`; `/systems/{id}/subsystems` | GeoJSON and SensorML JSON |
| Deployments | `/deployments`, `/deployments/{id}`; `/deployments/{id}/subdeployments`; System deployment association where provided | GeoJSON and SensorML JSON |
| Procedures | `/procedures`, `/procedures/{id}` | GeoJSON without position geometry; SensorML JSON |
| Sampling Features | `/samplingFeatures`, `/samplingFeatures/{id}`; `/systems/{id}/samplingFeatures` | GeoJSON |
| Property Definitions | `/properties`, `/properties/{id}` | SensorML JSON; not a GeoJSON Feature |
| DataStreams | `/datastreams`, `/datastreams/{id}`; applicable System/Deployment lists; schema and sampling-feature/feature-of-interest associations | Ordinary JSON; schema wrappers in JSON |
| Observations | `/observations`, `/observations/{id}`; `/datastreams/{id}/observations` and transaction-qualified nested items | Ordinary JSON or supported SWE JSON/Text/Binary |
| ControlStreams | `/controlstreams`, `/controlstreams/{id}`; applicable System/Deployment lists; schema and sampling-feature/feature-of-interest associations | Ordinary JSON; schema wrappers in JSON |
| Commands | `/commands`, `/commands/{id}`; `/controlstreams/{id}/commands`; `/commands/{id}/status` and `/result`, with subordinate items | Ordinary JSON or applicable SWE command encoding; JSON status/results |
| Feasibility | `/feasibility`, `/feasibility/{id}`; `/controlstreams/{id}/feasibility`; `/feasibility/{id}/status` and `/result`, with subordinate items | Command-shaped parameters and schema-defined feasibility result |
| System Events | `/systemEvents`, `/systemEvents/{id}`; `/systems/{id}/events`, with transaction-qualified nested items | Published JSON schema with the documented mapping (§13) |

Subsystems and subdeployments have canonical System/Deployment identities. Root lists use the standard top-level/default-recursion behavior; nested lists default to direct children. Required recursive associations also aggregate applicable descendants' streams/features, not only the immediate parent's own records. Do not invent nested item routes merely by appending an ID where the standard instead supplies a canonical link.

Media types are `application/geo+json`, `application/sml+json`, `application/json`, `application/swe+json`, `application/swe+text`, and `application/swe+binary` for their applicable resource/operation combinations. Text is not silently renamed CSV. The SWE vendor-prefixed conflict and alias treatment are in §13.

`Accept` negotiates the returned representation; request `Content-Type` identifies the submitted body. Honor quality values, exclusions, parameters, and wildcards. Missing `Accept` uses a documented family default: GeoJSON for the applicable Part 1 features, SensorML JSON for Properties, and ordinary JSON for Part 2 where supported. A parent stream's actual formats constrain value responses. Use appropriate `Vary` headers and representation-specific ETags. Return `406` or `415` when negotiation/request format is unsupported; do not return a different explicit format silently.

The schema routes are `/datastreams/{id}/schema?obsFormat=...` and `/controlstreams/{id}/schema?cmdFormat=...`; media-type query values must be URL encoded. The selected payload format and the JSON schema-wrapper response are different concepts. Do not add the previously optional `f` format selector in this baseline: use standard content negotiation and the specified schema selectors. This removes an unnecessary second negotiation path, not a CSAPI capability.

`cmdFormat` is optional. When omitted, select the first actually supported format for that ControlStream/revision in this Glaux preference order: `application/json`, `application/swe+json`, `application/swe+text`, `application/swe+binary`. Return one JSON schema-wrapper document with `commandFormat` identifying the selected payload format. The preference order is not an OGC rule. Explicitly unsupported `cmdFormat` is a `400` query error, while unacceptable wrapper `Accept` is `406`; omission alone is not an error. Missing required `obsFormat` remains `400`. Do not advertise a stream with no usable schema/format. [S2 requirement 25 and §16][S2]

### 6.3 Query rules and limits

| Query family | Required treatment |
|---|---|
| `limit` and next links | Publish minimum/default/maximum; use `1/10/10000` as the reference configuration from IDR-011, clamp a valid above-maximum integer, reject malformed/below-minimum values, and return no more than the effective limit. Other cost/byte limits still apply |
| `id`, `q` | Implement the specified local-ID/UID/list/prefix and keyword grammar on applicable routes. Do not invent a separate standard `uid` parameter. Preserve source identifiers |
| `bbox`, `geom`, `datetime` | Preserve the spatial/temporal and geometry-less-feature rules; use the correct family-specific temporal field |
| `recursive`, `parent`, `system`, `procedure`, `foi` | Apply where the resource class requires them; preserve direct-parent versus descendant and relationship semantics |
| `observedProperty`, `controlledProperty`, `baseProperty`, `objectType` | Use explicit property/object-type identity and required derivation/association traversal, not label/unit similarity; include the Property Definition `objectType` filter required by Part 1 requirement 58 |
| `phenomenonTime`, `resultTime` | Observation values or stream extents according to endpoint; special `resultTime=latest` only where specified |
| `issueTime`, `executionTime`, `statusCode`, `sender` | Command/ControlStream and applicable feasibility/status predicates; `currentStatus` is not the query parameter name |
| `eventType` and status/event `datetime` | Event type and relevant occurrence/report time, with published ambiguities documented in §13 |
| `filter`, `filter-lang`, `filter-crs` | Additional observation-only Features Part 3/CQL2 contract in §6.3.1; combine with native predicates rather than replacing them |

Keep a typed parameter definition per applicable route with its syntax, default, predicate, and source requirement. Public examples and OpenAPI must use that definition. Bound list sizes, hierarchy traversal, WKT/parser work, total response bytes, database execution time, and concurrent work independently of `limit`. A query that exceeds a documented resource budget fails explicitly; it does not silently drop predicates or truncate a record. A smaller valid page can carry a continuation.

Use a documented Unicode case-insensitive matching rule for `q` that satisfies the published requirement; pin its library/data behavior and test non-ASCII text. The exact normalization implementation is not yet verified. No arbitrary relevance ranking, client sort contract, or synonym ontology is implied.

#### 6.3.1 Enhanced observation filter contract

Support GET filtering on `/observations` and `/datastreams/{id}/observations`. Equivalent observation collection views may advertise the same capability only with matching scope and queryable mappings. Do not silently enable the language on every metadata, command or event endpoint. The following paths and property names are Glaux's CSAPI mapping, not standardized CSAPI additions:

| Filter scope | Linked queryables document | Declared searchable values |
|---|---|---|
| All authorized observations | `/observations/queryables` | `id`: observation local-ID string; `samplingGeometry`: §4.4.1's direct, time-applicable geometry |
| One authorized datastream | `/datastreams/{id}/observations/queryables` | The common values above plus selected `result.<name>` scalar aliases bound to that stream's contract |

Each `result.<name>` is a complete advertised property identifier mapped explicitly to one scalar component path, not a dotted join/traversal syntax. Record its type, property definition, unit, missing/nil mapping and applicable contract revisions in the queryables description and API examples. Keep mappings stable across compatible revisions; do not reinterpret historical values after schema changes or expose a universal cross-stream `resultValue`. Unsupported scalar mappings remain absent from discovery, without making otherwise valid observations unstoreable.

Return queryables as JSON Schema Draft 2020-12 with `application/schema+json` and `additionalProperties:false`. Use the prescribed spatial `format` representation without imposing a JSON `type` or `$ref` on the geometry property. Provide the `http://www.opengis.net/def/rel/ogc/1.0/queryables` link and an HTTP `Link` header on applicable filterable resource responses (the header is recommended in the prose and checked by the Queryables ATS). Protect per-stream schemas and queryables like the data contracts they disclose. Do not confuse this document with the stream's SWE `resultSchema`.

Illustrative queryables document; the host, `ds-temp`, property URI and revisions are fixtures, not existing resources. The deployed `$id` is the absolute queryables URL without query parameters:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.org/api/datastreams/ds-temp/observations/queryables",
  "title": "Observation queryables for ds-temp",
  "type": "object",
  "properties": {
    "id": { "type": "string" },
    "samplingGeometry": {
      "format": "geometry-any",
      "description": "Direct geometry established for phenomenon time under Section 4.4.1."
    },
    "result.temperature": {
      "type": "number",
      "description": "Temperature component, definition urn:example:air-temperature, unit Cel, contract revisions r1/r2. Missing, nil and non-finite values are NULL in this view."
    }
  },
  "additionalProperties": false
}
```

Accept a URL-encoded CQL2 JSON expression in `filter`; advertise `cql2-json` as both the only initial language and its default. `filter-lang=cql2-text` is unsupported. Accept the standard CRS84/CRS84h filter-coordinate behavior and corresponding identifiers; reject other requested CRSs in this initial binding. Validate operators, argument counts, types, calendar values and geometry beyond generic JSON-schema acceptance. CQL2 timestamp literals use UTC `Z`; native CSAPI time parameters keep their own grammar. Unknown names, unsupported language/CRS/operators and malformed expressions return `400` through §6.4, not an ignored or partly evaluated filter. Authorization denials retain §4.10's independent policy behavior.

Implement Boolean literals, `and`/`or` (at least two arguments), unary `not`/`isNull`, binary `=`, `<>`, `<`, `>`, `<=`, `>=`, and the selected `s_intersects`. Adopt the permitted property-left/literal-right comparison restriction; no implicit casts, property-to-property comparisons, LIKE/IN/BETWEEN, arithmetic or additional function classes. Validate the complete expression even on empty data or inside a short-circuited branch. Check calendar dates, finite numeric literals, coordinates and geometry semantics beyond schema pattern matching. A Count threshold such as `3.5` is a numeric comparison, not an instruction to round; a threshold need not be a valid measured value within the component's declared range. Bare NaN/Infinity is invalid JSON; quoted `"NaN"` is a string, not a number. Use `isNull`, not an invented JSON-null comparison literal. Preserve CQL2 three-valued logic: comparison with unavailable data and its negation remain NULL; only TRUE selects. [CQL2], [CQLSchema]

Authorize all requested queryables and required relationships before evaluation, including those inside `OR true` or `isNull`. When protected context prevents a permitted evaluation, use a consistent scope-level denial/concealment policy, not row-dependent NULL or an error contingent on a hidden value.

Example decoded expression for a stream advertising `result.temperature` in Cel; the real GET request URL-encodes the expression and sets `filter-lang=cql2-json`:

```json
{
  "op": "and",
  "args": [
    {
      "op": "s_intersects",
      "args": [
        { "property": "samplingGeometry" },
        { "type": "Polygon", "coordinates": [[[0,0],[2,0],[2,2],[0,2],[0,0]]] }
      ]
    },
    { "op": ">", "args": [{ "property": "result.temperature" }, 25] }
  ]
}
```

No POST search route is introduced. Ordinary native `foi`/time filters can accompany this expression, with AND semantics and the evaluation order in §4.4.1. Publish finite expression-byte/depth, geometry-complexity and database-work limits, independently of page size. Exhaustion fails safely without partial-success claims; a page limit or outer SQL LIMIT is not a traversal-work safeguard. Exact tested limit values remain implementation measurements rather than invented performance guarantees. [F3 §§6, 8][F3], [CQL2 §§6–8][CQL2], [IDR-059][R059]

### 6.4 Success, error, and command response contracts

Use RFC 9457 Problem Details (`application/problem+json`) for ordinary HTTP errors where compatible with the applicable standard. Give problems stable documented type identifiers, safe detail, and a request correlation value. HTTP failures remain separate from an otherwise successful command-status response reporting domain rejection/failure. [IDR-013][R013]

| Condition | Response behavior |
|---|---|
| Resource creation committed | `201` and `Location` identifying the created canonical resource; body per operation contract |
| Replace/patch/delete completed | `200` with the specified representation or `204` without a body, according to the documented operation |
| Malformed request/query or prescribed parent-schema failure | `400`; no mutation |
| Missing/invalid credentials; authenticated denial | `401` with applicable challenge; `403`, or consistent non-disclosure `404` where policy requires |
| Absent resource; method unsupported on existing route | `404`; `405` with `Allow` respectively |
| Unsatisfied response media preference; unsupported request media/coding | `406`; `415` respectively |
| Conflicting identity, state, or reused retry key | `409`, without protected competing values |
| Failed supplied HTTP precondition | `412`; missing `If-Match` is not an automatic error in this baseline |
| Request too large; semantically invalid body not governed by a specific `400`/`409` | `413`; `422` only for the latter bounded case |
| Rate/capacity limit; temporary dependency failure | `429` or `503` as applicable, with safe retry guidance |
| Unexpected server/serialization failure | `500`; no stack trace, SQL, secrets, or falsely successful response |

**Command/feasibility POST interpretation.** Select `201` plus canonical Command/Feasibility `Location`, a CommandStatus body, and `Content-Location` identifying the individual persisted status resource (for example `/commands/{id}/status/{statusId}`), not its collection. An asynchronous stream returns its atomically recorded initial `PENDING` report; a synchronous stream returns its single terminal report. This combination is an explicit Glaux interpretation, to describe in OpenAPI and verify with independent clients during implementation. `202` is not returned merely because a stream is asynchronous.

For synchronous work, retain a private durable admission/attempt record before dispatch, then publish the Command/Feasibility and its terminal status/results atomically when the authoritative outcome is known and the public parent has not been deleted. Do not expose temporary `PENDING` history or hold a database transaction across execution. Distinguish the HTTP waiting budget from the work's execution deadline. Disconnect or waiting-budget exhaustion neither cancels admitted work nor proves physical failure. If the server unexpectedly cannot obtain the synchronous outcome within its bound, return a safe `500` problem when still able to respond; use `504` only when actually acting as an upstream gateway. Retain the admission for reconciliation, describing uncertainty without a fabricated CommandStatus or rollback promise. `503` remains an overload/unavailability response, not an execution-status substitute. [RFC 9110 §15.6](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.6)

Serialize admission, schema mutation and public deletion through the same application transaction boundary. Private admissions pin their contract and prevent schema mutation while unresolved work depends on it, but do not veto an authorized cascade required by §4.6. If deletion precedes admission, admission fails. If deletion follows admission, retain private work/evidence for reconciliation, suppress later public resource/status resurrection, and apply the adapter's pre-dispatch authority checks to any work not yet sent. Deletion is not execution cancellation or proof of a physical outcome. If concurrent deletion prevents a synchronous success response before publication, return a safe `409` problem identifying the state conflict rather than `201` with nonexistent resources; keep the outcome/uncertainty privately. This exceptional response is a Glaux choice. Test the race through terminal publication, including already-dispatched work.

A same-key, same-intent retry reauthorizes disclosure and rejoins existing work within its own wait budget or returns the recorded outcome; it never dispatches again just because a response was lost. Do not expire/recycle its key while admitted work is unresolved; bounded retention applies after terminal resolution, subject to §4.7's restore limitations. Reconciliation establishes one terminal execution outcome and, unless public deletion has intervened, its one synchronous terminal report. Conflicting late callbacks remain restricted evidence, not a transition from `FAILED` to `COMPLETED`.

Unkeyed submissions remain permitted. After an ambiguous disconnect, matching payloads do not establish retry identity: a new POST may be a distinct action. Ordinary authorized discovery or operator reconciliation may help, but unique client recovery is not guaranteed and automatic resubmission is unsafe. No private jobs API or exactly-once physical-effect claim is introduced. The deterministic reference adapter exercises these boundaries; operational reconciliation remains adapter-specific. [S2 §§10.11, 11][S2], [R037]

### 6.5 Versioning and compatibility

Version server software independently of standards versions, schema-contract revisions, API documentation, database migrations, the Part 3/4 experiments and queryable mappings. Keep ordinary canonical URLs stable. Track breaking changes to request syntax, default behavior, query meaning, representations, and errors as well as Rust types. [IDR-010A][R010a]

Before a stable contract changes incompatibly, document the affected behavior, reason, migration path, and any required overlap period. Do not adopt the research's proposed calendar deprecation duration as a new project obligation without an actual release-support decision. Preserve supported older client behavior through regression tests. Unknown draft aliases are not accepted automatically, particularly for writes or tasking.

## 7. Conformance and Verification Strategy

### 7.1 Full direct class coverage

Each suffix below is appended to the appropriate Part 1 or Part 2 `/conf/` base in §1.2. Source requirement ranges are from [IDR-006][R006], [IDR-007][R007], and [IDR-008][R008]; the published text remains authoritative. This table connects the entire target to implementation and proof without duplicating the standards' requirement text.

| Part | Class suffix | Direct requirements | Implementation and primary proof |
|---|---|---|---|
| 1 | `api-common` | 1–3 | §§4.1–4.2, §6; identity, documents, inherited HTTP/time behavior |
| 1 | `system` | 4–8 | §4.2; System routes, links, collections, representations |
| 1 | `subsystem` | 9–13 | §4.2, §6.2; hierarchy, recursion, descendant associations |
| 1 | `deployment` | 14–18 | §4.2; Deployment identity, context, and System relationships |
| 1 | `subdeployment` | 19–23 | §4.2, §6.2; nested/recursive deployment behavior |
| 1 | `procedure` | 24–28 | §§4.2–4.3; procedure identity and non-positioned descriptions |
| 1 | `sf` | 29–33 | §§4.2, 4.5; Sampling Features, parent links, dynamic context |
| 1 | `property` | 34–37 | §§4.2–4.3; Property Definitions and semantic relationships |
| 1 | `advanced-filtering` | 38–59 | §4.4, §6.3; exact result-set tests across applicable routes |
| 1 | `create-replace-delete` | 60–71 | §4.6; transactions, hierarchy deletion, collection membership |
| 1 | `update` | 72–76 | §4.6; patch result validation, atomicity, and restrictions |
| 1 | `geojson` | 77–88 | §4.3, §6.2; applicable GeoJSON reads/writes and mappings |
| 1 | `sensorml` | 89–103 | §4.3, §6.2; applicable SensorML reads/writes and inherited rules |
| 2 | `api-common` | 1–2 | §4.1, §6; inherited feature-to-resource adaptations |
| 2 | `datastream` | 3–16 | §§4.3–4.4; DataStreams, Observations, schemas, and associations |
| 2 | `controlstream` | 17–34 | §4.9; ControlStreams, Commands, status/results, schemas |
| 2 | `feasibility` | 35–39 | §4.9; parameters, execution modes, status/result and discovery |
| 2 | `system-event` | 40–44 | §4.5; event identity, System association, and representations |
| 2 | `advanced-filtering` | 45–62 | §4.4, §6.3; dynamic/command/event filters plus Part 1 dependency |
| 2 | `create-replace-delete` | 63–78 | §§4.6, 4.9; all triggered resource and subordinate operations |
| 2 | `update` | 79–92 | §§4.6, 4.9; all triggered patch operations and schema protection |
| 2 | `json` | 93–106 | §4.3, §6.2; ordinary JSON and SWE component/schema bindings |
| 2 | `swecommon-json` | 107–114 | §4.3; observation/command schemas and JSON value encoding |
| 2 | `swecommon-text` | 115–122 | §4.3; observation/command schemas and Text value encoding |
| 2 | `swecommon-binary` | 123–130 | §4.3; observation/command schemas and Binary value encoding |

The 25 direct classes contain 233 requirements and five recommendations; the accepted research identifies 240 direct abstract tests. These counts exclude inherited obligations. There are no separate approved `observation`, `command`, or `system-history` classes to add. Applicable inherited classes in §1.2 must also be tested; a direct class is not complete while a prerequisite is unimplemented. Recommendations remain recommendations unless explicitly selected as Glaux behavior.

### 7.2 Requirement-to-test connections

Keep the class/capability mapping here and add exact requirement/test identifiers beside executable cases in the server repository. A compact test inventory may be generated from those annotations: source version and identifier, implementation module/operation, test case, result, and any interpretation from §13. Do not maintain a second prose requirements specification containing copied standard text. [IDR-050–052][R050]

Implement the published abstract tests as independent HTTP checks where possible. Extend them with negative and semantic tests when an abstract test is weak, ambiguous, or copied incorrectly. Preserve both the source procedure and the documented correction; never silently turn a server defect into a passed standard test. Distinguish unimplemented, not run, failed, harness error, and genuinely inapplicable cases. A skip or warning is not conformance evidence.

The conformance runner must not import server selection logic, serializers, or fixture-derived expected responses as its sole oracle. It may use ordinary HTTP/format libraries and pinned schemas, but expected resource relationships, query answers, and value meaning must be independently authored or calculated.

Distinguish requirements coverage (which obligations have current executable evidence), source coverage (which code ran), and assertion strength (which plausible wrong behaviors the tests detect). Use source coverage to investigate untested branches and error paths, not as a correctness percentage. Neither a test count, a global coverage threshold nor a mutation score establishes conformance. Record meaningful gaps and exclusions; do not redefine the denominator to conceal them. [IDR-052 §9.4][R052]

### 7.3 Release declarations and evidence

Declare a class only when the enabled implementation satisfies all its applicable requirements and prerequisites with adequate tests. Keep the declared class list, OpenAPI, actual routes, representations, and runtime configuration consistent. A release may accurately be a partial implementation while the project still targets the complete set.

Normal CI artifacts should identify the server commit/build, configuration used, database/migration version, standards/schema pins, fixture version, executed cases and outcomes, and known deviations. Use existing test output formats plus a small summary; no dedicated evidence platform is required. Do not label local conformance tests as OGC certification or organizational accreditation.

Experimental Parts 3/4, SSE, retry headers, synchronization exchange, and other Glaux extensions have separate tests and documentation. Passing those tests does not add an approved CSAPI conformance URI. The published Features/CQL2 classes below can be declared separately only when their full obligations and dependencies pass; their declaration does not standardize Glaux's observation-property mapping.

### 7.4 Additional filtering targets and experimental Part 4 evidence

The selected filtering classes are additional project targets, not six more CSAPI classes. Append each suffix to its stated `/conf/` prefix:

| Standard prefix | Class suffix | Implementation and proof |
|---|---|---|
| `http://www.opengis.net/spec/ogcapi-features-3/1.0/conf/` | `queryables` | Scoped schema/discovery links, actual available property names/types and access behavior |
| Same Features prefix | `filter` | Language/CRS selection, errors, AND with native filters and correct selection |
| `http://www.opengis.net/spec/cql2/1.0/conf/` | `basic-cql2` | Complete basic comparisons, Boolean and NULL semantics, including valid typed temporal literals |
| Same CQL2 prefix | `basic-spatial-functions` | Correct intersection predicates, including missing values and boundaries |
| Same CQL2 prefix | `basic-spatial-functions-plus` | All required spatial literal types, not merely polygons |
| Same CQL2 prefix | `cql2-json` | Complete JSON encoding obligations for the selected language classes |

Use [Features Part 3 Annex A][F3] and [CQL2 Annex A][CQL2] plus the mapping-specific tests below. Filter depends on Queryables; the Queryables ATS also requires Common Part 1 JSON and its Core/Landing Page dependencies. Spatial-plus depends on basic-spatial and Basic CQL2; JSON depends on Basic CQL2 and conditionally on the selected spatial classes/GeoJSON geometry rules. Basic CQL2 also incorporates SFA Architecture, Time Ontology, RFC 3339, JSON Schema and Unicode as specified. Verify applicable dependencies rather than adding unrelated APIs.

Support the complete spatial-plus literal set, including multi-geometries and GeometryCollection, and required 2D/3D coordinate inputs. SFA intersection is horizontal, not an implied volumetric operation; preserve height information without advertising 3D volume evaluation. These literal requirements do not expand the Part 4 specialization set. Features Filter's feature-specific class, CQL2 Text, full Spatial/Temporal Functions, arrays, arithmetic and general joins are not selected. No claim to the separate broader OGC CQL2 Reference Implementation qualification follows.

For Part 4, keep the pinned draft clauses, adapted-schema checks, project interpretations and independent expected results distinct from approved Part 1 tests. Complete Point/Curve/Surface evidence must include specialized type/shape validation and round trips, not just generic feature parsing. Disabled experimental support must not remove approved generic SamplingFeature behavior or advertise unimplemented specialization.

## 8. Testing Strategy

### 8.1 Test layers and fixtures

| Layer | What it establishes | Tooling approach |
|---|---|---|
| Domain/codec unit tests | Resource invariants, time/units, status rules, exact encoding behavior, invalid input | Rust unit/property tests with deterministic clocks and inputs |
| Database integration | Constraints, relationships, PostGIS queries, concurrency, transactions, migrations, restore | Real pinned PostgreSQL/PostGIS; isolated test databases |
| HTTP contract/conformance | Routes, discovery, queries, representations, errors, writes, standards tests | Independent HTTP runner against a running server |
| External-client interoperability | Actual client navigation, parsing, query/use workflows and meaning | Pinned OS4CSAPI TypeScript client plus an independently implemented Python client such as supported OWSLib CSAPI functionality |
| Recovery/security/performance | Access isolation, resource limits, interruption, replay, uncertain effects, representative cost | Fault injection, controlled synthetic workloads, documented measurements |

Use a small synthetic dataset rich enough to distinguish behaviors: two permitted/denied source groups, a parent and child System, a Procedure, nested Deployments, multiple Sampling Features, property derivation, two observation streams with out-of-order/tied times, a status stream, and synchronous/asynchronous ControlStreams. Include positive and negative SensorML/SWE documents and independently specified expected values. Keep fixtures source-attributed and versioned; do not depend on live operational information or external servers for routine tests. [IDR-053][R053]

Arrange cleanup when test resources are created, including failure paths. Report setup, reset and cleanup failures; failed setup or reset must prevent the affected test from continuing with stale state. Tests must not depend on another test's mutable data or execution order. Record the seeds, clock inputs and dependency versions needed to reproduce failures. [IDR-062][R062]

Code formatting, linting, compilation, ordinary `cargo test` including applicable doctests, dependency checks, and deterministic fixture validation run in CI. Add suite-management tools when needed; a large named test taxonomy is not a prerequisite. Test instructions distinguish routine offline tests from separately provisioned client/broker runs. Those environments may be optional for a local quick run, but evidence required for an implemented capability or release is not optional.

#### 8.1.1 Test strength, review and reliable execution

These are implementation practices within existing tasks, not a separate test framework or approval process. They apply in proportion to the behavior being changed; documentation and prerequisite-inspection tasks need applicable document/inspection checks, not invented runtime tests. [IDR-052 §§5, 9–10, 14–15][R052], [IDR-053 §13][R053]

- **Show that a test can detect the intended mistake.** State the controlling requirement, independently expected answer and a plausible wrong behavior. Normally run the new test before implementing a behavior change, observe failure for that behavioral reason, then implement and rerun focused and affected regression tests. A setup, compilation or unrelated failure is not that proof. For already-correct behavior, characterization or another justified exception, record why and demonstrate sensitivity with a suitable known-bad input/output or controlled fault where practical; disclose any remaining limitation. Intentional faults stay in disposable test runs and must not remain in the merged implementation.
- **Review assertions and expected data.** The PR review examines test inputs, expected answers and failure evidence as well as production code. Check exact identities, values, order and forbidden facts where the contract requires them; status/count/schema validity alone cannot establish selection or semantic correctness. Encoder/decoder agreement alone cannot exclude a shared bug. Review changed goldens (checked-in expected outputs) against the source contract; do not auto-accept generated output, weaken assertions or remove contractual IDs/timestamps/links merely to pass. Normalize only documented non-contractual variation. Independent reasoning and source data matter, not merely a separate file or assistant.
- **Use the layer that can observe the claim.** Small pure tests and narrow fakes are useful, but database mocks do not prove SQL/PostGIS/transaction behavior, and a recording publisher does not prove broker delivery or crash recovery. Retain real database, listener, packaged-startup and broker checks where those boundaries are claimed. Use explicit barriers/readiness signals and bounded diagnostic waits for concurrent work, rather than sleeps as evidence of ordering. A paused application clock does not control database or operating-system time.
- **Exercise more than hand-picked happy paths.** Add bounded property/model tests for numeric/time boundaries, query/paging rules, codec framing, command/retry and replay state transitions as their owning capabilities arrive. Choose generators that reach both valid and invalid/boundary cases; check the generator's determinism and intended partitions. Add resource-bounded fuzz targets for exposed query/cursor, schema, SWE and exchange parsers, using synthetic seed cases and semantic invariants beyond no panic where applicable. Record tool/generator versions and seeds, minimize failures and retain concrete regression cases; a seed alone may cease reproducing a failure after generator changes.
- **Sample assertion strength at critical logic.** Use targeted controlled faults or mutation testing on predicates, authorization, exact value handling, codec bounds and duplicate-effect/state transitions, starting from a passing, reproducible unmodified baseline. Pre-existing failures cannot count as detected mutations. Start with changed high-risk logic, not every function or a universal score. Inspect meaningful survivors: repair a missing assertion or document why the change is behaviorally equivalent or outside the tested contract. Compilation failures are not detected behavioral defects; timeouts require investigation. An identified defect or missing required proof remains open even when a diagnostic job itself is advisory. Mutation tools do not replace standards-derived expectations or find requirements absent from the implementation.
- **Prevent false-green execution.** Establish and recheck CI discovery and failure propagation: expected suites actually run; an empty or accidentally filtered required suite, unavailable required service, runner error or deliberately failing assertion cannot yield success. Retain passed/failed/skipped/ignored outcomes and the relevant selection/configuration. No retry-until-green: diagnostic retries preserve the initial failure; an unexplained flaky required check remains unresolved. A quarantined check has a linked owner/follow-up and remains an explicit evidence gap, not a pass. Never silently discard a required test to meet a runtime budget.
- **Report understandable evidence.** In the existing issue/PR record, state what behavior was proved, where the expected answer came from, what meaningful mistake was detected, what actually ran and what remains uncertain. Link normal test output and the tested commit/configuration. Explain a justified non-applicability or alternative honestly. Assistant review is not independent human approval, and neither review nor a tool score is proof of defect-free software.

Use the ordinary Cargo/real-database/independent-HTTP baseline. Suitable focused tools are `proptest`, `cargo-fuzz`, `cargo-mutants` and diagnostic `cargo-llvm-cov`; select and pin actual versions/configurations when their owning tasks establish compatibility, license and platform suitability. A suite runner such as nextest is optional, not evidence of quality by itself. Specialized tooling uses an approved supported environment; this does not authorize installation on the company laptop or require every tool for every issue. See §12.3 for primary-source support and limits.

Every implementation PR runs applicable fast checks, bounded property/regression cases and affected real-database/HTTP tests. Introduce each relevant scheduled method incrementally with its first owning target: broader generated-input/fuzz runs and targeted mutation/coverage checks need not all start in one task. Record budgets, supported build features and exclusions, then expand with capability growth. A scheduled result belongs to its actual commit and cannot substitute for a missing required PR check. Phase completion reruns affected workflows; the release candidate runs the complete applicable suite against its actual configuration, including required independent-client, broker, security and recovery cases. Known failures return to their owning issues; larger campaigns complement, rather than postpone, the tests needed to accept a capability.

### 8.2 Representative end-to-end scenarios

1. **Register and discover.** Create a Procedure, System, subsystem, Deployment, and Sampling Feature through the public interface. Start a separate client at the root, follow links, and recover the same identities and associations in each supported representation.
2. **Publish and retrieve.** Register a schema and DataStream; submit distinguishable observations; retrieve exact expected values with feature/time/property filters and paging. Send an invalid value and prove that neither data nor publication changed.
3. **Status under delay.** Publish current and delayed older status values; query history and current evidence. Disconnect the source and verify that last-known state remains timestamped and does not become false current availability.
4. **Live delivery and recovery.** Subscribe, publish changes, interrupt delivery, reconnect, and recover within the documented retention/authorization boundary. Exercise duplicate messages, deletion events, expired history, and revoked access. Test the optional MQTT binding too, using its HTTP/snapshot recovery path rather than assuming MQTT-native replay for a disconnected subscriber.
5. **Feasibility and command execution.** Run an infeasible analysis, a feasible analysis, an authorized command, and a denied command. Verify distinct outcomes, standard statuses/results, required time fields, synchronous/asynchronous behavior, and safe interruption during dispatch.
6. **Synchronization and restore.** Export state and a replay position, add concurrent changes, import/catch up, and inject a conflicting revision. Restore a backup and verify resource meaning, identity, deletion handling, and held command work. A cursor observed after the backup must fail in the new recovery epoch instead of skipping new events; resubmitting a post-backup command's key must not silently redispatch an effect whose receipt was lost.
7. **Experimental sampling descriptions.** Create/read/replace/patch all three selected Part 4 types with full URI and CURIE inputs. Check canonical type output, preserved associations/coordinates, ordinary spatial matches and rejection of wrong/null geometry. Confirm that generic or unsupported types are not mislabeled as specialized support.
8. **Enhanced observation selection.** Seed inside/boundary/outside direct sampling points and an intersecting ancestor whose child point is outside; assert different direct-geometry and recursive `foi` results. Use explicit retained geometry at phenomenon time that differs from current geometry, plus an unknown-history case and an interval not covered by one unambiguous geometry. Assert no current fallback. Combine spatial and scalar predicates with native time, `foi` and latest selection; check exact IDs before paging. Include declared nil, absent values, number/text and Celsius/Fahrenheit distinctions, equivalent SWE response encodings, discovery, invalid names/operators/CRS, NULL/negation, stale mapping cursors and multiple relationship paths. Repeat with protected related facts changed but the permitted view unchanged; membership, counts and errors must not reveal those facts. Specimen-property examples from research remain outside the adopted scope.
9. **Provenance and quality preservation.** Compare a direct measurement with a derived result whose exact input revisions and method revision are supplied. Change the current Procedure and prove historical context is not rewritten. Separate uploader from creator/delegated role assertions; test unknown input/role context without invented defaults. Preserve genuinely shared metadata and per-item differences. Round-trip all four quality component types, multiple assertions and per-record local references through schema JSON and JSON/Text/Binary values. Test missing/nil quality, invalid units/types, malformed `quality`, wrong/duplicate reference targets and cycles. Changing protected contributor/quality/label facts must not alter the permitted response or reveal them indirectly. A transformed payload must not inherit an unverified signature claim.

The second-pass contracts add these focused checks to the scenarios above:

- **Writes/schema discovery:** minimal stream POST with embedded schema, generated response fields and schema retrieval; observation POST without generated IDs; allowed description PATCH versus rejected schema modification after data exists. Also test ignored body-local IDs versus protected UID/parent/schema changes, PUT field removal/lossless conversion, Merge Patch null/array behavior, atomic validation failure, and distinct selector/media errors.
- **Command recovery:** exact `201`/Location/Content-Location/body; durable admission, external effect, terminal publication and lost-response crash boundaries; concurrent same-key retries never create another dispatch; timeout followed by completion creates one terminal report unless an authorized deletion intervenes. PUT/PATCH/DELETE of terminal reports must not restart execution. Test concurrent cascade, private outcome retention and the exceptional synchronous conflict; unkeyed ambiguity makes no deduplication promise, and successful feasibility analysis can answer NO without actuation.
- **Filtering:** values `20`, `25`, `30`, a declared nil sentinel, absence, NaN and infinities: `>25` selects only `30`, its negation selects `20`/`25`, and `isNull` selects unavailable states. Test zero/false, undeclared sentinels, exact large Counts, fractional numeric thresholds, compatible revisions, authoritative corrections versus staged conflicts, Boolean root literals and every selected operator/literal family. Include singleton GeometryCollection and invalid dates/types/branches on empty data.
- **Delivery/exchange:** browser authentication, cursor tampering/expiry/scope changes, ordered replay, slow subscribers and policy revocation; exact topic/media/token/AsyncAPI agreement; live-only MQTT recovery without persistent-session completeness claims; snapshot/outbox timing (snapshot r5, delayed r1–r4, then r6), validated overlap ancestry, missing ancestry, matched versus incompatible SSE bootstrap context, changed access scope and protected omissions; no completeness checkpoint across unresolved records. Race command admission/dispatch/publication against parent deletion and schema changes.

These scenarios connect the sections of the guide; they do not replace the complete class tests. The third drafting pass used them as design walkthroughs; executable verification remains implementation work.

### 8.3 Performance and regression expectations

Measure ingestion throughput/latency, concurrent filtered reads, latest-value queries, hierarchy/spatial searches, codec costs, bounded streaming, and replay recovery on a stated dataset and machine. Record correctness alongside latency, memory, and queue growth. Do not adopt invented production service levels or claim scale from a microbenchmark. [IDR-054][R054]

Include regressions from independent-client research: missing discovery links, different identities across formats, wrong envelopes, ignored query parameters, defective nested paths, incomplete feasibility descriptions, schema/value mismatch, stale latest selection, and overstated conformance. Pin client versions and record their supported subsets. A client workaround is evidence of interoperability behavior, not permission to change the standard contract. [IDR-014E–014G][R014e], [IDR-056][R056]

### 8.4 Whole-guide walkthrough and Roadmap handoff

The third drafting pass checked these paths from input to result against Goal v1.7. This records a design review, not executed integration tests; §8.2 supplies the scenarios to implement.

| Workflow | Connected design path | Result and failure boundary checked |
|---|---|---|
| Register and discover a System | Authorized description POST → operation-specific validation → identity/associations and source revision commit → canonical Location and linked alternate representations (§§4.1–4.3, 4.6, 6.2) | One identity across views; no generated-input requirement, source-content loss or access bypass |
| Register a stream and ingest observations | Nested stream POST with one schema → immutable compiled contract and generated format discovery → schema-bound observation POST → resource/audit/outbox transaction (§§4.3–4.4, 4.6–4.7) | Valid values become durable together; invalid input does not publish; later schema modification cannot reinterpret accepted data |
| Retrieve, filter and assess status | Authorized scope → native/selected CQL2 predicates → latest selection where requested → stable paging → selected encoder (§§4.4–4.5, 6.3) | Direct historical sampling geometry and value semantics preserved; stale/unknown status stays distinct; hidden related facts do not influence the permitted view |
| Stream and recover | Committed outbox → retained ordered log → authorized SSE or optional MQTT → bounded reconnect/resnapshot (§§4.8, 4.11) | SSE cursors and administrative exchange tokens are distinct; MQTT live delivery does not promise disconnected-subscriber replay |
| Evaluate feasibility and issue a command | ControlStream contract/permission → durable admission → adapter analysis or dispatch → authoritative outcome → status/results and HTTP response (§§4.9, 6.4) | Feasibility is not actuation; timeouts do not invent failure; retries, terminal report edits and cascade deletion cannot silently repeat execution or resurrect resources |
| Exchange and restore | Scoped snapshot/ancestry → validated import/conflict handling → ordered catch-up; isolated backup restore → new recovery epoch and command reconciliation (§§4.7, 4.11) | No stale overwrite, protected-omission deletion, skipped post-restore event or automatic replay of an uncertain physical action |

Across these paths, provenance/quality disclosure follows §4.10 and the same write/query boundaries; it is not another server subsystem. Experimental static Part 4 validation feeds the existing SamplingFeature path, and the selected filtering classes remain additional to the 25 CSAPI targets. Developer setup and independent-client use follow §§4.12 and 5.

The [Roadmap](glaux-server-roadmap.md) assigns implementation order and concrete tasks to these capabilities, §7's complete class/dependency coverage, §8's tests and §9.2's remaining proofs. Its complete initial issue set is published. This walkthrough remains a design review, not executed verification, a separate requirements inventory or an additional approval process.

## 9. Risks and Remaining Implementation Checks

### 9.1 Principal risks and mitigations

| Risk | Concrete mitigation |
|---|---|
| Published prose, schema, examples, and tests disagree | Keep the specific interpretation in §13, retain both source fixtures, and qualify affected claims until resolved |
| A rich SensorML/SWE model passes simple examples but fails real component combinations | Test recursive and composite schemas and all required codecs independently before advertising their classes |
| Data loses precision or meaning during storage/conversion | Preserve source/schema binding; test units, time precision, field order, nil/optional values, and cross-format equivalence |
| Resource/query authorization leaks through links, schemas, counts, latest selection, or events | Apply one access model before selection and test indirect disclosures and policy changes |
| Database and transport/device effects diverge under failure | Transactional outbox/work records, ordered replay, stable identities, and explicit uncertain-command reconciliation |
| Synchronization overwrites correct local state or resurrects deletions | Source-scoped revisions, receipt deduplication, retained tombstones, scoped snapshot reconciliation, and explicit conflicts |
| Full-scope delivery becomes a collection of permanently deferred features | Keep the 25-class map visible; Roadmap tasks must cover outstanding behavior, including Text/Binary, writes, and tasking |
| Research recommendations grow into unnecessary infrastructure | Adopt only mechanisms justified in this guide; keep the ordinary reference deployment small |
| Dependency or performance assumptions prove wrong | Compile the actual pinned dependency set and measure representative workloads before claiming support or scale |
| Experimental static types are confused with generic storage or whole-Part-4 support | Test exact specialized contracts; label the pinned interpretation and excluded types/behaviors |
| A plausible filter selects the wrong sampling location or leaks hidden related data | Test direct/ancestor and historical/current differences, unavailable history, unit/nil semantics and authorization before evaluation |
| Production context or quality is lost, overstated or disclosed without authority | Bind supplied context to exact revisions/components, distinguish roles and uncertainty measures, validate the quality interpretation and test independent disclosure |

### 9.2 Design dispositions and remaining implementation checks

The drafting questions have the following dispositions, checked together in the third-pass walkthrough (§8.4). Selecting a contract does not establish interoperability or conformance; remaining executable proofs belong to implementation. The Roadmap must assign these checks alongside the capabilities they protect, not leave them as an unowned research backlog.

| Former open item | Design disposition | Remaining implementation proof |
|---|---|---|
| Inherited transaction revision | Earlier pin retained; local body IDs ignored; later Common Part 5 changes not imported (§1.2) | Per-operation fixtures using the selected requirements and namespace |
| Cross-format writes/schema selector | Lossless replacement or explicit conflict; fixed Merge Patch projections; supported-format preference when `cmdFormat` is omitted (§§4.6, 6.2) | Full operation/media fixtures, especially richer source content and non-JSON record values |
| Command response/recovery | Exact header/body combination, private admission, bounded HTTP wait, reconciliation and optional-key limitations (§6.4) | Independent client checks and crash tests; adapter-specific external-effect evidence |
| Source contradictions | Explicit treatments and qualifications in §13, including System Event JSON and quality | Test both source fixtures and selected interpretations; unresolved normative contradictions may still prevent an unqualified affected claim |
| Dependencies/schema/time/text | Retain the small Rust stack, offline resolution and exact comparisons; no software installed or versions falsely verified | Compile/pin versions/features, prove recursive schemas, exact time representation and Unicode keyword matching; include source precision, calendar/offset/leap-second and non-ASCII cases. An unsupported input/precision limitation must be explicit and assessed against conformance, never silently rounded or ignored |
| Optional transport | SSE scope/cursor/authentication; exact MQTT paths/tokens/media, live-only data recovery, rumqttc/Mosquitto and AsyncAPI contract (§4.8) | Pin client/broker versions; executable AsyncAPI and access/recovery fixtures |
| Recovery and limits | Versioned bounded manifest, source epoch/revision/predecessor, scoped snapshots and explicit conflict operations (§4.11); restore boundaries in §4.7 | Generate schema/fixtures; prove snapshot/catch-up and post-restore continuity, enforce finite limits, measure reference values and publish recovery/deduplication limits |
| Enhanced queries | Scalar/revision/NULL mappings, queryables example, literal validation and authoritative correction behavior (§§4.4.1, 6.3.1) | Independent exact-result and complete selected-class tests, including source-artifact qualifications |
| Provenance/quality and Part 5 | Preserve known context and distinct quality meaning within existing scope; defer Part 5 implementation (§1.5) | Context/disclosure/quality fixtures; no new public lineage or Part 5 contract assumed |

Production workloads, an operational identity provider, sensor procurement and federation agreements remain deployment-specific, not prerequisites for Roadmap drafting. Implementation proofs should answer bounded library/encoding questions within the relevant tasks, not become new project deliverables in their own right. If a selected dependency or interpretation fails its tests, update the affected design here; do not silently drop an approved capability or declare its class anyway.

## 10. Quality Gates and Exit Criteria

These are checks on implementation and release claims, not additional drafting approval stages.

- **For an implemented capability:** required operations and representations exist; relevant standards and negative tests run; access/error behavior is covered; API documentation matches the implementation; any interpretation is explicit.
- **For a partial reference release:** build/run/test instructions and examples work on documented prerequisites; included capabilities pass their tests; disabled/deferred behavior is listed; no unsupported conformance class is advertised.
- **For full server completion:** all 25 direct CSAPI classes and applicable prerequisites are implemented and verified; all approved goal capabilities have evidence; experimental Part 3 and the selected static Part 4 types are implemented and accurately labeled; all six selected Features/CQL2 classes and their applicable dependencies pass with the agreed observation mapping; independent client workflows, recovery/security tests, and developer examples pass; no unresolved issue invalidates the stated conformance or safety boundaries. Deployment disablement is not a substitute for implementing an adopted capability.
- **For guide finalization before the Roadmap:** architecture, contracts, verification, and scope agree; consequential open design questions have a defensible disposition; remaining implementation-time choices are bounded and do not conceal missing capability design. This does not assert that the software has been completed.

No numeric performance target, production accreditation, or unrelated Glaux product completion is added to those conditions.

Version 1.0 meets the document-finalization condition through the scope map (§1.1), full target tables (§7), workflow review (§8.4) and explicit dispositions (§§9.2, 13). The other conditions remain future implementation/release checks. Baseline status does not assert that every upstream ambiguity is resolved or that a conformance class has passed.

## 11. Change Control

Version 0.1 was drafting iteration 1. Version 0.2 incorporated the approved Part 4/querying scope in Goal v1.7. Version 0.3 completed drafting iteration 2 after the project lead's `proceed` on the combined provenance/Part 5 discussion: keep Goal v1.7, incorporate focused provenance clarifications, and defer Part 5 implementation while preserving codec separation. It disposed the first-draft technical questions in §9.2. Research reports and synthesis addenda remain the historical evidence, not retroactively rewritten scope decisions.

Version 1.0 completes the third pass authorized by the project lead's subsequent `proceed`. It checks scope and end-to-end workflows, clarifies direction-aware validation and stream-schema registration, corrects cascade/terminal-status interactions, and makes backup-rollback continuity and command uncertainty explicit. It removes the unselected `f` negotiation extension and leftover proposal wording for selected choices. These refine the design within Goal v1.7; they do not add scope or claim implemented software.

Version 1.1 incorporates the project lead's September 18, 2026 `proceed` after discussion of IDR-SRV-062 and synthesis Addendum E. It clarifies test-harness failure/isolation/reproduction behavior (§8.1) and maintenance of contributor/assistant guidance (§4.12), without changing architecture or scope. Roadmap v1.2 retains the same nine phases, 41 capability groups and 286 issue-sized tasks.

Version 1.2 implements the project lead's `proceed` after the focused test-quality assessment. It restores useful daily practices from IDR-052/053, informed by primary-source checks and specific CS-GO assertion examples, without adopting the research's larger evidence machinery or mandatory tool portfolio. Roadmap v1.4 strengthens the existing CI task and assigns targeted methods to existing capability owners. No Goal change, new research topic, implementation issue, software installation or runtime verification is part of this document revision.

Version 1.3 records the project lead's September 19, 2026 `proceed` to pair the Goal's capability descriptions with understandable implementation explanations. Section 1.1 now connects each Goal heading to the selected technologies, their purpose and the detailed design, while retaining the capability index and acceptance boundary. Goal v1.8 adds navigation links only; Roadmap v1.18 updates the current planning references. The previous §1.1 anchor is retained for compatibility. Historical research and all 286 issue preparation pins remain unchanged; there are no new capabilities, technical contracts, technology selections or task definitions.

The Guide remains baselined. Issue publication is complete; the next implementation iteration is [task 1.1.1 / issue #3](https://github.com/DGIWG-P507/glaux-server/issues/3), the approved-prerequisite inspection described in [Roadmap §5.3](glaux-server-roadmap.md#53-immediate-next-step), without implicit installation. This documentation iteration does not execute that issue. Continue to push and summarize one completed iteration at a time; implementation proceeds one ready issue per authorized iteration.

Record material technical changes here with their reason and affected behavior/tests. A change that expands the approved goal must first be addressed in the Goal and Definition; an internal implementation improvement need not reopen mission scope. Update standards interpretations when authoritative corrections arrive and test compatibility before changing a published contract.

Keep requirement-to-test references near the code and this guide's capability/class tables current. Research remains supporting evidence. A report's acceptance, recommendation number, or confidence statement does not override the approved goal or prove a code path correct.

## 12. References and Research Use

### 12.1 Controlling planning documents and practical examples

- [Approved Goal and Definition](glaux-server-goal-and-definition.md)
- [Initial Planning Guidance](../../Governance/initial-planning-guidance.md)
- [Implementation Guide template and drafting instructions](../../Governance/implementation-guide-template.md)
- [Final initial-design research synthesis][RSynthesis] and its linked topic reports
- [OS4CSAPI main implementation guide][ExampleMain] and [parser-completion guide][ExampleParser]

The OS4CSAPI examples inform the use of concrete component boundaries, input/output contracts, implementation notes, and representative workflows. Their client architecture, historical API examples, estimates, and testing restrictions are not imported as server requirements. In particular, this server must be tested with real HTTP/database behavior and semantic expected results.

### 12.2 How the research informed this guide

| Research area | Applied here |
|---|---|
| [001–005: framework, responsibilities, standards, terminology, boundaries][R001] | §1 and the approved goal; no unrelated NATO service family or enterprise infrastructure added |
| [006–008: requirement and class baseline][R008] | Exact all-class target, inherited scope, and §7 coverage |
| [009–014: discovery, HTTP, queries, representations, API descriptions][R010] | §§4.1–4.4 and §6, including source-conflict handling |
| [014A–014H: implementations, clients, and Part 3][R014h] | Interoperability regressions and the bounded experimental design; peers are not normative authority |
| [015–024: models, identity, time, status, SensorML/SWE, semantics][R015] | Typed shared resources, precise time/schema meaning, validation, and preservation |
| [025–030: persistence and lifecycle][R025] | PostgreSQL/PostGIS, transactions, measured partitioning, deletion/retention distinctions |
| [031–038: writes, Publisher/Simulator, dynamic data, streaming, commands][R031] | One write boundary, ordinary ecosystem APIs, durable publication, full tasking/feasibility |
| [039–043, including 039A: security and interrupted operation][R039] | Server-enforced access, accountability, honest freshness, replay and conflict handling |
| [044–049: Rust, architecture, deployment, configuration, operations][R044] | Small workspace, one server, explicit dependencies, runnable reference, migration/restore |
| [050–056: verification and interoperability][R050] | Standards-to-test connections, real database/HTTP tests, independent clients, bounded workload measurements; v1.2 makes IDR-052/053's assertion-strength, generated-input, fixture-review and false-green controls explicit |
| [058: draft Part 4][R058] | The project lead selected the report's bounded implementation alternative, not blanket draft adoption: §§1.4, 4.2.1, 7.4 and 13 |
| [059: enhanced querying][R059] | Adopted six-class JSON filtering and explicit direct-geometry/scalar mappings: §§4.4.1, 6.3.1, 7.4 and 8.2; qualifies the earlier CQL2 deferral |
| [060: Part 5 Protobuf-first assessment][R060] | §1.5 records implementation deferral with existing codec/schema separation; no Protobuf replacement for SWE Binary |
| [061: provenance and related metadata][R061] | §§4.3, 4.10, 6.1 and 8 distinguish production/audit context, exact known inputs, roles, quality and disclosure; no new provenance platform or security regime |
| [062: CS-GO engineering practices and development history][R062] | §§4.12 and 8.1 clarify guidance maintenance and reliable test lifecycle/reproduction; other lessons reinforce existing work, without importing Go-specific mechanisms or claiming established peer TDD/runtime results |

Consequential source checks during this drafting pass included the published CSAPI resource/encoding clauses, the exact conformance identifiers, Features transaction draft pins and conditional-request permissions, SWE optional/array behavior, command/feasibility semantics, and the current pinned Part 3 source. This is targeted checking of findings used in the design, not a claim that every research paragraph has been re-audited.

The v0.2 scope update additionally checked the pinned Part 4 spatial clauses/schemas and published Features Part 3/CQL2 clauses against the accepted supplements. Their reports and synthesis addenda remain the historical research record; the later scope approval is recorded in Goal v1.7 and this revision, not by rewriting their recommendations as past adoption decisions.

The September 18 second pass compared both transaction pins, inspected published command/schema/System Event requirements, checked the pinned Part 3 event/format material, CQL2/queryables rules and SWE quality evidence, and used the SSE/MQTT/database primary references for the selected delivery/recovery contracts. Open issues cited below remain proposals, not normative fixes. Dependency compilation, generated contract validation, runtime performance and independent-client tests were not run as part of document drafting.

The third pass checked whole-Goal coverage and the workflows in §8.4. Targeted source checks covered the pinned stream/observation schemas, published schema-mutation and cascade rules, and terminal status versus report mutation; accepted restore research informed the recovery-epoch and missing-command-history limits. Document/link/example checks and independent read-only design reviews are not substitutes for future runtime tests.

The Guide deliberately does not adopt every proposed research mechanism: no compulsory private Publisher envelope, simulator management API, eight-package skeleton, graph/evidence database, universal policy or trust engine, mandatory conditional-write header, or separate requirements/decision-document set. The capability remains required where the approved goal requires it; these particular mechanisms do not.

### 12.3 Test-quality clarification sources and limits

The September 18, 2026 assessment compared existing testing research with executable peer-test source and primary guidance. The following support §8.1.1's selected practices, not a claim that one tool portfolio is universally best or that Glaux tests have run:

- [NIST SSDF v1.1, PW.8](https://nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-218.pdf): risk-appropriate executable testing, input fuzzing and regression handling; not an additional accreditation target.
- [Inozemtseva and Holmes, ICSE 2014](https://cs.uwaterloo.ca/~m2nagapp/courses/CS846/1171/papers/inozemtseva_icse14.pdf): coverage alone was a weak effectiveness proxy in the studied Java systems; not a measured Rust/Glaux result.
- [Practical Mutation Testing at Scale, 2021](https://research.google/pubs/practical-mutation-testing-at-scale-a-view-from-google/) and [cargo-mutants interpretation](https://mutants.rs/using-results.html): selective mutation diagnostics and careful survivor interpretation, not score-driven acceptance.
- [Proptest failure persistence](https://proptest-rs.github.io/proptest/proptest/failure-persistence.html), [state-machine testing](https://proptest-rs.github.io/proptest/proptest/state-machine.html) and the [Rust Fuzz Book](https://rust-fuzz.github.io/book/cargo-fuzz/guide.html): generators, replay limits and focused fuzz targets; verify actual supported toolchains during implementation.
- [OWASP authorization testing](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Testing_Automation_Cheat_Sheet.html): exercise roles, operations and data access, supplementing the existing protected-view tests rather than substituting a scanner result.
- CS-GO at `b1fd2e0e9bd69e222d05258d659a842ca24502cb`: its [latest-observation test](https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/e2e/observations_test.go#L465-L500) checks the expected ID, while the adjacent [time-range test](https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/e2e/observations_test.go#L506-L539) checks one returned item but not which one. The latter could accept the wrong observation; this is source analysis, not an observed runtime defect or a conclusion about the author's TDD process.

## 13. Appendix: Standards Interpretations and Project Choices

This table records known consequential seams in one place. An interpretation is not an amendment to an OGC standard. It needs focused tests, transparent release documentation, and reconsideration when authoritative corrections become available. A materially unresolved contradiction can prevent an unqualified affected conformance claim.

| Source issue | Selected implementation treatment | Source and remaining qualification |
|---|---|---|
| CSAPI transaction dependency cites Features Part 4, while later research uses a renamed/newer draft | Keep the earlier explicit pin and original dependency identifiers; ignore body-local IDs on PUT/PATCH. Later permission to reject mismatching IDs is not imported | [008][R008], [031][R031], [earlier transaction source][F4], [later source][F4Later]; §1.2 supersedes the research's blanket local-ID rejection |
| Cross-format replacement and PATCH target remain open upstream | Lossless replacement or `409`; fixed writable projections and whole-result validation in §4.6 | [Replacement issue][ReplaceIssue], [Patch issue][PatchIssue]; selected Glaux treatment, not an upstream resolution |
| Mandatory `If-Match` in research 029/031 is a project choice | Honor supplied conditions; permit unconditional standard writes with the lost-update tradeoff documented | [029][R029], [031][R031]; both reviewed draft revisions permit a missing-header success path |
| Required response members can also be annotated read-only; schema is write-only on stream descriptions | Use operation/direction projections with unchanged upstream artifacts; compile the schema supplied with stream registration (§§4.3–4.4) | [023][R023], [DataStreamSchema], [ControlStreamSchema], [ObservationSchema]; generic JSON Schema validation alone is insufficient |
| Cascade prose says parameter presence while abstract tests distinguish true/false | Use validated Boolean `cascade=true` for cascading deletion, not mere presence; false/default remains non-cascading | [S2 requirements 65/70 and tests A.65/A.70][S2]; public deletion retains private evidence but does not cancel device execution |
| Singular `/controlstream`, `/command`, isolated `/controls/{id}`, and inconsistent System Event paths | Use `/controlstreams`, `/commands`, and `/systems/{id}/events` consistently; no automatic unsafe-method aliases | [010][R010]; competing published/ATS/artifact templates remain documented |
| Nested feasibility replace/delete template omits the item ID; tagged OpenAPI omits feasibility | Implement explicit canonical `/feasibility/{id}` and qualified nested item operations with the ID; describe and test feasibility in Glaux's API contract | [007][R007], [010][R010], [037][R037] |
| Exact command/feasibility POST header/body combination is not stated consistently | Select `201` + canonical `Location` + status body + individual status `Content-Location`; bounded waiting/reconciliation in §6.4 | [036][R036], [037][R037]; verify with independent clients during implementation, not a claimed resolved OGC ambiguity |
| Terminal lifecycle rules coexist with report CRUD/PATCH and parent cascade | Preserve execution evidence independently of mutable public reports; honor cascade without resurrection; concurrent deletion can prevent synchronous public success (§§4.9, 6.4) | [S2 §§10.11, 14.4, 14.6, 15.6][S2]; explicit Glaux reconciliation, including the exceptional `409` response |
| SensorML/CSAPI Property and System Event media/schema differences | Properties use CSAPI SensorML; Events use `application/json`, `definition`/`label`/`time` and authorized parent relation (§4.5) | [012][R012], [020][R020], [021][R021], [EventIssue]; conceptual message/parent mapping remains explicitly qualified |
| System Event type examples contain `x-OGC/TBD` identifiers | Preserve submitted valid identifiers; use explicitly Glaux/example vocabulary where a stable demonstration definition is needed, without claiming it is an approved OGC term | [020][R020]; no invented definitive OGC vocabulary |
| SWE media types differ between CSAPI and SWE Common | Use CSAPI `application/swe+json`, `+text`, `+binary`; support vendor-prefixed equivalents only as declared aliases over the same codecs, returning the negotiated token | [012][R012], [S2], [SWE]; aliases do not erase the source conflict |
| `cmdFormat` query name/default versus `commandFormat` response property | Use the optional query parameter, explicit supported-format preference in §6.2 and actual selected `commandFormat`; wrapper remains JSON | [012][R012], [022][R022], Part 2 requirement 25; default preference is Glaux's choice |
| SWE array-flag requirements conflict with model/schema/examples | Use `recordsAsArrays`/`vectorsAsArrays` with true meaning arrays, recording the contradictory requirement text and both fixture shapes | [022][R022], SWE §8.7.1 and §10.2.3; needs explicit conformance interpretation |
| Optional SWE fields and nil sentinels can be confused | Permit absent or null optional object fields and required positional null placeholders as specified; do not treat every JSON null as a declared nil-reason value | [022][R022], SWE JSON encoding rules |
| Published SWE simple-component JSON schema omits conceptual quality | Separate semantic validation of the array/component/reference interpretation in §4.3; retain unmodified schemas and exact source | [022][R022], [061][R061], [QualityPeer]; legacy peer evidence is not complete SWE 3.0 conformance |
| Open resource `validTime` bounds are less clear than query bounds | Accept `..` in query intervals; do not emit it as a core resource bound. Use finite bounds or the published ongoing-period `now` shape only where valid, without treating `now` as infinity | [018][R018]; a genuinely unrepresentable bound remains an explicit limitation/interpretation |
| System Event/status `datetime` inheritance lacks a clear family field mapping | Use event occurrence time and status `reportTime`, respectively, with explicit tests and API documentation | [011][R011], [018][R018], [020][R020] |
| Observed/controlled-property conceptual URI lists differ from JSON object summaries | Store property identities explicitly; generate the selected published JSON object form and document derivation of summaries | [024][R024]; do not infer identities from labels or units |
| SensorML DataInterface, qualifiers, and input/output binding gaps | Preserve valid source constructs; implement only mappings supported by the chosen schema/interpretation and surface unsupported executable bindings explicitly | [021][R021], [024][R024]; do not silently discard richer source meaning |
| Part 3 MQTT/discovery incomplete; `parentId` conflicts with CloudEvents attribute spelling | Publish a versioned Glaux MQTT/AsyncAPI experiment with explicit topics/discovery; use `parentid` as a listed deviation. SSE stays separate | [014H][R014h], [035][R035], [pinned Part 3 source][P3] |
| Part 4 static prose permits URI/CURIE while subtype schemas use full-URI constants | Accept only the three explicit alias/full-URI pairs in §4.2.1; dispatch by expanded URI and serialize the full URI, preserving source spelling | [058][R058], [pinned spatial clauses][P4Types]; experimental encoding choice, not an amended standard |
| Part 4's spatial schema references missing `samplingFeature.json` and stale inherited requirement names | Compose the selected validator with the pinned approved Part 1 SamplingFeature contract/dependencies; use current Part 1 `/req/sf` inheritance; retain originals and adapted-schema provenance | [058][R058], [spatial schema][P4Spatial], [approved common schema][SFSchema]; do not import the unfinished all-types bundle |
| Part 4 schemas allow null/pose alternatives while spatial prose requires shape/location | Require non-null matching top-level GeoJSON geometry for the selected static types; no pose-only fallback. Generic Part 1 geometry-less handling is unchanged | [058][R058], [P4Types]; O&M spatial metadata's Clause 9 citation is reconciled to O&M 2.0 Clause 10, not a new dependency scope |
| Features Part 3/CQL2 does not standardize CSAPI observation queryables | Use the scoped paths/names in §6.3.1 and direct phenomenon-time geometry rule in §4.4.1; absence maps to NULL only in the authorized view | [059][R059], [F3], [CQL2]; Glaux mapping, no ancestor/current fallback, arbitrary joins or implied Part 4 mobile support |
| CQL2 JSON schema sets GeometryCollection minimum to two members, while spatial-plus prose permits one or more | Accept singleton GeometryCollection under the selected spatial-plus semantics; preserve the upstream schema and test the explicit adaptation | [CQLSchema], [CQL2 §7.6][CQL2]; generic schema acceptance alone does not establish literal conformance |

These treatments are part of the technical baseline; baselining does not resolve upstream contradictions or certify an implementation. Where evidence cannot support an unqualified claim, retain the qualification. A new authoritative correction or failed implementation proof can require a versioned Guide change without reopening unrelated scope or creating another planning process.

[R001]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md
[R006]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-006-csapi-part-1-requirement-baseline-report.md
[R007]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-007-csapi-part-2-requirement-baseline-report.md
[R008]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-008-conformance-class-and-requirement-mapping-report.md
[R009]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-009-landing-page-api-definition-and-conformance-declaration-behavior-report.md
[R010]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-010-collections-resources-links-and-navigation-behavior-report.md
[R010a]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-010a-api-versioning-backward-compatibility-and-deprecation-strategy-report.md
[R011]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-011-query-filtering-sorting-pagination-and-selection-semantics-report.md
[R012]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-012-content-negotiation-media-types-and-encoding-selection-report.md
[R013]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-013-error-model-http-status-codes-and-failure-semantics-report.md
[R014]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-014-openapi-description-and-api-documentation-strategy-report.md
[R014e]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-014e-os4csapi-client-smoke-test-findings-study-report.md
[R014h]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-014h-draft-csapi-part-3-publish-subscribe-and-implementation-study-report.md
[R015]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-015-canonical-glaux-server-resource-model-report.md
[R018]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-018-temporal-validity-and-freshness-model-report.md
[R020]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-020-status-availability-and-system-event-model-report.md
[R021]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-021-sensorml-representation-strategy-report.md
[R022]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-022-swe-common-data-component-strategy-report.md
[R023]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-023-schema-and-encoding-validation-strategy-report.md
[R024]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-024-units-observed-properties-and-semantic-binding-strategy-report.md
[R025]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-025-database-and-persistence-architecture-options-report.md
[R027]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-027-time-series-observation-storage-strategy-report.md
[R029]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-029-transaction-consistency-idempotency-and-concurrency-strategy-report.md
[R031]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-031-server-write-and-ingestion-model-report.md
[R034]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-034-datastream-observation-and-status-update-semantics-report.md
[R035]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-035-streaming-and-event-publication-strategy-report.md
[R036]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-036-control-stream-and-command-lifecycle-model-report.md
[R037]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-037-feasibility-and-asynchronous-tasking-strategy-report.md
[R039]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-039-authentication-authorization-and-api-security-threat-model-report.md
[R039a]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-039a-zero-trust-architecture-alignment-and-enforcement-model-report.md
[R040]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-040-policy-releasability-and-cross-boundary-access-constraints-report.md
[R042]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-042-ddil-informed-server-semantics-report.md
[R043]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-043-server-synchronization-and-conflict-handling-boundary-report.md
[R044]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-044-rust-implementation-language-and-framework-strategy-report.md
[R045]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-045-service-architecture-and-modularization-strategy-report.md
[R046]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-046-reference-deployment-strategy-report.md
[R049]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-049-migration-upgrade-backup-and-restore-strategy-report.md
[R050]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-050-conformance-harness-strategy-report.md
[R052]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-052-rust-test-driven-architecture-and-multi-layer-test-strategy-report.md
[R053]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-053-test-data-fixtures-golden-files-and-scenario-corpus-strategy-report.md
[R054]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-054-performance-load-stress-and-streaming-test-strategy-report.md
[R056]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-056-interoperability-test-matrix-for-external-csapi-clients-report.md
[R058]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-058-draft-csapi-part-4-sampling-features-study-report.md
[R059]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study-report.md
[R060]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-060-csapi-part-5-protobuf-first-implementation-study-report.md
[R061]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-061-provenance-and-related-metadata-interoperability-study-report.md
[R062]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-062-cs-go-engineering-practices-and-development-history-study-report.md
[RSynthesis]: ../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/final-idr-research-report.md
[S1]: https://docs.ogc.org/is/23-001/23-001.html
[S2]: https://docs.ogc.org/is/23-002/23-002.html
[SML]: https://docs.ogc.org/is/23-000/23-000.html
[SWE]: https://docs.ogc.org/is/24-014/24-014.html
[P3]: https://github.com/opengeospatial/ogcapi-connected-systems/tree/6f529a15bfa63259febc3620378d3e5a06305333/api/part3
[P4]: https://github.com/opengeospatial/ogcapi-connected-systems/tree/05a3c62d198ee52d0cf81a734b700967b7d864a1/api/part4
[P4Types]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/05a3c62d198ee52d0cf81a734b700967b7d864a1/api/part4/sections/clause_13_sampling_feature_types.adoc
[P4Spatial]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/05a3c62d198ee52d0cf81a734b700967b7d864a1/api/part4/openapi/schemas/samplingSpatial.json
[SFSchema]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/schemas/geojson/samplingFeature.json
[DataStreamSchema]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/schemas/json/dataStream.json
[ControlStreamSchema]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/schemas/json/controlStream.json
[ObservationSchema]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/schemas/json/observation.json
[F3]: https://docs.ogc.org/is/19-079r2/19-079r2.html
[CQL2]: https://docs.ogc.org/is/21-065r2/21-065r2.html
[CQLSchema]: https://schemas.opengis.net/cql2/1.0/cql2.json
[QualityPeer]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/lib-ogc/swe-common-core/src/main/java/org/vast/swe/SWEJsonBindings.java
[ReplaceIssue]: https://github.com/opengeospatial/ogcapi-connected-systems/issues/166
[PatchIssue]: https://github.com/opengeospatial/ogcapi-connected-systems/issues/170
[EventIssue]: https://github.com/opengeospatial/ogcapi-connected-systems/issues/201
[SSE]: https://html.spec.whatwg.org/multipage/server-sent-events.html
[CloudEvents]: https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/formats/json-format.md
[MQTT]: https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html
[MQTTClient]: https://docs.rs/rumqttc/latest/rumqttc/v5/index.html
[Broker]: https://mosquitto.org/documentation/authentication-methods/
[PGIsolation]: https://www.postgresql.org/docs/18/transaction-iso.html
[F4]: https://github.com/opengeospatial/ogcapi-features/tree/9ca25f56a58ed822ea8a685a7a41afa7181aaa8b/extensions/transactions
[F4Later]: https://github.com/opengeospatial/ogcapi-features/blob/4e30324a14b682ff4a26ee43aad1eb6428c846a3/extensions/transactions/create-replace-update-delete/standard/20-002.adoc
[ExampleMain]: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/planning/csapi-implementation-guide.md
[ExampleParser]: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/planning/phase-5/P5-parser-completion-implementation-guide.md

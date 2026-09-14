# Section 021: SensorML Representation Strategy - Research Report

**Topic ID:** IDR-SRV-021<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-021 SensorML Representation Strategy](../IDR%20Plans/idr-srv-021-sensorml-representation-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core questions, all detailed-question groups, all 6 methodology phases, and all 10 success criteria<br>
**Methodology Used:** Authority-ranked extraction from SensorML 3.0 and its JSON schemas; direct mapping against approved CSAPI Parts 1 and 2, tagged schemas, examples, and abstract tests; accepted AEP/STANAG and IDR-SRV-001 through IDR-SRV-020 carry-forward; bounded refresh of SensorML-owned official-repository history; and representation-boundary, import, preservation, security, validation, fixture, and interoperability analysis<br>
**Research Time:** Approximately 10 hours of AI-assisted execution on September 13–14, 2026<br>
**Official Standards Source Pin:** [`opengeospatial/ogcapi-connected-systems` `v1.0.0` at `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)<br>
**SensorML Edition:** OGC 23-000, Version 3.0, approved June 2, 2025 and published July 16, 2025<br>
**Shared Register Baseline:** OGC API - Connected Systems upstream-history register version 1.9; SensorML-owned entries and official `master` rechecked September 13, 2026 with no material register change required<br>
**Document Purpose:** Establish the server-side SensorML representation, normalization, source-preservation, transformation, validation, security, fixture, and downstream-design baseline for the Rust Glaux reference server<br>
**Author:** OpenAI Codex<br>
**Accepted By:** TBD pending Glaux Project Lead review<br>
**Acceptance Date:** TBD pending acceptance<br>
**Date:** September 14, 2026<br>
**Last Updated:** September 14, 2026

---

## Reading Guide and Evidence Labels

| Label | Meaning |
|---|---|
| **N** | Normative or standards-derived finding from an approved applicable source or incorporated artifact |
| **A** | Project-controlling AEP/STANAG adoption or operational-context finding carried from an accepted report |
| **P** | Glaux project decision or recommendation proposed for acceptance here |
| **I** | Informative implementation, test, interoperability, or community evidence |
| **D** | Official draft or post-publication direction that is not an approved requirement |
| **X** | Published ambiguity, artifact conflict, evidence gap, or unresolved project decision |

“SensorML document,” “CSAPI resource,” “Glaux domain entity,” and “stored artifact” are not synonyms. A SensorML document is one representation of an applicable resource. It can contribute authoritative evidence, but it does not become the entire canonical resource graph, an executable command contract, or a trusted policy decision merely because it parses.

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. SensorML Extraction Methodology
5. SensorML Concept Inventory
6. SensorML-to-CSAPI / Glaux Resource Mapping
7. Representation Boundary Findings
8. Import, Generation, Transformation, and Preservation
9. Identifiers, Classifiers, Contacts, and Metadata
10. Capabilities, Characteristics, Modes, Positions, Inputs, and Outputs
11. SWE Common, Semantic Binding, Temporal, Status/Event, and Command Dependencies
12. Validation, Profile, Schema, and Conformance
13. Security, Policy, and Releasability
14. Fixtures, Golden Files, and Interoperability
15. Downstream Topic Handoff Matrix
16. Recommendations
17. Risks, Constraints, and Open Questions
18. Validation Against Plan Success Criteria
19. References

---

## 1. Executive Summary

Glaux should implement SensorML 3.0 as a **typed, policy-aware representation over one encoding-neutral canonical resource graph**, not as the database schema and not as an opaque JSON blob that bypasses domain validation. The same System, Procedure, Deployment, or Property must retain one identity, lifecycle, relationship set, and revision history across SensorML, GeoJSON, JSON, HTML, and future representations. [N,A,P]

The recommended representation pipeline has five distinct records or views:

1. an immutable source artifact containing exact received bytes, media type, digest, source, retrieval/receipt time, declared version/profile, and handling controls;
2. a parsed SensorML document graph retaining recognized members, unknown extensions, references, and parse diagnostics;
3. a normalized Glaux projection containing the fields and relationships needed for identity, query, linking, authorization, cross-resource consistency, and deterministic generation;
4. an authorized generated SensorML view for the selected resource revision and request context; and
5. immutable validation and transformation records tied to the source artifact, normalized revision, schemas, profile, software version, and result. [P; IDR-SRV-015–019]

This design solves the two failure modes exposed by prior implementations. Storing independent `smljson` and `geojson` copies allows resources to disappear or disagree by representation; storing unvalidated heterogeneous JSON as the canonical record turns extension tolerance into an integrity bypass. Glaux instead keeps source fidelity without making source bytes the public truth, and it normalizes useful facts without discarding rich or unknown SensorML content. [I,P]

CSAPI Part 1's SensorML conformance class applies to Systems, Deployments, Procedures, and Property definitions. Sampling Features are not SensorML-encoded by that class. Part 2 DataStreams, Observations, ControlStreams, Commands, status records, results, feasibility resources, and System Events are not additional native SensorML feature representations, although they depend on SensorML-described processes, SWE Common components, and—in the published System Event schema—a SensorML Event shape. Glaux must not broaden the advertised class beyond the approved resource/encoding combinations. [N]

For Systems and Procedures, `PhysicalComponent`, `PhysicalSystem`, `SimpleProcess`, and `AggregateProcess` are representation classes, not endpoint identities. CSAPI requires physical classes for hardware/human observers or hardware datasheets and non-physical classes for simulations/processes or human procedures; Procedures must not contain position information. `typeOf` represents the System-to-Procedure system-kind relationship. Permanent addressable subsystems remain canonical System resources; inline SensorML components remain document-level process components unless they independently qualify for resource identity and lifecycle. [N,P; IDR-SRV-015–017]

SensorML metadata and execution descriptions also require disciplined boundaries:

- `validTime` qualifies the description/configuration and must not be collapsed with transaction, observation, result, event, or evaluation time;
- capabilities describe discoverable performance or operating envelopes, not present availability;
- characteristics describe properties such as dimensions, material, power demand, or expected lifetime and are not automatically CSAPI Property resources;
- modes/configuration describe allowed or selected settings but never authorize or execute a command;
- position may contain static geometry, 3D pose, a trajectory, or a process-produced dynamic state and therefore cannot always be flattened into one geometry column;
- inputs, outputs, parameters, capabilities, and characteristics depend on SWE Common, but their detailed component/encoding policy belongs to IDR-SRV-022;
- identifiers, classifiers, semantic `definition` URIs, and contacts carry authority and disclosure context; strings must not be treated as equivalent identifier types or trusted vocabulary solely by syntax; and
- SensorML history Events are descriptive source evidence and must not automatically create or replace canonical CSAPI System Event resources. [N,P]

Imported SensorML is untrusted input. The ordinary standards-facing write path should accept only representations that satisfy JSON syntax, the selected SensorML schema class, the CSAPI wrapper/mapping rules, Glaux cross-resource invariants, and applicable authorization/policy. A separate privileged import/quarantine workflow may preserve invalid, partial, external, legacy, or profile-divergent documents and diagnostics without creating a conformant public resource. Legacy SensorML 2.x XML may be retained or transformed by an explicitly versioned adapter, but it is not a SensorML 3.0 JSON representation and must not support a 3.0 conformance claim. [N,P]

Four upstream gaps require explicit adapters and fixtures rather than intuitive repair:

1. `DataInterface` exists in the SensorML conceptual model but is absent from the published JSON I/O choice;
2. Part 2 `outputName` and analogous `inputName` are not fully specified as bindings to SensorML output/input names;
3. SensorML `DerivedProperty.qualifier` and schema member `qualifiers` are omitted from the CSAPI Property conceptual/mapping tables; and
4. Part 2's conceptual System Event fields differ from its SensorML-derived published JSON schema. [N,X,D]

The project lead is asked to accept the layered representation architecture, normalization/preservation rules, resource mappings, import and generation contract, security posture, validation ladder, and downstream handoffs in this report. Acceptance would make IDR-SRV-022 the next eligible topic; it would not select a database, finalize SWE Common or semantic profiles, authorize arbitrary SensorML process execution, implement draft Part 3, or begin server implementation.

---

## 2. Scope and Plan Alignment

### 2.1 Completed Scope

This report:

- inventories the SensorML 3.0 concepts relevant to Glaux Server;
- maps SensorML classes and members to the accepted CSAPI/Glaux resource families;
- separates native representation, canonical domain state, source artifact, normalized index, derived view, and validation evidence;
- defines handling for generated, imported, partial, invalid, external, inherited, legacy, and profile-specific SensorML;
- establishes identity, classifier, contact, capability, characteristic, mode, configuration, position, component, input, output, and process-method boundaries;
- identifies SWE Common, semantic, temporal, provenance, status/event, command, persistence, security, conformance, fixture, and interoperability dependencies;
- incorporates accepted OSH, CS-Go, pygeoapi, SECD, smoke-test, interoperability, and community findings as nonnormative evidence; and
- records published/artifact conflicts and explicit downstream ownership.

### 2.2 Excluded Decisions

This report does not:

- choose Rust crates, database tables, document stores, search engines, resolver infrastructure, or cache products;
- define the complete SWE Common component and encoding subset (IDR-SRV-022);
- select the final schema-validation stack or repair every published schema issue (IDR-SRV-023);
- select unit, observed-property, controlled-property, ontology, or semantic-registry policy (IDR-SRV-024);
- define write transaction APIs, storage layout, or synchronization protocol (IDR-SRV-025–033, 042);
- define dynamic-data or command state machines (IDR-SRV-034, 036–038);
- define authentication, authorization, NATO releasability, audit, or redaction policy (IDR-SRV-039–041);
- adopt draft Part 3 or treat transport envelopes as SensorML documents; or
- implement the server.

### 2.3 Research Question Coverage

| Plan question group | Status | Evidence |
|---|---|---|
| Relevant SensorML concepts and authority | Complete | §§3–5 |
| Mapping to canonical resource families | Complete | §6 |
| Representation, storage, normalization, query, and opacity boundaries | Complete | §§7–8 |
| Identifiers, classifiers, contacts, metadata, trust | Complete | §9 |
| Capabilities, characteristics, modes, positions, I/O | Complete | §10 |
| SWE, semantics, temporal, status/event, command relationships | Complete | §11 |
| Import, generation, transformation, partial/legacy behavior | Complete | §8 |
| Validation, schema, profile, conformance | Complete with bounded downstream detail | §12 |
| Security, policy, releasability | Complete as requirements/handoffs | §13 |
| Implementation and interoperability lessons | Complete | §§3.4, 14 |

---

## 3. Evidence Base and Authority Classification

### 3.1 Primary Sources

| Source | Version/pin | Role | Accessed | Authority and limitation |
|---|---|---|---|---|
| [SensorML](https://docs.ogc.org/is/23-000/23-000.html) | OGC 23-000, 3.0; approved/published 2025 | Process, metadata, physical/process, configuration, deployment, derived-property, and JSON rules | 2026-09-13 | Controlling [N] |
| [Official SensorML schemas](https://schemas.opengis.net/sensorML/3.0/json/) | Versioned 3.0 registry | JSON implementation schemas | 2026-09-13 | Normative-support artifact; schema validity is not complete semantic/CSAPI validity [N] |
| [CSAPI Part 1](https://docs.ogc.org/is/23-001/23-001.html) | OGC 23-001, 1.0 | SensorML media type, resource mappings, classes, links, schemas, ATS | 2026-09-13 | Controlling for API representation [N] |
| [CSAPI Part 2](https://docs.ogc.org/is/23-002/23-002.html) | OGC 23-002, 1.0 | Dynamic resources and SensorML/SWE seams | 2026-09-13 | Controlling; identified prose/schema gaps remain [N,X] |
| [Tagged CSAPI source](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2) | `v1.0.0`, `8e03b236...` | Reproducible schemas, OAS, examples, SensorML files | 2026-09-13 | Tagged published-artifact provenance [N,X] |
| [SWE Common](https://docs.ogc.org/is/24-014/24-014.html) | OGC 24-014, 3.0 | Components used in I/O, parameters, characteristics, capabilities, settings, streams | 2026-09-13 | Controlling dependency; detailed strategy deferred [N] |
| [OGC API - Features Part 1](https://docs.ogc.org/is/17-069r4/17-069r4.html) | OGC 17-069r4, 1.0 | Feature/collection/HTTP inheritance | 2026-09-13 | Inherited where incorporated [N] |
| [SSN/SOSA](https://www.w3.org/TR/vocab-ssn/) | W3C/OGC Recommendation, 2017 with published namespace | Semantic cross-check for systems, procedures, capabilities, deployment | 2026-09-13 | Supporting semantics; does not create extra CSAPI members [N/I] |

The tagged tree records exact Git object IDs for the principal SensorML schemas, including `DescribedObject.json` (`557184f...`), `AbstractProcess.json` (`8d9f246...`), `Deployment.json` (`a525602...`), and `DerivedProperty.json` (`1288456...`). The tagged SensorML/CSAPI source entered the publication merge at commit `8e03b236` on July 16, 2025. These pins make the artifact findings reproducible even if registry metadata or repository `master` changes.

### 3.2 Controlled AEP/STANAG Boundary

The controlled project source remains NATO package `AC/224(JCGISR)D(2026)0005`, dated April 27, 2026, SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`. This report neither redistributes nor newly quotes it. Accepted IDR-SRV-001 through 003 establish that AEP-4789 Volume II adopts CSAPI Parts 1 and 2, SensorML 3.0, and SWE Common 3.0 as a coherent technical package and assigns SensorML the rich system/process-description role. The accepted findings support trustworthy, secure, linked, temporally qualified descriptions under operational and DDIL conditions; they do not create undocumented SensorML members, mandate arbitrary process execution, or supply a complete security/profile vocabulary. [A]

### 3.3 Accepted Project Baselines

| Accepted source | Controlling carry-forward for this topic |
|---|---|
| IDR-SRV-003 | Part 1 owns resource graph; Part 2 owns dynamic interaction; SensorML owns rich descriptions; SWE owns data structures/encodings |
| IDR-SRV-004 | Qualify `System`, `Procedure`, SensorML Process, capability, status, Property, `definition`, and identifier terms |
| IDR-SRV-008/012 | Advertise only tested class/media combinations; use `application/sml+json`; preserve editorial conflicts |
| IDR-SRV-015 | One canonical encoding-neutral resource graph; representation objects and schemas are not independent domain entities |
| IDR-SRV-016 | Distinguish UUIDv7 local `ResourceId`, UID, canonical URL, external identifier, alias, revision, event, and storage identities |
| IDR-SRV-017 | Typed authoritative relationships generate representation links; exact Part 1 `ogc-rel:` vocabulary; policy-aware graph |
| IDR-SRV-018 | Separate valid/effective, phenomenon, result, event, transaction, evaluation, and freshness time; request-scoped current/as-of |
| IDR-SRV-019 | Preserve source fidelity and transformations; use scoped quality/trust evidence, not a universal trust score |
| IDR-SRV-020 | Capabilities are not current availability; history Event, System Event, status Observation, and transport event remain distinct |

### 3.4 Implementation and Community Evidence

- OSH demonstrates separate resource handlers, SensorML/GeoJSON/SWE bindings, typed stores, and broad tests. Its member-order-dependent parser defect and legacy XML breadth show why Glaux needs unordered-JSON and declared-profile tests. [I]
- CS-Go demonstrates typed format adapters, strict JSON decoding, schema validation, heterogeneous SensorML/SWE persistence, and substantial formatter/E2E fixtures. JSONB is useful for preserved structures but is not an integrity boundary by itself. [I]
- The pygeoapi/52°North PoC stored independent SensorML and GeoJSON documents; a captured deployment exposed populated SensorML collections and empty JSON views for the same roots. This is direct evidence against representation-dependent populations and canonical truth. [I]
- SECD returned ordinary JSON for SensorML and invalid media requests in bounded probes. This supports exact negotiation, body-schema, and `Content-Type` tests; it does not define acceptable behavior. [I]
- OS4CSAPI evidence shows clients depend on consistent IDs/counts/relationships across representations and that recursive SensorML/SWE schemas can exceed naïve tool limits. [I]
- The reported “SensorML publisher loss” was adjudicated as a client payload/decoder mismatch, not a new CS-Go server regression. Its retained lesson is to preserve finding dispositions and test the complete producer-server path. [I]

### 3.5 Upstream-History Refresh and Conflicts

The official repository's `master` remained `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f` on September 13, 2026. The SensorML-owned register entries were rechecked; no issue state or merged source change altered the version 1.9 register. The following still matter:

| Evidence | Status | Meaning here |
|---|---|---|
| #39 | Closed/published | Position alternatives restored; retain full position union, not geometry only |
| #45 | Open | Conceptual `DataInterface` is missing from JSON I/O choice; do not invent a standard JSON form |
| #46 | Closed/published | SensorML/SWE 3.0 migration and XML removal control the baseline |
| #109–113, #125–126, #153 | Mixed/mostly editorial | Publication corrections exist; residual editorial items do not change this architecture |
| #147 | Open | `outputName`/`inputName` stream-to-SensorML binding remains incompletely published |
| #162 | Open | `qualifier`/`qualifiers` is schema-valid but omitted from CSAPI conceptual/mapping tables |
| PR #156/#158 | Published | Registry schema references and final SensorML/SWE references were corrected before publication |

Authority resolution follows three rules: approved requirements and incorporated schemas control; a prose/schema conflict remains visible; and a project adapter is labeled and tested rather than represented as OGC behavior. [N,X,P]

---

## 4. SensorML Extraction Methodology

### 4.1 Extraction Unit

Each candidate concept was recorded with:

- class/member and source anchor;
- authority class and relevant conformance class;
- applicable CSAPI/Glaux resource family;
- representation placement and class-selection rule;
- canonical normalization, indexing, preservation, and reference-resolution need;
- structural, semantic, profile, cross-resource, and policy validation need;
- SWE/semantic/temporal/status/command dependency;
- sensitivity and releasability risk;
- positive, negative, round-trip, and interoperability fixture implication; and
- downstream owner and unresolved issue.

### 4.2 Decision Tests

A SensorML fact is normalized only when at least one condition holds:

1. CSAPI maps it to an API resource attribute or association;
2. Glaux needs it for identity, lifecycle, authorization, policy, query, link generation, validation, or cross-representation equivalence;
3. a downstream operation must bind to it reliably, such as a stream output name or a Procedure reference; or
4. a selected Glaux/AEP profile makes it a governed field.

A fact remains source-preserved or document-native when it is rich, recursive, rarely queried, vendor/profile-specific, unsafe to interpret, semantically unresolved, or not needed to maintain a canonical invariant. “Not normalized” does not mean discarded: the source bytes, parsed member, validation result, provenance, and policy controls remain available according to authorization. [P]

### 4.3 Classification Vocabulary

Mappings are classified as **direct**, **profiled**, **derived**, **partial**, **document-only**, or **unresolved**. Storage roles are described logically as **canonical field/relation**, **preserved source**, **parsed document**, **derived index/view**, **validation artifact**, or **quarantine artifact**. These are design roles, not selected database technologies.

---

## 5. SensorML Concept Inventory

### 5.1 Core Process Model

SensorML models physical devices, sensors, actuators, platforms, human observers, and computational processes through a shared process abstraction: inputs plus parameters and a method produce outputs. Its metadata supports identification, discovery, and qualification but must not be required for process execution. Extensions must use a non-SensorML namespace and must not alter or be required for execution. These rules make SensorML powerful for description while also establishing why Glaux must not execute arbitrary imported content or hide command semantics in extensions. [SensorML §§1, 7, 8.2; Requirements 4–5, 9–10] [N]

| Class | Standards meaning | Glaux representation use | Boundary |
|---|---|---|---|
| `DescribedObject` | Base identity/label/metadata/extension model | Common metadata substrate | Not independently a CSAPI resource class |
| `AbstractProcess` | Common process I/O, parameters, `typeOf`, FOI, configuration, modes | Shared process projection | Abstract; not accepted as a concrete wire class |
| `SimpleProcess` | Indivisible non-physical/logical process; location unimportant | Simulation or human/computational Procedure/System | Method required by concrete schema/model |
| `AggregateProcess` | Non-physical process composed of subprocesses and explicit connections | Workflow/process chain | Components are not automatically API resources |
| `PhysicalComponent` | Indivisible physical processing device; location important | Hardware/human observer or hardware datasheet | Physical class; supports attachment/position |
| `PhysicalSystem` | Physical aggregate with components/connections and spatial context | Composite hardware/platform/System/datasheet | Permanent addressable subsystems remain canonical resources |
| `Deployment` | When, where, why, and how systems are deployed | Native CSAPI Deployment representation | `deployedSystems` are embedded association descriptors, not separate CSAPI family |
| `DeployedSystem` | System reference plus local deployment description/configuration | Qualified Deployment→System association | No separate canonical resource family per accepted model |
| `DerivedProperty` | Domain property derived from base property/object/statistic/qualifiers | Native CSAPI Property representation | Property identity distinct from semantic URIs |

### 5.2 Metadata Inventory

The published `DescribedObject.json` requires `type`, `label`, and `uniqueId` and exposes `id`, description, language, keywords, identifiers, classifiers, valid time, security/legal constraints, characteristics, capabilities, contacts, documents, and history. CSAPI wrappers further constrain class and require resource-specific `definition`/`uniqueId` combinations. Requiredness must be evaluated through the complete composed schema and CSAPI mapping, not from a single local fragment. [N]

Identifiers use SWE `Term`: `definition` identifies the identifier type, `codeSpace` its authority, and `value` the designation. Classifiers serve categorization and discovery. Keywords are unqualified tokens. Contacts use ISO 19115 responsible-party structures; documents use online-resource structures. Security constraints can mark the whole document, with property-level marking possible through extensions. Legal constraints cover privacy, intellectual-property, and ethical use. [SensorML §§8.2.2–8.2.7] [N]

### 5.3 Process Detail Inventory

- `inputs`, `outputs`, and `parameters` accept `ObservableProperty` or applicable SWE Common data components/streams. Tightly related values must use an aggregate such as `DataRecord`. [N]
- `typeOf` is a resolvable link to a more general process. With no configuration, the complete description combines the specific and referenced documents; with configuration, settings restrict or select the inherited options. [N]
- `featuresOfInterest` assists discovery and explains purpose; it is not a substitute for CSAPI Sampling Feature resources or their typed relationships. [N,P]
- `modes`, if present, contain at least two Mode entries; each Mode may carry configuration. Settings can set values/arrays/constraints/modes and enable or disable I/O. [N]
- `components` and `connections` express composition and data flow for aggregate/physical systems. They do not by themselves create CSAPI resource identity, persistence ownership, or command wiring. [N,P]
- `attachedTo` means parent movement affects child position; it is the SensorML mapping for CSAPI parent System in the applicable representation. [N]
- position can be location/point, GeoPose/relative pose, trajectory, or process/datastream-produced dynamic state. Every nontrivial pose requires explicit reference-frame semantics. [N]
- `method`/`ProcessMethod` describes how a simple/physical component transforms input and parameters into output. A description can reference protected methods; possession of a method description is not authorization to execute it. [N,P]

### 5.4 Deployment, History, and Property Inventory

A Deployment inherits descriptive metadata and adds location, platform, and deployed-system descriptions. A DeployedSystem requires a system reference and may include deployment-local configuration. This supports a qualified many-to-many Deployment→System fact, not ownership or cascade deletion of the System. [N; IDR-SRV-017]

SensorML history is an Event collection covering calibration, maintenance, algorithm/parameter changes, and deployment history. It is part of a process description and can provide source evidence. CSAPI Part 2 System Event is a durable API resource with its own identity, association, query, and published schema gap. Glaux must preserve both concepts and explicitly transform only when a profile says a source history event warrants a canonical System Event. [N,P; IDR-SRV-020]

DerivedProperty supplies identifier, label, description, base property, object type, statistic, and qualifier. CSAPI Part 1 maps the first three semantic dimensions but its published mapping table omits qualifier, while the incorporated SensorML schema includes plural `qualifiers`. Glaux should accept/preserve schema-valid qualifiers and expose them only under the selected published-schema/compatibility interpretation until an upstream/profile decision is recorded. [N,X,P]

---

## 6. SensorML-to-CSAPI / Glaux Resource Mapping

### 6.1 Applicable Native Representations

| CSAPI/Glaux family | Native `application/sml+json` under Part 1 class | SensorML class/shape | Canonical rule |
|---|---|---|---|
| System | Yes | `PhysicalComponent`, `PhysicalSystem`, `SimpleProcess`, or `AggregateProcess` by subject | One System resource; `id` is local ResourceId; `uniqueId` is UID |
| Procedure | Yes | Physical classes for hardware datasheets; non-physical classes for human/method processes | One Procedure resource; no `position` |
| Deployment | Yes | `Deployment` | One Deployment plus qualified platform/deployed-System relationships |
| Property | Yes | `DerivedProperty` | One Property identity distinct from base/statistic/object/qualifier semantic URIs |
| Sampling Feature / generic Feature | No | None under Part 1 SensorML class | Use GeoJSON/other advertised representation; SensorML FOI links are context only |
| DataStream / Observation / ControlStream / Command / Feasibility / result/status | No native Part 1 SensorML feature mapping | Depend on process/SWE descriptions | Remain Part 2 resources |
| System Event | No general Part 1 mapping | Published Part 2 schema composes SensorML `Event` | Use explicit event wire adapter from IDR-SRV-020 |

### 6.2 Required Mapping Matrix

Abbreviations: **CF** canonical field; **CR** canonical relationship; **PS** preserved source; **PD** parsed document; **DV** derived view/index; **VA** validation artifact; **Q** quarantine. “Index” means policy-scoped indexing, never unconditional disclosure.

| SensorML concept | Source anchor | Authority | Related family | Representation pattern | Internal normalization | Source preservation | Query/index | Validation | SWE dependency | Semantic binding | Security | Test | Downstream | Notes/unresolved |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Concrete process class | SML §§8.3–8.6; P1 R95/R100 | N | System, Procedure | `type` discriminator | CF class/subject-kind | PS+PD | type/profile | class allowed for endpoint/subject | indirect | class/type URI | reveals asset kind | all 4 classes × endpoints | 023,024,053 | Procedure position prohibition |
| Local `id` | P1 R92 | N | all four | JSON `id` equals URL `{id}` | CF `ResourceId` | PS | exact lookup | route/body equality | no | no | identifier leakage | create/read/replace | 023,031,032 | Never trust imported external `id` |
| `uniqueId` | P1 R93; SML base schema | N | all four | URI-valued UID | CF UID with collision policy | PS | exact authorized UID | URI, uniqueness, resource-kind scope | no | identity URI | correlation risk | collisions/aliases | 023,031,053 | Not canonical URL or DB key |
| `label`/`description` | P1 Table 48 | N | all four | common names | CF | PS | text where profile permits | length/language/policy | no | optional language | may reveal mission | cross-format equality | 023,024,040 | Preserve original language form |
| `definition` | SML/P1 Tables 49/51/53 | N | System, Procedure, Deployment | type URI/CURIE | CF resolved + lexical source | PS | semantic/type index | allowed vocabulary/profile | no | central | asset/mission inference | URI/CURIE variants | 024,040,053 | CURIE context must be explicit |
| Keywords | SML §8.2.2.2 | N | all four | unqualified tokens | selected DV | PS+PD | opt-in text/facet | syntax/profile | no | weak/unqualified | disclosure/poisoning | case/lang/duplicates | 024,028,040 | Do not treat as ontology terms |
| Identifiers | SML §8.2.2.3 | N | all four | typed `Term` list | CF alternate identifiers where governed | PS+PD | exact type/authority/value | type/codeSpace/collision | SWE Term | authority URI | high correlation | duplicate/conflict | 024,028,040 | Map through IDR-016 types |
| Classifiers / asset type | SML §8.2.2.4; P1 Table 49 | N | System primarily; all described objects | Term list; `cs:AssetType` mapping | selected CF/DV | PS+PD | controlled facets | vocabulary/profile | SWE Term | required | capability inference | known/unknown code | 024,040,053 | Generic classifiers not all first-class |
| `validTime` | SML §8.2.2.6; P1 mappings | N | System, Procedure, Deployment | description/resource interval | CF temporal fact | PS | current/as-of | interval/profile consistency | time component | temporal semantics | historical location/capability | bounds/revisions | 023,025,028 | Not transaction/freshness time |
| Security/legal constraints | SML §§8.2.2.5, 8.2.3 | N | all four/artifacts | external model / metadata | policy cues, not truth | exact PS essential | no general index; secure policy index | recognized marking + fail-safe unknown | extension/SWE possible | external vocabulary | critical | mixed markings/redaction | 039–041 | Marking does not enforce access |
| Contacts/documents | SML §§8.2.6–8.2.7 | N | all four | responsible party/online resource | normalize governed references minimally | PS+PD | restricted | URI/contact/profile | no | roles | PII/endpoints | field-level authorization | 028,039,040 | Avoid public full-text by default |
| Capabilities | SML §8.2.4 | N | System, Procedure | named SWE `DataRecord` groups | selected typed DV/profile fields | PS+PD | controlled discovery | SWE structure/semantics/range | high | definitions/units | sensitive performance | bounds/profile | 022–024,040 | Never current availability |
| Characteristics | SML §8.2.5 | N | System, Procedure, Deployment | named SWE `DataRecord` groups | selective DV | PS+PD | profile-specific | SWE/schema/profile | high | definitions/units | asset signature | physical/power examples | 022–024,040 | Not automatically CSAPI Property |
| `typeOf` | SML §8.2.9.3; P1 Table 50 | N | System→Procedure | resolvable weblink | CR plus resolution snapshot | exact PS | relationship query | target type, cycles, revision, policy | inherited content | target semantics | SSRF/link disclosure | local/external/cycle | 023,025,028,040 | Preserve base+overlay; derived resolved view |
| `attachedTo` | SML §8.5.1.1; P1 Table 50 | N | System→System | direct parent weblink | CR parent relation | PS | hierarchy | no self/cycle/cardinality | no | physical attachment | graph disclosure | reparent/cycle | 025,030,040 | Stronger meaning than arbitrary containment |
| Components/connections | SML §§8.4, 8.6 | N | System/Procedure document; possible System graph | inline/ref process graph | normalize only independent resources and critical refs | PS+PD | optional names/types | recursive graph, endpoints, cycles, port refs | high | component/port definitions | topology sensitive | nested/cycle/order | 022,023,028,040 | Inline component != canonical subsystem automatically |
| Position/reference frames | SML §8.5.1.2–.4; P1 Table 49 | N | System | GeoJSON, pose, trajectory, process/stream | CF static geometry/pose summary; CR dynamic source | PS+PD | spatial only when safe/defined | CRS/frame/time/class | possible DataRecord/Array | CRS/axis semantics | highly sensitive | point/pose/trajectory/process | 022–026,034,040 | Procedures prohibit position |
| Inputs/outputs/parameters | SML §8.2.9.1 | N | System, Procedure; stream/command seam | named ObservableProperty/SWE component | names, definitions, selected typed schema refs | PS+PD | semantic/name index | SWE class/name/aggregate consistency | central | property/unit | command/capability exposure | scalar/record/unknown | 022–024,034,036 | Description does not create stream automatically |
| `DataInterface` | SML conceptual model; issue #45 | N,X,D | System/Procedure output | model permits interface; JSON schema omits it | none beyond preserved evidence until profile | PS+PD/Q | no | flag unsupported JSON form | SWE DataStream | interface semantics | endpoint leakage/SSRF | unresolved fixtures | 022,023,035 | Do not invent OGC JSON encoding |
| Modes/configuration/settings | SML §§8.2.9.3, 8.8 | N | System, Procedure, Deployment association | base options + instance restrictions/selections | selected declarative facts | PS+PD | capability/mode facet if governed | references, min two modes, allowed restrictions | high | definitions/constraints | command/safety sensitive | inherited/missing/ref errors | 022–024,036–040 | Never authorization or execution by itself |
| FOI references | SML §8.2.9.2; P1 associations | N | System/Deployment context; Sampling Feature/Feature | references/generic links | CR only where CSAPI relationship exists | PS | relationship query | target kind/scope/policy | no | feature semantics | mission/location | local/external/hidden | 024,026,040 | Do not synthesize resource from embedded URI |
| Deployment location/platform | SML §8.9; P1 Tables 51–52 | N | Deployment | GeoJSON location + platform link | CF + CR | PS | spatial/relationship | geometry, target, validity | no | deployment type | mission sensitive | temporal/platform | 023,025,026,040 | Platform can be System or external Feature |
| Deployed systems/configuration | SML §8.9; P1 Table 52 | N | Deployment→System | embedded qualified association | CR/fact with config evidence | PS+PD | deployment/system | local/external ref, duplicates, validity | settings | role/config semantics | operationally sensitive | multi-system/config | 022,025,028,040 | No separate DeployedSystem resource |
| History Event | SML §8.2.8 | N | source description; System Event seam | inline Event list | source evidence; optional explicit transform | PS+PD | restricted derived event index | event/time/type/provenance | Event properties may use SWE | event vocabulary | operational history | idempotent transform | 020,023,025,028,040 | Not automatically CSAPI System Event |
| DerivedProperty dimensions | SML §8.10; P1 R102–103 | N,X | Property | `baseProperty`, `objectType`, `statistic`, `qualifiers` | CF semantic refs/qualifiers | PS | semantic facets | URI, cycle, profile | qualifier may use SWE | central | semantic inference | schema/table conflict | 023,024,053 | P1 tables omit qualifier(s) |
| General links | P1 R91, Tables 50/52/54 | N,X | applicable relations | `links[]`, direct members, `typeOf`, `attachedTo` | CR is authority | PS | relationship | exact rel/target/policy | no | link relation | graph disclosure | prefixed/bare fixtures | 023,040,053 | Follow accepted IDR-017 exact `ogc-rel:` output |
| Extension | SML R9–10 | N | any described object | non-SensorML namespace content | only selected registered projection | exact PS+PD | not indexed by default | namespace, size, no execution dependency | possible | profile-owned | code/data injection | unknown round-trip | 023,028,039 | Preserve but do not trust/execute |

### 6.3 Relationship Projection Rules

The canonical relationship graph is authoritative. SensorML generation projects it into the encoding-specific location:

- System `systemKind` → `typeOf` Procedure link;
- parent System → `attachedTo`;
- System association collections → `links[]` with accepted exact `ogc-rel:` relations;
- Deployment platform → `platform`; participating Systems → `deployedSystems[].system`;
- Procedure implementing Systems → `links[]`; and
- Property base reference → `baseProperty` semantic/resource reference according to the selected profile.

Inbound direct members and links are parsed into candidate relationship facts, checked against endpoint/resource kind, target visibility, cardinality, lifecycle, cycles, and authorization, then committed atomically. They are not stored as independent truth alongside a conflicting graph. [P; IDR-SRV-017]

---

## 7. Representation Boundary Findings

### 7.1 Five-Layer Representation Architecture

| Layer | Purpose | Mutability/authority | Must not become |
|---|---|---|---|
| Source artifact | Exact received bytes and acquisition context | Immutable evidence | Public response by default |
| Parsed SensorML document | Navigable standards/profile content and unknown extensions | Derived from exact artifact; parser-versioned | Canonical relationship/lifecycle truth |
| Canonical Glaux projection | Identity, resource kind, lifecycle, relationships, temporal facts, governed metadata | Transactionally authoritative inside Glaux | SensorML-specific persistence schema |
| Authorized wire view | Deterministic SensorML for resource revision/request policy | Regenerable projection | Blind echo of imported source |
| Validation/transformation evidence | Reproducible diagnostics and lineage | Append-oriented/immutable | One Boolean `valid` or `trusted` flag |

This architecture is logical. IDR-SRV-028 will choose persistence mechanisms and retention, but must preserve the distinctions and atomic linkage among artifact, normalized revision, transformation, and validation result. [P]

### 7.2 Canonical Ownership

The Glaux resource owns local ID, UID policy, canonical URL, resource family, lifecycle, revision, name/description, mapped type, valid-time fact, and accepted relationships. SensorML fields are projections of those facts. Rich SensorML members can be canonically owned by a document revision when no broader Glaux behavior depends on them; the ownership registry must say which side wins on write. A cross-encoding update cannot leave two authoritative copies. [P]

Recommended ownership classes:

- **Always normalized:** `id`, `uniqueId`, `label`, description, concrete class, mapped `definition`, `validTime`, `typeOf`, `attachedTo`, platform, deployed-System associations, and standard relationship links.
- **Normalize selected structure:** identifier/classifier typed terms, static geometry/pose summary, I/O names and definition references, component references that are independent resources, and profile-governed capability/characteristic keys.
- **Preserve and parse, usually not flatten:** full SWE component trees, process methods, inline process graphs, settings/modes, reference frames, trajectories, contacts/documents/history, extensions, and unprofiled metadata.
- **Never infer as facts solely from syntax:** trust, current availability, executable authority, semantic equivalence, releasability, resource ownership, or creation of streams/subresources. [P]

### 7.3 Inline Components Versus Canonical Resources

An inline SensorML component is promoted to a canonical System only through an explicit registration decision that allocates a local ID, validates UID collisions, establishes parent/relationship facts, records provenance, and applies authorization. Conversely, an addressable CSAPI subsystem is emitted as a link/reference or bounded representation according to the wire profile; it is not duplicated as an independently mutable anonymous subtree. Inline logical steps within AggregateProcess ordinarily remain document nodes. [P]

### 7.4 Inheritance and Resolution

Glaux must preserve the original `typeOf` link, base document/revision, overlay document, and configuration separately. A **resolved description** is a derived, provenance-bearing view—not a destructive flattening. Resolution must be bounded by allowed schemes/origins, authorization, response size, redirect count, timeout, recursion depth, and cycle detection; it records retrieved bytes, digest, effective source/revision, cache/freshness state, and unresolved reason. External content is never fetched synchronously as an uncontrolled side effect of ordinary public GET or write validation. [N,P]

If the base is unavailable, hidden, changed, invalid, or cyclic, the source document can remain preserved while the resolved view is incomplete. Public generation must either use a validated pinned base, omit nonrequired derived content with an explicit profile rule, or fail deterministically; it must not silently resolve against mutable latest content. [P]

### 7.5 Query and Index Boundary

Normalize/index only deliberate query surfaces. Index values with their type URI, code space, language, units, source document/revision, validity, and policy label. Unknown extension strings, contact fields, opaque methods, and ungoverned capability names are not automatically full-text indexed. Counts/facets must be authorization-filtered because even a hidden value's existence can be sensitive. [P]

---

## 8. Import, Generation, Transformation, and Preservation

### 8.1 Import Classes

| Input class | Handling | Canonical/public effect |
|---|---|---|
| Valid CSAPI SensorML write | Parse, validate all applicable layers, normalize, authorize, commit atomically | Creates/replaces resource per later write rules |
| Valid generic SensorML but not valid CSAPI representation | Preserve in privileged import workflow with diagnostics; map only through explicit profile/transformation | No conformant public resource until accepted normalization |
| Partial/invalid SensorML | Preserve exact bytes in quarantine if authorized; produce bounded diagnostics | No public resource or conformance effect |
| Externally authored referenced SensorML | Fetch only through controlled resolver; preserve retrieval evidence and pin digest/revision | Never trusted by origin alone |
| Legacy SensorML 2.x XML | Preserve with exact media/version; optionally transform using versioned adapter | Not a SensorML 3.0 JSON representation/claim |
| Profile-specific extension | Preserve exact extension; parse/normalize only registered profile vocabulary | Expose only if profile and policy permit |
| Generated Glaux SensorML | Deterministic projection from one canonical revision and policy context | Preferred public representation |

### 8.2 Ordinary Write Versus Quarantine Import

The standards-facing create/replace route for `application/sml+json` must reject a payload that fails applicable structural, CSAPI mapping, cross-resource, or authorization rules. Preserving bad bytes is not the same as accepting a resource. A separate administrative import endpoint/job may accept untrusted material into quarantine, but it must be capability-scoped, size-limited, malware/content scanned as appropriate, isolated from normal queries, and explicit that no CSAPI resource exists until review/normalization succeeds. [P]

Errors should use the accepted problem-details model and distinguish unsupported media/version, malformed JSON, schema violation, mapping violation, semantic/profile failure, target conflict, external-reference failure, and policy denial without echoing sensitive source fragments. Detailed validation artifacts may be available through privileged diagnostics. [P; IDR-SRV-013]

### 8.3 Identifier and Link Import

The server allocates or confirms the local UUIDv7 ResourceId according to IDR-SRV-016. An inbound `id` from an external document never silently becomes the local ID. The UID is checked for syntax, collision, alias/replacement history, resource-family scope, and caller authority. URLs, `typeOf`, component refs, external feature refs, documents, and links are untrusted network identifiers subject to SSRF-safe resolution policy; parsing a URL does not dereference it. [P]

### 8.4 Deterministic Generation

Generated SensorML must:

- select the concrete class using subject and endpoint rules;
- emit canonical `id`, UID, mapped common fields, and links from one revision/snapshot;
- satisfy the exact selected schema/profile class;
- omit Procedure position even if an internal procedure-related record has geospatial provenance;
- apply policy before serialization, including relationship, extension, contact, capability, history, and geometry visibility;
- emit stable ordering where useful for diffs while never making order semantically significant;
- record serializer/profile/schema versions and source revision;
- produce truthful `Content-Type`, `Content-Location`/links, ETag, cache controls, and `Vary` behavior from accepted HTTP baselines; and
- validate representative generated outputs continuously and all outputs at a risk-appropriate boundary determined in IDR-SRV-023. [N,P]

### 8.5 Round-Trip Contract

The required invariant is **semantic and evidence-preserving equivalence**, not byte equality:

- common identity, resource kind, mapped attributes, relationships, validity, and supported rich members survive SensorML read→write→read;
- unknown/profile extensions survive when the operation promises preservation and policy permits;
- exact original bytes remain retrievable as a restricted source artifact even if generated JSON changes whitespace/order;
- transformations identify every dropped, defaulted, coerced, redacted, resolved, or synthesized value; and
- alternate encodings return the same authorized resource population and common facts, though format-inapplicable members may differ. [P]

### 8.6 Legacy and Version Transformation

SensorML 3.0 supersedes 2.1, adds JSON/Deployment/DerivedProperty, and removes XML encodings while claiming model-level backward compatibility. Glaux therefore treats XML conversion as an explicit, potentially lossy migration—not as native 3.0 parsing. The transformation record must name input/output versions, adapter/software version, mapping policy, lost/unmapped content, warnings, actor, times, source/output digests, and validation results. Failed conversion leaves the original intact. [N,P]

---

## 9. Identifiers, Classifiers, Contacts, and Metadata

### 9.1 Identifier Algebra

SensorML/CSAPI fields map into the accepted identifier taxonomy as follows:

| Wire member | Glaux type | Rule |
|---|---|---|
| `id` | local `ResourceId` | UUIDv7 canonical route segment; URL/body equality |
| `uniqueId` | resource UID | URI syntax; uniqueness/collision/replacement policy |
| `identifiers[].value` | alternate designation | Qualified by identifier-type `definition` and optional `codeSpace` authority |
| `definition` | class/property semantic URI | Never a resource ID merely because it is a URI |
| `typeOf.href`, links, document URLs | resource/reference locator | May be local or external; resolution and trust separate |
| artifact digest | content identity | Identifies exact bytes, not the described System |
| revision ID/ETag | representation/resource revision evidence | Not interchangeable with UID or URL |

Lexical source values are preserved even when Glaux stores a resolved URI form, so generation, audit, and profile migration can distinguish original CURIEs from expanded identifiers. [P]

### 9.2 Discovery Metadata

Classifiers and identifiers can support exact/faceted discovery only with type and authority retained. Keywords support unqualified text search. Capabilities support further filtering after candidate discovery. Glaux should publish an explicit index allow-list and profile; it must not make every extension/property queryable or imply semantic equivalence from matching labels. [N,P]

### 9.3 Contacts, Documents, and History

Contact records can contain PII, organization, address, phone/email, operational roles, maintainers, owners, or pilots. Document links can expose internal endpoints, manuals, credentials in URLs, or classified technical detail. History can expose maintenance cycles and operational tempo. They require field/relationship-aware authorization and source-specific provenance. Redaction must operate on the parsed/canonical projection before serialization, not through textual substitution of final JSON. [N,P]

When policy removes a member, the output must remain schema/profile valid and semantically honest. If a required field or relationship cannot be released, return a policy-defined whole-resource concealment/error outcome or a separately valid releasable representation; never emit a structurally misleading object. [P; IDR-SRV-017]

### 9.4 Metadata Authority

Imported labels, manufacturer identifiers, classifiers, calibration claims, capabilities, and security markings are assertions by a source. Store assertion provenance, scope, validity, verification state, and conflicts. A valid `securityConstraints` object can inform handling but cannot self-authorize broader access or downgrade an existing policy label. [P; IDR-SRV-019]

---

## 10. Capabilities, Characteristics, Modes, Positions, Inputs, and Outputs

### 10.1 Non-Collapse Rules

| Concept | Means | Must not imply |
|---|---|---|
| Capability | Discoverable performance/operating property qualifying I/O | Current availability, readiness, feasibility, or authorization |
| Characteristic | Descriptive property not directly qualifying output | CSAPI Property resource or dynamic status |
| Mode | Named collection of permitted/configured settings | Current observed operating state unless separately evidenced |
| Configuration | Selected/restricted settings relative to a process description | Authorized command or proof device accepted/applied it |
| Input/output | Declared process interface/property structure | Existing ControlStream/DataStream |
| Position | Location/orientation or dynamic-state description relative to a frame | Always a static GeoJSON geometry |
| Component | Node in a SensorML process/system graph | Automatically a canonical subsystem resource |

These distinctions continue IDR-SRV-020's rule that descriptive capability is not current status. Runtime facts come from authorized observations, stream state, command/status records, or other traced evidence. [N,P]

### 10.2 Capability and Characteristic Normalization

Preserve the complete named SWE records. Normalize only profile-recognized definitions whose units, constraints, scope, and semantics are known. A numeric value without definition/unit/context is not safe for cross-system comparison. Indexing a declared range should retain whether it is design, measurement, operating, survival, or configured range; Glaux must not infer these categories from labels. Detailed component and unit rules belong to IDR-SRV-022/024. [P]

Capabilities and characteristics do not automatically become CSAPI Property resources. A Property resource defines a reusable observed/controlled semantic property; a capability/characteristic record supplies values about a process. A profile may link them, but it must preserve the distinction between property definition and property value/assertion. [N,P]

### 10.3 Modes and Configuration

Glaux should preserve base modes, instance configuration, deployment-local configuration, reference paths, and source provenance. A resolved effective configuration can be derived only against a pinned base revision with cycle-safe path validation. References must resolve to declared configurable parameters/I/O/modes; constraints must narrow rather than silently expand the base; and conflicting settings must be reported. [N,P]

Command generation or execution requires the later ControlStream/Command schema, authorization, feasibility, safety, idempotency, and audit pipeline. SensorML configuration can inform discovery and validation but cannot bypass those gates. [P]

### 10.4 Position Strategy

Glaux should preserve the full SensorML position union and normalize a queryable spatial projection only when transformation is well-defined:

- GeoJSON location can populate the canonical spatial footprint under the accepted CRS/axis policy;
- GeoPose/relative pose retains orientation and external/intrinsic frame identifiers in addition to any derived point;
- trajectory is time-varying evidence and should link to a dynamic representation/stream rather than become a single “current” point;
- by-process/by-datastream position retains a typed relationship to the producing process/stream and uses IDR-SRV-018 current/as-of evaluation; and
- an attached child's effective pose can be derived only from parent pose, relative transform, time alignment, and compatible frames, with transformation provenance and uncertainty. [N,P]

No position is emitted for a Procedure. Deployment `location` is separate from a System `position`; one describes the deployment area/place, the other the physical process pose. [N]

### 10.5 Inputs, Outputs, Parameters, and Streams

SensorML I/O names and structures describe process interfaces. Part 2 stream resources describe API-accessible dynamic channels. Glaux should use explicit binding facts rather than infer a stream from any matching label:

- DataStream → producing System/Procedure plus a selected SensorML output path/name and compatible result schema;
- ControlStream → receiving System/Procedure plus a selected SensorML input/parameter path/name and compatible command schema;
- observed/controlled Property references → semantic definitions used within those component trees; and
- Observation/Command values → validated against the parent stream schema, not merely the generic SensorML description. [P]

Issue #147 records maintainer direction that top-level `outputName` should match a SensorML output and that analogous `inputName` consistency should be documented, but the approved prose remains incomplete. Glaux should define this as a versioned profile invariant, preserve exact component paths when names are nested/ambiguous, and never call it an unqualified Part 2 obligation until corrected. [X,D,P]

`DataInterface` remains a model/schema gap. The public SensorML JSON writer must not emit an invented link-only or hardware-port form under the approved class. Glaux may preserve such content as a source/profile extension and should maintain fixtures for a future upstream correction. [X,P]

---

## 11. SWE Common, Semantic Binding, Temporal, Status/Event, and Command Dependencies

### 11.1 SWE Common Handoff

The SensorML core mandates SWE Common simple and record components for applicable `AbstractDataComponent` content; arrays, choices, matrices, streams, and encodings belong to advanced support. IDR-SRV-022 must select the Glaux-supported component/encoding breadth, recursion limits, name/path rules, validation strategy, value constraints, and compact/expanded representation behavior. This report requires the selected profile to support at least the components necessary for compliant SensorML I/O, parameters, capability, characteristic, configuration, event, and position content. [N,P]

### 11.2 Semantic Binding Handoff

`definition`, identifier/classifier types, `codeSpace`, units, base properties, statistics, object types, qualifiers, roles, process types, event types, CRS/reference frames, and extension vocabularies require governed semantic identifiers. IDR-SRV-024 must define URI/CURIE expansion, ontology/version pins, local/external registries, cache/offline behavior, equivalence rules, deprecation, unknown semantics, and observed/controlled-property linkage. Syntax-valid URIs remain source assertions until these checks succeed. [P]

SSN/SOSA can explain relationships among System, Procedure, deployment, observation, actuation, property, feature of interest, and capability. It should annotate or validate selected semantics without forcing every SensorML node into an RDF store or adding non-CSAPI wire fields. [N,P]

### 11.3 Temporal and Provenance Handoff

SensorML `validTime` is description validity; Deployment valid time is operational deployment applicability; history Event time is occurrence time; a trajectory carries phenomenon/position times; source receipt and transformation have transaction times; a generated response has evaluation/snapshot time. Each must retain its axis and provenance from IDR-SRV-018/019. Multiple SensorML descriptions for the same UID can represent different valid periods but do not create new Systems automatically. [N,P]

### 11.4 Status and Event Handoff

Capabilities, configured modes, settings, and characteristics are descriptive. Current status is an observation-derived/policy-derived projection. A SensorML Mode name may also appear as a reported status category, but the description and observation remain separate linked evidence. [P; IDR-SRV-020]

SensorML history Events and Part 2 System Events require an explicit transformation contract with source event identity/digest, target System, vocabulary mapping, time mapping, deduplication key, provenance, and policy result. The published Part 2 `name`/`eventTime` versus schema `label`/`time` mismatch remains governed by the IDR-SRV-020 adapter decision. [N,X,P]

### 11.5 Command and Control Handoff

SensorML inputs, parameters, allowed constraints, modes, and configuration can help validate whether a requested command shape is describable. They do not establish current feasibility, command acceptance, operator authority, safe execution, or result success. IDR-SRV-036–038 must bind ControlStream schemas to SensorML paths, detect description/schema drift, define command lifecycle and feasibility, and apply authorization/safety/audit independently. [P]

Draft Part 3 messages may reference or transport resource representations later, but SensorML identity, revision, redaction, and content-type decisions must remain transport-neutral. A published event about a SensorML change is not the source document, canonical resource, or transformation record. [D,P]

---

## 12. Validation, Profile, Schema, and Conformance

### 12.1 Validation Ladder

| Layer | Question | Example failure | Required record |
|---|---|---|---|
| Transport/media | Is body delivered under supported media/version/size? | XML under `application/sml+json` | request/media diagnostic |
| JSON syntax | Is JSON well formed and within parser budgets? | duplicate-key policy, depth/size exhaustion | parser/version/result |
| SensorML schema class | Does object satisfy applicable 3.0 JSON schema composition? | missing `type`/`label`/`uniqueId`, wrong Mode cardinality | schema IDs/digests/errors |
| CSAPI wrapper/mapping | Is class/resource mapping valid? | Procedure position; wrong system class; `id` mismatch | requirement/profile errors |
| Cross-resource integrity | Do IDs, links, targets, cardinalities, validity, and cycles agree? | `typeOf` target is not Procedure; parent cycle | graph validation result |
| SWE structure/value | Do embedded components and settings satisfy selected SWE profile? | invalid component path/constraint/unit | deferred profile result |
| Semantic/profile | Are definitions, code spaces, qualifiers, roles, and extensions allowed? | unknown asset type or ambiguous CURIE | vocabulary/profile result |
| Security/operational | May actor register/expose/use it? Is command use safe? | self-asserted downgrade; forbidden capability | policy decision/audit |
| Generated response | Does authorized projection remain valid and truthful? | redaction removes required field; stale link | serializer/profile result |

A single `valid=true` flag loses the validator, schema/profile version, layer, warnings, unresolved references, and policy context. Store each result as immutable evidence with input/output digest and software/configuration identity. [P]

### 12.2 Conformance Class Posture

Claiming Part 1 `/req/sensorml` means supporting `application/sml+json` reads and, conditionally when create/replace/delete is implemented, writes, plus Requirements 89–103 for the implemented resource families. Glaux should advertise the class only when every enabled applicable resource representation, mapping, negotiation behavior, schema, and relevant abstract test passes. Presence of a serializer or acceptance of generic JSON is not proof. [N,P]

SensorML itself is modular. The Glaux capability registry must state the concrete JSON/model classes and any advanced/configurable support actually implemented and tested. It must not imply that support for the CSAPI minimum means complete execution of arbitrary AggregateProcesses, advanced SWE encodings, all position forms, or every community extension. [N,P]

### 12.3 Published Conflicts and Interpretation Records

At minimum, IDR-SRV-023 must retain:

1. the stale preliminary vendor-media-type note versus approved `application/sml+json` requirements;
2. Part 1 Annex A's stale SensorML 2.1 prerequisite versus Clause 19.2's 3.0 prerequisites;
3. Requirement 91's association-name wording versus mapping-table instructions/accepted Table 3 `ogc-rel:` output;
4. missing `DataInterface` JSON choice;
5. quaternion UML/schema issue #43;
6. `DerivedProperty.qualifier`/`qualifiers` omission from CSAPI tables;
7. `outputName`/`inputName` mapping gap; and
8. System Event conceptual/schema mismatch. [N,X]

Tests must identify which source each expectation represents. A validator/tool failure caused by recursive schemas or unsupported JSON Schema 2020-12 features is a tool limitation, not proof that the instance is invalid. [I,P]

### 12.4 Profile Descriptor

The eventual Glaux SensorML profile should be machine-readable and versioned, naming:

- standards/schema pins and permitted concrete classes;
- resource-family mappings and required/forbidden members;
- supported SWE component/conformance subset;
- recognized vocabularies/CURIE maps and extension namespaces;
- inheritance/resolution and external-reference policy;
- normalization/index fields and source-preservation promise;
- write/import/legacy behavior;
- redaction/releasable-view rules;
- known published conflicts and adapters; and
- linked conformance/fixture evidence. [P]

---

## 13. Security, Policy, and Releasability

### 13.1 Threat Surface

SensorML can disclose identity, ownership, operator contacts, precise position/trajectory, platform attachment, topology, component inventory, interfaces/endpoints, performance ranges, operating modes, command inputs, configuration limits, maintenance history, documentation locations, and source lineage. Even classifiers, counts, link existence, error detail, or schema choices can reveal mission or system capability. [N,A,P]

Inbound documents also create parser/resource-exhaustion, recursive-reference, external-fetch/SSRF, malicious URL, oversized aggregate, duplicate-key, Unicode, extension injection, stored-content, semantic poisoning, and self-asserted marking risks. Process methods and settings must be treated as data, never deserialized into executable code or automatically applied to equipment. [P]

### 13.2 Enforcement Rules

1. Authenticate and authorize import, registration, update, source-artifact access, reference resolution, transformation, indexing, and response generation separately. [P]
2. Apply policy to canonical facts, edges, nested components, metadata, and extensions before serialization. [P]
3. Treat source-provided `securityConstraints` as an assertion that can raise handling requirements but never lower authoritative policy. Unknown/malformed markings fail safely into quarantine/restricted handling. [P]
4. Use HTTPS in transit and appropriate encryption/key separation at rest, consistent with SensorML security considerations and later threat modeling. [N,P]
5. Restrict resolvers by scheme, DNS/IP class, origin allow-list, credentials, redirects, time/size/depth, content type, and cache key; prevent access to loopback, link-local, metadata-service, and internal addresses unless explicitly administered. [P]
6. Never leak hidden target identities through links, reverse counts, validation errors, ETags, alternate views, cached raw documents, or search facets. [P]
7. Generated variants with different policy results need private/no-store or correctly partitioned caches; `Vary: Authorization` alone is not a complete shared-cache security design. [P]
8. Record who supplied, validated, approved, transformed, released, and retrieved sensitive artifacts. [A,P]

### 13.3 Source Versus Public View

The exact source artifact can be more sensitive than a generated representation because it preserves removed fields, original identifiers, internal URLs, and unrecognized markings. Source access therefore has an independent privilege. A user authorized for a System summary is not automatically authorized for the imported SensorML document or its validation diagnostics. [P]

---

## 14. Fixtures, Golden Files, and Interoperability

### 14.1 Required Corpus

| Fixture family | Required examples |
|---|---|
| Class/resource | All four process classes as valid/invalid System and Procedure; Deployment; DerivedProperty; forbidden Procedure position |
| Common identity | ID/URL equality, URI UID, duplicate/colliding UID, aliases, CURIE/URI, multilingual label |
| Metadata | identifiers/code spaces, classifiers/asset type, keywords, contacts, documents, security/legal markings, unknown extensions |
| Inheritance | local/external `typeOf`, pinned base, overlay, configuration, missing/hidden/stale base, cycles, depth/size limits |
| Composition | inline components, linked canonical subsystems, connections, invalid port paths, recursive graphs, reordered members |
| I/O/SWE | scalar/record and later advanced components, output/input path binding, constraints, modes (0/1/2+), settings references |
| Position | GeoJSON point/geometry, GeoPose YPR/quaternion, relative pose, trajectory, by-process/by-stream, incompatible frames |
| Deployment | platform System/external Feature, multiple deployed systems, per-system config, subdeployment links, time conflicts |
| Property | base/object/statistic, qualifiers, local/external bases, semantic cycles, table/schema conflict variants |
| Event/history | source history Event, explicit transformation to System Event, duplicate replay, prose/schema adapter variants |
| Import | malformed JSON, wrong media/version, partial schema, legacy 2.x XML, profile extension, quarantine promotion |
| Policy | whole/member markings, hidden link/target, redacted contact/capability/position, required-field concealment |
| Negotiation | `Accept`/`f`, q-values/wildcards, 406/415, truthful type, alternate links, same IDs/counts across encodings |
| Robustness | duplicate keys, extreme depth/width, huge arrays/strings, Unicode, malicious URLs, redirect/DNS rebinding, timeout |

### 14.2 Golden-File Rules

Every fixture records source/license, immutable pin/digest, standards/profile/schema versions, expected validation layer/result, intentional deviation, expected normalized facts, expected generated representation, redaction variant, and provenance. Official examples and implementation fixtures are seed evidence, not golden truth; they must be revalidated because published examples and schemas can disagree. [P,I]

Golden comparison has three modes:

- byte equality only for exact source retention or deterministic serializer snapshots;
- normalized JSON equality ignoring member order where syntax is irrelevant; and
- semantic equivalence for cross-representation identity, relationships, temporal facts, and common attributes. [P]

### 14.3 Interoperability Matrix

Test at least Glaux↔CSAPI Explorer, OS4CSAPI clients, OSH-derived clients, CS-Go fixtures/clients, generic HTTP clients, chosen JSON Schema validators, OpenAPI generators, and any AEP profile validator available to the project. Include read, write, reject, round-trip, alternate representation, external reference, restart, migration, and policy-view cases. Never adjust server conformance merely to imitate one peer's quirk; use named compatibility adapters with telemetry and sunset criteria. [I,P]

Particular regressions required by evidence are:

- no representation-dependent resource population or identity;
- no JSON member-order dependency;
- no silent field/extension/link loss on supported writes;
- no `application/sml+json` request answered as ordinary JSON without truthful negotiation;
- no unsupported media fallback that hides 406/415 behavior;
- bounded recursive-schema tooling;
- complete source-to-client publisher integration for prior loss scenarios; and
- static conformance declarations mechanically consistent with enabled codecs and passing tests. [I,P]

---

## 15. Downstream Topic Handoff Matrix

| Topic | Mandatory handoff | Decision retained here |
|---|---|---|
| IDR-SRV-022 SWE Common | Select component/encoding breadth, recursion/path rules, settings and position structures, value validation | SensorML embeds SWE; layered ownership/preservation remains |
| IDR-SRV-023 Validation | Implement validation ladder, schema pins, conflict register, diagnostics, generated-output gates | Invalid ordinary writes reject; quarantine separate |
| IDR-SRV-024 Semantics | Govern URI/CURIE, definitions, code spaces, units, roles, property/qualifier semantics, registries/offline cache | Syntax is not semantic trust |
| IDR-SRV-025/027 | Apply validity/revision/provenance to descriptions, bases, transformations, and resolved views | Source/base/overlay/resolved view remain separate |
| IDR-SRV-028 Storage | Choose storage/index/cache/retention for artifact, parsed graph, canonical projection, generated view, validation evidence | Five-layer logical model controls |
| IDR-SRV-029/030 | Define atomic cross-encoding update, conflict, relationship, cascade, and restart behavior | One canonical owner; no independent representation truth |
| IDR-SRV-031/032 | Define write/import/promotion/precondition and transformation APIs | Standards write is strict; quarantine import privileged |
| IDR-SRV-034 | Bind DataStreams/output paths and dynamic position/status descriptions | Description does not create live/current facts |
| IDR-SRV-035 | Preserve SensorML representation IDs/revisions/redaction across transport; monitor DataInterface | No Part 3 adoption here |
| IDR-SRV-036–038 | Bind ControlStreams/input paths; validate modes/config while preserving safety/auth/feasibility separation | Imported settings never execute directly |
| IDR-SRV-039–041 | Threat-model parsing/resolution; define policy/releasable projections, marking authority, audit | Source and generated view have separate access |
| IDR-SRV-042 | Resolve/cache external descriptions safely under DDIL; provenance and stale states | No uncontrolled synchronous fetch |
| IDR-SRV-043 | Bound external catalog/document/semantic resolution and policy | External URL is not trust |
| IDR-SRV-044–049 | Select Rust/tool architecture supporting recursive typed/opaque hybrid, capability registry, deterministic codecs | No crate/framework selection here |
| IDR-SRV-050/051 | Map SensorML/CSAPI requirements, adapters, schema conflicts, and evidence to tests/claims | Claims require complete applicable evidence |
| IDR-SRV-053 | Build the fixture corpus in §14 with pins, validity classes, and intentional deviations | Examples are not automatically golden |
| IDR-SRV-056 | Execute cross-client/server representation, negotiation, write, and policy tests | Named compatibility only |
| IDR-SRV-057 | Recheck open issues #45/#147/#162 and all material corrections/releases | Approved 1.0/3.0 baseline controls until changed |

---

## 16. Recommendations

| ID | Recommendation | Priority | Authority |
|---|---|---|---|
| R-021-01 | Adopt the five-layer source/parsed/canonical/generated/validation representation architecture. | Critical | P; IDR-015/019 |
| R-021-02 | Maintain exactly one canonical resource identity, lifecycle, relationship graph, and population across all representations. | Critical | N/A/P |
| R-021-03 | Implement Part 1 SensorML only for System, Procedure, Deployment, and Property combinations actually enabled and tested. | Critical | N |
| R-021-04 | Use subject/endpoint rules to select concrete process classes; never infer CSAPI endpoint identity from SensorML `type` alone. | High | N/P |
| R-021-05 | Normalize identity, mapped types, validity, and relationships; selectively normalize governed discovery/operational seams; preserve rich/unknown source content. | Critical | P |
| R-021-06 | Make generated policy-aware SensorML the normal public view; grant exact source-artifact access separately. | Critical | P |
| R-021-07 | Separate strict conformant writes from privileged quarantine import/promotion of partial, invalid, legacy, or profile-divergent material. | Critical | P |
| R-021-08 | Preserve `typeOf` base, overlay, configuration, resolution snapshot, and resolved view separately with cycle/SSRF/size/time controls. | Critical | N/P |
| R-021-09 | Keep inline components distinct from canonical subsystem resources unless explicit registration allocates identity/lifecycle/relationships. | High | P; IDR-015–017 |
| R-021-10 | Preserve the full position union and derive query geometry/current pose only with frame, time, uncertainty, and transformation provenance. | High | N/P |
| R-021-11 | Treat capabilities/characteristics/modes/configuration as descriptive evidence, not availability, authorization, feasibility, or execution. | Critical | N/P; IDR-020 |
| R-021-12 | Bind streams and commands to explicit SensorML I/O component paths through a versioned profile; do not infer resource creation from descriptions. | High | X/P |
| R-021-13 | Carry DataInterface, qualifier(s), output/input-name, relation wording, and System Event gaps as explicit interpretation/fixture records. | High | N/X/P |
| R-021-14 | Use a layered validator and machine-readable profile; derive conformance claims from enabled codecs plus passing evidence. | Critical | N/P |
| R-021-15 | Apply policy before projection and secure raw artifacts, nested metadata, indexes, links, errors, resolvers, and caches independently. | Critical | N/A/P |
| R-021-16 | Require exact-source, normalized-JSON, semantic round-trip, cross-format population, recursive robustness, and external-client fixtures. | High | I/P |

### 16.1 Acceptance Decision

Acceptance establishes these recommendations as the SensorML planning baseline. Later topics may choose mechanisms within it. A material departure—such as making imported SensorML the database truth, exposing raw documents by default, executing configuration, or maintaining independent per-format resources—requires an explicit superseding decision with standards, security, migration, and test impact. [P]

---

## 17. Risks, Constraints, and Open Questions

### 17.1 Risks and Mitigations

| Risk | Impact | Mitigation/handoff |
|---|---|---|
| Opaque-only storage | Weak query, validation, linking, policy, equivalence | Selective normalization plus exact preservation |
| Full flattening | Loss of extensions, recursive structure, provenance, round-trip fidelity | Parsed graph/source artifact plus bounded canonical projection |
| Dual per-format truth | Different populations/relationships by representation | One canonical owner and semantic equivalence tests |
| Recursive/hostile content | Resource exhaustion or parser/tool failure | Budgets, streaming/bounded parsing, fuzz/negative corpus, tool pinning |
| External references | SSRF, mutable inheritance, stale/hidden bases | Controlled resolver, pinned digest/revision, explicit states |
| Semantic poisoning | False unit/type/capability conclusions | Governed registries and source-qualified assertions |
| Self-asserted markings | Disclosure or denial manipulation | Authoritative policy can only maintain/raise handling |
| Overexposed rich metadata | Reveals position, topology, contacts, command affordances | Pre-serialization policy projections and separate raw access |
| Schema/prose gaps | False conformance or incompatible writers | Interpretation register and variant fixtures |
| Legacy transformation | Silent loss or false 3.0 claim | Versioned adapter, loss report, preserved original |
| Description/runtime collapse | Capability mistaken for current readiness or configuration for execution | Explicit non-collapse types and downstream gates |

### 17.2 Constraints

- The accepted four-standard AEP package and approved OGC editions control; future drafts do not silently update it.
- SensorML is extensible and semantically dependent, so schema validation alone cannot establish trust, meaning, or policy compliance.
- JSON object order is not semantic; exact-byte preservation and semantic comparison serve different purposes.
- Policy filtering can make a once-valid document invalid; releasable projection rules must be designed with schemas/profiles.
- This topic defines logical storage roles but cannot select physical persistence before IDR-SRV-028.
- This topic cannot finalize SWE or semantic breadth before IDR-SRV-022/024.

### 17.3 Open Questions and Owners

1. Which exact SWE component and encoding conformance classes will Glaux support initially? — IDR-SRV-022.
2. Which validator stack can safely handle recursive JSON Schema 2020-12 artifacts and report stable locations? — IDR-SRV-023/044.
3. What Glaux/AEP SensorML profile fields are mandatory, indexable, or releasable by deployment? — IDR-SRV-023/024/040.
4. How will qualifiers and `outputName`/`inputName` be advertised pending upstream correction? — IDR-SRV-023/024/034/036.
5. What storage/index/cache technologies and retention apply to exact artifacts and parsed graphs? — IDR-SRV-028.
6. What is the public/admin API for source artifacts, validation reports, quarantine, and promotion? — IDR-SRV-031/032/046.
7. What precise policy applies when required SensorML content cannot be released? — IDR-SRV-039/040.
8. Will Glaux support legacy SensorML 2.x XML import at launch, migration-only, or later? — roadmap after IDR-SRV-023/028/044.
9. Will upstream close #45, #147, and #162 or publish a corrigendum/new version? — monitor in owning topics and IDR-SRV-057.

None prevents this planning baseline. They are mechanism/profile decisions intentionally assigned downstream.

---

## 18. Validation Against Plan Success Criteria

### 18.1 Methodology Completion

| Phase | Result | Evidence |
|---|---|---|
| 1. Source collection/framework | Complete | §§3–4 |
| 2. Standards and CSAPI mapping | Complete | §§5–6 |
| 3. Representation boundary/normalization | Complete | §§7–9 |
| 4. SWE/semantic/temporal/status/command dependencies | Complete | §§10–11 |
| 5. Validation/security/fixture/interoperability | Complete | §§12–14 |
| 6. Synthesis | Complete | §§15–17 |

### 18.2 Success Criteria

| Topic-plan criterion | Status | Evidence |
|---|---|---|
| Relevant concepts identified with source anchors | Met | §§3, 5, 6.2 |
| Concepts mapped to canonical resource families | Met | §6 |
| Representation/domain/document/persistence/validation boundaries distinguished | Met | §§7, 12; persistence mechanism deferred as planned |
| Import/generation/transformation/preservation/invalid handling documented | Met | §8 |
| SWE/semantic/temporal/status/event/command/persistence dependencies documented | Met | §§10–11, 15 |
| Validation/security/conformance/fixture/interoperability implications documented | Met | §§12–14 |
| Implementation/community lessons incorporated nonnormatively | Met | §§3.4, 14 |
| Recommendations decision-usable and server-bounded | Met | §16 |
| Downstream handoffs explicit | Met | §15 and §17.3 |
| References explicit and reproducible | Met | §19 and source pins in §3 |

### 18.3 Report Completion Checklist

- [x] Topic ID and research-plan link match the overall index.
- [x] All core and detailed question groups are answered or assigned explicitly.
- [x] Normative, AEP, project, implementation, draft, and unresolved evidence are separated.
- [x] Mutable technical evidence is pinned and date-checked.
- [x] Controlled-source limits are explicit; no controlled content is redistributed or invented.
- [x] Conflicts with accepted reports are reconciled.
- [x] Recommendations, risks, fixtures, and downstream owners are explicit.
- [x] Report is ready for plan-owner review.
- [ ] Plan-owner acceptance and date recorded.

### 18.4 Next Two Actions

1. Glaux Project Lead reviews and accepts IDR-SRV-021.
2. In the same instruction, the project lead authorizes execution of exactly IDR-SRV-022.

Combined response pattern: **`accept IDR-SRV-021 and proceed`**. Under the established shorthand, a bare **`proceed`** at this review boundary carries that combined meaning. Acceptance does not authorize server implementation, draft Part 3 implementation, or IDR-SRV-023.

---

## 19. References

### 19.1 Controlling Standards and Artifacts

- Open Geospatial Consortium, [OGC 23-000, *OGC SensorML Encoding Standard*, Version 3.0](https://docs.ogc.org/is/23-000/23-000.html), approved June 2, 2025; published July 16, 2025; accessed September 13, 2026.
- Open Geospatial Consortium, [official SensorML 3.0 JSON schemas](https://schemas.opengis.net/sensorML/3.0/json/), accessed September 13, 2026.
- Open Geospatial Consortium, [OGC 23-001, *OGC API - Connected Systems - Part 1: Feature Resources*, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html), accessed September 13, 2026.
- Open Geospatial Consortium, [OGC 23-002, *OGC API - Connected Systems - Part 2: Dynamic Data*, Version 1.0](https://docs.ogc.org/is/23-002/23-002.html), accessed September 13, 2026.
- Open Geospatial Consortium, [OGC 24-014, *OGC SWE Common Data Model Encoding Standard*, Version 3.0](https://docs.ogc.org/is/24-014/24-014.html), accessed September 13, 2026.
- Open Geospatial Consortium, [OGC 17-069r4, *OGC API - Features - Part 1: Core*, Version 1.0](https://docs.ogc.org/is/17-069r4/17-069r4.html), accessed September 13, 2026.
- W3C/OGC, [*Semantic Sensor Network Ontology*](https://www.w3.org/TR/vocab-ssn/), accessed September 13, 2026.
- Open Geospatial Consortium, [`ogcapi-connected-systems` published source tag `v1.0.0`, commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2), checked September 13, 2026.

### 19.2 Official History Evidence

- [OGC API - Connected Systems issue #39, position options](https://github.com/opengeospatial/ogcapi-connected-systems/issues/39).
- [Issue #45, SensorML DataInterface](https://github.com/opengeospatial/ogcapi-connected-systems/issues/45).
- [Issue #46, SensorML/SWE Common 3.0 refactor](https://github.com/opengeospatial/ogcapi-connected-systems/issues/46).
- [Issue #147, outputName inconsistencies](https://github.com/opengeospatial/ogcapi-connected-systems/issues/147).
- [Issue #162, Property qualifier omission](https://github.com/opengeospatial/ogcapi-connected-systems/issues/162).
- [PR #156, final SensorML schema/reference publication changes](https://github.com/opengeospatial/ogcapi-connected-systems/pull/156).
- [PR #158, final SensorML publication review](https://github.com/opengeospatial/ogcapi-connected-systems/pull/158).
- [Glaux upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md), Version 1.9, relevant entries refreshed September 13, 2026.

### 19.3 Project and Controlled Sources

- NATO Consultation, Command and Control Board Joint Capability Group Intelligence, Surveillance and Reconnaissance, `AC/224(JCGISR)D(2026)0005`, April 27, 2026, project-controlled package, SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`; used only through accepted IDR findings.
- [IDR-SRV-003 Standards Package Baseline](idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline-report.md).
- [IDR-SRV-004 Terminology Crosswalk](idr-srv-004-terminology-and-concept-crosswalk-report.md).
- [IDR-SRV-012 Content Negotiation](idr-srv-012-content-negotiation-media-types-and-encoding-selection-report.md).
- [IDR-SRV-015 Canonical Resource Model](idr-srv-015-canonical-glaux-server-resource-model-report.md).
- [IDR-SRV-016 Identifier and Lifecycle Strategy](idr-srv-016-identifier-uri-and-resource-lifecycle-strategy-report.md).
- [IDR-SRV-017 Relationship Model](idr-srv-017-relationship-and-linkage-model-report.md).
- [IDR-SRV-018 Temporal Model](idr-srv-018-temporal-validity-and-freshness-model-report.md).
- [IDR-SRV-019 Provenance, Quality, and Trust Model](idr-srv-019-provenance-lineage-quality-and-trust-metadata-model-report.md).
- [IDR-SRV-020 Status, Availability, and System Event Model](idr-srv-020-status-availability-and-system-event-model-report.md).

### 19.4 Implementation and Interoperability Evidence

- [IDR-SRV-014A OSH Study](idr-srv-014a-osh-csapi-server-implementation-study-report.md).
- [IDR-SRV-014B Connected Systems Go Study](idr-srv-014b-connected-systems-go-csapi-server-implementation-study-report.md).
- [IDR-SRV-014C pygeoapi/52°North Study](idr-srv-014c-pygeoapi-csapi-server-implementation-study-report.md).
- [IDR-SRV-014D SECD Study](idr-srv-014d-secd-csapi-server-implementation-study-report.md).
- [IDR-SRV-014E OS4CSAPI Client Smoke-Test Study](idr-srv-014e-os4csapi-client-smoke-test-findings-study-report.md).
- [IDR-SRV-014F SECD Interoperability Study](idr-srv-014f-secd-interoperability-findings-study-report.md).
- [IDR-SRV-014G OS4CSAPI Discussions Study](idr-srv-014g-os4csapi-discussions-lessons-learned-study-report.md).
- [IDR-SRV-014H Draft Part 3 and Implementation Study](idr-srv-014h-draft-csapi-part-3-publish-subscribe-and-implementation-study-report.md).

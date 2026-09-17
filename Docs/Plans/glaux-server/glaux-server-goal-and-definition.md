# Glaux Server Goal and Definition
**Version:** 1.6<br>
**Date:** 17 September 2026<br>
**Status:** Approved

**Revision summary:** Applies the completed initial design research to clarify the reference-implementation goal, full standards scope, experimental Part 3 intent, server responsibilities, and verification expectations within the existing ten-section structure.

---

## 1. Purpose
This document defines the goal, scope, and planning baseline for **Glaux Server**, the server-side implementation component of the DGIWG Glaux ecosystem.

Glaux Server is intended to be a full-scope, open-source Rust reference implementation of OGC API - Connected Systems through which the Glaux software suite operationalizes the STANAG 4789 / AEP-4789 standards framework. Its resources and APIs shall support connected-system discovery, description, access, exchange, streaming, status, and tasking workflows across NATO, national, coalition, federated, and tactical environments.

This document is not an implementation guide, roadmap, or software design specification. It establishes the goal and definition baseline from which those later artifacts shall be developed.

---

## 2. Goal
Deliver a full-scope, open-source Rust reference implementation of OGC API - Connected Systems, including its SensorML and SWE Common requirements, for the Glaux ecosystem and its STANAG 4789 / AEP-4789 commitments.

Glaux Server shall support authorized users, applications, AI-enabled services, publishers, simulators, client software, and interoperable external systems that need to discover, describe, or task heterogeneous connected systems, or access, exchange, and stream the information they produce.

The server shall provide coherent, machine-readable, standards-correct API behavior for connected-system resources, metadata, observations, dynamic data, status information, system events, and command/tasking workflows while preserving compatibility with the broader Glaux ecosystem. Its implementation shall be robust, interoperable, testable, maintainable, and useful to other implementers.

---

## 3. Definition
**Glaux Server** is the open-source Rust reference server for OGC API - Connected Systems within the Glaux ecosystem.

It is the server-side implementation component through which Glaux operationalizes the STANAG 4789 / AEP-4789 Volume II standards package: OGC API - Connected Systems Part 1, OGC API - Connected Systems Part 2, SensorML, and SWE Common.

It shall provide the standards-based resource and API contract used by Glaux components and external clients. Its responsibilities cover the correctness of the resources it manages, its API behavior, validation, authorization, and integration interfaces.

The server shall expose, manage, validate, and control access to API resources and interactions needed to make connected systems and their information discoverable, accessible, understandable, linked, trustworthy, interoperable, and secure for authorized users, AI, applications, and software services.

---

## 4. Standardization Basis
Glaux Server shall be planned and implemented against the STANAG 4789 / AEP-4789 framework and the open standards adopted by AEP-4789 Volume II.

The core standards package and versions for this planning baseline are:

- **[OGC API - Connected Systems Part 1: Feature Resources, version 1.0](https://docs.ogc.org/is/23-001/23-001.html)**
- **[OGC API - Connected Systems Part 2: Dynamic Data, version 1.0](https://docs.ogc.org/is/23-002/23-002.html)**
- **[OGC SensorML Encoding Standard, version 3.0](https://docs.ogc.org/is/23-000/23-000.html)**
- **[OGC SWE Common Data Model Encoding Standard, version 3.0](https://docs.ogc.org/is/24-014/24-014.html)**

These standards shall be treated as a coherent implementation package rather than as unrelated specifications. CSAPI Part 1 provides the resource-oriented foundation for connected-system feature resources and associated metadata. CSAPI Part 2 provides the dynamic-data and interaction layer for datastreams, observations, status information, control streams, commands, command status, feasibility exchanges, and system events. SensorML provides rich machine-readable descriptions of systems and processes. SWE Common provides common data structures and encodings for sensor-related data, including observations, status information, command inputs, and tasking parameters.

Full scope means implementing all 25 conformance classes identified in CSAPI Parts 1 and 2 by the [conformance mapping research](../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-008-conformance-class-and-requirement-mapping-report.md), together with their applicable prerequisite requirements. This is Glaux's intended completion target; OGC permits smaller conforming implementations. References to other standards bring in the requirements needed for that target, rather than every independent capability those standards offer. The Implementation Guide shall identify the exact inherited dependencies, including any incorporated drafts.

Authoritative technical requirements, schemas, conformance classes, and encoding rules remain in the adopted standards themselves. Glaux Server planning and implementation shall maintain traceability to those authoritative sources. Where published requirements, schemas, and examples disagree, the selected interpretation and its rationale shall be documented and revisited as authoritative corrections become available. Upstream proposals, implementation examples, and project choices shall be distinguished from published standards requirements.

Glaux Server is also planned to include experimental support for OGC API - Connected Systems Part 3 Publish/Subscribe, with the implemented draft revision and supported behavior clearly documented. This work is distinct from the adopted Parts 1 and 2 standards package and shall not be presented as conformance to an approved Part 3 standard. Protocol choices and implementation sequencing belong in the Implementation Guide and Roadmap.

---

## 5. Core Capability Scope
Glaux Server planning and implementation shall address the following full-scope capability areas.

### 5.1 Connected-System Discovery and Navigation
Glaux Server shall support standards-aligned discovery, resource navigation, landing-page behavior, conformance declaration, collection/resource discovery, and API description behavior needed by human users, client software, AI-enabled services, and interoperable external systems.

### 5.2 Registration and Description
Glaux Server shall support the registration, description, update, and retrieval of connected-system resources and associated metadata, including systems, platforms, sensors, actuators, samplers, procedures, deployments, sampling features, observed or controlled properties, and related contextual resources.

Descriptions shall be sufficient to support persistent identification, capability understanding, deployment context, provenance, validity, lineage, and machine interpretation.

Alternate representations of a resource shall preserve consistent identity, relationships, and descriptive meaning according to the applicable standards mappings.

### 5.3 Access and Exchange
Glaux Server shall support standards-aligned access to connected-system information and related data, including structured resource retrieval, observation access, metadata access, historical query behavior, exchange of sensor-derived information, and preservation of contextual binding between data, producing systems, observed properties, features of interest, time, location, provenance, and validity.

Storage, retrieval, and supported conversions shall preserve the applicable schema bindings, units, and temporal meaning. Any limitations of a supported conversion shall be documented; conversion shall not silently change the meaning of the data.

### 5.4 Streaming and Dynamic Data
Glaux Server shall support dynamic data workflows associated with connected systems, including datastreams, observations, status information, event-driven updates, time-varying information, and streaming or near-real-time exchange patterns where applicable.

Dynamic data behavior shall preserve temporal context, sequencing, freshness, operational relevance, and machine-readable structure sufficient for interoperable use.

Publish/subscribe work includes the planned experimental Part 3 support described in Section 4, with its draft status and supported capabilities made explicit.

### 5.5 Tasking and Control
Glaux Server shall support tasking and control workflows where applicable, including control streams, commands, command status, feasibility-related exchanges, controllable parameters, tasking lifecycle behavior, and governance of authorized command interactions.

Tasking and control shall be treated as first-order server capabilities, not as optional user-interface behavior or later application-layer decoration.

Tasking behavior shall preserve the standard's command and feasibility semantics and clearly distinguish feasibility assessments, accepted requests, execution status, and confirmed outcomes. Device-specific actuation and safety arrangements are defined with the connected systems and deployments that carry out commands.

### 5.6 Status and Availability
Glaux Server shall support the exposure and exchange of system status, operational state, availability, health, configuration state, lifecycle state, degraded operation, and other system events required to understand whether connected systems can support operational use.

Status and availability information shall preserve temporal and validity context so consumers can distinguish current information from stale, delayed, or last-known state.

### 5.7 Security, Authorization, and Trust
Glaux Server shall be designed with explicit treatment of security, authorization, validation, trust, access governance, failure semantics, and policy-aware interoperability.

Security and authorization shall not be treated as deployment afterthoughts. The server design shall account for cross-organizational, coalition, federated, and differently accredited environments in which access to information or tasking authority may vary by user, system, organization, mission, role, policy, or operational context.

Glaux Server shall enforce configured access rules and provide documented integration with identity and policy services. The surrounding organizations and deployments retain responsibility for identity administration, policy ownership, authorization to release operational information, and accreditation.

### 5.8 Cross-Environment and DDIL-Informed Operation
In this planning baseline, **DDIL-informed** refers broadly to disconnected, denied, degraded, intermittent, and limited-bandwidth operating conditions; it does not assert that one fixed operating-state taxonomy has already been selected.

Glaux Server shall be designed with awareness of enterprise, coalition, federated, tactical, constrained, and DDIL-informed operating environments.

The server architecture and implementation planning shall account for degraded connectivity, intermittent synchronization, constrained exchange, asynchronous updates, last-known state reporting, staged metadata enrichment, freshness and validity assessment, and efficient handling of updates where continuous high-quality connectivity cannot be assumed.

The server shall preserve source identity and timestamps, handle repeated or delayed updates consistently, identify synchronization conflicts, and distinguish last-known information from current evidence. This scope concerns correct server behavior during interruption and recovery. Network topology, connectivity provision, and federation arrangements are defined by the deployments that require them.

### 5.9 Validation, Conformance, and Verification
Glaux Server shall be planned using a conformance-first and verification-first approach.

Implementation work shall include explicit test, validation, and conformance strategies for standards behavior, resource models, encodings, API behavior, error handling, security behavior, tasking workflows, ecosystem integration, and operationally representative scenarios.

Verification shall include tests derived from the standards' requirements and test procedures, together with practical use through independent external clients. Checks shall establish correct discovery, query results, resource relationships, representations, and data meaning, beyond successful transport or parsing alone. Each release's conformance declarations and API documentation shall accurately describe its implemented and tested capabilities.

---

## 6. Role in the Glaux Ecosystem
Glaux Server provides the canonical server-side contract for the wider Glaux ecosystem.

It shall support integration with:

- **Glaux Web App** for operational visualization, discovery, status display, observation access, and authorized interaction workflows
- **Glaux Mobile** for tactical-edge situational awareness and authorized interaction with connected systems
- **Glaux Publisher** for publication, mediation, and ingestion of connected-system resources and dynamic data
- **Glaux Simulator** for standards-aligned generation, replay, validation, and stress-testing workflows
- **External OGC API - Connected Systems clients and implementations** where interoperability is required
- **NATO, national, coalition, and federated mission-system environments** where STANAG 4789-aligned sensor integration is required

The server shall define its API and behavioral contracts clearly enough that other Glaux components can be designed, implemented, tested, and validated against them without ambiguity.

The server shall offer the same published CSAPI interfaces to Glaux components and external clients for operations covered by the standards. Any additional integration interface shall be explicitly documented and justified. An implementer shall be able to build, run, and exercise the server with documented dependencies and example data without first completing the other Glaux applications.

---

## 7. Implementation Character
Glaux Server shall be planned as a full-scope reference implementation component, not as a minimal subset, toy prototype, or temporary demonstration-only service.

Implementation may be sequenced, but sequencing decisions shall not reduce the intended capability model. Each implementation increment shall preserve architectural integrity and remain aligned to the complete Glaux Server definition.

Where a capability is not implemented immediately, the planning record shall identify whether it is deferred, dependent on another design decision, blocked by a standards interpretation question, or assigned to another Glaux component.

Glaux Server shall be implemented in **Rust** and released as open-source software. The implementation target is a robust, best-of-breed OGC API - Connected Systems reference server whose behavior is standards-correct, interoperable, testable, maintainable, and useful to other implementers.

Research and planning serve two equal purposes:

- reduce the risk that AI-assisted development misunderstands the standards or makes poorly informed implementation decisions; and
- equip AI-assisted development with the standards knowledge, implementation evidence, and engineering analysis needed to build the strongest reference implementation practical.

The completed [Glaux Server initial design research](../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/final-idr-research-report.md) and its linked topic reports provide supporting evidence, recommendations, and unresolved questions for implementation planning. The Goal and Definition establishes the intended outcome and scope; the Implementation Guide translates that goal into technical design; the Roadmap sets the order of delivery. Governance defines how these documents are reviewed and changed. Procedures for future research belong in governance.

Developer documentation shall explain how to build, configure, run, and test the reference server and exercise its capabilities using examples. Specific frameworks, libraries, databases, internal mechanisms, and test tools belong in the Implementation Guide; delivery increments and their sequencing belong in the Roadmap.

---

## 8. Out of Scope for Glaux Server
The Glaux Server project does not include:

- Building the full product functionality of Glaux Web App, Glaux Mobile, Glaux Publisher, or Glaux Simulator
- Replacing the authoritative OGC or NATO standards with project-specific technical definitions
- Implementing unrelated service families as primary server scope, including WMS, WFS, WMTS, STAC, TMS, NSILI, TAK/COT, or other adjacent service models, except where documented integration or interoperability hooks are required
- Creating a general-purpose enterprise platform unrelated to connected-system interoperability
- Providing production hosting, managed-service operations, 24/7 SRE, SOC operations, or cloud-account administration as project deliverables
- Providing enterprise identity infrastructure or cross-domain transfer guards, authorizing an organization's release of operational information, or accrediting deployments
- Providing network connectivity infrastructure or establishing federation agreements on behalf of deployments
- Defining operator tactics, techniques, and procedures
- Defining modality-specific sensor profiles unless assigned through a later Glaux planning artifact, AEP volume, SRD, or project decision

These exclusions do not prevent Glaux Server from supporting integration patterns, deployment guidance, security interfaces, mediation hooks, or interoperability behavior required by the wider Glaux ecosystem.

---

## 9. Non-Negotiable Requirements

- Glaux Server shall remain aligned to STANAG 4789 / AEP-4789 and the adopted Volume II standards package.
- CSAPI Part 1, CSAPI Part 2, SensorML, and SWE Common shall be treated as a coherent standards package.
- Standards references, interpretations, implementation decisions, and deviations shall be explicit, documented, and traceable. Draft support and Glaux extensions shall be clearly identified and shall not silently change the published CSAPI contract.
- Security, authorization, validation, error handling, and failure semantics shall be treated as core server design requirements.
- DDIL-informed operation, degraded connectivity, freshness, validity, synchronization, and constrained exchange shall be addressed in the architecture and planning baseline.
- The server shall preserve ecosystem compatibility with Glaux Web App, Glaux Mobile, Glaux Publisher, and Glaux Simulator.
- The server shall expose machine-readable behavior suitable for authorized users, applications, AI-enabled services, and interoperable software clients.
- Planning decisions shall be reviewable, decision-usable, and evidence-backed.
- No silent scope drift shall occur; major planning changes shall be versioned and recorded. A research recommendation for an internal mechanism or additional capability shall be identified and justified when incorporated into implementation planning. Acceptance of a research report does not by itself make every proposed mechanism a new project objective.
- Sequencing decisions shall not be described as reductions of the intended Glaux Server capability model.

---

## 10. Summary Statement
Glaux Server is intended to be a full-scope, open-source, standards-correct Rust reference implementation of OGC API - Connected Systems and its applicable SensorML and SWE Common requirements, serving the Glaux ecosystem and the STANAG 4789 / AEP-4789 core APIs and encodings package.

Its resources and APIs shall support connected-system discovery, description, access, exchange, streaming, status, events, and tasking across NATO, national, coalition, federated, tactical, and DDIL-informed environments. Planned experimental Part 3 support extends the publish/subscribe work with an explicit draft status.

The resulting server shall be understandable, maintainable, independently usable with documented dependencies, and verifiable through standards-based tests and external clients. The full intended capability remains the goal throughout incremental implementation.


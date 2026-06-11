# Section 032: External Source Registration and Trust Model - Research Plan

**Status:** Planned  
**Last Updated:** June 11, 2026  
**Estimated Research Time:** 12-16 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-032-external-source-registration-and-trust-model-report.md`

---

## Usage Instructions

Before executing this plan, review the exemplar corpus and align the report to the proven standards for content nature, type, level, and detail:

- Full research-plans corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans
- Blueprint-first exemplar: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/01-pr114-blueprint-analysis.md
- Inventory/sourcing rigor exemplar: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/15-fixture-sourcing-organization.md
- End-state synthesis exemplar: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/38-testing-playbook-synthesis.md

This plan is limited to research planning. It does not execute the research and does not produce the research report.

---

## 1. Research Objective

This topic research must define the Glaux Server planning baseline for the **external source registration and trust model** across publishers, adapters, simulators, gateways, federated sources, legacy feeds, direct API submitters, brokered message sources, source identities, source metadata, credentials, provenance, authorization, validation posture, trust levels, policy/releasability constraints, source health, source lifecycle, DDIL synchronization, and auditability.

The research must answer:

- What does Glaux Server need to know about an external source before accepting metadata, observations, status updates, system events, command-status reports, or command/control interactions from that source?
- How should external sources, publishers, adapters, simulators, gateways, brokers, federated CSAPI services, and command-capable gateways be registered, identified, authenticated, authorized, classified, trusted, monitored, disabled, retired, or quarantined?
- Which source-registration and trust concepts are required or implied by CSAPI, SensorML, SWE Common, AEP-4789 server responsibilities, NATO JISR operational constraints, and Glaux Server full-scope goals?
- What trust levels, source authority concepts, provenance records, source credentials, validation policies, data-acceptance policies, replay policies, and policy/releasability controls should Glaux Server support?
- How should Glaux Server distinguish trusted source identity, adapter identity, source-system identity, data-owner identity, command authority, validation status, source health, and provenance of individual records?
- How should the trust model support high-rate ingestion, metadata updates, source conflicts, source revocation, stale sources, DDIL reconnect, replay, federation, command/control safety, audit, conformance testing, fixtures, and interoperability?

The output must be an external source registration and trust model with source anchors, source-class taxonomy, source registration data model, trust-state model, credential/trust boundary guidance, provenance requirements, validation and policy implications, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows `IDR-SRV-031: Publisher and Adapter Integration Boundary Model`.

`IDR-SRV-031` defines the boundary between Glaux Server and external publishers/adapters. This topic defines how Glaux Server registers, identifies, authenticates, trusts, constrains, monitors, and audits those external contributors. It must precede dynamic data ingestion, datastream/observation semantics, streaming/event publication, command lifecycle, feasibility validation, command authorization/safety/audit, DDIL behavior, security threat modeling, policy/releasability enforcement, and source-oriented fixtures.

### Critical Constraints

- Treat CSAPI Part 1 and Part 2, SensorML 3.0, SWE Common 3.0, AEP-4789 server responsibilities, prior IDR findings, and Glaux Server full-scope objectives as controlling.
- Do not design the full authentication and authorization architecture here. Identify source-registration and trust requirements and hand full security architecture to Category G.
- Do not design the full ingestion pipeline here. Identify source-trust inputs to ingestion and hand detailed ingestion design to `IDR-SRV-033`.
- Do not assume all external sources are trusted, standards-compliant, continuously connected, non-malicious, synchronized, or stable.
- Do not collapse source identity, adapter identity, user identity, organization identity, publisher identity, data owner, and command authority into one concept.
- Do not treat simulated/test sources as operationally trusted sources.
- Do not define a trust model that requires always-online external validation or cloud-only identity systems.
- Do not expose sensitive source registration details, credentials, topology, classification/releasability labels, or command authority through public CSAPI responses unless explicitly allowed by downstream policy decisions.
- Keep the research bounded to Glaux Server behavior and server-side registration/trust contracts.

---

## 2. Research Questions

### Core Questions

1. What source-registration information must Glaux Server maintain before accepting metadata or dynamic data from external sources?
2. What trust states, source classes, source authority concepts, credential models, and provenance records should Glaux Server support?
3. How should source trust affect validation, ingestion, metadata updates, observations, status updates, events, command interactions, DDIL replay, and conflict handling?
4. How should Glaux Server monitor, revoke, quarantine, retire, or degrade trust for external sources?
5. What downstream implications follow for ingestion, streaming, command/control, security, DDIL, deployment, fixtures, conformance, and interoperability?

### Detailed Questions

#### Standards and Trust Baseline

- What source identity, source authority, provenance, or trust concepts are explicitly defined or implied by CSAPI Part 1?
- What source identity, source authority, provenance, or trust concepts are explicitly defined or implied by CSAPI Part 2?
- What SensorML and SWE Common metadata can identify source systems, procedures, owners, contacts, operators, or provenance?
- What AEP-4789 server responsibilities imply source registration, source trust, provenance, or cross-boundary data acceptance requirements?
- Which trust concepts are inherited from HTTP, API security, OGC API patterns, or deployment architecture rather than CSAPI itself?
- Which trust concepts are implementation-defined but required for a robust Glaux Server?

#### Source Class and Source Identity Taxonomy

- What external source classes must be represented:
  - standards-compliant CSAPI publisher,
  - OSH/OSH-Connected publisher,
  - legacy feed adapter,
  - simulator,
  - file/batch importer,
  - brokered publisher,
  - tactical gateway,
  - federated CSAPI source,
  - command/control gateway,
  - test fixture source,
  - administrative/manual source?
- How should Glaux Server distinguish source system identity, adapter identity, publisher identity, organization identity, operator identity, data owner, data steward, and command authority?
- Which source classes can be authoritative for metadata, observations, status, or commands?
- Which source classes should be allowed only in test/demo profiles?

#### Source Registration Data Model

- What fields should be recorded when an external source is registered:
  - source identifier,
  - display name,
  - source type/class,
  - organization/owner/operator,
  - adapter/publisher identifier,
  - protocol/integration pattern,
  - endpoint or broker topic,
  - credential reference,
  - supported resource families,
  - supported datastreams/control streams,
  - profile/version claims,
  - validation profile,
  - semantic/vocabulary profile,
  - trust state,
  - authority scope,
  - policy/releasability labels,
  - command authority scope,
  - expected heartbeat/health reporting,
  - retry/replay capability,
  - DDIL behavior,
  - provenance rules,
  - audit requirements,
  - lifecycle state?
- Which fields are public, administrator-only, sensitive, or secret references?
- Which fields are static configuration versus persisted administrative metadata?
- Which fields are required, optional, profile-specific, or deployment-specific?

#### Trust State and Lifecycle Model

- What trust states should be supported:
  - proposed,
  - registered,
  - pending validation,
  - trusted,
  - partially trusted,
  - constrained,
  - test-only,
  - simulated,
  - degraded,
  - suspended,
  - quarantined,
  - revoked,
  - retired?
- What events cause trust-state transitions?
- How should trust-state changes affect ingestion, metadata updates, event acceptance, command interactions, and public visibility?
- How should trust-state changes be audited?
- How should trust be restored after failure, revocation, reconnect, or DDIL operation?
- Which transitions require human/operator approval?

#### Credentials, Authentication, and Source Claims

- What credential patterns should be evaluated:
  - API keys,
  - mutual TLS certificates,
  - OAuth/OIDC client credentials,
  - signed tokens,
  - broker credentials,
  - file/import signatures,
  - local test credentials,
  - pre-shared tactical credentials,
  - offline credential bundles?
- Which credential material is secret versus metadata?
- How should credential references be stored without exposing secrets?
- How should source claims be validated and mapped to source registration records?
- How should source identity differ from user identity?
- How should credentials be rotated, revoked, expired, or scoped?

#### Source Authority and Provenance

- How should Glaux Server record source authority for metadata, observations, status updates, events, command status reports, and feasibility results?
- How should Glaux Server distinguish source-authored content, adapter-derived content, server-normalized content, inferred content, simulated content, and manually entered content?
- How should provenance be attached to individual resources and individual time-series records?
- How should source authority affect conflict resolution among multiple sources?
- What provenance is required for audit, conformance, debugging, and interoperability?
- Which provenance should be client-visible versus administrator-only?

#### Validation, Acceptance, and Quarantine Policy

- How should source trust affect validation strictness and acceptance behavior?
- Which data from untrusted, partially trusted, or constrained sources should be rejected, quarantined, accepted with warnings, or accepted only into a staging area?
- Which source claims should be independently validated by Glaux Server?
- How should Glaux Server handle invalid metadata, invalid observations, inconsistent datastream references, unsupported SWE/SensorML structures, unknown units, conflicting semantic terms, duplicate records, and source timestamp anomalies?
- Which validation failures affect source trust?
- Which findings should be handed to ingestion, validation, and conformance topics?

#### Metadata Update and Authority Scope

- Which sources may create, update, retire, or supersede systems, procedures, deployments, sampling features, properties, datastreams, SensorML, SWE Common definitions, and semantic mappings?
- How should source authority scope be represented?
- How should Glaux Server prevent unauthorized source updates to resources outside scope?
- How should conflicting source metadata be handled?
- How should source-supplied metadata updates trigger validation, resource lifecycle changes, system events, and audit records?

#### Dynamic Data, Status, Event, and Replay Trust

- Which sources may submit observations, status updates, dynamic property values, source health records, and system events?
- Which fields in submitted records should be trusted, normalized, overridden, or assigned by Glaux Server?
- How should source sequence numbers, offsets, batch IDs, message IDs, and replay claims be validated?
- How should source trust affect duplicate detection, out-of-order data, late data, stale data, and replay?
- How should source health and source trust interact?

#### Command/Control Trust and Authority

- Which sources or gateways may participate in command/control workflows?
- How should command authority be represented separately from data-submission authority?
- How should command-capable sources be registered, validated, constrained, audited, suspended, or revoked?
- How should Glaux Server verify that a command gateway may act for a given system or control stream?
- How should command status reports from external gateways be trusted?
- Which findings should be handed to command lifecycle, feasibility validation, command authorization, and security-test topics?

#### DDIL, Federation, and Offline Trust

- How should source trust operate when external identity validation is unavailable?
- What source registration and credential material must be available locally for DDIL operation?
- How should Glaux Server treat records received after reconnect or delayed synchronization?
- How should source trust and source authority be maintained across federated CSAPI servers?
- How should stale, revoked, suspended, or unknown sources be handled during disconnected operation?
- Which findings should be handed to DDIL and federation topics?

#### Source Health, Monitoring, and Diagnostics

- What source health fields should be tracked:
  - last contact,
  - heartbeat status,
  - latency,
  - backlog,
  - error rate,
  - validation failure rate,
  - replay lag,
  - credential status,
  - schema/profile mismatch,
  - source clock drift,
  - quarantine count?
- Which source health metrics affect trust state?
- Which source health information should be publicly visible, administrator-only, or internal-only?
- How should source health differ from sensor/system status?

#### Security, Policy, and Releasability

- How should source registration interact with authentication, authorization, policy, releasability, command authority, and audit?
- Which source registration fields are sensitive?
- How should public CSAPI responses avoid leaking source topology, internal adapters, source names, policy labels, credential references, or command authorities?
- How should policy filtering affect source-specific metadata, provenance, and query results?
- Which findings should be handed to Category G and security-test topics?

#### Configuration and Deployment

- What source trust settings are static configuration versus persisted administrative metadata?
- How should local development, CI, demo, operational, and tactical-edge profiles manage source registration?
- How should test sources, simulated sources, and demo sources be isolated from operational sources?
- How should source registration be bootstrapped?
- Which findings should be handed to deployment, configuration, and CI topics?

#### Fixtures, Conformance, and Interoperability

- What source-registration fixtures are needed:
  - trusted CSAPI source,
  - untrusted adapter,
  - simulator source,
  - command gateway,
  - revoked source,
  - stale source,
  - duplicate source,
  - federated source,
  - DDIL reconnect source,
  - source with invalid metadata,
  - source with conflicting observations?
- What conformance tests should verify source trust behavior?
- What interoperability tests should involve OSH, connected-systems-go, pygeoapi, SECD, CSAPI Explorer, OS4CSAPI clients, and external CSAPI clients?
- Which tests should be server-boundary tests versus full adapter tests?

#### Implementation and Interoperability Lessons

- What source registration, trust, provenance, credential, health, and adapter-identity lessons were identified in OSH, Connected Systems Go, pygeoapi, and SECD?
- What smoke-test or interoperability findings indicate source trust, source identity, provenance, duplicate handling, invalid source metadata, or command-gateway issues?
- What OS4CSAPI discussion lessons affect source registration and trust strategy?
- Which implementation patterns should Glaux Server adopt, avoid, or investigate further?
- Which findings must be treated as implementation-specific rather than standards-derived?

---

## 3. Primary Resources

The future research report must analyze these sources directly.

### Project and Governance Sources

- Glaux Server Overall IDR Research Plan: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Creation Prompt: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-creation-prompt.md
- Research Plan Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Overall Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/overall-research-report-template.md
- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md

### Prior IDR Topic Sources

- `IDR-SRV-001` through `IDR-SRV-031` research reports, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`

### Controlling Standards and API Sources

- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- W3C / OGC Semantic Sensor Network Ontology: https://www.w3.org/TR/vocab-ssn/
- OGC schemas: https://schemas.opengis.net/

### Security, Identity, and Provenance Candidate Sources

Use current official documentation and primary-source technical material when executing the research. Candidate sources include, but are not limited to:

- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9457 - Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457
- OAuth 2.0 RFC 6749: https://www.rfc-editor.org/rfc/rfc6749
- OAuth 2.0 Bearer Token RFC 6750: https://www.rfc-editor.org/rfc/rfc6750
- OAuth 2.0 Token Introspection RFC 7662: https://www.rfc-editor.org/rfc/rfc7662
- OpenID Connect Core: https://openid.net/specs/openid-connect-core-1_0.html
- TLS 1.3 RFC 8446: https://www.rfc-editor.org/rfc/rfc8446
- W3C PROV Overview: https://www.w3.org/TR/prov-overview/
- CloudEvents specification: https://cloudevents.io/
- NATS documentation: https://docs.nats.io/
- MQTT v5 specification: https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html

These are candidates for evaluation, not assumed selections.

### AEP-4789 and STANAG 4789 Sources

Use source material available to the Glaux project team and record exact title, volume, version, date, status, storage location, and classification/handling constraints:

- STANAG 4789
- AEP-4789 Volume I
- AEP-4789 Volume II
- Any approved or draft AEP-4789 schemas, profiles, implementation guidance, source registration guidance, source trust guidance, security guidance, provenance guidance, DDIL guidance, or standards-package annexes relevant to external sources, publishers, adapters, source authority, and NATO JISR sensor integration

### Implementation and Lessons-Learned Sources

Use implementation-study outputs and source repositories as non-normative evidence:

- OSH / OpenSensorHub sources and `IDR-SRV-014A`
- Connected Systems Go sources and `IDR-SRV-014B`
- pygeoapi sources and `IDR-SRV-014C`
- SECD sources and `IDR-SRV-014D`
- OS4CSAPI Client Smoke Test Findings and `IDR-SRV-014E`
- SECD Interoperability Findings and `IDR-SRV-014F`
- OS4CSAPI Discussions Lessons-Learned and `IDR-SRV-014G`
- Category C findings from `IDR-SRV-015` through `IDR-SRV-020`
- Category D findings from `IDR-SRV-021` through `IDR-SRV-024`
- Category E findings from `IDR-SRV-025` through `IDR-SRV-030`
- Publisher and Adapter Integration Boundary Model findings from `IDR-SRV-031`
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/

---

## 4. Supporting Resources

Use these sources to interpret project context, downstream dependencies, expected research depth, and report style.

- DGIWG P5.07 Glaux main repository: https://github.com/DGIWG-P507/glaux
- Glaux project website: https://dgiwg-p507.github.io/glaux/
- DGIWG P5.07 GitHub organization: https://github.com/DGIWG-P507
- Glaux Server repository, if available or created: https://github.com/DGIWG-P507/glaux-server
- Glaux Publisher repository, if available or created: https://github.com/DGIWG-P507/glaux-publisher
- Glaux Simulator repository, if available or created: https://github.com/DGIWG-P507/glaux-simulator
- OS4CSAPI testing research-plan corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

---

## 5. Research Methodology

### Phase 1: Source Collection and Trust Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and extraction framework for external source registration and trust research.

**Tasks:**

1. Gather prior IDR reports, standards sources, implementation studies, identity/provenance/security documentation, integration-boundary findings, and project architecture sources.
2. Extract source registration and trust requirements from each prior topic and classify them by source class, identity concept, trust state, authority scope, credential requirement, provenance need, and downstream dependency.
3. Define inventory fields:
   - source class,
   - identity concept,
   - registration field,
   - trust state,
   - authority scope,
   - credential/authentication pattern,
   - provenance requirement,
   - validation/acceptance policy,
   - security/policy implication,
   - downstream handoff.
4. Define evaluation criteria:
   - standards alignment,
   - trust clarity,
   - provenance strength,
   - security,
   - DDIL/offline suitability,
   - command safety,
   - operational usability,
   - source flexibility,
   - auditability,
   - fixture/testability,
   - interoperability.
5. Establish evidence hierarchy for standards, AEP material, prior reports, implementation evidence, and security/provenance documentation.

**Expected Output:** Source registration/trust extraction framework and evaluation rubric.

### Phase 2: Source Class, Identity, and Registration Data Inventory (3-4 hours)

**Objective:** Determine source classes, identity concepts, and registration metadata.

**Tasks:**

1. Inventory source classes and identity concepts from `IDR-SRV-031`, implementation studies, and standards sources.
2. Identify registration fields required for each source class and integration pattern.
3. Classify registration fields by public/sensitive/secret/admin-only status.
4. Identify source authority scopes for metadata, observations, status, events, commands, and feasibility interactions.
5. Build a source-registration data matrix.

**Expected Output:** Source class and registration data matrix.

### Phase 3: Trust State, Credential, Provenance, and Validation Analysis (3-4 hours)

**Objective:** Define source trust, authentication, provenance, and data-acceptance behavior.

**Tasks:**

1. Analyze trust states, lifecycle transitions, suspension/revocation/quarantine behavior, and audit requirements.
2. Analyze credential and authentication patterns appropriate for source registration and source claims.
3. Analyze provenance requirements for source-authored, adapter-derived, server-normalized, inferred, simulated, and manually entered content.
4. Analyze validation and acceptance behavior by trust state and source class.
5. Identify unresolved questions requiring prototype validation or security review.

**Expected Output:** Trust/credential/provenance/validation strategy matrix.

### Phase 4: Dynamic Data, Commands, DDIL, and Conflict Analysis (2.5-3.5 hours)

**Objective:** Analyze high-risk source trust interactions.

**Tasks:**

1. Analyze observation, status, event, and replay trust behavior.
2. Analyze source sequence, offset, batch, message ID, and duplicate-detection trust behavior.
3. Analyze command-capable source and gateway trust requirements.
4. Analyze DDIL, offline credential, stale trust, reconnect, synchronization, and federation implications.
5. Analyze source conflict resolution and authority scope implications.
6. Map findings to Category F, G, and DDIL topics.

**Expected Output:** High-risk source trust interaction matrix.

### Phase 5: Security, Fixtures, Conformance, Deployment, and Interoperability Implication Analysis (2.5-3 hours)

**Objective:** Prepare source-registration/trust findings for downstream implementation and verification work.

**Tasks:**

1. Identify security, policy, releasability, source-health, and audit implications.
2. Identify source-registration fixtures and simulated trust-state scenarios.
3. Identify conformance tests for server-side source trust behavior.
4. Identify interoperability tests with OSH, connected-systems-go, pygeoapi, SECD, CSAPI Explorer, OS4CSAPI clients, and external CSAPI clients.
5. Identify deployment, configuration, bootstrap, admin workflow, and local development implications.
6. Map findings to Category F, G, H, and I topics.

**Expected Output:** Source trust downstream implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert source registration and trust research into a decision-usable baseline.

**Tasks:**

1. Consolidate source class taxonomy, registration data model, trust-state model, credential/provenance guidance, validation/acceptance guidance, and downstream implications.
2. Produce recommended external source registration and trust model with rationale and unresolved questions.
3. Identify sequencing for ingestion, streaming, command, DDIL, security, deployment, fixture, conformance, and performance topics.
4. Produce downstream handoff matrix.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] External source classes, identity concepts, and registration fields are identified with source anchors.
- [ ] Source identity, adapter identity, publisher identity, user identity, data-owner identity, and command authority are distinguished.
- [ ] Trust states, source authority scopes, credential patterns, lifecycle transitions, revocation, quarantine, and health-monitoring implications are documented.
- [ ] Provenance, validation/acceptance, metadata update, observation/status/event, command, replay, DDIL, and conflict implications are documented.
- [ ] Security, policy, releasability, fixture, conformance, deployment, and interoperability implications are documented.
- [ ] Implementation-study and community-lesson findings are incorporated as non-normative evidence.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** External Source Registration and Trust Model Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-032-external-source-registration-and-trust-model-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Source registration/trust extraction methodology
5. External source class taxonomy
6. Source identity and authority concept taxonomy
7. Source registration data model findings
8. Trust state and lifecycle findings
9. Credential and authentication pattern findings
10. Provenance and source authority findings
11. Validation, acceptance, quarantine, and conflict handling findings
12. Metadata update, observation/status/event, and replay trust findings
13. Command/control gateway and command authority findings
14. DDIL, federation, offline credential, and synchronization findings
15. Security, policy, releasability, source health, and audit implications
16. Fixture, conformance, deployment, performance, and interoperability test implications
17. Downstream topic handoff matrix
18. Recommendations
19. Risks, constraints, and open questions
20. Validation against this plan's success criteria
21. References

The source registration and trust matrix should include, at minimum:

- Source class
- Identity concept
- Registration field
- Public/sensitive/secret/admin-only classification
- Trust state
- Authority scope
- Credential/authentication pattern
- Provenance requirement
- Validation/acceptance policy
- Replay/DDIL implication
- Command authority implication
- Security/policy implication
- Test/conformance implication
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` through `IDR-SRV-031` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, OGC API - Features, SensorML, SWE Common, relevant OGC schemas/OpenAPI artifacts, identity/provenance/security sources, and project-available AEP-4789 material must be reachable or explicitly marked unavailable.
- Implementation-study and interoperability findings must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-033: Dynamic Data Ingestion and Normalization Pipeline`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-035: Streaming and Event Publication Strategy`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-037: Feasibility and Command Validation Strategy`
- `IDR-SRV-038: Command Authorization, Safety, and Audit Strategy`
- `IDR-SRV-039: Authentication, Authorization, and API Security Threat Model`
- `IDR-SRV-040: Policy, Releasability, and Cross-Boundary Access Constraints`
- `IDR-SRV-041: DDIL Behavior, Caching, and Synchronization Semantics`
- `IDR-SRV-045: Service Packaging, Containerization, and Deployment Topology`
- `IDR-SRV-046: Local Development and CI Environment Strategy`
- `IDR-SRV-049: Observability, Logging, Metrics, and Operational Diagnostics`
- `IDR-SRV-053: Test Data, Fixtures, Golden Files, and Scenario Corpus Strategy`
- `IDR-SRV-054: Performance, Load, Stress, and Streaming Test Strategy`
- `IDR-SRV-055: Security, Authorization, and Command-Control Test Strategy`
- `IDR-SRV-056: Interoperability Test Matrix for External CSAPI Clients`
- Final overall IDR synthesis report

---

## 9. Research Status Checklist

Update this section as work progresses.

- [ ] Phase 1 complete
- [ ] Phase 2 complete
- [ ] Phase 3 complete
- [ ] Phase 4 complete
- [ ] Phase 5 complete
- [ ] Phase 6 synthesis complete
- [ ] Deliverable draft complete
- [ ] Deliverable reviewed
- [ ] Deliverable accepted

**Actual Research Time:** TBD until complete  
**Completion Date:** TBD until complete

---

## 10. Notes and Open Questions

- This topic defines source registration and trust strategy, not final authentication/authorization implementation.
- Source identity, adapter identity, publisher identity, data owner, and command authority must remain distinct.
- Implementation-study findings are useful but must not override standards-derived server responsibilities or security requirements.
- Open question: Which trust states are required for first implementation versus full-scope readiness?
- Open question: Which registration fields should be public, administrator-only, sensitive, or secret references?
- Open question: How should trust state degrade automatically based on validation failures or source-health signals?
- Open question: How should command-capable gateways be registered and constrained?
- Open question: How should source trust operate during disconnected/DDIL conditions?
- Risk: Treating all registered sources as fully trusted could create data-quality, policy, and command-safety failures.
- Risk: Weak provenance could make audit, debugging, conformance, and conflict resolution unreliable.
- Risk: Exposing source registration details could leak topology, policy, or command-authority information.
- Risk: Overly strict trust requirements could prevent useful legacy adapters or tactical gateways from participating.

---

## References

- Glaux Server Overall IDR Research Plan: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Creation Prompt: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-creation-prompt.md
- Research Plan Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Overall Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/overall-research-report-template.md
- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md
- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- W3C / OGC Semantic Sensor Network Ontology: https://www.w3.org/TR/vocab-ssn/
- OGC schemas: https://schemas.opengis.net/
- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9457 - Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457
- OAuth 2.0 RFC 6749: https://www.rfc-editor.org/rfc/rfc6749
- OAuth 2.0 Bearer Token RFC 6750: https://www.rfc-editor.org/rfc/rfc6750
- OAuth 2.0 Token Introspection RFC 7662: https://www.rfc-editor.org/rfc/rfc7662
- OpenID Connect Core: https://openid.net/specs/openid-connect-core-1_0.html
- TLS 1.3 RFC 8446: https://www.rfc-editor.org/rfc/rfc8446
- W3C PROV Overview: https://www.w3.org/TR/prov-overview/
- CloudEvents specification: https://cloudevents.io/
- NATS documentation: https://docs.nats.io/
- MQTT v5 specification: https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

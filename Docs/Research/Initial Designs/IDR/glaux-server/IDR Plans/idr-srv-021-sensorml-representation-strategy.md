# Section 021: SensorML Representation Strategy - Research Plan

**Status:** Planned  
**Last Updated:** June 8, 2026  
**Estimated Research Time:** 12-16 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-021-sensorml-representation-strategy-report.md`

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

This topic research must define the Glaux Server planning baseline for **SensorML representation strategy** across systems, procedures, deployments, platforms, components, capabilities, characteristics, positions, contacts, inputs, outputs, modes, process descriptions, and CSAPI resources.

The research must answer:

- What SensorML 3.0 concepts must Glaux Server support, produce, ingest, preserve, transform, validate, and expose?
- How should Glaux Server map SensorML concepts to canonical CSAPI resource families from `IDR-SRV-015` through `IDR-SRV-020`?
- Which SensorML content belongs in externally exposed API representations, internal domain model fields, document storage, metadata records, validation rules, or derived views?
- How should Glaux Server distinguish SensorML system descriptions, procedures/processes, deployed systems, platforms, components, capabilities, characteristics, contacts, identifiers, classifiers, positions, inputs, outputs, and modes?
- How should SensorML representations interact with SWE Common data components, observed properties, controlled properties, datastreams, observations, status, events, and command/control resources?
- How should Glaux Server handle imported SensorML, generated SensorML, partial SensorML, externally authored SensorML, legacy SensorML, and STANAG/AEP profile expectations?
- What downstream implications follow for SWE Common, schema/encoding validation, semantic binding, metadata/document storage, dynamic data, command lifecycle, security, fixtures, conformance, and interoperability testing?

The output must be a SensorML representation strategy baseline with source anchors, mapping tables, validation implications, representation guidance, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows the Category C resource/domain-model block:

- `IDR-SRV-015: Canonical Glaux Server Resource Model`
- `IDR-SRV-016: Identifier, URI, and Resource Lifecycle Strategy`
- `IDR-SRV-017: Relationship and Linkage Model`
- `IDR-SRV-018: Temporal, Validity, and Freshness Model`
- `IDR-SRV-019: Registration, Update, and State Change Semantics`
- `IDR-SRV-020: Status, Availability, and System Event Model`

Those topics define the server's resource families, identity, relationships, lifecycle, temporal behavior, and status/event concepts. This topic starts Category D: SensorML, SWE Common, and Semantic Representation. It determines how SensorML should represent descriptive system/procedure/deployment metadata and how those representations should support CSAPI interoperability, validation, and downstream server design.

### Critical Constraint(s)

- Treat SensorML 3.0, OGC API - Connected Systems Part 1 and Part 2, SWE Common 3.0, OGC API - Features, AEP-4789 Volume II adoption context, and earlier IDR findings as controlling.
- Do not treat SensorML as the entire Glaux Server domain model. Distinguish SensorML representation, CSAPI API resources, internal domain entities, and persistence records.
- Do not finalize SWE Common data-component strategy here. Identify SWE dependencies and hand detailed analysis to `IDR-SRV-022`.
- Do not finalize schema/encoding validation here. Identify SensorML validation needs and hand detailed validation strategy to `IDR-SRV-023`.
- Do not finalize observed-property/unit/semantic binding strategy here. Identify semantic dependencies and hand them to `IDR-SRV-024`.
- Do not design metadata/document storage here. Identify storage implications and hand them to `IDR-SRV-028`.
- Do not assume imported SensorML can be trusted, complete, profile-conformant, or directly exposed without validation and policy checks.
- Do not force every CSAPI field into SensorML or every SensorML element into first-class Glaux Server fields.
- Keep the research bounded to Glaux Server behavior and server-side contracts.

---

## 2. Research Questions

### Core Questions

1. What SensorML concepts must Glaux Server support for CSAPI and AEP-4789 server responsibilities?
2. How should SensorML map to Glaux Server canonical resources and internal domain entities?
3. Which SensorML content should be stored, validated, normalized, indexed, exposed, generated, or preserved as source documents?
4. How should SensorML interact with SWE Common, observed properties, controlled properties, status, events, datastreams, observations, and commands?
5. What downstream implications follow for validation, persistence, security, fixtures, conformance, and interoperability?

### Detailed Questions

#### SensorML Source and Concept Baseline

- What SensorML 3.0 constructs are relevant to Glaux Server?
- Which SensorML constructs are required, recommended, optional, or profile-specific for CSAPI?
- Which SensorML constructs are directly referenced by CSAPI Part 1 or Part 2?
- Which SensorML constructs are relevant to AEP-4789 server responsibilities?
- Which SensorML constructs appear in implementation-study examples, fixtures, demos, or client smoke tests?
- Which SensorML constructs are likely unnecessary or out of scope for initial Glaux Server server-side behavior?

#### Mapping to Canonical Resource Families

- How should SensorML concepts map to systems, procedures, deployments, sampling features, properties, datastreams, observations, status and dynamic properties, system events, control streams, commands, feasibility resources, and links?
- Which SensorML concepts represent systems versus procedures/processes versus deployments?
- How should components, subsystems, platforms, attached sensors, and aggregations be represented?
- How should SensorML inputs and outputs map to datastreams, observed properties, controlled properties, command parameters, and result structures?
- Which mappings are direct, derived, partial, ambiguous, or profile-dependent?

#### Representation Boundary Analysis

- Which SensorML content should be exposed as native SensorML documents?
- Which SensorML content should be represented in CSAPI JSON/GeoJSON resource fields?
- Which SensorML content should be preserved as source documents but not normalized?
- Which SensorML content should be normalized into Glaux Server internal fields for query, validation, linking, or policy?
- Which SensorML content should be indexed for discovery?
- Which SensorML content should remain opaque to Glaux Server?
- How should Glaux Server support both standards-conformant representation and practical API usability?

#### Identifiers, Classifiers, Contacts, and Metadata

- How should SensorML identifiers relate to Glaux Server canonical identifiers and alternate identifiers?
- How should SensorML classifiers, keywords, contacts, documentation, security markings, and metadata be represented?
- Which metadata elements should be discoverable through CSAPI queries or links?
- How should imported SensorML identifiers and externally supplied metadata be trusted, preserved, normalized, or rejected?
- Which findings should be handed to `IDR-SRV-016`, `IDR-SRV-024`, `IDR-SRV-028`, and Category G topics?

#### Capabilities, Characteristics, Modes, and Positions

- How should SensorML capabilities and characteristics be represented in Glaux Server?
- Which capabilities and characteristics should map to CSAPI properties?
- How should operational modes, configuration modes, and sensor/platform positions be represented?
- How should static, time-varying, derived, and status-like characteristics be distinguished?
- Which position, orientation, geometry, CRS, and geospatial implications should be handed to storage/query and representation topics?

#### Inputs, Outputs, Results, and SWE Common Dependencies

- How should SensorML inputs and outputs be modeled relative to SWE Common data components?
- Which outputs map to datastream definitions?
- Which inputs map to command parameters, control streams, feasibility requests, or controlled properties?
- How should result structures, encodings, units, observed properties, and controlled properties be represented?
- Which findings should be handed to `IDR-SRV-022` and `IDR-SRV-024`?

#### Procedures, Deployments, and Provenance

- How should Glaux Server distinguish a procedure/process description from an installed/deployed system?
- How should SensorML support provenance of observations, deployments, data streams, and commands?
- How should deployment-specific metadata override or supplement procedure metadata?
- How should versioned or updated SensorML descriptions affect resource identity and lifecycle?
- Which findings should be handed to `IDR-SRV-019`, `IDR-SRV-027`, and `IDR-SRV-028`?

#### Import, Generation, and Transformation

- Should Glaux Server ingest externally authored SensorML?
- Should Glaux Server generate SensorML from internal records?
- Should Glaux Server transform SensorML into CSAPI resource summaries or other encodings?
- How should partial, invalid, legacy, or profile-divergent SensorML be handled?
- What validation, warning, normalization, and preservation behaviors are needed?
- How should transformations preserve source provenance and avoid lossy behavior?

#### Validation and Conformance Implications

- What SensorML validation rules are needed for Glaux Server?
- What should be validated structurally, semantically, profile-specifically, and operationally?
- What validation should happen on import, registration, update, response generation, and conformance testing?
- What errors or warnings should be produced when SensorML is incomplete or inconsistent with CSAPI resources?
- Which findings should be handed to `IDR-SRV-023`, `IDR-SRV-050`, and `IDR-SRV-051`?

#### Security, Policy, and Releasability Implications

- What sensitive information may appear in SensorML descriptions?
- How should contacts, locations, capabilities, characteristics, command inputs, configuration details, or provenance be filtered or protected?
- How should policy and releasability affect SensorML source documents versus derived CSAPI views?
- How should Glaux Server avoid leaking restricted sensor capabilities, locations, system associations, or command/control affordances?
- Which findings should be handed to Category G and security-test topics?

#### Implementation and Interoperability Lessons

- What SensorML lessons were identified in OSH, Connected Systems Go, pygeoapi, and SECD?
- What smoke-test or interoperability findings indicate SensorML parsing, validation, representation, link, identifier, or schema issues?
- What OS4CSAPI discussion lessons affect SensorML strategy?
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

- `IDR-SRV-001` through `IDR-SRV-020` research reports, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`

### Controlling Standards Sources

- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC schemas: https://schemas.opengis.net/
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- W3C / OGC Semantic Sensor Network Ontology: https://www.w3.org/TR/vocab-ssn/

### AEP-4789 and STANAG 4789 Sources

Use source material available to the Glaux project team and record exact title, volume, version, date, status, storage location, and classification/handling constraints:

- STANAG 4789
- AEP-4789 Volume I
- AEP-4789 Volume II
- Any approved or draft AEP-4789 schemas, profiles, implementation guidance, or standards-package annexes relevant to SensorML, system descriptions, procedures, deployments, server responsibilities, and NATO JISR sensor metadata

### Implementation and Lessons-Learned Sources

Use implementation-study outputs and source repositories as non-normative evidence:

- OSH / OpenSensorHub sources and `IDR-SRV-014A`
- Connected Systems Go sources and `IDR-SRV-014B`
- pygeoapi sources and `IDR-SRV-014C`
- SECD sources and `IDR-SRV-014D`
- OS4CSAPI Client Smoke Test Findings and `IDR-SRV-014E`
- SECD Interoperability Findings and `IDR-SRV-014F`
- OS4CSAPI Discussions Lessons-Learned and `IDR-SRV-014G`
- Canonical and Category C findings from `IDR-SRV-015` through `IDR-SRV-020`
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
- OS4CSAPI testing research-plan corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

---

## 5. Research Methodology

### Phase 1: Source Collection and SensorML Extraction Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and extraction framework for SensorML strategy.

**Tasks:**

1. Gather SensorML, SWE Common, CSAPI, AEP-4789, prior IDR reports, implementation studies, smoke-test findings, interoperability findings, and discussion lessons.
2. Extract SensorML constructs, examples, schema references, profiles, mappings, and implementation observations.
3. Define inventory fields:
   - SensorML concept,
   - source authority,
   - related CSAPI resource family,
   - internal/external exposure,
   - representation pattern,
   - normalization need,
   - validation need,
   - storage need,
   - query/index need,
   - security/policy implication,
   - test implication,
   - downstream handoff.
4. Define classification values:
   - normative,
   - inherited,
   - standards-package-specific,
   - representation-specific,
   - implementation-support,
   - implementation-specific,
   - optional,
   - unresolved.
5. Establish evidence hierarchy for standards, AEP material, prior reports, implementation observations, and community discussion.

**Expected Output:** Source inventory and SensorML extraction framework.

### Phase 2: SensorML Standards and CSAPI Mapping Analysis (3-4 hours)

**Objective:** Extract SensorML concepts and map them to CSAPI and Glaux Server resources.

**Tasks:**

1. Review SensorML 3.0 constructs relevant to systems, processes, procedures, deployments, components, capabilities, characteristics, contacts, identifiers, classifiers, positions, inputs, outputs, and modes.
2. Map SensorML constructs to CSAPI Part 1 and Part 2 resources.
3. Identify where SensorML is referenced, required, optional, or implied by CSAPI and AEP-4789 context.
4. Identify direct, indirect, derived, partial, ambiguous, and unresolved mappings.
5. Build a SensorML-to-Glaux resource mapping table.

**Expected Output:** SensorML concept and CSAPI mapping matrix.

### Phase 3: Representation Boundary and Normalization Analysis (2.5-3.5 hours)

**Objective:** Determine which SensorML content should be stored, normalized, indexed, exposed, generated, or preserved.

**Tasks:**

1. Distinguish SensorML source documents, CSAPI representations, internal domain fields, document-storage records, derived metadata, and validation artifacts.
2. Identify SensorML elements that should be normalized for discovery, query, linking, authorization, and interoperability.
3. Identify SensorML elements that should remain opaque or source-preserved.
4. Identify import, generation, transformation, and round-trip preservation concerns.
5. Identify metadata/document storage implications for `IDR-SRV-028`.

**Expected Output:** Representation boundary and normalization guidance.

### Phase 4: SWE, Semantic, Temporal, Status, and Command Dependency Analysis (2.5-3 hours)

**Objective:** Identify cross-topic dependencies created by SensorML representation.

**Tasks:**

1. Identify SensorML dependencies on SWE Common data components and encodings.
2. Identify observed-property, controlled-property, unit, semantic, and SSN/SOSA dependencies.
3. Identify temporal, deployment, lifecycle, status, event, and position implications.
4. Identify command/control, input/output, feasibility, and mode implications.
5. Map findings to `IDR-SRV-022`, `IDR-SRV-023`, `IDR-SRV-024`, `IDR-SRV-034`, `IDR-SRV-036`, and `IDR-SRV-038`.

**Expected Output:** Cross-topic dependency and handoff matrix.

### Phase 5: Validation, Security, Fixture, and Interoperability Implication Analysis (2.5-3 hours)

**Objective:** Prepare SensorML findings for downstream validation and implementation work.

**Tasks:**

1. Identify structural, semantic, profile, and operational validation needs.
2. Identify security, policy, releasability, and source-trust concerns.
3. Identify fixtures, sample SensorML documents, golden files, invalid examples, and transformation test cases needed.
4. Identify conformance, interoperability, generated-client, CSAPI Explorer, and external-tool compatibility tests.
5. Incorporate non-normative lessons from OSH, CS-GO, pygeoapi, SECD, OS4CSAPI smoke tests, interoperability findings, and discussions.

**Expected Output:** SensorML validation/security/test implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert SensorML research into a decision-usable representation strategy baseline.

**Tasks:**

1. Consolidate standards-derived SensorML inventory, CSAPI mapping, representation boundaries, normalization findings, dependencies, and downstream implications.
2. Produce SensorML representation recommendations by Glaux Server resource family.
3. Resolve conflicts and ambiguities where possible.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] SensorML concepts relevant to Glaux Server are identified with source anchors.
- [ ] SensorML concepts are mapped to canonical CSAPI resource families.
- [ ] SensorML representation, internal domain model, document storage, persistence, and validation boundaries are distinguished.
- [ ] Import, generation, transformation, preservation, and partial/invalid SensorML handling implications are documented.
- [ ] SWE Common, semantic binding, temporal, status/event, command/control, and persistence dependencies are documented.
- [ ] Validation, security, conformance, fixture, and interoperability implications are documented.
- [ ] Implementation-study and community-lesson findings are incorporated as non-normative evidence.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** SensorML Representation Strategy Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-021-sensorml-representation-strategy-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. SensorML extraction methodology
5. SensorML concept inventory
6. SensorML-to-CSAPI / Glaux resource mapping
7. Representation boundary findings
8. Import, generation, transformation, and preservation findings
9. Identifiers, classifiers, contacts, and metadata findings
10. Capabilities, characteristics, modes, positions, inputs, and outputs findings
11. SWE Common, semantic binding, temporal, status/event, and command dependencies
12. Validation, profile, schema, and conformance implications
13. Security, policy, and releasability implications
14. Fixture, golden-file, and interoperability test implications
15. Downstream topic handoff matrix
16. Recommendations
17. Risks, constraints, and open questions
18. Validation against this plan's success criteria
19. References

The SensorML mapping matrix should include, at minimum:

- SensorML concept
- Source standard / source anchor
- Authority classification
- Related CSAPI / Glaux resource family
- Representation pattern
- Internal normalization need
- Source-document preservation need
- Query/index need
- Validation implication
- SWE Common dependency
- Semantic binding implication
- Security/policy implication
- Test implication
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` through `IDR-SRV-020` research reports should be complete or explicitly marked unavailable/deferred.
- Official SensorML 3.0, SWE Common 3.0, CSAPI Part 1 and Part 2, OGC API - Features, OGC schemas, OpenAPI artifacts, SSN/SOSA, and project-available AEP-4789 material must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-022: SWE Common Data Component Strategy`
- `IDR-SRV-023: Schema and Encoding Validation Strategy`
- `IDR-SRV-024: Units, Observed Properties, and Semantic Binding Strategy`
- `IDR-SRV-028: Metadata and Document Storage Strategy`
- `IDR-SRV-033: Dynamic Data Ingestion and Normalization Pipeline`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-038: Command Authorization, Safety, and Audit Strategy`
- `IDR-SRV-039: Authentication, Authorization, and API Security Threat Model`
- `IDR-SRV-040: Policy, Releasability, and Cross-Boundary Access Constraints`
- `IDR-SRV-050: Conformance Harness Strategy`
- `IDR-SRV-051: Requirement-to-Test Traceability Strategy`
- `IDR-SRV-053: Test Data, Fixtures, Golden Files, and Scenario Corpus Strategy`
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

- This topic defines SensorML representation strategy, not the whole Glaux Server domain model.
- SensorML should be treated as a standards representation and metadata model that interacts with CSAPI resources, not as a substitute for all server-internal state.
- Implementation-study findings are useful but must not override SensorML and CSAPI semantics.
- Open question: Which SensorML elements should be normalized into first-class Glaux Server fields?
- Open question: How much round-trip fidelity should be required for imported SensorML documents?
- Open question: How should partial, invalid, legacy, or profile-divergent SensorML be handled?
- Open question: Which SensorML fields are operationally sensitive and require policy filtering?
- Open question: Which SensorML examples should become canonical fixtures?
- Risk: Treating SensorML as opaque only could limit discovery, validation, and interoperability.
- Risk: Over-normalizing SensorML could create brittle mappings and loss of source fidelity.
- Risk: Exposing imported SensorML without validation or policy checks could leak sensitive information or break clients.
- Risk: Treating one implementation's SensorML profile as canonical could distort Glaux Server design.

---

## References

- Glaux Server Overall IDR Research Plan: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Creation Prompt: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-creation-prompt.md
- Research Plan Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Overall Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/overall-research-report-template.md
- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC schemas: https://schemas.opengis.net/
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- W3C / OGC Semantic Sensor Network Ontology: https://www.w3.org/TR/vocab-ssn/
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

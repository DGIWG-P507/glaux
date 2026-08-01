# Section 022: SWE Common Data Component Strategy - Research Plan

**Status:** Planned  
**Last Updated:** June 10, 2026  
**Estimated Research Time:** 12-16 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-022-swe-common-data-component-strategy-report.md`

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

This topic research must define the Glaux Server planning baseline for **SWE Common data component strategy** across datastreams, observations, dynamic properties, status values, control streams, commands, feasibility requests, result structures, units, encodings, and CSAPI resource representations.

The research must answer:

- What SWE Common 3.0 concepts must Glaux Server support, produce, ingest, preserve, transform, validate, and expose?
- How should SWE Common data components map to canonical Glaux Server resource families, SensorML inputs/outputs, observed properties, controlled properties, datastream schemas, observation result structures, status values, command parameters, and feasibility inputs?
- Which SWE Common content belongs in externally exposed CSAPI representations, internal domain fields, validation artifacts, schema/encoding rules, persistence metadata, or derived views?
- How should Glaux Server distinguish data components, data records, quantities, categories, booleans, text fields, vectors, arrays, choices, ranges, time fields, nil values, constraints, units of measure, semantic definitions, and encodings?
- How should SWE Common interact with SensorML representation strategy, schema/encoding validation, unit/observed-property semantics, dynamic-data ingestion, time-series storage, command lifecycle, and security/audit behavior?
- How should Glaux Server handle imported SWE Common structures, generated SWE Common structures, partial or profile-divergent structures, legacy/example structures, and AEP-4789 profile expectations?

The output must be a SWE Common data component strategy baseline with source anchors, mapping tables, component-type guidance, validation implications, encoding implications, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows `IDR-SRV-021: SensorML Representation Strategy`.

SensorML defines system/procedure metadata and refers to inputs, outputs, parameters, characteristics, and capabilities that frequently depend on SWE Common data components. This topic defines how Glaux Server should handle those components across dynamic data, observations, status, command/control, and schema validation. It must precede schema/encoding validation, units/observed-property/semantic binding strategy, time-series storage, ingestion normalization, datastream/observation semantics, command lifecycle modeling, test-data fixtures, conformance harness design, and interoperability testing.

### Critical Constraint(s)

- Treat SWE Common 3.0, SensorML 3.0, OGC API - Connected Systems Part 1 and Part 2, OGC API - Features, AEP-4789 Volume II adoption context, and earlier IDR findings as controlling.
- Do not treat SWE Common as the entire Glaux Server data model. Distinguish SWE Common representation, CSAPI API resources, internal domain entities, persistence records, and validation artifacts.
- Do not finalize unit, observed-property, controlled-property, or semantic binding policy here. Identify semantic dependencies and hand detailed analysis to `IDR-SRV-024`.
- Do not finalize schema/encoding validation here. Identify SWE Common validation needs and hand detailed validation strategy to `IDR-SRV-023`.
- Do not design time-series storage here. Identify storage and indexing implications and hand them to `IDR-SRV-027`.
- Do not finalize command lifecycle semantics here. Identify command-parameter and feasibility dependencies and hand detailed command lifecycle work to `IDR-SRV-036`.
- Do not assume imported SWE Common definitions can be trusted, complete, profile-conformant, or directly exposed without validation and policy checks.
- Keep the research bounded to Glaux Server behavior and server-side contracts.

---

## 2. Research Questions

### Core Questions

1. What SWE Common concepts must Glaux Server support for CSAPI and AEP-4789 server responsibilities?
2. How should SWE Common map to datastreams, observations, status values, command parameters, feasibility requests, and SensorML inputs/outputs?
3. Which SWE Common content should be stored, validated, normalized, indexed, exposed, generated, or preserved?
4. How should SWE Common interact with units, observed properties, controlled properties, semantic definitions, encodings, and schema validation?
5. What downstream implications follow for persistence, ingestion, dynamic data, commands, security, fixtures, conformance, and interoperability?

### Detailed Questions

#### SWE Common Source and Concept Baseline

- What SWE Common 3.0 constructs are relevant to Glaux Server?
- Which SWE Common constructs are required, recommended, optional, or profile-specific for CSAPI?
- Which SWE Common constructs are directly referenced by CSAPI Part 1 or Part 2?
- Which SWE Common constructs are relevant to AEP-4789 server responsibilities?
- Which SWE Common constructs appear in implementation-study examples, fixtures, demos, or client smoke tests?
- Which SWE Common constructs are likely unnecessary or out of scope for initial Glaux Server behavior?

#### Data Component Type Strategy

- How should Glaux Server support SWE Common component types such as Quantity, Count, Boolean, Category, Text, Time, DataRecord, DataChoice, Vector, DataArray, Matrix, Range, nil values, constraints, and quality metadata?
- Which component types are expected in observation results, status values, command parameters, and feasibility requests?
- Which component types should be fully supported, partially supported, deferred, or explicitly rejected?
- Which component types require special validation, serialization, persistence, indexing, or policy handling?

#### Mapping to CSAPI and Glaux Resource Families

- How should SWE Common concepts map to properties, datastreams, observations, status and dynamic properties, system events, control streams, commands, command status, feasibility resources, and SensorML inputs/outputs?
- Which mappings are direct, derived, partial, ambiguous, or profile-dependent?
- How should result structure, component definitions, encodings, observed properties, and controlled properties be linked to datastreams and control streams?
- How should Glaux Server distinguish data component definitions from actual observation values or command values?

#### Representation Boundary Analysis

- Which SWE Common content should be exposed in CSAPI JSON responses?
- Which SWE Common content should be represented as schemas, component definitions, or linked metadata?
- Which SWE Common content should be preserved as source structures but not normalized?
- Which SWE Common content should be normalized into internal fields for query, validation, conversion, or policy?
- Which SWE Common content should be indexed for discovery or filtering?
- How should Glaux Server support both standards-conformant representation and practical API usability?

#### Encodings and Serialization

- What SWE Common encoding concepts are relevant to JSON, binary encodings, arrays, records, and future pub/sub or streaming behavior?
- Which encodings are required now, expected later, optional, or out of scope?
- How should encoding definitions relate to content negotiation and media types from `IDR-SRV-012`?
- How should encoding definitions relate to OpenAPI and schema generation from `IDR-SRV-014`?
- How should encoding definitions support observations, latest values, status, streaming, and command/control?
- Which findings should be handed to `IDR-SRV-023`, `IDR-SRV-034`, and `IDR-SRV-035`?

#### Units, Observed Properties, Controlled Properties, and Semantics

- How should SWE Common units of measure be represented?
- How should observed properties and controlled properties be referenced?
- How should semantic definitions, URIs, labels, definitions, and vocabularies be represented?
- Which unit and semantic validation responsibilities belong in this topic versus `IDR-SRV-024`?
- How should Glaux Server handle missing, ambiguous, non-standard, conflicting, or profile-specific units and properties?

#### Observation, Status, and Dynamic Data Implications

- How should observation result structures be represented with SWE Common?
- How should result values be validated against data component definitions?
- How should status values and dynamic properties be represented?
- How should latest values, historical observations, and event payloads use or refer to SWE Common structures?
- How should null/nil/missing/error/out-of-range values be represented and validated?
- Which findings should be handed to `IDR-SRV-027`, `IDR-SRV-031`, `IDR-SRV-034`, and `IDR-SRV-035`?

#### Command, Control Stream, and Feasibility Implications

- How should command parameters be represented with SWE Common?
- How should control stream input structures be defined?
- How should feasibility request and response structures relate to command parameters and constraints?
- How should parameter constraints, allowed values, ranges, units, defaults, nil values, and validation errors be represented?
- Which command/control component definitions are operationally sensitive?
- Which findings should be handed to `IDR-SRV-036`, `IDR-SRV-037`, and `IDR-SRV-038`?

#### Import, Generation, Transformation, and Normalization

- Should Glaux Server ingest externally authored SWE Common structures?
- Should Glaux Server generate SWE Common definitions from internal datastream, observation, status, or command metadata?
- Should Glaux Server transform SWE Common into JSON Schema/OpenAPI schemas or internal validation rules?
- How should partial, invalid, legacy, or profile-divergent SWE Common be handled?
- What warnings, normalization, preservation, and provenance behavior are needed?
- How should transformations avoid lossy behavior?

#### Validation and Conformance Implications

- What SWE Common validation rules are needed for Glaux Server?
- What should be validated structurally, semantically, profile-specifically, and operationally?
- What validation should happen on import, registration, update, observation ingestion, command submission, feasibility requests, response generation, and conformance testing?
- What errors or warnings should be produced when SWE Common structures are incomplete or inconsistent with CSAPI resources?

#### Security, Policy, and Releasability Implications

- What sensitive information may appear in SWE Common component definitions?
- How could command parameters, controlled properties, allowed values, ranges, system capabilities, or status component definitions reveal sensitive operational information?
- How should policy and releasability affect SWE Common source definitions versus derived CSAPI views?
- How should Glaux Server avoid leaking restricted capabilities, command/control affordances, or sensitive dynamic values?

#### Implementation and Interoperability Lessons

- What SWE Common lessons were identified in OSH, Connected Systems Go, pygeoapi, and SECD?
- What smoke-test or interoperability findings indicate SWE Common parsing, validation, encoding, unit, semantic, command-parameter, or schema issues?
- What OS4CSAPI discussion lessons affect SWE Common strategy?
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

- `IDR-SRV-001` through `IDR-SRV-021` research reports, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`

### Controlling Standards Sources

- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api
- OGC schemas: https://schemas.opengis.net/
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- W3C / OGC Semantic Sensor Network Ontology: https://www.w3.org/TR/vocab-ssn/

### AEP-4789 and STANAG 4789 Sources

Use source material available to the Glaux project team and record exact title, volume, version, date, status, storage location, and classification/handling constraints:

- STANAG 4789
- AEP-4789 Volume I
- AEP-4789 Volume II
- Any approved or draft AEP-4789 schemas, profiles, implementation guidance, or standards-package annexes relevant to SWE Common, observations, datastreams, status, dynamic data, command/control, encodings, and NATO JISR data components

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
- SensorML Representation Strategy findings from `IDR-SRV-021`
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

### Phase 1: Source Collection and SWE Common Extraction Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and extraction framework for SWE Common strategy.

**Tasks:**

1. Gather SWE Common, SensorML, CSAPI, AEP-4789, prior IDR reports, implementation studies, smoke-test findings, interoperability findings, and discussion lessons.
2. Extract SWE Common constructs, examples, schema references, profiles, mappings, encoding definitions, and implementation observations.
3. Define inventory fields:
   - SWE Common concept,
   - source authority,
   - related CSAPI resource family,
   - component type,
   - internal/external exposure,
   - representation pattern,
   - validation need,
   - encoding need,
   - unit/semantic dependency,
   - storage/index need,
   - security/policy implication,
   - test implication,
   - downstream handoff.
4. Define classification values: normative, inherited, standards-package-specific, representation-specific, implementation-support, implementation-specific, optional, deferred, and unresolved.
5. Establish evidence hierarchy for standards, AEP material, prior reports, implementation observations, and community discussion.

**Expected Output:** Source inventory and SWE Common extraction framework.

### Phase 2: SWE Common Standards and CSAPI Mapping Analysis (3-4 hours)

**Objective:** Extract SWE Common concepts and map them to CSAPI and Glaux Server resources.

**Tasks:**

1. Review SWE Common 3.0 constructs relevant to quantities, counts, booleans, categories, text, time, records, arrays, vectors, matrices, choices, ranges, nil values, constraints, units, semantics, and encodings.
2. Map SWE Common constructs to CSAPI Part 1 and Part 2 resources.
3. Identify where SWE Common is referenced, required, optional, or implied by CSAPI, SensorML, and AEP-4789 context.
4. Identify direct, indirect, derived, partial, ambiguous, and unresolved mappings.
5. Build a SWE Common-to-Glaux resource mapping table.

**Expected Output:** SWE Common concept and CSAPI mapping matrix.

### Phase 3: Data Component, Encoding, and Representation Boundary Analysis (2.5-3.5 hours)

**Objective:** Determine how SWE Common components should be represented, validated, transformed, and exposed.

**Tasks:**

1. Distinguish SWE Common component definitions, CSAPI representations, internal domain fields, validation artifacts, OpenAPI/schema artifacts, and stored values.
2. Identify supported, partial, deferred, and out-of-scope component types.
3. Identify encoding patterns for JSON, arrays, records, streaming, and future binary representations.
4. Identify component definitions that should be normalized, indexed, preserved, generated, or treated as opaque.
5. Identify schema/encoding validation implications for `IDR-SRV-023`.

**Expected Output:** Component/encoding/representation boundary guidance.

### Phase 4: Observation, Status, Command, and Semantic Dependency Analysis (2.5-3 hours)

**Objective:** Identify cross-topic dependencies created by SWE Common data components.

**Tasks:**

1. Analyze observation result structures, datastream schemas, latest values, and status values.
2. Analyze control stream input structures, command parameters, feasibility requests, parameter constraints, and command-status outputs.
3. Identify unit, observed-property, controlled-property, semantic definition, and SSN/SOSA dependencies.
4. Identify temporal, quality, nil-value, and uncertainty implications.
5. Map findings to `IDR-SRV-024`, `IDR-SRV-027`, `IDR-SRV-031`, `IDR-SRV-034`, `IDR-SRV-036`, and `IDR-SRV-037`.

**Expected Output:** Observation/status/command/semantic dependency matrix.

### Phase 5: Validation, Security, Fixture, and Interoperability Implication Analysis (2.5-3 hours)

**Objective:** Prepare SWE Common findings for downstream validation and implementation work.

**Tasks:**

1. Identify structural, semantic, profile, unit, encoding, observation-value, status-value, command-parameter, and feasibility-validation needs.
2. Identify security, policy, releasability, and source-trust concerns.
3. Identify fixtures, sample SWE Common structures, golden files, invalid examples, transformation tests, and command/status test cases needed.
4. Identify conformance, interoperability, generated-client, CSAPI Explorer, and external-tool compatibility tests.
5. Incorporate non-normative lessons from OSH, CS-GO, pygeoapi, SECD, OS4CSAPI smoke tests, interoperability findings, and discussions.

**Expected Output:** SWE Common validation/security/test implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert SWE Common research into a decision-usable data component strategy baseline.

**Tasks:**

1. Consolidate standards-derived SWE Common inventory, CSAPI mapping, component support strategy, encoding strategy, dependencies, and downstream implications.
2. Produce SWE Common recommendations by Glaux Server resource family and component type.
3. Resolve conflicts and ambiguities where possible.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] SWE Common concepts relevant to Glaux Server are identified with source anchors.
- [ ] SWE Common concepts are mapped to canonical CSAPI resource families.
- [ ] Data component definitions, observation values, command values, internal domain fields, validation artifacts, and persistence concerns are distinguished.
- [ ] Component-type support scope is identified.
- [ ] Encoding, nil-value, constraint, unit, observed-property, controlled-property, and semantic implications are documented.
- [ ] Observation, status, dynamic-data, command, feasibility, and SensorML dependencies are documented.
- [ ] Validation, security, conformance, fixture, and interoperability implications are documented.
- [ ] Implementation-study and community-lesson findings are incorporated as non-normative evidence.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** SWE Common Data Component Strategy Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-022-swe-common-data-component-strategy-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. SWE Common extraction methodology
5. SWE Common concept inventory
6. SWE Common-to-CSAPI / Glaux resource mapping
7. Data component type support strategy
8. Encoding and representation boundary findings
9. Observation, datastream, status, and dynamic-data findings
10. Command, control stream, and feasibility findings
11. Units, observed properties, controlled properties, and semantic dependency findings
12. Validation, profile, schema, encoding, and conformance implications
13. Security, policy, and releasability implications
14. Fixture, golden-file, and interoperability test implications
15. Downstream topic handoff matrix
16. Recommendations
17. Risks, constraints, and open questions
18. Validation against this plan's success criteria
19. References

The SWE Common mapping matrix should include, at minimum:

- SWE Common concept / component type
- Source standard / source anchor
- Authority classification
- Related CSAPI / Glaux resource family
- Representation pattern
- Internal normalization need
- Encoding implication
- Unit/semantic dependency
- Validation implication
- Persistence/query implication
- Security/policy implication
- Test implication
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` through `IDR-SRV-021` research reports should be complete or explicitly marked unavailable/deferred.
- Official SWE Common 3.0, SensorML 3.0, CSAPI Part 1 and Part 2, OGC API - Features, OGC schemas, OpenAPI artifacts, SSN/SOSA, and project-available AEP-4789 material must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-023: Schema and Encoding Validation Strategy`
- `IDR-SRV-024: Units, Observed Properties, and Semantic Binding Strategy`
- `IDR-SRV-027: Time-Series Observation Storage Strategy`
- `IDR-SRV-028: Metadata and Document Storage Strategy`
- `IDR-SRV-031: Server Write and Ingestion Model`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-035: Streaming and Event Publication Strategy`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-037: Feasibility and Asynchronous Tasking Strategy`
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

- This topic defines SWE Common data component strategy, not the whole Glaux Server domain model.
- SWE Common should be treated as a standards data component and encoding model that interacts with CSAPI resources, SensorML, validation, observations, status, and commands.
- Implementation-study findings are useful but must not override SWE Common and CSAPI semantics.
- Open question: Which SWE Common component types require full initial support?
- Open question: How much round-trip fidelity should be required for imported SWE Common structures?
- Open question: How should partial, invalid, legacy, or profile-divergent SWE Common structures be handled?
- Open question: Which SWE Common command parameter definitions are operationally sensitive and require policy filtering?
- Open question: Which SWE Common examples should become canonical fixtures?
- Risk: Treating SWE Common as opaque only could limit validation, command safety, and interoperability.
- Risk: Over-normalizing SWE Common could create brittle mappings and loss of source fidelity.
- Risk: Exposing imported SWE Common structures without validation or policy checks could leak sensitive command/control or capability information.
- Risk: Treating one implementation's SWE Common profile as canonical could distort Glaux Server design.

---

## References

- Glaux Server Overall IDR Research Plan: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Creation Prompt: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-creation-prompt.md
- Research Plan Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Overall Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/overall-research-report-template.md
- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api
- OGC schemas: https://schemas.opengis.net/
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- W3C / OGC Semantic Sensor Network Ontology: https://www.w3.org/TR/vocab-ssn/
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

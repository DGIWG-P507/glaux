# Section 024: Units, Observed Properties, and Semantic Binding Strategy - Research Plan

**Status:** Planned  
**Last Updated:** June 10, 2026  
**Estimated Research Time:** 12-16 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-024-units-observed-properties-and-semantic-binding-strategy-report.md`

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

This topic research must define the Glaux Server planning baseline for **units, observed properties, controlled properties, semantic identifiers, vocabulary binding, and semantic interoperability strategy** across CSAPI resources, SensorML descriptions, SWE Common data components, datastreams, observations, status values, commands, feasibility checks, metadata, validation, query, and conformance testing.

The research must answer:

- What unit, observed-property, controlled-property, semantic-definition, vocabulary, and ontology-binding responsibilities must Glaux Server support?
- How should Glaux Server represent, validate, normalize, preserve, and expose units of measure, observed properties, controlled properties, semantic URIs, labels, definitions, vocabularies, and concept schemes?
- How should semantic binding interact with SensorML, SWE Common, CSAPI Part 1, CSAPI Part 2, SSN/SOSA, OGC schemas, AEP-4789 profile expectations, and NATO JISR sensor interoperability needs?
- How should Glaux Server handle missing, ambiguous, conflicting, non-standard, locally defined, externally supplied, or profile-specific units and properties?
- Which semantic bindings are externally visible API behavior, which are internal validation/indexing aids, and which are preserved source metadata?
- How should semantic consistency support discovery, query, validation, transformation, data normalization, command safety, conformance tests, and external-client interoperability?
- What downstream implications follow for persistence, metadata/document storage, ingestion normalization, datastream/observation semantics, command validation, security, fixtures, conformance harnesses, and interoperability testing?

The output must be a units, observed properties, and semantic binding strategy baseline with source anchors, vocabulary and URI guidance, resource-family mappings, validation implications, normalization recommendations, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows:

- `IDR-SRV-021: SensorML Representation Strategy`
- `IDR-SRV-022: SWE Common Data Component Strategy`
- `IDR-SRV-023: Schema and Encoding Validation Strategy`

SensorML and SWE Common establish the representation structures that carry units, observed properties, controlled properties, semantic definitions, and identifiers. The validation topic establishes where structural and semantic validation occur. This topic defines the semantic binding strategy needed to support interoperable interpretation of sensor observations, status values, command parameters, and metadata. It must precede detailed metadata/document storage, dynamic-data ingestion, observation semantics, command validation, conformance harness planning, fixture strategy, and interoperability testing.

### Critical Constraint(s)

- Treat OGC API - Connected Systems Part 1 and Part 2, SensorML 3.0, SWE Common 3.0, SSN/SOSA, OGC API - Features, AEP-4789 Volume II adoption context, and earlier IDR findings as controlling.
- Do not invent a mandatory vocabulary regime that conflicts with CSAPI, SensorML, SWE Common, or AEP-4789 expectations.
- Do not assume all units or properties can be normalized losslessly. Preserve source metadata and distinguish normalized views from source values.
- Do not finalize schema/encoding validation here. Build on `IDR-SRV-023` and identify additional semantic-validation needs.
- Do not design database schema here. Identify persistence, indexing, vocabulary-cache, and metadata-storage implications and hand them to Category E topics.
- Do not finalize dynamic-data ingestion or command lifecycle behavior here. Identify semantic dependencies and hand them to Category F topics.
- Do not finalize policy or releasability behavior here. Identify semantic and vocabulary disclosure risks and hand them to Category G.
- Distinguish observed properties, controlled properties, feature properties, system characteristics, status properties, command parameters, units, and semantic definitions.
- Keep the research bounded to Glaux Server behavior and server-side contracts.

---

## 2. Research Questions

### Core Questions

1. What unit, observed-property, controlled-property, and semantic binding concepts must Glaux Server support?
2. Which semantic concepts are normative, inherited, profile-specific, implementation-support, optional, or unresolved?
3. How should semantic URIs, labels, definitions, vocabularies, units, observed properties, and controlled properties be represented, validated, preserved, and normalized?
4. How should semantic binding support discovery, query, ingestion, observation interpretation, status interpretation, command validation, and interoperability?
5. What downstream implications follow for persistence, metadata storage, dynamic data, command validation, security, fixtures, conformance, and interoperability?

### Detailed Questions

#### Semantic Source and Authority Baseline

- What semantic binding concepts are defined or implied by CSAPI Part 1?
- What semantic binding concepts are defined or implied by CSAPI Part 2?
- What unit and property concepts are defined or implied by SensorML?
- What unit, observed-property, controlled-property, and semantic-definition concepts are defined or implied by SWE Common?
- What concepts are provided or constrained by SSN/SOSA?
- What concepts are inherited from OGC API - Features or OGC API conventions?
- What semantic requirements or expectations are implied by STANAG 4789 / AEP-4789 Volume II server responsibilities?
- Which semantic concepts appear in implementation studies or community lessons but are not clearly standards-defined?

#### Units of Measure Strategy

- What unit representations must Glaux Server support?
- Which unit identifiers are expected: UCUM, QUDT, OGC URI, local URI, free-text labels, SensorML/SWE Common codes, or profile-specific identifiers?
- How should Glaux Server handle unit aliases, symbols, labels, definitions, conversion factors, dimensions, and incompatible units?
- Should Glaux Server perform unit conversion, unit normalization, unit validation, or unit preservation only?
- What unit behavior is required for observations, status values, command parameters, feasibility checks, capabilities, characteristics, and metadata?
- How should missing, ambiguous, conflicting, or invalid units be handled?

#### Observed Property Strategy

- How should observed properties be represented and referenced?
- How should observed-property definitions be linked to datastreams, observations, SensorML outputs, SWE Common result structures, metadata, and query/filter behavior?
- Which observed-property identifiers or vocabularies are expected?
- How should Glaux Server handle multiple observed properties in one datastream or result structure?
- How should observed properties relate to features of interest, sampling features, units, result types, and status values?
- How should missing, ambiguous, local, conflicting, or profile-specific observed properties be handled?

#### Controlled Property and Command Parameter Strategy

- How should controlled properties be represented and referenced?
- How should controlled-property definitions be linked to control streams, command parameters, feasibility requests, SensorML inputs, SWE Common input structures, command validation, command authorization, and audit behavior?
- Which controlled-property identifiers or vocabularies are expected?
- How should Glaux Server distinguish command parameters, controlled properties, system settings, tasking inputs, feasibility constraints, and command status fields?
- How should missing, ambiguous, local, conflicting, or sensitive controlled properties be handled?
- Which findings should be handed to command lifecycle, feasibility validation, command authorization, and security topics?

#### Semantic URI, Vocabulary, and Concept Scheme Strategy

- What kinds of semantic identifiers must Glaux Server support?
- Should Glaux Server support dereferenceable HTTP URIs, URNs, local identifiers, ontology terms, vocabulary concepts, profile-specific terms, or mappings among these?
- How should Glaux Server preserve labels, definitions, broader/narrower relationships, synonyms, alternate IDs, mappings, source vocabularies, and provenance?
- Which vocabularies should be treated as authoritative, recommended, optional, local, or profile-specific?
- How should vocabulary versioning, offline availability, cache validity, and DDIL behavior be handled?
- Which semantic resources should be discoverable or exposed to clients?

#### SensorML and SWE Common Integration

- How do SensorML identifiers, classifiers, capabilities, characteristics, inputs, outputs, modes, and positions depend on semantic binding?
- How do SWE Common units, component definitions, data records, categories, quantities, constraints, nil values, and encodings depend on semantic binding?
- What semantic consistency checks are needed between SensorML and SWE Common structures?
- Which semantic bindings should be normalized into Glaux Server internal fields?
- Which semantic bindings should remain as source-preserved representation details?
- Which findings should be handed to validation, metadata storage, and fixture topics?

#### Discovery, Query, and Interoperability Implications

- How should semantic bindings support discovery by observed property, controlled property, unit, capability, system type, procedure type, phenomenon, sensor modality, or status type?
- Which semantic fields should be queryable?
- Which semantic fields should be indexed?
- How should semantic matching handle aliases, mappings, vocabulary versions, local terms, or equivalent concepts?
- Which query behavior is required or expected by CSAPI, AEP-4789, CSAPI Explorer, OS4CSAPI clients, or other external clients?
- Which semantic interoperability risks should Glaux Server avoid?

#### Validation and Normalization Implications

- What semantic validation should occur at registration, import, update, ingestion, command submission, feasibility evaluation, response generation, CI, and conformance testing?
- What should be validated structurally versus semantically?
- Which semantic inconsistencies should be warnings rather than errors?
- How should Glaux Server handle unknown terms, unknown vocabularies, non-dereferenceable URIs, invalid units, missing definitions, or conflicting mappings?
- How should validation avoid excessive online dependencies or DDIL fragility?
- Which findings should be handed to `IDR-SRV-023`, `IDR-SRV-050`, and `IDR-SRV-051`?

#### Persistence, Caching, and Metadata Storage Implications

- What semantic information must be persisted?
- What semantic information should be indexed?
- What vocabulary caches, lookup tables, mappings, profiles, or term registries are needed?
- How should vocabulary source provenance and versioning be preserved?
- How should semantic bindings be stored for imported SensorML/SWE definitions versus normalized views?
- Which findings should be handed to `IDR-SRV-025`, `IDR-SRV-028`, and `IDR-SRV-047`?

#### Security, Policy, and Releasability Implications

- Which semantic terms, observed properties, controlled properties, capabilities, command parameters, or vocabulary mappings may reveal sensitive operational information?
- How should policy and releasability affect semantic metadata exposure?
- How should semantic filtering affect discovery, queries, links, SensorML, SWE Common, command definitions, and status values?
- How should Glaux Server avoid leaking sensitive capabilities or command/control affordances through semantic identifiers?
- Which findings should be handed to Category G and security-test topics?

#### Implementation and Interoperability Lessons

- What unit, property, and semantic binding lessons were identified in OSH, Connected Systems Go, pygeoapi, and SECD?
- What smoke-test or interoperability findings indicate unit, observed-property, controlled-property, semantic URI, vocabulary, or schema issues?
- What OS4CSAPI discussion lessons affect semantic binding strategy?
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

- `IDR-SRV-001` through `IDR-SRV-023` research reports, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`

### Controlling Standards Sources

- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- W3C / OGC Semantic Sensor Network Ontology: https://www.w3.org/TR/vocab-ssn/
- OGC schemas: https://schemas.opengis.net/
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html

### Semantic Vocabulary and Unit Sources

Use these as candidate sources to evaluate, not as assumed mandatory project choices:

- UCUM: https://ucum.org/
- QUDT: https://qudt.org/
- OGC Definitions Server: https://defs.opengis.net/
- OGC Naming Authority resources: https://www.ogc.org/ogcna/
- W3C SKOS Reference: https://www.w3.org/TR/skos-reference/
- W3C RDF Concepts and Abstract Syntax: https://www.w3.org/TR/rdf11-concepts/
- W3C OWL 2 Overview: https://www.w3.org/TR/owl2-overview/

### AEP-4789 and STANAG 4789 Sources

Use source material available to the Glaux project team and record exact title, volume, version, date, status, storage location, and classification/handling constraints:

- STANAG 4789
- AEP-4789 Volume I
- AEP-4789 Volume II
- Any approved or draft AEP-4789 schemas, profiles, implementation guidance, vocabularies, controlled term lists, semantic guidance, or standards-package annexes relevant to observed properties, controlled properties, units, semantic identifiers, and NATO JISR sensor interoperability

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
- SensorML Representation Strategy findings from `IDR-SRV-021`
- SWE Common Data Component Strategy findings from `IDR-SRV-022`
- Schema and Encoding Validation Strategy findings from `IDR-SRV-023`
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

### Phase 1: Source Collection and Semantic Binding Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and extraction framework for units, properties, and semantic binding.

**Tasks:**

1. Gather CSAPI, SensorML, SWE Common, SSN/SOSA, AEP-4789, vocabulary, prior IDR, implementation-study, smoke-test, interoperability, and discussion sources.
2. Extract unit, observed-property, controlled-property, semantic-definition, vocabulary, ontology, mapping, and validation concepts.
3. Define inventory fields:
   - concept type,
   - source authority,
   - related resource family,
   - URI/identifier pattern,
   - vocabulary/source,
   - label/definition behavior,
   - normalization need,
   - validation need,
   - query/index need,
   - security/policy implication,
   - test implication,
   - downstream handoff.
4. Define classification values: normative, inherited, profile-specific, vocabulary-specific, implementation-support, implementation-specific, optional, local, deferred, and unresolved.
5. Establish evidence hierarchy for standards, AEP material, official vocabularies, prior reports, implementation observations, and community discussion.

**Expected Output:** Source inventory and semantic binding extraction framework.

### Phase 2: Standards and Vocabulary Inventory (2.5-3.5 hours)

**Objective:** Identify semantic binding sources and their authority by resource family.

**Tasks:**

1. Extract CSAPI Part 1 and Part 2 semantic binding references and requirements.
2. Extract SensorML unit, identifier, classifier, capability, characteristic, input, and output semantic expectations.
3. Extract SWE Common unit, observed-property, controlled-property, component-definition, and semantic expectations.
4. Extract SSN/SOSA concepts relevant to systems, observations, properties, procedures, features of interest, and deployments.
5. Review candidate vocabulary and unit sources such as UCUM, QUDT, OGC Definitions Server, OGC Naming Authority, SKOS, RDF, and OWL.
6. Extract AEP-4789 profile and vocabulary implications from project-available material.
7. Identify conflicts, gaps, unresolved vocabulary choices, and versioning concerns.

**Expected Output:** Semantic source and vocabulary authority matrix.

### Phase 3: Resource-Family Semantic Mapping Analysis (3-4 hours)

**Objective:** Define semantic binding behavior by Glaux Server resource family.

**Tasks:**

1. Use `IDR-SRV-015` through `IDR-SRV-023` findings as the organizing baseline.
2. Map units, observed properties, controlled properties, semantic identifiers, vocabularies, and definitions to each resource family.
3. Distinguish source-preserved metadata, normalized semantic fields, queryable/indexed fields, validation fields, and API response fields.
4. Identify direct, derived, partial, ambiguous, profile-specific, and unresolved mappings.
5. Identify semantic consistency rules among SensorML, SWE Common, datastreams, observations, status values, commands, and feasibility resources.

**Expected Output:** Resource-family semantic binding matrix.

### Phase 4: Validation, Normalization, Query, and Offline/DDIL Analysis (2.5-3 hours)

**Objective:** Define how semantic bindings should be validated, normalized, queried, and maintained.

**Tasks:**

1. Identify semantic validation rules for units, observed properties, controlled properties, URIs, labels, definitions, vocabulary membership, and cross-resource consistency.
2. Identify normalization and preservation strategies for local terms, aliases, mappings, and external vocabularies.
3. Identify query and discovery behavior involving semantic fields and aliases.
4. Identify vocabulary caching, offline lookup, DDIL operation, vocabulary versioning, and source provenance implications.
5. Identify handoffs to persistence, metadata storage, configuration, ingestion, and DDIL topics.

**Expected Output:** Semantic validation/normalization/query implication matrix.

### Phase 5: Security, Fixtures, Conformance, and Interoperability Implication Analysis (2.5-3 hours)

**Objective:** Prepare semantic binding findings for downstream verification and implementation work.

**Tasks:**

1. Identify semantic disclosure risks involving capabilities, controlled properties, command parameters, sensitive observed properties, source provenance, and vocabulary mappings.
2. Identify fixtures, sample vocabularies, unit examples, valid/invalid semantic bindings, local-term examples, alias/mapping examples, command-parameter examples, and observation examples needed.
3. Identify conformance, generated-client, CSAPI Explorer, external-client, semantic-query, validation, and interoperability tests.
4. Incorporate non-normative lessons from OSH, CS-GO, pygeoapi, SECD, OS4CSAPI smoke tests, interoperability findings, and discussions.
5. Map findings to Category E, F, G, and I topics.

**Expected Output:** Semantic security/test/interoperability implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert semantic binding research into a decision-usable baseline.

**Tasks:**

1. Consolidate semantic source inventory, vocabulary authority findings, resource-family mapping, validation/normalization/query guidance, and downstream implications.
2. Produce semantic binding recommendations by resource family and concept type.
3. Resolve conflicts and ambiguities where possible.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Unit, observed-property, controlled-property, semantic identifier, vocabulary, and ontology-binding concepts are identified with source anchors.
- [ ] Semantic concepts are mapped to canonical CSAPI resource families.
- [ ] Source-preserved metadata, normalized semantic fields, validation fields, query/index fields, and API response fields are distinguished.
- [ ] Candidate unit and vocabulary sources are evaluated without prematurely declaring them mandatory unless standards/profile evidence supports it.
- [ ] Semantic validation, normalization, alias/mapping, vocabulary versioning, offline/DDIL, and query implications are documented.
- [ ] Security, policy, conformance, fixture, and interoperability implications are documented.
- [ ] Implementation-study and community-lesson findings are incorporated as non-normative evidence.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Units, Observed Properties, and Semantic Binding Strategy Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-024-units-observed-properties-and-semantic-binding-strategy-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Semantic binding extraction methodology
5. Standards and vocabulary source inventory
6. Semantic concept taxonomy
7. Unit strategy findings
8. Observed-property strategy findings
9. Controlled-property and command-parameter semantic findings
10. Resource-family semantic mapping
11. SensorML and SWE Common semantic consistency findings
12. Validation, normalization, vocabulary-cache, query, and DDIL findings
13. Persistence, metadata storage, and configuration implications
14. Security, policy, and releasability implications
15. Fixture, golden-file, conformance, and interoperability test implications
16. Downstream topic handoff matrix
17. Recommendations
18. Risks, constraints, and open questions
19. Validation against this plan's success criteria
20. References

The semantic binding matrix should include, at minimum:

- Concept type
- Unit/property/semantic identifier
- Source standard / vocabulary / source anchor
- Authority classification
- Related CSAPI / Glaux resource family
- Representation pattern
- Normalization need
- Source-preservation need
- Query/index need
- Validation implication
- Vocabulary/cache implication
- Security/policy implication
- Test implication
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` through `IDR-SRV-023` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, SensorML 3.0, SWE Common 3.0, OGC API - Features, SSN/SOSA, OGC schemas, OpenAPI artifacts, relevant semantic/vocabulary sources, and project-available AEP-4789 material must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-027: Time-Series Observation Storage Strategy`
- `IDR-SRV-028: Metadata and Document Storage Strategy`
- `IDR-SRV-047: Configuration, Secrets, and Environment Strategy`
- `IDR-SRV-031: Server Write and Ingestion Model`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-037: Feasibility and Asynchronous Tasking Strategy`
- `IDR-SRV-038: Command Authorization, Safety, and Audit Strategy`
- `IDR-SRV-039: Authentication, Authorization, and API Security Threat Model`
- `IDR-SRV-040: Policy, Releasability, and Cross-Boundary Access Constraints`
- `IDR-SRV-042: DDIL-Informed Server Semantics`
- `IDR-SRV-043: Server Synchronization and Conflict Handling Boundary`
- `IDR-SRV-050: Conformance Harness Strategy`
- `IDR-SRV-051: Requirement-to-Test Traceability Strategy`
- `IDR-SRV-053: Test Data, Fixtures, Golden Files, and Scenario Corpus Strategy`
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

- This topic defines semantic binding strategy, not the full vocabulary registry, database schema, or final ontology profile.
- Semantic binding should improve interoperability without erasing source metadata or forcing premature vocabulary choices.
- Implementation-study findings are useful but must not override standards-derived semantics.
- Open question: Which vocabulary sources should be required, recommended, optional, or merely supported?
- Open question: Should Glaux Server perform unit conversion or only validation and preservation?
- Open question: How should local and mission-specific property terms be represented without breaking interoperability?
- Open question: Which semantic identifiers are operationally sensitive and require policy filtering?
- Open question: Which semantic examples should become canonical fixtures?
- Risk: Weak semantic binding could reduce discovery, validation, and cross-client interoperability.
- Risk: Overly rigid vocabulary enforcement could reject useful operational data and prevent local adaptation.
- Risk: Online vocabulary dependencies could fail during DDIL or disconnected operation.
- Risk: Semantic metadata may reveal sensitive capabilities, observed phenomena, controlled properties, or command affordances.

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
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- W3C / OGC Semantic Sensor Network Ontology: https://www.w3.org/TR/vocab-ssn/
- OGC schemas: https://schemas.opengis.net/
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- UCUM: https://ucum.org/
- QUDT: https://qudt.org/
- OGC Definitions Server: https://defs.opengis.net/
- OGC Naming Authority resources: https://www.ogc.org/ogcna/
- W3C SKOS Reference: https://www.w3.org/TR/skos-reference/
- W3C RDF Concepts and Abstract Syntax: https://www.w3.org/TR/rdf11-concepts/
- W3C OWL 2 Overview: https://www.w3.org/TR/owl2-overview/
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

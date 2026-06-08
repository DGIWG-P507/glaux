# Section 015: Canonical Glaux Server Resource Model - Research Plan

**Status:** Planned  
**Last Updated:** June 8, 2026  
**Estimated Research Time:** 14-18 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-015-canonical-glaux-server-resource-model-report.md`

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

This topic research must define the **canonical Glaux Server resource model**: the authoritative server-side model of resources, entities, relationships, collections, resource boundaries, lifecycle boundaries, and representation responsibilities needed to implement STANAG 4789 / AEP-4789 Volume II through OGC API - Connected Systems Parts 1 and 2, SensorML, and SWE Common.

The research must answer:

- What canonical resource families must Glaux Server expose and manage?
- What internal entities, external API resources, collections, subresources, relationships, and lifecycle boundaries should be distinguished?
- Which resource concepts are inherited from OGC API - Features, defined by CSAPI Part 1, defined by CSAPI Part 2, represented through SensorML, represented through SWE Common, or introduced as Glaux Server implementation concepts?
- How should Glaux Server model systems, procedures, deployments, sampling features, properties, datastreams, observations, status, events, control streams, commands, feasibility, command status, and related dynamic data?
- What resource-model implications follow from implementation-study findings, client smoke-test findings, SECD interoperability findings, and OS4CSAPI discussion lessons?
- What handoffs are required for identifiers, relationships, temporal modeling, status/event modeling, SensorML, SWE Common, persistence, validation, testing, and interoperability?

The output must be a canonical resource-model baseline with source anchors, concept definitions, resource/entity boundaries, relationship and lifecycle implications, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows the standards baseline and implementation-observation block:

- `IDR-SRV-001` through `IDR-SRV-005`: STANAG 4789 / AEP-4789 and standards-package baseline
- `IDR-SRV-006` through `IDR-SRV-014`: CSAPI, API behavior, conformance, query, representation, error, and OpenAPI baseline
- `IDR-SRV-014A` through `IDR-SRV-014G`: implementation studies, smoke-test findings, interoperability findings, and community lessons

Those topics establish what must be implemented, how the API behaves, and what practical implementation lessons should be considered. This topic starts Category C: Server Resource and Domain Model. It synthesizes prior findings into the canonical Glaux Server resource model that later topics will refine into identifier strategy, relationships, temporal semantics, registration/update behavior, status/events, persistence, validation, and tests.

### Critical Constraint(s)

- Treat OGC API - Connected Systems Part 1 and Part 2, SensorML, SWE Common, OGC API - Features, AEP-4789 Volume II adoption context, and earlier IDR findings as controlling.
- Do not invent a resource model that conflicts with CSAPI resource semantics, OGC API - Features collection/item behavior, SensorML system/procedure semantics, or SWE Common data component semantics.
- Do not design the database schema here. Identify persistence implications and hand them to Category E topics.
- Do not finalize URI/identifier rules here. Identify identifier needs and hand them to `IDR-SRV-016`.
- Do not finalize relationship/link behavior here. Identify relationship needs and hand them to `IDR-SRV-017`.
- Do not finalize temporal/freshness rules here. Identify temporal needs and hand them to `IDR-SRV-018`.
- Do not finalize registration/update APIs here. Identify lifecycle needs and hand them to `IDR-SRV-019`.
- Do not finalize status/event semantics here. Identify status/event needs and hand them to `IDR-SRV-020`.
- Do not collapse API resources, domain entities, persistence records, schemas, and serialized representations into one concept; distinguish them explicitly.
- Keep the research bounded to Glaux Server behavior and server-side contracts.

---

## 2. Research Questions

### Core Questions

1. What is the canonical set of Glaux Server resource families and internal domain entities?
2. Which resource families are normative CSAPI resources, inherited OGC API resources, SensorML/SWE representations, implementation-support entities, or future/extension concepts?
3. What are the boundaries between resources, subresources, collections, relationships, observations, events, commands, and status objects?
4. What lifecycle responsibilities does Glaux Server have for each resource family?
5. What downstream implications follow for identifiers, links, temporal modeling, persistence, validation, conformance, fixtures, and interoperability?

### Detailed Questions

#### Standards and Concept Sources

- What resource concepts are defined by CSAPI Part 1?
- What resource concepts are defined by CSAPI Part 2?
- What resource concepts are inherited from OGC API - Features?
- What resource concepts are represented by SensorML?
- What resource concepts are represented by SWE Common?
- What concepts come from AEP-4789 Volume II adoption context or NATO JISR usage?
- What concepts are implementation-support constructs rather than externally visible API resources?
- What concepts appear in existing implementations or community discussions that are not clearly defined in standards?

#### Canonical Resource Families

- What canonical model should Glaux Server use for:
  - landing page,
  - API definition,
  - conformance declaration,
  - collections,
  - systems,
  - procedures,
  - deployments,
  - sampling features,
  - properties,
  - datastreams,
  - observations,
  - status and dynamic properties,
  - system events,
  - control streams,
  - commands,
  - command status,
  - feasibility resources,
  - links and relationship resources,
  - schema/profile/documentation resources?
- Which resource families are mandatory, optional, conditional, future-facing, or implementation-specific?
- Which resources are collections, collection items, nested resources, related resources, or stateful workflow resources?
- Which resources represent relatively stable descriptions versus dynamic state or historical event/data records?

#### API Resource vs Domain Entity Boundaries

- What is the difference between a CSAPI API resource and a Glaux Server internal domain entity?
- Should systems, procedures, deployments, datastreams, control streams, and commands be modeled as distinct entities internally?
- Which API resources may be views over multiple internal entities?
- Which internal entities should not be exposed directly?
- What resource boundaries are needed to support authorization, policy, audit, validation, lifecycle, and interoperability?
- How should the model handle implementation-source resources such as publisher feeds, simulator resources, external systems, cached observations, and derived status objects?

#### Lifecycle Boundaries

- Which resource families are registered, updated, archived, deleted, derived, observed, streamed, commanded, or generated?
- Which resources are authoritative records versus derived views?
- Which resources have lifecycle states, such as active, inactive, retired, stale, unavailable, pending, accepted, rejected, completed, failed, or canceled?
- Which lifecycle questions must be handed to `IDR-SRV-019`, `IDR-SRV-020`, `IDR-SRV-036`, `IDR-SRV-037`, and `IDR-SRV-038`?
- How should the canonical model preserve full-scope capability without prematurely reducing scope?

#### Relationship and Linkage Implications

- What relationships exist among systems, procedures, deployments, sampling features, properties, datastreams, observations, commands, control streams, features of interest, events, and status resources?
- Which relationships are direct, derived, temporal, many-to-many, hierarchical, or contextual?
- Which relationships are represented as links, embedded objects, identifiers, query filters, or separate resources?
- Which relationship questions must be handed to `IDR-SRV-017`?
- Which relationship findings affect graph/query/persistence topics?

#### Temporal, Status, and Event Implications

- Which resources have valid time, phenomenon time, result time, event time, ingestion time, publication time, update time, or expiration/freshness time?
- Which resources represent current state versus historical state?
- Which resources require status, availability, health, freshness, or last-known-state semantics?
- Which resources produce system events?
- Which temporal/status/event questions must be handed to `IDR-SRV-018` and `IDR-SRV-020`?

#### SensorML and SWE Common Implications

- Which parts of the canonical resource model map to SensorML systems, processes, procedures, capabilities, characteristics, inputs, outputs, modes, positions, and contacts?
- Which parts of the canonical resource model map to SWE Common data components, data records, quantities, categories, vectors, arrays, units, observed properties, controlled properties, and command parameters?
- What boundaries should be maintained between CSAPI resources and SensorML/SWE representations?
- Which findings should be handed to `IDR-SRV-021`, `IDR-SRV-022`, and `IDR-SRV-024`?

#### Implementation-Study and Community-Lesson Implications

- What resource-model lessons were identified in OSH, Connected Systems Go, pygeoapi, and SECD?
- What smoke-test or interoperability findings indicate resource-shape, identifier, link, relationship, or schema problems?
- What OS4CSAPI discussion lessons affect the canonical model?
- Which implementation patterns should Glaux Server adopt, avoid, or investigate further?
- Which findings must be treated as implementation-specific rather than standards-derived?

#### Validation, Test, and Interoperability Implications

- What resource-model decisions need explicit schema validation?
- What resource-model decisions require conformance tests?
- What fixtures and golden files are needed to validate canonical resource shapes?
- What external-client interoperability cases are sensitive to resource-model choices?
- What traceability is needed from standards requirements to resource model to tests?
- Which findings should be handed to `IDR-SRV-050`, `IDR-SRV-051`, `IDR-SRV-053`, and `IDR-SRV-056`?

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

- `IDR-SRV-001` through `IDR-SRV-014G` research reports, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`

### Controlling Standards Sources

- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC schemas: https://schemas.opengis.net/
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- W3C / OGC Semantic Sensor Network Ontology: https://www.w3.org/TR/vocab-ssn/

### AEP-4789 and STANAG 4789 Sources

Use source material available to the Glaux project team and record exact title, volume, version, date, status, storage location, and classification/handling constraints:

- STANAG 4789
- AEP-4789 Volume I
- AEP-4789 Volume II
- Any approved or draft AEP-4789 schemas, profiles, implementation guidance, or standards-package annexes relevant to server resource responsibilities

### Implementation and Lessons-Learned Sources

Use the implementation-study outputs and source repositories to inform non-normative implementation lessons:

- OSH / OpenSensorHub sources and `IDR-SRV-014A`
- Connected Systems Go sources and `IDR-SRV-014B`
- pygeoapi sources and `IDR-SRV-014C`
- SECD sources and `IDR-SRV-014D`
- OS4CSAPI Client Smoke Test Findings and `IDR-SRV-014E`
- SECD Interoperability Findings and `IDR-SRV-014F`
- OS4CSAPI Discussions Lessons-Learned and `IDR-SRV-014G`
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

### Phase 1: Source Collection and Resource-Model Framework Setup (2-2.5 hours)

**Objective:** Establish the evidence base and define the extraction framework for canonical resource modeling.

**Tasks:**

1. Gather standards, schemas, OpenAPI artifacts, prior IDR reports, implementation studies, smoke-test findings, interoperability findings, and community lessons.
2. Identify every resource, concept, entity, object, schema, collection, endpoint, and relationship mentioned across sources.
3. Define resource-model inventory fields:
   - concept name,
   - source,
   - API resource / domain entity / representation / persistence concern / implementation support concept,
   - resource family,
   - lifecycle,
   - relationships,
   - temporal properties,
   - security/policy relevance,
   - validation and test implications.
4. Define classification values:
   - normative,
   - inherited,
   - standards-package-specific,
   - representation-specific,
   - implementation-support,
   - optional,
   - future candidate,
   - implementation-specific,
   - unresolved.
5. Establish evidence hierarchy for standards, AEP material, prior reports, implementation observations, and community discussion.

**Expected Output:** Source inventory and canonical resource-model extraction framework.

### Phase 2: Standards-Derived Resource Inventory (3-4 hours)

**Objective:** Extract the canonical resource candidates from CSAPI, OGC API - Features, SensorML, SWE Common, and AEP-4789 context.

**Tasks:**

1. Extract Part 1 resource families, collections, relationships, and behavior.
2. Extract Part 2 dynamic data, observation, status, event, command, control, and feasibility concepts.
3. Extract inherited OGC API - Features collection/item behavior.
4. Extract SensorML concepts that must be represented or referenced by Glaux Server resources.
5. Extract SWE Common concepts that affect data components, observation values, properties, command parameters, and validation.
6. Extract AEP-4789 server-responsibility concepts from project-available material.
7. Build a standards-derived resource inventory with source anchors.

**Expected Output:** Standards-derived resource inventory and authority classification.

### Phase 3: Canonical API Resource and Domain Entity Boundary Analysis (3-4 hours)

**Objective:** Define boundaries among API resources, domain entities, representations, and implementation-support objects.

**Tasks:**

1. Group resources and concepts into canonical resource families.
2. Distinguish external API resources from internal domain entities.
3. Identify which API resources are direct entities, views, derived records, state records, or workflow records.
4. Identify lifecycle boundaries for each resource family.
5. Identify which concepts belong in later topics rather than this canonical model.
6. Document unresolved boundary questions.

**Expected Output:** Canonical resource/entity boundary matrix.

### Phase 4: Relationship, Lifecycle, Temporal, and Status Implication Analysis (2.5-3.5 hours)

**Objective:** Identify model implications that must be carried forward to later Category C topics.

**Tasks:**

1. Identify relationships among systems, procedures, deployments, sampling features, properties, datastreams, observations, control streams, commands, status, events, and features of interest.
2. Identify lifecycle states and ownership/update responsibilities.
3. Identify temporal dimensions and freshness concepts.
4. Identify status, availability, event, and dynamic-property implications.
5. Identify handoffs to `IDR-SRV-016`, `IDR-SRV-017`, `IDR-SRV-018`, `IDR-SRV-019`, and `IDR-SRV-020`.

**Expected Output:** Relationship/lifecycle/temporal/status handoff matrix.

### Phase 5: Implementation Lessons, Persistence, Validation, and Test Implication Analysis (2.5-3 hours)

**Objective:** Incorporate implementation-study and interoperability lessons into the canonical model without making them normative.

**Tasks:**

1. Review `IDR-SRV-014A` through `IDR-SRV-014G` findings for resource-model lessons.
2. Identify implementation patterns to adopt, avoid, or further investigate.
3. Identify persistence and indexing implications without designing the database.
4. Identify schema validation and resource-shape implications.
5. Identify fixture, golden-file, conformance, and interoperability test implications.
6. Map findings to Category D, Category E, Category F, and Category I topics.

**Expected Output:** Implementation-lesson and downstream-implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert resource-model research into a decision-usable canonical model baseline.

**Tasks:**

1. Consolidate standards-derived resource inventory, boundary analysis, relationship/lifecycle implications, and implementation lessons.
2. Produce a canonical resource family list with definitions, authority classification, API exposure, internal entity implications, lifecycle notes, and downstream handoffs.
3. Resolve conflicts and ambiguities where possible.
4. Produce recommendations for downstream IDR topic use.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Canonical resource families are identified with source anchors.
- [ ] API resources, domain entities, serialized representations, persistence concerns, and implementation-support concepts are distinguished.
- [ ] CSAPI Part 1, CSAPI Part 2, OGC API - Features, SensorML, SWE Common, and AEP-4789 concepts are mapped to the resource model.
- [ ] Resource lifecycle boundaries are identified.
- [ ] Relationship, identifier, temporal, status, event, SensorML, SWE Common, persistence, validation, and testing implications are documented.
- [ ] Implementation-study and community-lesson findings are incorporated as non-normative evidence.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Canonical Glaux Server Resource Model Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-015-canonical-glaux-server-resource-model-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Resource-model extraction methodology
5. Standards-derived resource inventory
6. Canonical Glaux Server resource family definitions
7. API resource versus domain entity boundary analysis
8. Lifecycle boundary findings
9. Relationship and linkage implications
10. Temporal, freshness, status, and event implications
11. SensorML and SWE Common implications
12. Implementation-study and interoperability lesson findings
13. Persistence, validation, fixture, and test implications
14. Downstream topic handoff matrix
15. Recommendations
16. Risks, constraints, and open questions
17. Validation against this plan's success criteria
18. References

The canonical resource-model matrix should include, at minimum:

- Resource or concept name
- Resource family
- Source standard / source anchor
- Authority classification
- API resource / internal entity / representation / persistence concern / implementation-support concept
- External API exposure
- Primary relationships
- Lifecycle responsibility
- Temporal/status/freshness implication
- SensorML/SWE implication
- Persistence implication
- Validation implication
- Test implication
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` through `IDR-SRV-014G` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, OGC API - Features, SensorML, SWE Common, OGC schemas, OpenAPI artifacts, and project-available AEP-4789 material must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-016: Identifier, URI, and Resource Lifecycle Strategy`
- `IDR-SRV-017: Relationship and Linkage Model`
- `IDR-SRV-018: Temporal, Validity, and Freshness Model`
- `IDR-SRV-019: Registration, Update, and State Change Semantics`
- `IDR-SRV-020: Status, Availability, and System Event Model`
- `IDR-SRV-021: SensorML Representation Strategy`
- `IDR-SRV-022: SWE Common Data Component Strategy`
- `IDR-SRV-023: Schema and Encoding Validation Strategy`
- `IDR-SRV-024: Units, Observed Properties, and Semantic Binding Strategy`
- `IDR-SRV-025: Database and Persistence Architecture Options`
- `IDR-SRV-026: Geospatial Storage and Query Strategy`
- `IDR-SRV-027: Time-Series Observation Storage Strategy`
- `IDR-SRV-028: Metadata and Document Storage Strategy`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
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

- This topic defines the canonical Glaux Server resource model, not the database schema.
- This topic should preserve a full-scope implementation baseline while clearly identifying sequencing and downstream handoffs.
- Implementation-study findings are useful but must not override standards-derived resource semantics.
- Open question: Which concepts should be represented as first-class internal entities versus derived API views?
- Open question: Which dynamic-data and tasking concepts require first-class lifecycle state?
- Open question: Which SensorML and SWE Common constructs should remain representation details versus canonical domain-model concepts?
- Open question: Which resource-model questions require later prototype validation?
- Risk: Over-collapsing standards concepts into simplified entities could create future conformance or interoperability problems.
- Risk: Over-modeling every standards concept as a first-class server entity could create unnecessary complexity.
- Risk: Treating one implementation's model as canonical could distort Glaux Server design.
- Risk: Failing to distinguish resource model from persistence model could prematurely constrain architecture.

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
- OGC schemas: https://schemas.opengis.net/
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- W3C / OGC Semantic Sensor Network Ontology: https://www.w3.org/TR/vocab-ssn/
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

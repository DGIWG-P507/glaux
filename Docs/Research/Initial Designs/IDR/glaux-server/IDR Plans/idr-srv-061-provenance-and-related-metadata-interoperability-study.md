# Section 061: Provenance and Related Metadata Interoperability Study - Research Plan

**Topic ID:** IDR-SRV-061<br>
**Status:** Complete and accepted<br>
**Last Updated:** September 18, 2026<br>
**Estimated Research Time:** Not yet estimated; one focused research/report iteration is planned, subject to source availability.<br>
**Actual Research Time:** One AI-assisted research/report iteration, September 18, 2026; no human-hours estimate inferred<br>
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-061-provenance-and-related-metadata-interoperability-study-report.md`

---

## Usage Instructions

Follow the [Research Plan Template](../../../../../Governance/research-plan-template.md) for this document and the [Research Report Template](../../../../../Governance/research-report-template.md) for the later report, preserving each template's applicable section order. Use the pinned OS4CSAPI exemplars for question-led analysis, direct evidence, concrete examples and usable recommendations, not their client-specific implementation scope or effort estimates.

The project lead's first September 18, 2026 `proceed` authorized this plan and its minimal supplemental index registration, published in commit `2458771`. The subsequent `proceed` authorized the research/report iteration. The [report](../IDR%20Reports/idr-srv-061-provenance-and-related-metadata-interoperability-study-report.md), published in `3283efd`, was accepted by the next `proceed` on September 18, 2026, which also authorized its separate synthesis addendum. [Addendum D](../IDR%20Reports/final-idr-research-report.md#addendum-d-provenance-and-related-metadata-interoperability) is prepared for review; Goal/Guide discussion alongside Part 5 is next. No special approval phrase, implementation adoption, or Goal/Guide change follows automatically.

---

## 1. Research Objective

Determine how a standards-faithful Glaux Server can preserve and expose useful provenance and related metadata across connected-system workflows, what the existing CSAPI standards package already supports, and whether any demonstrated interoperability gaps justify additional planning. Examine origin, derivation, responsibility and stewardship together with their quality, grouping, exchange and disclosure touchpoints, without treating these as one interchangeable concept.

Start from CSAPI, its supporting standards and practical consumer questions. OMS and SOSA/SSN explain important conceptual relationships; PROV and other relevant work are sources to assess, not a preselected replacement model or exchange schema. Recommend existing-standard use, a specifically justified addition, external integration, or no change/deferral as the evidence warrants.

### Why This Topic Order

The original 67-topic IDR and supplements 058/059/060 are complete and accepted. IDR-SRV-019 already studied provenance, lineage, quality and trust; IDR-SRV-021/022 and 040/041 cover relevant representations, policy and audit. The [approved Goal v1.7](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md) includes provenance context, while the [draft Guide v0.2](../../../../../Plans/glaux-server/glaux-server-implementation-guide.md), particularly §4.10, preserves practical origin/transformation records without requiring a general-purpose provenance graph database.

The new question is how these capabilities support interoperable, consumer-usable provenance in real observation workflows, including relationships to quality and controlled disclosure. This is a targeted extension and re-evaluation of relevant findings, not a claim that provenance or NATO security labeling was never researched.

**Planning context:** The project lead supplied preliminary stakeholder concerns about tracing results, responsibility, uncertainty, grouping, large-volume exchange and handling restrictions. Treat their generalized substance as illustrative, unvalidated use cases within a broader assessment. Do not reproduce the supplied wording or identify the originating application or its architect in project documentation. Do not adopt its proposed schemas, metadata placement, terminology, identifiers, database relationships or marking granularity as requirements. Public OGC/NGA-related provenance work is a separate evidence lead, not proof that those stakeholder preferences are OGC or NATO obligations.

The agreed sequence remains:

1. Draft and push this plan with its overall-index entry.
2. On the next `proceed`, conduct the research and push one report for review.
3. After report acceptance, add its findings to the final synthesis through a separately authorized addendum, preserving the original synthesis and existing addenda.
4. Discuss any Goal/Guide implications, including the still-pending Part 5 discussion; make only subsequently agreed changes and resume Guide drafting pass 2.

### Critical Constraints

- Preserve the approved CSAPI/SensorML/SWE scope and selected experiments/filtering. Research acceptance does not adopt every recommendation or add a new conformance claim.
- Distinguish conceptual models, ontology vocabularies, concrete encodings, API contracts and deployment policy. Inspect CSAPI Part 1 Annex C and its actual normative references; conceptual alignment is not automatic full implementation of every OMS concept or newer SOSA/SSN edition.
- Keep data-production history separate from server ingestion/mutation history, provenance assertions separate from verified identity/integrity, and quality assessments separate from authorization or proof of truth.
- No presumed custom schema, report resource, provenance endpoint, RDF service, graph database, universal confidence score, uncertainty-fusion engine or mandatory field-by-field marking. Demonstrate a gap before proposing an addition.
- Examine IC-EDH/ISM and ADatP/STANAG 4774/4778 only for relevant applicability, representation, binding and disclosure questions. Do not import an entire security regime, invent equivalence between policies, authorize release, or undertake accreditation/cross-domain guard implementation.
- Do not install software, implement server features, deploy services, contact stakeholders or publish upstream issues. Optional small checks may use installed tools; no benchmark campaign, new framework or parallel requirements/decision-document system is a deliverable.

---

## 2. Research Questions

### Core Questions

1. **Q1 - Existing support and standards roles:** What provenance capabilities and reusable mappings exist in the adopted CSAPI package, its conceptual foundations, relevant W3C work and practical OGC/implementation evidence?
2. **Q2 - Origin, derivation and responsibility:** What facts and relationships let a consumer trace a result to its inputs, producing system, method/version and responsible parties, without confusing those roles or overstating evidence?
3. **Q3 - Quality and uncertainty:** How can distinct assessments of identity, property values and evidence quality be represented and interpreted without conflating likelihood, confidence, probability, precision and measurement uncertainty?
4. **Q4 - Grouping and scalable exchange:** How can individual and grouped observations retain interpretable provenance through storage, retrieval, publication, conversion and exchange without imposing a new report model or excessive repeated metadata?
5. **Q5 - Protection and disclosure boundaries:** Which label, binding, granularity and access-control considerations affect that exchange, and which belong to a deployment-specific policy or external integration rather than CSAPI core?
6. **Q6 - Recommendation and verification:** What, if anything, should change in Glaux planning, with what bounded implementation consequences, alternatives and independent tests?

### Detailed Questions

**Standards and prior evidence (Q1)**

- Establish the roles of OMS/O&M conceptual models, legacy XML bindings and services, SOSA/SSN, CSAPI, SensorML and SWE Common. Identify which concepts and relationships the adopted versions actually implement; do not turn historical lineage into an obligation to implement legacy services or all newer ontology modules.
- Inspect existing SOSA/SSN-PROV alignment before proposing mappings. Distinguish informative alignments from normative constraints, published editions from drafts, and semantic compatibility from a usable public exchange contract.
- Reuse the accepted reports in Section 4, then inspect only relevant OGC testbed outputs and peer implementations. Identify concrete resource/schema examples, handlers, tests and consumer behavior; a named standard, research demonstration or internal field alone does not prove interoperable CSAPI support. Record gaps in public evidence without claiming global absence.

**Origin, derivation, identity and responsibility (Q2)**

- Trace producing devices/software, process descriptions and actual process executions, model/method versions, calibration/configuration changes, raw source artifacts and antecedent observations. Distinguish persistent identity from the exact revision used, a sampling relationship from causal derivation, and event/result/receipt times. Assess incomplete, remote or unavailable history honestly.
- Distinguish a producer, operator, author, publisher, custodian and accountable person/organization. Assess attribution, association and delegation meanings, including what evidence substantiates an assertion. Neither an authenticated caller nor a recorded delegation alone proves scientific correctness or satisfies every accountability policy. Do not require every system to be modeled as the same kind of agent.
- Determine where each fact belongs and how it is reached through existing resources, descriptions and links; do not place all provenance in Procedure metadata by assumption. Consider source data outside Glaux and dataset/service context without presupposing mandatory database foreign keys, local ownership or a new catalog.

**Quality and uncertainty semantics (Q3)**

- Identify the assessed subject and method before naming a field: identity/class assessment, numerical property estimate, source/evidence quality or analytic judgment. Clarify informal confidence scores, statistical confidence levels, coverage probabilities, analytic confidence and technical likelihood functions. Determine which distinctions materially affect exchange; do not merely rename a stakeholder term.
- Assess SWE value-local quality and other existing mechanisms for units, bounds/distributions, stated coverage, categorical assessments, evaluator/method/version and absent/unknown information. Separate result precision/resolution from uncertainty or probability; preserve reported meaning and identify unsupported conversions.
- Determine what metadata consumers need to compare or combine assessments, including shared inputs and correlated evidence. Distinguish preserving those facts from performing fusion, recalibration or probability estimation. A common numeric range does not establish cross-producer comparability; no universal aggregation algorithm is to be designed here.

**Grouping, encoding and scale (Q4)**

- Compare individual observations, compound results, datastream context, observation collections and transport batches. Assess when shared provenance is valid, how per-item differences remain visible, and whether an application report adds anything the existing representations cannot provide. Shared packaging does not establish shared origin or identical quality.
- Examine stable references and versioned shared descriptions versus inline repetition, including retention/correction, access-controlled references, unavailable links, source identifiers and processing of large antecedent sets. Give qualitative storage, retrieval and bandwidth trade-offs; do not invent benchmark results or require preservation of every raw artifact indefinitely.
- Trace semantic preservation across supported representations and Part 3 publication. Consult Part 4 sampling and Part 5 research only where those relationships or encoding changes affect a demonstrated case; neither automatic Part 5 adoption nor a new recursive lineage-query language follows.

**Security-label interoperability (Q5)**

- Separate policy applicability, label syntax, association/binding to data, and access enforcement. Compare the roles and scope of IC-EDH/ISM and ADatP/STANAG 4774/4778 using identified editions/profiles; do not assert one-to-one policy equivalence or a universal per-JSON-field rule from an overview, stakeholder account or XML example.
- Compare whole-resource, grouped and finer-grained protection only where the applicable policy permits it. Examine binding preservation, inheritance/defaults if specified, mixed restrictions, derivation metadata and the risk of leaking protected information through links, counts, schemas or queries. Do not infer permission to strip markings or disclose derived information.
- Assess operational cost, interoperability and schema-valid disclosure. Use the Guide's existing complete-resource/projection/deny boundary; distinguish carrying labels from evaluating them and from independently verifying their authority. If a specific obligation depends on unavailable policy, report that limit instead of inventing it or blocking the entire study.

**Options and proof (Q6)**

- For each material gap, classify whether the standard already supports it, Glaux's draft design has not addressed it, an external profile/integration is needed, or the evidence remains insufficient. Compare existing representations/links, a narrowly identified extension or adapter, and no change/deferral before recommending one.
- State incremental API/representation, validation, persistence, Rust implementation and test implications against the existing Guide. Inspect particular libraries only if a supported recommendation needs them; do not repeat the general stack study or select a database because an ontology describes a graph.
- Identify positive, negative and cross-representation checks with independently expected outcomes. Explain how a consumer recovers the meaning, not just how a document parses. Separate observed evidence, proposed tests and unresolved upstream questions; identify exact prior conclusions and planning sections affected without editing them.

Use a small set of contrasting scenarios, not a domain-specific requirements catalog:

| Scenario | Questions it must expose |
|---|---|
| Direct environmental measurement or human-entered observation | Producing source, method, relevant times, responsible roles, value quality and genuinely unknown history. |
| Derived estimate or object identification from multiple inputs | Exact inputs, processing/model version, distinct identity/value assessments, shared-source dependence and accountability assertions. |
| Recalibration, software change or corrected result | Historical versus current method/configuration, input/output revisions and production history versus server update history. |
| High-volume grouped exchange with differently accessible sources | Shared versus per-item facts, efficient references, preserved meaning after conversion/publication, and safe disclosure when some provenance cannot be returned. |

These are explanatory cases and prospective verification inputs. They do not require four prototypes, an operational dataset or a new fixture suite during research. Adjust their detail to the actual evidence and record why.

---

## 3. Primary Resources

These are source entry points for execution, not findings or a claim that every source defines Glaux obligations. Record exact clauses, editions, repository commits and access dates in the report.

1. **Adopted standards package:** [CSAPI Part 1, OGC 23-001](https://docs.ogc.org/is/23-001/23-001.html), especially Annex C and Systems/Procedures/SamplingFeatures/encoding clauses; [Part 2, OGC 23-002](https://docs.ogc.org/is/23-002/23-002.html), especially DataStreams/Observations, associations, results and times; [SensorML 3.0, OGC 23-000](https://docs.ogc.org/is/23-000/23-000.html), for descriptions, processes, responsible parties, history and extension points; [SWE Common 3.0, OGC 24-014](https://docs.ogc.org/is/24-014/24-014.html), particularly §7.4 and component/encoding rules. Inspect the applicable normative schemas and examples reached from these sources, not prose alone.
2. **Conceptual foundations and provenance:** [OGC OMS entry point](https://www.ogc.org/standards/om/) and [OMS 3.0, OGC 20-082r4](https://docs.ogc.org/as/20-082r4/20-082r4.html); [SOSA/SSN 2017 Recommendation](https://www.w3.org/TR/2017/REC-vocab-ssn-20171019/); [newer SOSA/SSN edition entry point](https://www.w3.org/TR/vocab-ssn-2023/), whose publication status and alignment/module boundaries must be checked; [PROV-DM](https://www.w3.org/TR/prov-dm/) and [PROV-O](https://www.w3.org/TR/prov-o/). Review existing alignments and constraints only as needed for the cases; do not import newer draft modules into CSAPI's dated references silently.
3. **Quality terminology:** SWE Common above; [JCGM International Vocabulary of Metrology](https://jcgm.bipm.org/vim/en/), particularly measurement uncertainty and coverage; [JCGM uncertainty guides](https://www.bipm.org/en/publications/guides); [NIST confidence-interval explanation](https://www.itl.nist.gov/div898/handbook/prc/section1/prc14.htm) and [likelihood explanation](https://www.itl.nist.gov/div898/handbook/eda/section3/eda3652.htm). Use [ICD 203](https://www.dni.gov/files/documents/ICD/ICD-203.pdf), §D.6.e.(2), only to clarify analytic confidence versus event likelihood in that context, not to impose intelligence-analysis policy on generic observations. Consult [W3C Data Quality Vocabulary](https://www.w3.org/TR/vocab-dqv/) where metadata-quality assertions need comparison with value-local quality.
4. **OGC provenance work and sponsor context:** [Testbed-20 GDC Provenance Demonstration Engineering Report, OGC 24-036](https://docs.ogc.org/per/24-036.html), particularly concrete provenance exchanges and unresolved interoperability issues; [Testbed-21 initiative](https://www.ogc.org/initiatives/ogc-testbed-21/) and its linked public material on data quality, integrity, provenance and trust. Distinguish sponsor needs, study tasks, implemented demonstrations and published standards. Expand to another report only when a cited result can change a Glaux conclusion; do not audit the full OGC testbed portfolio or implement its other API families.
5. **Security-label source boundaries:** [Archived ODNI ISM overview](https://archive.dni.gov/index.php/who-we-are/organizations/ic-cio/ic-technical-specifications/information-security-marking-metadata) and [IC-EDH overview](https://archive.dni.gov/index.php/who-we-are/organizations/ic-cio/ic-technical-specifications/ic-enterprise-data-header), with release packages where needed and publicly accessible; [NISP ADatP-4774 entry](https://nisp.nw3.dk/standard/nato-adatp-4774-ed.a-v1.html) and [ADatP-4778 entry](https://nisp.nw3.dk/standard/nato-adatp-4778-ed.a-v1.html), with their official NATO references and relevant binding profiles. These archived/index sources establish leads, not current deployment authority or blanket marking requirements. Record access/version limitations and reuse IDR-SRV-040's boundary analysis.
6. **Official implementation artifacts and maintenance:** [OGC CSAPI repository](https://github.com/opengeospatial/ogcapi-connected-systems), published `v1.0.0` source commit [`8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2), and the [shared upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md). During execution, consult/date-check only topic-relevant entries and linked resolutions; pin any newly relevant draft/source. Issues and proposals remain informative unless incorporated into an applicable approved artifact.
7. **Practical implementation evidence:** [OpenSensorHub core](https://github.com/opensensorhub/osh-core), [OSH add-ons](https://github.com/opensensorhub/osh-addons) only when a relevant component is located, and [Connected Systems Go](https://github.com/SomethingCreativeStudios/connected-systems-go). Follow relevant CSAPI/SensorML/result-quality, source-link and disclosure paths/tests, with pinned commits and a bounded search. Do not assume these implement PROV exchange or every stakeholder scenario. Testbed implementations may supply additional comparison where explicitly documented.

---

## 4. Supporting Resources

Read relevant sections, not the entire completed research library:

- [IDR-SRV-019, provenance/lineage/quality/trust](../IDR%20Reports/idr-srv-019-provenance-lineage-quality-and-trust-metadata-model-report.md): §§5, 6.3, 7-11 and 13 for semantic distinctions, shared/per-item context, activities, quality and exposure; §17 for unresolved public-exchange choices. Reassess relevant recommendations against the current Goal/Guide rather than reinstating every older proposed mechanism.
- [IDR-SRV-021, SensorML representation](../IDR%20Reports/idr-srv-021-sensorml-representation-strategy-report.md), [IDR-SRV-022, SWE components](../IDR%20Reports/idr-srv-022-swe-common-data-component-strategy-report.md), and [IDR-SRV-024, semantic bindings](../IDR%20Reports/idr-srv-024-units-observed-properties-and-semantic-binding-strategy-report.md): descriptions, responsible parties, quality, units and extension boundaries.
- [IDR-SRV-034, observation semantics](../IDR%20Reports/idr-srv-034-datastream-observation-and-status-update-semantics-report.md), particularly §§6-7: composite values, observation identity and batch membership.
- [IDR-SRV-040, policy/releasability](../IDR%20Reports/idr-srv-040-policy-releasability-and-cross-boundary-access-constraints-report.md), particularly §§3.3, 5, 9.1-9.4 and 14.3: deployment roles, document/property protection, schema-valid disclosure and national/NATO adapter boundaries; [IDR-SRV-041, audit/accountability](../IDR%20Reports/idr-srv-041-audit-logging-and-accountability-strategy-report.md), for audit versus data-production history.
- [IDR-SRV-058, sampling](../IDR%20Reports/idr-srv-058-draft-csapi-part-4-sampling-features-study-report.md), [059, querying](../IDR%20Reports/idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study-report.md), and [060, Part 5](../IDR%20Reports/idr-srv-060-csapi-part-5-protobuf-first-implementation-study-report.md), only for affected sampling/relationship, visibility or encoding questions.
- [Final IDR Research Report and addenda](../IDR%20Reports/final-idr-research-report.md); [Goal v1.7](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md), §§4-5 and 8-9; [draft Guide v0.2](../../../../../Plans/glaux-server/glaux-server-implementation-guide.md), especially §§4.3, 4.10-4.11, 6.1 and 12.2. The Goal controls scope; the Guide remains a draft.

Consult another accepted report only to resolve a concrete identity, time, ingestion, publication or verification question. This is not a new general audit or reopening of completed topics.

---

## 5. Research Methodology

Use four phases within one focused research/report iteration. Do not infer human-hours estimates from AI execution; record actual work and source limitations.

### Phase 1: Establish questions, standards roles and reuse

**Objective:** Establish Q1 and the evidence baseline for Q2-Q5.

**Tasks:**

1. Confirm current scope and relevant prior findings. Refine the general scenarios and distinguish needs, tentative preferences and unsupported assumptions.
2. Identify applicable standard editions, existing mappings and the bounded official-history refresh; classify draft, normative, informative, implementation and stakeholder evidence.
3. Locate directly relevant OGC/peer examples and external-policy sources. Record unavailable material and limit affected claims without substituting invented content.

**Expected Output:** A concise standards-role/source inventory and a reusable prior-findings baseline, not a new normative framework.

### Phase 2: Trace representative information and gaps

**Objective:** Answer Q2-Q5 through concrete cases.

**Tasks:**

1. Trace each scenario's identities, inputs, methods, roles, times and consumer access through actual CSAPI/SensorML/SWE artifacts and relevant existing mappings.
2. Examine quality meaning, grouping and shared context; identify what can be preserved and interpreted, what is missing, and what cannot be verified.
3. Assess disclosure and label/binding boundaries alongside versioning, exchange and scale. Do not design an operational policy regime or uncertainty-fusion algorithm.

**Expected Output:** A compact capability/gap comparison with source-backed examples and explicit limits, distinguishing standard gaps from Glaux design gaps.

### Phase 3: Compare practical options and verification

**Objective:** Establish the implementation evidence and options for Q6.

**Tasks:**

1. Check relevant peer source/tests and testbed examples for actual exchange behavior; distinguish implemented, demonstrated, proposed and unverified support.
2. Compare existing-standard use, bounded additions/external integration and no change/deferral against the demonstrated gaps, operational cost and current Guide.
3. Define independent semantic, negative, cross-representation and access-control checks. Use optional installed-tool checks only when they resolve a consequential ambiguity; report them separately from proposed tests.

**Expected Output:** Bounded implementation/verification implications, qualitative complexity with assumptions, and viable alternatives.

### Phase 4: Synthesis

**Objective:** Produce one decision-usable report, not an implementation specification.

**Tasks:**

1. Answer Q1-Q6 with evidence or explicit unresolved limitations; reconcile affected earlier conclusions.
2. Recommend what to retain, clarify, add, integrate externally or defer, with precise scope and prerequisites. Identify potential Goal/Guide impacts without changing those documents.
3. Write and review the report in template order, check success-criteria coverage and prepare the separate acceptance/synthesis handoff.

**Expected Output:** The single research report for project-lead review. No implementation adoption or synthesis edit occurs in this phase.

---

## 6. Success Criteria

This topic research is complete when:

- [x] Q1-Q6 are answered with evidence or explicit limitations and their decision consequences.
- [x] Standards roles, editions and existing alignments are clear; conceptual lineage, drafts, stakeholder preferences and implementation precedent are not promoted into CSAPI obligations.
- [x] The broader use cases are covered without reproducing stakeholder wording or identifying the originating application; none is treated as an approved requirement merely because it was supplied.
- [x] Concrete cases distinguish production lineage, server history, responsibility assertions, verified identity and unavailable evidence.
- [x] Quality terminology and representation preserve meaningful distinctions; no universal probability, confidence conversion or fusion algorithm is assumed.
- [x] Grouping, shared references, revisions and large-volume exchange are assessed without presupposing a report resource, new catalog or database design.
- [x] IC and NATO label/binding roles, policy applicability, granularity and safe disclosure are assessed within the bounded scope, with no blanket per-field marking or whole-regime adoption inferred.
- [x] Existing standard support, Glaux design omissions, actual interoperability gaps and unresolved questions are distinguishable; relevant peer evidence and upstream-history checks are reproducible.
- [x] Alternatives, incremental implementation/test implications and prospective Goal/Guide changes are practical and explicit. No-change or deferral remains a valid outcome.
- [x] The report follows its template, reconciles relevant prior research and supplies inputs for a later synthesis addendum and planning discussion only.

Report completion, project-lead acceptance and implementation adoption are separate. Evidence limitations do not justify pretending a missing contract or policy has been established.

---

## 7. Deliverable

**Deliverable Name:** Provenance and Related Metadata Interoperability Study - Research Report<br>
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-061-provenance-and-related-metadata-interoperability-study-report.md`

Follow the [Research Report Template](../../../../../Governance/research-report-template.md). Include a plain-language executive recommendation; Q1-Q6 findings; standards/source roles; concrete capability/gap cases; options and bounded recommendations; implementation/verification implications; limitations and open questions; and exact references. Keep supporting comparisons within that report, not separate schemas, requirements documents or new governance artifacts.

Do not produce server code, a test framework, a security-policy implementation, a synthesis addendum, a Roadmap, or Goal/Guide edits during the research/report iteration.

---

## 8. Dependencies

### Must Complete Before Starting

**Internal project prerequisites (completion gates):**

- The original IDR and relevant reports in Section 4 are already complete and accepted. Supplements 058/059/060 are also accepted; they remain inputs, not new research assignments.
- Publish this plan and its matching overall-index entry before the next `proceed` authorizes research/report execution. No prerequisite exception is proposed.

**External evidence prerequisites:**

- Applicable CSAPI/SensorML/SWE artifacts and provenance/quality sources needed to substantiate the claims actually made.
- Identified public implementation and security-label material for any associated interoperability claims. A deployment's controlling policy is necessary for a deployment-specific obligation claim, not for completing a general capability assessment.

For inaccessible material, record the source, access attempt, affected question and resulting limit; do not infer its contents. A stakeholder schema, operational dataset, controlled document, SWG minutes or author response is not a general completion gate. Request additional clarification only when it would materially change a specific recommendation; do not solicit protected operational examples.

### Blocks (What This Topic Unlocks)

- A later authorized accepted-findings addendum to the final synthesis.
- Informed discussion of provenance/related-metadata implications for the Goal/Guide, alongside the pending Part 5 discussion, before resuming Guide drafting pass 2.
- No new gate on unrelated work and no invalidation of the original IDR or earlier supplements.

---

## 9. Research Status Checklist

- [x] Phase 1 complete
- [x] Phase 2 complete
- [x] Phase 3 complete
- [x] Phase 4 synthesis complete
- [x] Deliverable draft complete
- [x] Deliverable reviewed
- [x] Deliverable accepted

**Actual Research Time:** One AI-assisted research/report iteration, September 18, 2026; source investigation began at 14:12 UTC; no human-hours estimate inferred<br>
**Completion Date:** Research/report completed and accepted September 18, 2026

The research/report iteration produced the report, topic/overall status updates and a bounded upstream-history refresh. The report includes direct artifact/peer evidence, a reproduced quality-schema probe, qualitative alternatives and prospective verification cases. It recommends existing-standard use with focused Guide clarifications and conditional deferral of public provenance/policy integrations. The subsequent `proceed` accepted the report and authorized the separate synthesis addendum, now prepared for review. The original synthesis and Addenda A–C are preserved; the Goal, Guide, Roadmap and server code remain unchanged. Acceptance of research does not adopt its implementation recommendations.

---

## 10. Notes and Open Questions

- The stakeholder's use of quality terms and desired accountability evidence remains tentative. Use precise neutral language and compare interpretations; do not silently replace one ambiguous word with another or impose one sector's terminology everywhere.
- A method/model reference may describe physical sensing, calibration, signal processing or an inference model. Determine the relevant meaning and version before assigning a resource/property or treating the device and method as identical.
- Report-level grouping and identical provenance are application preferences to examine, not established CSAPI rules. Stable identification is a need; a particular UUID scheme or storage foreign key is a separate choice.
- Field-level markings may be meaningful in a particular profile, but no universal mandate has been established here. Examine actual policy and binding rules before assuming either mandatory granularity or permission to coarsen it. Assess practicality without attributing institutional motives or treating informal criticism as evidence.
- Preserve the full server goal and prior accepted research as historical evidence. Any recommended addition must earn its place through a demonstrated need and a later planning decision, not merely through the availability of another standard.

---

## References

- [Overall IDR Research Plan](overall-idr-research-plan.md), [Research Planning Approach](../../../../../Governance/research-planning-approach.md) and [Initial Planning Guidance](../../../../../Governance/initial-planning-guidance.md).
- [Research Plan Template](../../../../../Governance/research-plan-template.md), [Research Report Template](../../../../../Governance/research-report-template.md) and [Overall Research Report Template](../../../../../Governance/overall-research-report-template.md).
- [OS4CSAPI research-plan exemplar corpus at audited commit `754411897173c2ec4debaa9bcf4ed9e0f8a9e230`](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans), particularly plans 01, 15 and 38 for source-led analysis, concrete verification and useful synthesis.
- Topic-specific primary entry points and accepted research inputs are listed in Sections 3-4. The project-lead discussion is attributed planning context, not an independently verified standards source or an adopted requirements catalog.

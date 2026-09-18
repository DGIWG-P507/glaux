# Section 060: CSAPI Part 5 Protobuf-First Implementation Study - Research Plan

**Topic ID:** IDR-SRV-060<br>
**Status:** Complete<br>
**Last Updated:** September 18, 2026<br>
**Estimated Research Time:** Not yet estimated; one focused research/report iteration is planned, subject to source availability.<br>
**Actual Research Time:** One AI-assisted research/report iteration, September 17–18, 2026, America/New_York, including an overnight pause; no human-effort estimate inferred<br>
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-060-csapi-part-5-protobuf-first-implementation-study-report.md`

---

## Usage Instructions

Follow the existing [Research Plan Template](../../../../../Governance/research-plan-template.md) structure and use the [Research Report Template](../../../../../Governance/research-report-template.md) for the later report. The pinned OS4CSAPI exemplars inform concrete questions, source inventories, meaningful verification and practical recommendations; their client-specific scope, metrics and estimates are not Glaux requirements.

The plan and its supplemental index entry were published in commit `488b9ba`. The user's next `proceed` on September 17, 2026 accepted the plan for execution and authorized the research/report iteration. A later synthesis addendum and any Goal/Guide changes remain separate iterations under the established `proceed` workflow. No Part 5 implementation is adopted by this plan or research authorization.

The [report](../IDR%20Reports/idr-srv-060-csapi-part-5-protobuf-first-implementation-study-report.md) was published in `d5ef3b8`. The user's subsequent `proceed` accepted the report on September 18, 2026 and authorized the separately prepared [synthesis addendum](../IDR%20Reports/final-idr-research-report.md#addendum-c-csapi-part-5-protobuf-first-implementation). Research acceptance does not select an implementation option or change the Goal/Guide.

---

## 1. Research Objective

Determine whether Glaux Server should plan a bounded experimental implementation of OGC API - Connected Systems Part 5, **prioritizing Protocol Buffers (Protobuf)**. Establish what specification and implementation material actually exists, what it adds beyond the server's existing SWE Common binary commitments, and whether a faithful, interoperable and testable Rust implementation can be defined without inventing missing OGC behavior.

The outcome is one evidence-backed recommendation: adopt a specifically bounded experiment, preserve identified compatibility points without implementing it yet, or defer pending specific upstream material. State the benefits, costs, limitations and planning consequences of each viable option. Do not assume adoption is the desired finding.

### Why This Topic Order

The original 67-topic IDR and supplements IDR-SRV-058/059 are complete and accepted. [Goal and Definition v1.7](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md) now includes experimental Part 3, selected static Part 4 types and bounded enhanced observation filtering. The [Implementation Guide v0.2](../../../../../Plans/glaux-server/glaux-server-implementation-guide.md) remains a draft, with its broader second drafting pass pending. Encoding and schema choices can still be informed before that guide is finalized.

Existing research noticed Part 5 but did not assess its implementation: IDR-SRV-014G recorded future binary-encoding interests; IDR-SRV-007 treated Protobuf artifacts as informative, not a Part 2 conformance class. This topic fills that specific gap rather than reopening the completed IDR.

**Project-lead meeting context, September 17, 2026:** The project lead reported that Protobuf was the room's preferred encoding at that morning's CSAPI SWG meeting, and that time, resources and prioritization may limit Part 5's first publication to Protobuf. The project lead also reported intended later inclusion of Parts 3, 4 and 5 in the STANAG after publication. These are attributed planning inputs, not independently verified minutes, a finalized OGC scope decision or a change to the currently adopted NATO/OGC baseline. They justify a Protobuf-first study, not an equal-depth survey of every encoding in the repository overview.

The agreed sequence is:

1. Draft and push this plan with minimal supplemental index registration.
2. On the next `proceed`, conduct the study and push its report for review.
3. In a later authorized iteration, add accepted findings to the final synthesis as a separate addendum, preserving the original synthesis and existing supplements.
4. Discuss any Goal and Definition or Implementation Guide changes; make only subsequently agreed changes, then resume the guide's drafting sequence.

### Critical Constraints

- Preserve the full approved Parts 1 and 2 target, applicable SensorML/SWE obligations, and the already selected experiments/filtering. Protobuf does not replace required SWE JSON, Text or Binary support.
- Distinguish Connected Systems Part 5 from similarly numbered Features/Common parts. Distinguish Protobuf serialization from SWE Binary, compression, ProtoJSON and gRPC; adoption of an encoding does not automatically adopt another API or transport.
- Establish the actual Part 5 evidence baseline first. The preliminary discussion found an overview and experimental/example artifacts but no dedicated Part 5 directory or working branch in the inspected official public repository. Recheck this bounded search at execution; do not treat that observation as proof that no newer, private or unpublished draft exists.
- Separate published requirements, draft text, proposed schemas, observed implementation behavior, meeting context and Glaux recommendations. A compilable `.proto` file or working codec alone does not establish a CSAPI binding or Part 5 conformance.
- Give FlatBuffers, FlatGeobuf, video and other proposed encodings only a brief status/boundary check. If new evidence materially changes the Protobuf-first premise, report the scope question instead of silently expanding the study.
- Do not install software, implement server features, deploy services, publish issues or contact developers. Use source analysis and optional small checks with already available tools. A new benchmark, SDK, schema registry service or separate requirements/decision-document system is not a research deliverable.

---

## 2. Research Questions

### Core Questions

1. **Q1 - Available specification and maturity:** What Part 5 text, schemas, examples, decisions and implementation evidence exist, and what do they actually specify for a Protobuf-first release?
2. **Q2 - Added value and interoperable binding:** What does Protobuf add beyond approved SWE Binary, and what resource coverage, schema discovery, wire format and HTTP/Part 3 behavior would an experiment need?
3. **Q3 - Meaning, compatibility and safety:** Can the proposed mapping preserve CSAPI/SWE meaning across encodings and schema evolution, with bounded validation and authorization?
4. **Q4 - Implementation and verification:** What do existing implementations and Rust libraries demonstrate, and what incremental work and independent tests would a bounded Glaux experiment require?
5. **Q5 - Recommendation and planning impact:** Should Glaux implement an identified experiment now, retain compatibility points, or defer? Which exact existing conclusions or planning sections would need qualification or change?

### Detailed Questions

**Specification and evidence boundary (Q1)**

- Locate and pin relevant official branches, documents, schemas, examples, releases and recorded discussion. Identify actual requirements/conformance material if present; otherwise say which elements are missing. Do not manufacture class identifiers, a version, a mandatory resource set or a publication schedule.
- Distinguish an OGC-proposed CSAPI Protobuf binding from generic Protobuf tooling or a vendor's unrelated internal serialization. What syntax/edition, imports and schema conventions do the actual artifacts use? Are examples internally consistent and usable, or merely exploratory?
- Seek public corroboration of the reported Protobuf-first direction in relevant SWG records/issues where available. Retain attribution and uncertainty if no public record is found. Search the official repository and directly linked material; do not make unprovided minutes or an author response a prerequisite for reporting the evidence gap.

**Capability and wire contract (Q2)**

- Determine whether each artifact covers resource metadata, an observation/command envelope, SWE schema descriptions, data values, or some combination. Inventory applicable resource types and read/write/publication directions; do not assume observations, commands, status, events and feature descriptions all share one complete mapping.
- Compare Protobuf with the already planned SWE Binary for the actual supported cases: semantic coverage, schema availability, ecosystem interoperability, message framing, compatibility and implementation complexity. Treat reduced size/CPU or bandwidth benefit as a hypothesis unless backed by relevant measurements; do not invent performance results.
- How does a client discover the encoding and obtain the correct `.proto` or descriptor, dependencies and message type? Trace `obsFormat`/`cmdFormat`, schema response representations, media types, request `Content-Type`, response `Accept`, capability declarations and errors from their controlling sources. Separate proposed identifiers from registered/published ones.
- Identify fixed common messages versus per-stream/generated schemas, type identity and version binding. How are messages framed individually, in lists/pages, in supported multi-record requests and over Part 3? Does an envelope carry schema identity or rely on negotiated stream context? Separate binary payload selection from CloudEvents/event metadata and delivery guarantees.
- For Part 3, assess the relevant multi-encoding negotiation question without adding transports or assuming every supported encoding must be published continuously. No automatic gRPC endpoint follows from selecting Protobuf.

**Semantic mapping, evolution and safety (Q3)**

- Map only the SWE/CSAPI constructs actually covered by the available proposal: identifiers and links, phenomenon/result time, scalar types and precision, units/property definitions, absence/default/nil distinctions, composite values and associated geometry/reference frames. For arrays, choices or other unsupported constructs, identify the limitation rather than silently dropping information or narrowing the approved server baseline.
- Investigate field numbers, wire types, optional presence, unknown fields/enums, schema revision changes and old stored data. Distinguish wire compatibility from preservation of meaning, original-byte retention from decode/re-encode preservation, and semantic equality from deterministic/canonical byte assumptions.
- How would new schemas and descriptors bind to the existing immutable stream contracts? What validation remains necessary after successful decoding? Identify supported versus rejected conversions and safe handling of incompatible revisions.
- Bound message size, nesting, allocation, repeated content and descriptor/import processing. Examine malformed/truncated inputs and parser differences. Prevent arbitrary schema/import fetching or request-time code execution; preserve access controls on data, schema discovery, errors and publication.
- Existing enhanced observation filters must evaluate the same logical values regardless of accepted or returned encoding. Assess compatibility with those mappings, not a new filtering language or expanded Part 4 scope.

**Implementation, tests and recommendation (Q4-Q5)**

- Inspect only relevant OSH and CS-Go sources/branches/tests. Distinguish claimed support, codec helpers, public routes and actual interoperability evidence. If no relevant implementation is found within the documented search, record that limit without claiming none exists elsewhere.
- Compare the necessary Rust options for generated messages and dynamic descriptors/reflection, including maintained versions, licensing, required build tools, compiler/runtime compatibility and offline operation. Inspect official Protobuf Rust support and relevant `prost`/`prost-reflect` documentation; select no library merely because it can serialize a fixed example. Do not repeat the general Rust stack study.
- Show incremental effects on existing domain/codec boundaries, schema storage, HTTP/publication handling and tests. Keep a single logical resource model; additional representations do not inherently require another database or service. Give qualitative effort and assumptions, not unsupported delivery dates or speed claims.
- Define independently expected cases that prove both meaning and rejection behavior. If an executable check cannot run with installed tools, distinguish a proposed test from an observed result and report the limitation.

Use these representative cases to make the recommendation concrete; their precise coverage depends on the inspected artifacts:

| Case | What the report must establish |
|---|---|
| Scalar observation exchange | Discover the stream/schema, exchange one observation and compare identity, times, value and unit with existing representations; include absent/default/nil distinctions. |
| Composite data and schema evolution | Use one representative structured result and a compatible/incompatible revision to expose mapping limits, unknown-field behavior and historical binding. |
| Command exchange | Where supported by the proposal, preserve parameter meaning and existing command/status semantics; otherwise record the missing contract rather than inventing it. |
| Publication and invalid inputs | Explain framing and encoding selection for Part 3; include mismatched schema, truncated/oversized data and denied schema/data access. |

These are report examples and future verification inputs, not a requirement to build four prototypes or a new test suite during research.

---

## 3. Primary Resources

1. **Official CSAPI Part 5 entry points:** [Repository overview](https://github.com/opengeospatial/ogcapi-connected-systems), [public branches](https://github.com/opengeospatial/ogcapi-connected-systems/branches), and [API source tree](https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/api). The preliminary source snapshot is commit [`3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f), not an adopted Part 5 draft. Recheck at execution. Starting artifacts include [Part 2 schema examples](https://github.com/opengeospatial/ogcapi-connected-systems/tree/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f/api/part2/openapi/examples/schemas), `observationSchemaProtobuf.json` / `commandSchemaProtobuf.json` under `api/part2/openapi/schemas/json`, and [SWE Common experiments](https://github.com/opengeospatial/ogcapi-connected-systems/tree/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f/swecommon/experiments). Directory placement does not make those artifacts approved requirements.
2. **Approved baseline for comparison:** [CSAPI Part 1, OGC 23-001](https://docs.ogc.org/is/23-001/23-001.html), [Part 2, OGC 23-002](https://docs.ogc.org/is/23-002/23-002.html), and [SWE Common 3.0, OGC 24-014](https://docs.ogc.org/is/24-014/24-014.html); corresponding official `v1.0.0` CSAPI snapshot [`8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2). Review only affected encoding, schema, resource and operation clauses. Consult [SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html) where an inspected mapping actually uses it.
3. **Part 3 and relevant history:** The Guide's [pinned Part 3 baseline](https://github.com/opengeospatial/ogcapi-connected-systems/tree/6f529a15bfa63259febc3620378d3e5a06305333/api/part3), [issue 190 on multi-encoding publication](https://github.com/opengeospatial/ogcapi-connected-systems/issues/190), and the [shared upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md). During execution, consult and refresh only entries that materially affect this topic; follow relevant linked resolutions and classify approved, draft and unresolved evidence separately.
4. **Protocol Buffers controlling documentation:** [Encoding](https://protobuf.dev/programming-guides/encoding/), [field presence](https://protobuf.dev/programming-guides/field_presence/), and the [official documentation entry point](https://protobuf.dev/) for the applicable syntax/edition, descriptors, unknown fields, schema evolution, serialization guarantees, media types and Rust/toolchain support. Record the actual versions/editions inspected; generic Protobuf rules do not define the missing CSAPI mapping.
5. **Implementation evidence:** [OpenSensorHub core](https://github.com/opensensorhub/osh-core), [OSH add-ons](https://github.com/opensensorhub/osh-addons), and [Connected Systems Go](https://github.com/SomethingCreativeStudios/connected-systems-go). Start from directly relevant CSAPI format/schema handlers and codec tests; inspect add-ons or branches only where they contain or link to pertinent work. Pin the inspected commits. These are source leads, not assertions that a Part 5 implementation exists in each repository.
6. **Rust feasibility sources:** [prost](https://github.com/tokio-rs/prost), [prost-reflect](https://github.com/andrewhickman/prost-reflect), and official Protobuf Rust guidance reached through the documentation above. Follow current maintainer documentation/releases for the narrowly required capabilities; record limitations rather than installing a toolchain to settle them in this study.

These are planning entry points. The report must cite the exact execution-time clauses, paths, commits and access dates supporting its conclusions.

---

## 4. Supporting Resources

Review the affected sections of these accepted reports, not the entire completed research library:

- [IDR-SRV-007, Part 2 baseline](../IDR%20Reports/idr-srv-007-csapi-part-2-requirement-baseline-report.md), §9.4 for the earlier Protobuf boundary; [IDR-SRV-014G, community lessons](../IDR%20Reports/idr-srv-014g-os4csapi-discussions-lessons-learned-study-report.md), LL-18 and related open questions for prior Part 5 awareness.
- [IDR-SRV-012, media types and encoding selection](../IDR%20Reports/idr-srv-012-content-negotiation-media-types-and-encoding-selection-report.md), §§6-9; [IDR-SRV-022, SWE component strategy](../IDR%20Reports/idr-srv-022-swe-common-data-component-strategy-report.md), especially component/encoding meaning and cross-format verification; [IDR-SRV-023, validation strategy](../IDR%20Reports/idr-srv-023-schema-and-encoding-validation-strategy-report.md), especially schema binding, safe resolution and semantic validation.
- [IDR-SRV-035, streaming/publication](../IDR%20Reports/idr-srv-035-streaming-and-event-publication-strategy-report.md), especially Resource Data versus event envelopes and encoding selection. Follow IDR-SRV-014H only for relevant Part 3 evidence, not a new Pub/Sub study.
- [IDR-SRV-059, enhanced querying](../IDR%20Reports/idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study-report.md), for encoding-independent logical result filtering; [Final IDR Research Report](../IDR%20Reports/final-idr-research-report.md) and existing addenda for later synthesis context.
- [Goal and Definition v1.7](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md) and [draft Implementation Guide v0.2](../../../../../Plans/glaux-server/glaux-server-implementation-guide.md), especially §§4.3, 4.4.1, 4.8, 6.2, 6.5 and 7-8. The current approved scope controls over older research recommendations; the technical guide remains a draft.

Other accepted reports may be consulted only when a concrete identity, time, security, command or testing question requires them. Their use does not create another topic or require another general audit.

---

## 5. Research Methodology

Phase durations are not estimated separately; execute these four phases within one focused research/report iteration and record actual work rather than inferred human hours.

### Phase 1: Establish the available baseline

**Objective:** Answer Q1 and identify exactly what can be assessed.

**Tasks:**

1. Confirm the approved project baseline and relevant prior findings; record the project-lead meeting account as attributed context.
2. Inspect the official source entry points, date-check relevant register entries and pin available Part 5/Protobuf artifacts and linked decisions. Record the bounded search and any missing draft, requirements, schema or tests.
3. Briefly classify other proposed encodings as context; retain Protobuf as the study's priority.

**Expected Output:** A concise source/maturity inventory and clearly identified specification gaps. If no coherent Part 5 binding is available, continue as a feasibility/readiness assessment, not an invented implementation specification.

### Phase 2: Trace the contract and preserve meaning

**Objective:** Answer Q2-Q3 against the actual artifacts and approved baseline.

**Tasks:**

1. Map covered resources, operations, schema discovery, media types, framing and Part 3 touchpoints; label missing or proposed behavior.
2. Compare with existing SWE Binary and trace the representative cases through schema/value mapping, revisions, validation and access controls.
3. Identify whether each gap is editorial, an implementation limitation or a substantive interoperability decision. Do not fill substantive gaps silently.

**Expected Output:** A compact coverage/gap comparison and concrete semantic/compatibility cases, with no invented benefit measurements.

### Phase 3: Assess implementation and verification

**Objective:** Answer Q4 with bounded peer and Rust evidence.

**Tasks:**

1. Inspect relevant peer codec/handler/test paths and the necessary Rust library capabilities, versions and build dependencies.
2. Identify incremental effects on the existing server design and the independent valid/invalid, cross-format and schema-evolution tests needed to establish support.
3. Use optional small checks only where installed tools can resolve a consequential question; clearly distinguish source inspection, executed checks and proposed future tests.

**Expected Output:** A practical feasibility assessment, implementation/test implications and qualitative complexity with explicit assumptions.

### Phase 4: Synthesis

**Objective:** Answer Q5 and produce the single report for review.

**Tasks:**

1. Compare bounded implementation, compatibility-only preparation and deferral against the evidence. State exact preconditions and exclusions for any recommended experiment.
2. Explain which prior conclusions remain valid and which need qualification; identify prospective Goal/Guide impacts without editing them.
3. Write the report in template order, verify Q1-Q5 and success-criteria coverage, and prepare the separate acceptance/addendum handoff.

**Expected Output:** The research report, with a plain-language recommendation and no automatic implementation adoption.

---

## 6. Success Criteria

This topic research is complete when:

- [x] Q1-Q5 are answered with evidence or explicit unresolved limitations and their decision consequences.
- [x] The actual Part 5/Protobuf source and maturity baseline is identified; meeting preference, draft artifacts and approved obligations are not conflated.
- [x] Protobuf is compared with existing SWE Binary commitments without replacing them or assuming performance benefits.
- [x] Covered resource/operation directions, schema discovery, framing, HTTP/publication behavior and substantive binding gaps are explicit.
- [x] Representative cases address value meaning, presence/nil/defaults, precision/time, schema evolution, safe parsing and authorization; unsupported mappings are visible.
- [x] Peer and Rust conclusions identify exact sources and support depth; unavailable tools or unpublished work are not presented as verified results.
- [x] Incremental implementation and independent verification needs are practical, bounded and grounded in the existing design, including encoding-independent observation filtering.
- [x] The recommendation explains what could be implemented faithfully, what would remain a Glaux experiment and what requires upstream clarification; alternatives and qualitative costs are stated.
- [x] The report follows the template, has reproducible references, records relevant upstream-history checks and supplies inputs for a later synthesis addendum and planning discussion only.

Report completion and project-lead acceptance remain separate. A well-supported recommendation to defer can complete this study; it cannot establish that missing specification content has been resolved or that Part 5 has been implemented.

---

## 7. Deliverable

**Deliverable Name:** CSAPI Part 5 Protobuf-First Implementation Study - Research Report<br>
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-060-csapi-part-5-protobuf-first-implementation-study-report.md`

Follow the [Research Report Template](../../../../../Governance/research-report-template.md) in its required section order. Include a plain-language executive recommendation; Q1-Q5 coverage; source/maturity inventory; comparison with SWE Binary; the representative cases; bounded peer/Rust evidence; implementation and verification implications; alternatives, limitations and prospective planning changes. Keep supporting tables within the report rather than creating parallel specifications or process documents.

Do not produce server code, a new test framework, the synthesis addendum, a Roadmap, or Goal/Guide edits during the research/report iteration.

---

## 8. Dependencies

### Must Complete Before Starting

**Internal project prerequisites (completion gates):**

- The original 67-topic IDR, including IDR-SRV-057, is complete and accepted. The relevant reports in Section 4 are existing inputs, not new assignments.
- IDR-SRV-058/059 are complete and accepted; the resulting approved scope is recorded in Goal v1.7. This study does not reopen their acceptance or selected scope.
- Publish this plan and its matching overall-index entry first. Completed in commit `488b9ba`; the user's subsequent `proceed` authorized research/report execution.

**External evidence prerequisites:**

- Access to the approved CSAPI/SWE baseline and available official Protobuf/Part 5 material needed for the claims actually made.
- Relevant implementation and Rust documentation for feasibility claims, with evidence limitations recorded when unavailable.

If a source cannot be accessed, identify it, the attempt, affected question and limitation; do not infer its contents. A new Part 5 working draft, SWG minutes or author response is not a mandatory gate: their absence may justify a qualified readiness assessment or deferral. Do not claim a complete wire contract without the necessary evidence, or treat absence from one searched snapshot as proof of global absence.

### Blocks (What This Topic Unlocks)

- An accepted-findings Part 5 addendum to the final synthesis in a later authorized iteration.
- Discussion of whether a bounded experimental Protobuf implementation belongs in the Goal and Definition and how it would affect the Implementation Guide.
- Informed resumption of Guide drafting pass 2 after that discussion; no new prerequisite is imposed on unrelated work or completed topics.

---

## 9. Research Status Checklist

- [x] Phase 1 complete
- [x] Phase 2 complete
- [x] Phase 3 complete
- [x] Phase 4 synthesis complete
- [x] Deliverable draft complete
- [x] Deliverable reviewed
- [x] Deliverable accepted

**Actual Research Time:** One AI-assisted iteration across September 17–18, 2026; source review, small checks, drafting and technical review, with an overnight pause. No human-hours estimate inferred.<br>
**Completion Date:** Research/report completed and accepted September 18, 2026

Research execution, technical review and project-lead acceptance are complete. The separately authorized synthesis addendum is prepared for review. The recommendation preserves existing compatibility points while deferring selection of a Part 5 binding; an OSH-compatible experiment remains an alternative for discussion, not adopted scope. The next `proceed` begins the Goal/Guide discussion, without automatically editing those documents or authorizing implementation.

---

## 10. Notes and Open Questions

- The September 17 meeting account is sufficient to prioritize Protobuf; the project lead need not supply a technical design. A later public draft, minutes or author-provided reference would improve the evidence and should be incorporated if available, without creating an indefinite monitoring task.
- First-release Protobuf-only scope remains possible, not settled by this plan. Record any execution-time evidence that confirms or contradicts it.
- A small example encoding is not automatically a reusable schema-generation rule for arbitrary datastreams. Identify that distinction explicitly before recommending a general implementation.
- Where the draft is incomplete, distinguish reasonable experimental choices from choices that would defeat cross-implementation interoperability. Do not call a locally invented binding exact implementation of an absent OGC contract.
- Preserve all prior reports and synthesis addenda as historical evidence. Any later adoption belongs in the familiar Goal/Guide/Roadmap sequence, not a retroactive rewrite of the research.

---

## References

- [Overall IDR Research Plan](overall-idr-research-plan.md), [Research Planning Approach](../../../../../Governance/research-planning-approach.md) and [Initial Planning Guidance](../../../../../Governance/initial-planning-guidance.md).
- [Research Plan Template](../../../../../Governance/research-plan-template.md), [Research Report Template](../../../../../Governance/research-report-template.md) and [Overall Research Report Template](../../../../../Governance/overall-research-report-template.md).
- [OS4CSAPI research-plan exemplar corpus at audited commit `754411897173c2ec4debaa9bcf4ed9e0f8a9e230`](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans), particularly plans 01, 15 and 38 for question-led analysis, source/fixture rigor and practical synthesis.
- Topic-specific source entry points and accepted report inputs are identified in Sections 3-4. The September 17, 2026 project-lead meeting account is recorded in Section 1 as user-supplied context, not a public standards citation.

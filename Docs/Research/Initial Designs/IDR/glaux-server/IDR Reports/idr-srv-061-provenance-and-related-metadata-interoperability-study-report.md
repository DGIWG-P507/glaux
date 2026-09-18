# Section 061: Provenance and Related Metadata Interoperability Study - Research Report

**Topic ID:** IDR-SRV-061<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-061][plan]<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan][overall]<br>
**Research Questions Covered:** Six; existing support, origin and responsibility, quality, grouping/exchange, protection, and recommendations/verification<br>
**Methodology Used:** Targeted prior-research reconciliation, primary-standard and pinned schema/code inspection, bounded upstream-history review, an installed-tool schema probe, and independent evidence reviews<br>
**Research Time:** One AI-assisted research/report iteration, September 18, 2026; source investigation began at 14:12 UTC. No human-hours or implementation-effort estimate is inferred from execution time.<br>
**Primary Source(s):**

- [CSAPI Part 1][part1], [Part 2][part2], [SensorML 3.0][sml], and [SWE Common 3.0][swe]
- [OMS 3.0][oms], [SOSA/SSN 2017][ssn], [PROV-DM][prov-dm], and [PROV-O][prov-o]
- Pinned schemas, implementation paths, quality references, and public policy/binding material identified below

**Supporting Resources:** [Goal v1.7][goal], [draft Guide v0.2][guide], accepted IDR-SRV-019/021/022/023/024/034/040/041/058/059/060, and the [upstream-history register][history]<br>
**Document Purpose:** Determine what the existing CSAPI package can convey, where practical provenance exchange remains incomplete, and which bounded planning clarifications or optional integrations deserve discussion<br>
**Author(s):** Glaux research workflow, AI-assisted<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** September 18, 2026<br>
**Date:** September 18, 2026<br>
**Last Updated:** September 18, 2026

## Usage Rules

This report follows the [Research Report Template](../../../../../Governance/research-report-template.md). Findings identify their evidence; interpretations and recommendations are project analysis, not additional standards requirements. The research/report iteration authorized execution of the published plan and publication of this report for review, not automatic acceptance or implementation. Preliminary stakeholder concerns inform the cases without becoming a requirements catalog.

The report was published in commit `3283efd`. The user's subsequent `proceed` accepted it on September 18, 2026 and authorized the separate synthesis addendum. Acceptance records the research as suitable for downstream use; it does not adopt a public provenance profile, security adapter, or other implementation option, or change the Goal/Guide.

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base
4. Findings by Research Question
5. Decision Analysis
6. Key Recommendations
7. Implementation Implications and Estimates
8. Risks, Constraints, and Open Questions
9. Validation Against Plan Success Criteria
10. Next Steps and Handoff
11. References
12. Appendices

## 1. Executive Summary

Provenance is the information that helps someone understand where a result came from and how it was produced. Glaux does not need a second observation model to begin supporting it. CSAPI already connects observations with datastreams, producing systems and procedures; SensorML describes processes and their context; SWE Common supports value-specific quality concepts. The approved Goal already includes provenance context, and the draft Guide already preserves practical origin/transformation information. [part1], [part2], [sml], [swe], [goal], [guide]

The consequential gap is more specific: **a consumer cannot infer the exact input results, actual processing execution, method revision, and accountable roles for an individual output merely from its current system/procedure links.** The inspected public artifacts and peer paths do not establish one interoperable contract for exchanging all those facts. PROV supplies reusable meanings for these relationships, including an existing informative SOSA/SSN alignment; it does not itself supply a CSAPI endpoint or require an RDF server. [obs-schema], [stream-schema], [ssn], [prov-o]

Recommend keeping the existing server goal and discussing targeted Guide clarifications: distinguish production from ingestion history, preserve exact references where supplied, retain quality meaning, avoid silent metadata loss, and test independently authorized disclosure of provenance. A public PROV-compatible export/profile is a conditional future option, not a prerequisite or an adopted feature. No separate report resource, universal confidence score, graph database, or national/NATO security platform is justified by this study.

Two practical cautions matter. First, quality terms are not interchangeable, and the published SWE JSON schema has a previously identified quality-validation gap, reproduced here. Second, inspected IC/NATO material does not establish a universal rule to repeat markings on every JSON field; deployment obligations still require an applicable authoritative policy and binding profile. These are bounded implementation/interpretation questions, not reasons to expand the whole project. [R022], [R023], [ism], [edh], [nato4774], [nato4778]

## 2. Scope and Plan Alignment

This is the single deliverable for IDR-SRV-061. It assesses broad, consumer-usable provenance against existing standards and four contrasting cases. It does not adopt preliminary preferences about schema placement, identifiers, storage relationships, grouping, probability, or marking granularity.

The study completed targeted standards/schema inspection, quality terminology comparison, public OGC/peer evidence review, a bounded history refresh, and options/test analysis. No server was implemented or deployed; no software was installed; no upstream issue was published. No operational stakeholder dataset or controlled policy was required or inspected.

### Research Question Coverage Matrix

| Plan question | Short form | Coverage | Evidence location |
|---|---|---|---|
| Q1 | Existing support and standards roles | Answered, with edition and exchange limits | §§3, 4.1 |
| Q2 | Origin, derivation and responsibility | Answered; exact public lineage contract remains open | §4.2 |
| Q3 | Quality and uncertainty | Answered; JSON quality mapping requires explicit validation treatment | §4.3; Appendix A |
| Q4 | Grouping and scalable exchange | Answered qualitatively; no performance claim | §4.4 |
| Q5 | Protection and disclosure | Answered within accessible-source limits; no deployment-policy determination | §4.5 |
| Q6 | Recommendation and verification | Answered; adoption deferred to planning discussion | §§4.6–10 |

## 3. Evidence Base

### 3.1 Primary Sources Reviewed

All sources were accessed September 18, 2026. Clause references below identify the inspected scope, not a claim to have audited every provision.

| Source | Version/status and authority | Anchor used | Availability / limits |
|---|---|---|---|
| [CSAPI Parts 1][part1] / [2][part2] | Approved 1.0, OGC 23-001 / 23-002 | Part 1 §3, Annex C; Part 2 datastream/observation model, particularly §9.7 | Controlling applicable API provisions; conceptual annex is not an extra implementation profile |
| [SensorML][sml] / [SWE Common][swe] | Approved 3.0, OGC 23-000 / 24-014 | SensorML §§8.2–8.3, 9.1; SWE §§7.4, 8.2.15 | Description/model provisions distinguished from concrete JSON artifacts |
| [Official source tree][ogc-tree] | Published-source commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2` | Observation/DataStream; SensorML description/process; SWE Quantity inheritance | Reproducible artifact evidence; not every permissive schema member has standardized semantics |
| [OMS][oms] | Abstract Specification Topic 20, 3.0.0, OGC 20-082r4 | §§8.1.3, 8.4.2–8.5.2, 9.2.5, 9.9 | Retrieved in memory after browser extraction rejected the large HTML; abstract model, not CSAPI wire schema |
| [SOSA/SSN][ssn] | W3C Recommendation, October 19, 2017 | §6.5 PROV alignment | Edition cited by CSAPI; alignment is non-normative |
| [Newer SOSA/SSN draft][ssn-draft] | “2023 Edition,” Working Draft September 14, 2026 | §§6.1, 7 | Draft; not silently substituted for the dated Recommendation |
| [PROV-DM][prov-dm] / [PROV-O][prov-o] | W3C Recommendations, April 30, 2013 | DM §§5.2–5.4, 5.6; O §§3–4 | Relationship model/ontology, not a ready-made CSAPI exchange contract |
| [DQV][dqv] | W3C Working Group Note, December 15, 2016 | §§3–4, 6.2–6.3 | Complementary dataset/metadata assessment vocabulary, not a Recommendation or universal quality model |
| [NIST likelihood][likelihood] / [confidence][confidence]; [VIM][uncertainty] / [GUM][gum] | Authoritative statistical/metrological explanations | NIST §§1.3.6.5.2, 7.1.4; VIM §§2.15, 2.26, 2.37, 2.41, 4.14; GUM §§5.2.4–5.2.5 | Different contexts must remain distinguishable |
| [ICD 203][icd203] | Published directive, inspected §D.6.e.(2) | Analytic uncertainty terminology | Context-specific distinction, not generic Glaux policy |
| [ISM][ism] / [IC-EDH][edh] | Archived ODNI overviews; listed releases 2021-NOVr2022-NOV / 2019-MAR | Overview, downloads, mission requirements | Overviews readable in browser; package retrieval failed. No package/schema validation or current deployment applicability established |
| [ADatP-4774][nato4774] / [4778][nato4778] | NISP entries: Edition A Version 1, 2017 / 2018 | Syntax versus binding; references to NATO publications | Index evidence, not a complete national-policy or implementation assessment |
| [NISP v15 profiles][nisp-profiles] / [NCIA TN-1491][ncia] | ADatP-4778.2 Ed A V1:2020 listings / historical technical note Ed 2, work concluded September 2017 | Profile chapters 7–9; note Annexes F–H | Historical concrete mechanisms must not be passed off as inspected 2020 normative text |

### 3.2 Supporting Sources Reviewed

| Source | Pin/status | Relevance and anchor | Limits |
|---|---|---|---|
| [CS-Go Observation][csgo-obs], [formatter][csgo-format], [tests][csgo-tests] | `b1fd2e0e9bd69e222d05258d659a842ca24502cb` | Associations, parameters, times, link normalization and decoding | Source/tests inspected, not executed; no complete lineage contract demonstrated |
| [CS-Go Procedure][csgo-procedure], [SensorML formatter][csgo-sml], [DataStream][csgo-stream] | Same pin | Descriptive context preservation; typed component quality limitation | Flexible result data does not prove metadata round-trip |
| [OSH observation binding][osh-obs], [internal interface][osh-interface], [DataStream binding][osh-stream] | `9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08` | Actual read/write paths versus internal fields | Not a whole-product conformance audit |
| [OSH SWE tests][osh-quality], [quality fixture][osh-quality-fixture], [dynamic-quality fixture][osh-dynamic-quality] | Same pin; fixtures under `examples_v20` | Value-local quality and per-record references | Legacy library examples/tests, not proof of complete SWE 3.0/CSAPI support |
| [OSH SensorML binding][osh-sml], [service delegation][osh-sml-service], [batch tests][osh-batch] | Same pin | Dropped metadata, method shapes, individual observation IDs | No runtime or interoperability test executed |
| [OGC 24-036][testbed20] | Testbed-20 Engineering Report, June 23, 2025 | §2.2; Annexes B–D | Demonstration evidence, expressly not an OGC Standard |
| [Testbed-21 initiative][testbed21] | Dated retrieval of sponsor/task page | NGA/NASA sponsorship; quality/provenance task | Planned research context, not completed CSAPI conformance evidence |
| [Shared history register][history] | Bounded IDR-SRV-061 refresh | Relevant issues/PRs and unchanged master/tag | Not a full register recensus; draft Part 3/4 pins retain their prior check dates |
| [Prior reports][R019], [Goal][goal], [Guide][guide] | Accepted research; approved Goal v1.7; draft Guide v0.2 at Glaux `24587715ee40a1d352df43c344a09e7182ee8f9d` | Reconciliation in §4.6 | Current Goal controls project scope; older research proposals are not automatically implementation commitments |

### 3.3 Evidence Quality Notes

Approved applicable standards control obligations. Informative alignment, abstract-model capability, source-code behavior, tests, engineering demonstrations, open issues and attributed stakeholder needs are different kinds of evidence.

The schema probe checks only Quantity and its five-file inheritance/reference closure, not all SWE schemas or Glaux conformance. Peer findings are path-specific static observations. Bounded searches did not identify an established exact-input provenance contract in the inspected paths; this does not prove that no other implementation, extension or unpublished work exists.

The [history register][history] records unchanged published/master pins and open issues #149, #162, #174, #178, #179, #182, #185 and PR #199. New issue [#201](https://github.com/opengeospatial/ogcapi-connected-systems/issues/201) identifies SystemEvent JSON mapping questions. A September 2 comment on [#9](https://github.com/opengeospatial/ogcapi-connected-systems/issues/9#issuecomment-5507022757) describes ongoing ontology-alignment work. None supplies an adopted provenance API or repairs an approved artifact merely by being discussed.

Security conclusions are deliberately narrower than a policy review: public overviews and profile examples establish roles and alternatives, not permission to omit required markings or proof of accreditation. No stakeholder policy, deployment authority, or validated cross-policy translation was supplied.

## 4. Findings by Research Question

### 4.1 Q1 — Existing support and standards roles

**Finding.** The standards already provide complementary pieces. Their different roles explain why “use OMS and PROV” is not yet a concrete implementation decision.

| Piece | Source-backed role | Consequence for Glaux — interpretation |
|---|---|---|
| OMS / older O&M | Abstract observation concepts; OMS includes contextual related observations, result quality and collections. The OGC distinguishes this from O&M XML implementation standards. [oms], [oms-overview] | No replacement JSON observation schema follows from the abstract model alone |
| SOSA/SSN | Semantic vocabulary. The 2017 informative alignment maps observation activity to PROV Activity, procedure to Plan, result to Entity. [ssn] | Reuse existing meanings before inventing an alignment; keep edition and authority explicit |
| CSAPI | Resource/API contract with relationships to its conceptual foundations; Part 1 Annex C explains those relationships and legacy-service mappings. [part1] | Implement the applicable CSAPI contract, not legacy SOS/SPS or every abstract association |
| SensorML | Describes systems/processes, I/O, methods, configuration, contacts and history. [sml] | Useful provenance context, but a reusable description is not automatically evidence of a particular execution |
| SWE Common | Describes values and associated quality concepts; §7.4.3 places fuller processing lineage outside SWE itself and points toward process descriptions. [swe] | Preserve value meaning; do not make a quality component carry the entire production history |
| PROV / DQV | PROV describes production/derivation and responsibility relations; DQV describes quality assessments and their provenance. [prov-dm], [prov-o], [dqv] | Candidate crosswalk/export vocabulary, not a mandatory graph database, API family, or universal score |

**Evidence and qualifications.** OMS §8.1.3 can relate observations with a contextual role. Its §§8.4–8.5 distinguish procedure description from observer identity; §9.2.5 supports multiple result-specific quality measures. Those conceptual affordances do not prove that every relation has a standard CSAPI JSON field. [oms]

SOSA/SSN 2017 also aligns `isSampleOf` with derivation. It would therefore be wrong to say sampling has no lineage meaning. The narrower limitation is that a sampling link alone does not identify the exact input observation results and processing execution that generated an output. The newer Working Draft has different Asset/System alignment choices and acknowledges incomplete OMS coverage; its axioms must not be silently mixed into the 2017 baseline. [ssn], [ssn-draft]

**Practical evidence.** OGC 24-036 demonstrates negotiated PROV bundles associated with processing jobs; supplementary paths were not all defined by the draft Processes specification. It also reports unresolved identity and remote-chain questions. Its deployment examples do not validate protected-provenance disclosure: security was outside the study's relevant scope, and one example protected provenance while leaving processing results unrestricted. These are useful interoperability lessons, not CSAPI endpoint or access-policy prescriptions. [testbed20]

Testbed-21 identifies NGA and NASA sponsorship and a quality/provenance research task. This substantiates institutional interest, not a claim that the preliminary stakeholder requests are NGA requirements, that the planned work is complete, or that it changes CSAPI conformance. [testbed21]

**Recommendation.** Build on the existing package and informative alignment. Do not adopt an OMS-specific replacement schema, Processes/STAC service, PROV serialization, or DQV export without an identified exchange need.

### 4.2 Q2 — Origin, derivation and responsibility

**Finding.** Existing links expose meaningful source context, but exact production evidence needs identities and scopes that cannot safely be guessed.

**Concrete artifact evidence.** The pinned Observation schema defines `datastream@id`, `samplingFeature@id`, `procedure@link`, `phenomenonTime`, `resultTime`, `parameters`, and inline/linked result forms. The DataStream schema has a required producing-system association and conditional shared procedure/deployment/feature associations. The Observation schema defines no dedicated antecedent-observation list or accountable-creator member. Its openness to unknown properties does not standardize such a member. [obs-schema], [stream-schema]

For example, a consumer can follow an observation's datastream to its producing system and a relevant procedure description. It cannot conclude that the current description is the immutable model/configuration used for an earlier result, that the publisher operated the sensor, or that the sampling feature was an antecedent observation. Those are analysis limits, not missing values the server should manufacture.

| Consumer question | Existing evidence to use first | Remaining boundary / project interpretation |
|---|---|---|
| What produced this result? | DataStream system association; applicable observation/shared procedure link | Device, software process and method are distinct even when packaged together |
| What method/configuration applied? | SensorML `typeOf`, configuration, inputs/outputs/parameters, method and history [process-schema], [simple-schema], [description-schema] | Bind the relevant revision/context to the result; current metadata alone does not establish past execution |
| Which inputs were actually used? | Process descriptions and aggregate connections explain intended processing [aggregate-schema]; source/result links can identify artifacts | A process diagram does not automatically enumerate actual input-result identities; an exact-input exchange contract remains open |
| Who was responsible? | SensorML contacts with role identifiers [party-schema]; PROV attribution, association and delegation [prov-dm] | Contact, publisher, custodian, authenticated submitter and accountable producer are not interchangeable |
| When did it happen? | CSAPI phenomenon/result times; description/history context | Receipt, persistence and later correction times are server facts, not substitutes for production time |
| What is unknown? | Available source assertions and resolvable references | Distinguish missing, unavailable, withheld and unverified information; none means “no antecedents” |

PROV distinguishes derivation, revision and primary source, as well as entity attribution, activity association and agent delegation. A “primary source” relationship is contextual; every raw input is not automatically entitled to that designation. PROV expresses assertions, not proof of authority or scientific truth. [prov-dm], [prov-o]

**Implementation evidence.** CS-Go's model/formatter carry observation links and parameters; inspected tests exercise link handling and decoding. Its SensorML formatter copies contacts/history/configuration/method in both directions. This is practical existing-standard support, not demonstrated recovery of exact processing runs and antecedents. [csgo-obs], [csgo-format], [csgo-tests], [csgo-sml]

OSH illustrates why an internal field is insufficient evidence: `IObsData` has parameters, while the inspected observation JSON binding skips unrecognized inputs and does not write parameters or a per-observation procedure link. Its DataStream binding does handle shared source associations. These findings concern those paths at the pinned commit, not the whole product or all formats. [osh-interface], [osh-obs], [osh-stream]

**Recommendation.** Preserve supplied source IDs, exact revisions or artifact references, method/configuration context and asserted role provenance when applicable. Do not invent missing humans, delegation chains or historical model identities. Keep authenticated ingestion identity separately. For remote data, preserve the reference and available version evidence without requiring local ownership, mandatory foreign keys, unlimited copying or indefinite raw-data retention. A digest over specified bytes can support integrity or identity checks; it does not by itself establish authorship or correctness.

### 4.3 Q3 — Quality and uncertainty

**Finding.** The appropriate term depends on what is assessed and how. Renaming every “likelihood” to “confidence” would preserve the original ambiguity.

| Meaning | Source-backed distinction | Exchange consequence — recommendation |
|---|---|---|
| Statistical likelihood | A likelihood function relates observed data to a parameterized model. [likelihood] | Do not interpret it as a generic probability that an output is true |
| Frequentist confidence level | Describes coverage under repeated interval construction. [confidence] | Preserve the method, interval and intended interpretation |
| Measurement uncertainty / coverage | Uncertainty characterizes dispersion attributed to a measurand; GUM also uses “level of confidence” for coverage probability. [uncertainty], [coverage] | Preserve quantity, units, coverage and assumptions; avoid a blanket definition of “confidence” |
| Analytic confidence | ICD 203 distinguishes confidence in the evidentiary/reasoning basis from event likelihood. [icd203] | Useful contextual distinction, not a policy imposed on every sensor |
| Precision / resolution | Agreement among replicate measurements under specified conditions and distinguishable change are different concepts. [precision], [resolution] | More decimal places do not imply lower uncertainty or greater truth probability |
| Informal classifier/quality score | Meaning depends on its stated producer/method contract | Do not label a normalized score a calibrated probability without evidence |

**Interpretation.** An identity/class assessment and an uncertainty estimate for a numerical property concern different propositions. They may belong to the same output, but neither should overwrite the other. For comparison, a consumer needs the assessed subject/component, metric/definition, method/version, units or scale, evaluator/source where supplied, relevant interval/coverage, and limitations. Unknown uncertainty is not zero. Comparable numeric ranges alone do not establish comparable meaning.

**Representation evidence.** SWE supports static and dynamic component quality, including numerical, range, categorical and textual forms. Its conceptual quality association and encoding must be distinguished. [swe] In the pinned/published JSON Quantity inheritance chain, no `quality` member is declared. The installed-tool probe accepted an arbitrary quality string while correctly rejecting an invalid unit structure: this demonstrates permissiveness, not standardized quality validation. [quantity-schema], [simple-component-schema] Appendix A documents inputs, reference rebasing and controls.

This is a reproduced **existing** IDR-SRV-022/023 gap, not a discovery that invalidates those reports. Their preservation and separate semantic-validation approach remains relevant. A supported concrete quality form needs an explicit, tested interpretation; preserve upstream schema bytes rather than silently editing them or mistaking unknown-member acceptance for support. [R022], [R023]

OSH has concrete quality tests and numerical/range/categorical fixtures, including per-record references; they are legacy-version library evidence, not complete current CSAPI validation. CS-Go's inspected typed component struct has no explicit quality member. Both findings strengthen the need for particular round-trip fixtures, not a conclusion that either product universally lacks quality support. [osh-quality], [osh-quality-fixture], [osh-dynamic-quality], [csgo-stream]

DQV's metric/measurement/subject pattern may help exchange dataset or metadata assessments, but it is not a replacement for existing result-component meaning. [dqv] Shared instruments or references can correlate uncertainty; GUM explicitly addresses that problem. Preserving shared inputs/calibration context helps a later analyst, but a lineage graph does not itself calculate covariance or justify combining probabilities. [gum] Generic lineage also does not prove metrological traceability, which concerns a documented calibration chain to a reference. [traceability]

**Recommendation.** Retain meaningful assessment distinctions and test their representation. Do not add a universal score, calibration service, statistical conversion or fusion engine.

### 4.4 Q4 — Grouping and scalable exchange

**Finding.** Compound results, common datastream context, collections and transport batches solve different problems. None establishes a universal “same object and identical provenance” report rule.

**Evidence.** OMS collections permit membership and optional collection constraints; they do not prescribe that universal rule. [oms] The DataStream schema expressly limits shared associations to cases where they apply across the stream. [stream-schema] OSH's batch test creates observations and verifies their individual IDs, demonstrating a batch without turning it into a separate report entity. [osh-batch] PROV collection membership likewise does not substitute for derivation. [prov-dm]

**Scenario analysis — prospective examples, not executed integrations:**

| Case | Existing-standard path and useful facts | Failure to prevent / independently expected result |
|---|---|---|
| Direct environmental measurement or human entry | Observation → datastream → producer/procedure; typed value and applicable quality; actual source and times | Missing history remains unknown. A user uploading a result is not automatically its creator. A method contact is not automatically its operator |
| Derived estimate or object identification | Reusable processing description plus supplied exact input references and method/configuration revision; separate identity/value assessments | Method description alone cannot recover the input set. Two outputs sharing source evidence cannot be assumed independent |
| Recalibration, software update or corrected result | Preserve the earlier context and identify the changed method/result; keep original production evidence separate from later write/correction evidence | Updating a current description must not silently reattribute earlier outputs. A server correction is not automatically a new physical observation |
| High-volume batch with differently accessible sources | Share genuinely common descriptions by stable references; retain each item's identity and exceptions; authorize context disclosure | Packaging cannot manufacture shared lineage. Hidden inputs must not leak or be represented as nonexistent. Conversion/publication must retain supported meaning |

**Qualitative trade-offs — analysis, not benchmarks.** Shared versioned descriptions reduce repeated bytes and allow caching, but introduce availability, authorization and retention dependencies. Inline facts improve local readability but increase volume and can diverge when copied. Large antecedent sets require bounded processing and a defined reference/list contract if exposed; a new recursive query language or unrestricted graph expansion does not follow. Retaining an identifier/digest without retaining the artifact is a legitimate evidence limit that must be visible to authorized consumers.

A compound observation result can carry related components under its result schema; making separate observations or grouping them is a semantic choice, not merely a bandwidth optimization. Per-item changes in model, quality or responsible role must remain distinguishable even when transport/shared context is optimized. UUIDs may be useful identifiers, but this study establishes no universal UUID or new catalog requirement. [R034]

The selected experimental Part 3 publication path should be checked against the same supported semantics as HTTP. Part 4 sampling relationships may explain what was sampled, without supplying exact processing inputs. The pending Part 5 decision does not authorize encoding selection here: any later codec must prove preservation or explicitly restrict unsupported cases, rather than silently dropping metadata. No new lineage filter is added to the already selected observation filtering scope. [R058], [R059], [R060], [guide]

**Recommendation.** Prefer existing observations and datastream context, with precise shared/per-item scope and bounded references. Defer a new report resource and public large-lineage exchange mechanism until a concrete consumer need demonstrates what existing representations cannot meet.

### 4.5 Q5 — Protection and disclosure boundaries

**Finding.** Four distinct questions must be answered: whose policy applies, how a label is represented, what data it is bound to, and how access is enforced. A label does not perform those other functions by itself.

**IC evidence.** The archived ISM overview describes XML markings at document and portion levels. IC-EDH describes an XML enterprise header and expressly bounds its applicability; Joint/Coalition use is not universally required by that specification. Neither inspected overview establishes a blanket per-JSON-field mandate, a JSON inheritance/default rule, or permission to omit markings required by a particular deployment. Package retrieval failed, so this report makes no package-validation or detailed binding claim. [ism], [edh]

**NATO evidence.** The NISP entries distinguish ADatP-4774 label syntax from ADatP-4778 metadata binding. NISP v15 lists later 4778.2 REST, generic-packaging and sidecar profiles. The older NCIA technical note describes binding an HTTP body through header-carried metadata (Annex F §§1, 5; the start line and headers themselves are outside that coverage), separate-file sidecars (Annex H §5), and packages with distinct metadata (Annex G §5). Binding therefore need not mean inserting labels into every payload field. These historical examples do not establish which current profile a deployment must use or authorize coarser protection. [nato4774], [nato4778], [nisp-profiles], [ncia]

**Representation evidence.** SensorML's description schema permits typed security-constraint objects. That extension capacity is not an implemented policy engine or a defined binding for every nested observation field. [description-schema] The inspected OSH SensorML path skips incoming security/legal constraints, showing why a model's existence cannot substitute for round-trip verification. [osh-sml], [osh-sml-service]

**Disclosure case — interpretation.** A derived numerical result may be releasable while an input ID, producing host, operator or method detail is protected. Authorize the result and its disclosed context separately under the applicable policy. Follow the Guide's existing rule: a complete conformant resource, an explicitly valid permitted projection/profile, or denial/concealment. Do not return schema-invalid output, infer that releasable output authorizes lineage, or describe a partial view as complete evidence. Even acknowledging that hidden sources exist may itself require permission. [guide], [R040]

Protection also affects counts, filters, schema discovery, errors, publication and alternate encodings. If conversion or projection changes signed/bound content, output association/verification needs its own applicable treatment; successful source verification does not verify transformed bytes. Carrying an asserted label is not proof of its authority, and mapping an IC label to a NATO label is not automatically policy equivalence.

**Recommendation.** Preserve the existing deployment-policy boundary. Consider a specific label/binding adapter only after identifying its authoritative profile, applicability, consumer and validation expectations. Do not mandate field-level markings, strip them on assumption, build a cross-domain guard, or claim accreditation. Whole-resource authorization is simpler; finer-grained protection and detached metadata bring additional validation, lifecycle and disclosure combinations. No cost benchmark was performed.

### 4.6 Q6 — Gap classification, reconciliation and verification

**Finding.** Most useful next work is clarification and proof within the existing design, not adoption of another platform.

| Need / gap | Classification | Bounded disposition recommended |
|---|---|---|
| Source system, procedure descriptions, times, component meaning | Existing standard support | Implement and verify the actual fields/links first |
| Bind historical output to supplied method/input revision and distinguish submitter | Guide detail to sharpen, building on §4.10 | Clarify data retained and expected tests; do not invent missing upstream evidence |
| Exchange exact input-result set, execution and role assertions consistently | Public contract not established by inspected artifacts | Conditional consumer-driven profile/export discussion; PROV is a candidate semantic basis |
| Validate/round-trip SWE quality JSON | Known standards-artifact/interpretation seam | Retain exact vendor schemas; explicit supported interpretation and semantic tests |
| National/NATO label binding and detailed disclosure | Deployment-specific integration | Require applicable profile/policy before adapter choice; no blanket core obligation |
| Historical description retrieval and SystemEvent mappings | Unresolved public artifacts | Keep version/execution evidence distinct; do not invent standard routes/fields |
| Public lineage querying or newer binary representation | Separate capability decision | No expansion of current filter scope or automatic Part 5 adoption |

**Reconciliation with accepted research.** IDR-SRV-019 §§5–6, 10 and 13 already distinguish origin, lineage, quality and exposure; §§13.4/17 leave public exchange choices open. This supplement adds concrete current-artifact/peer checks and narrower consumer cases. It does not reinstate every earlier graph/assertion-envelope proposal. [R019]

IDR-SRV-021's description/history boundaries and IDR-SRV-022/023's quality preservation/schema-gap handling are corroborated, not replaced. IDR-SRV-024's explicit semantic binding matters more than renaming a numeric field. IDR-SRV-034's composite/batch distinction remains intact. IDR-SRV-040/041 and Guide §4.10 continue to separate policy, audit and production provenance. Supplements 058–060 supply sampling/filter/encoding boundaries, not a new lineage subsystem. [R021], [R022], [R023], [R024], [R034], [R040], [R041], [R058], [R059], [R060]

**Planning interpretation.** No Goal change is needed to recognize provenance already in scope. Guide §§4.3, 4.4, 4.7–4.8, 4.10–4.11, 6.1 and 8 are candidates for concise clarification of supported data and tests. A later optional public provenance profile would require a separate explicit scope decision. This report changes none of those documents.

## 5. Decision Analysis

| Option | Benefits | Costs / risks | Compatibility impact | Recommendation |
|---|---|---|---|---|
| Existing-standard implementation plus focused Guide/test clarifications | Makes current origin and quality support dependable; fits present goal | Does not promise complete cross-server ancestry exchange | Preserves CSAPI/SensorML/SWE contracts and named interpretation seams | Preferred next planning discussion |
| Optional, explicitly specified PROV-compatible export/link profile | Could serve consumers needing exact input/run/role exchange | Requires discovery, identity/version, serialization, disclosure and lifecycle agreement | Additional profile, not automatic base conformance | Conditional; do not select a format or endpoint now |
| External provenance/policy integration | Allows a deployment to supply specialized analysis or binding | Additional integration/availability/semantic-mapping obligations | Base API can remain unchanged; integration still needs proof | Valid when a concrete deployment needs it |
| New universal report/schema/graph/security-and-confidence subsystem | Centralizes many application preferences | Large scope, uncertain semantics, unjustified policy/aggregation claims | Risks replacing or obscuring the reference contract | Not recommended |
| Defer all clarification | No immediate planning work | Leaves known preservation/meaning failures untested | No new conformance claim | Defer optional integrations, not known in-scope verification |

## 6. Key Recommendations

1. **Use the existing representations first and make their limits explicit.** High priority. Preserve available source/method/input context without inventing it; no prerequisite beyond the current scope. Rationale: existing capabilities are substantial, while exact-input exchange remains a narrower open question.
2. **Clarify production versus ingestion/correction history and responsible roles.** High priority for Guide discussion. Retain applicable exact identities/version evidence and distinguish asserted roles from verified submitter identity. Preconditions: concrete supported input/output forms; no invented immutable public history route.
3. **Preserve quality meaning and verify the supported JSON form.** High priority. Rationale: semantic distinctions and the reproduced permissive-schema gap. Preconditions: documented interpretation consistent with the existing validation strategy, positive/negative and round-trip fixtures; no universal percentage conversion.
4. **Test shared/per-item context, conversion/publication and disclosure together.** High priority for implementation planning. Preconditions: supported representations and configured access rules. Rationale: field presence in a model is insufficient, and hidden lineage can leak indirectly.
5. **Defer public PROV/DQV profiles and policy-specific adapters unless a concrete exchange warrants them.** Conditional priority. Preconditions: identified consumer questions, example exchanges, applicable profiles and a later project-lead decision. No new implementation dependency or governance document is proposed.

## 7. Implementation Implications and Estimates

### 7.1 Implications

These are inputs to later Guide discussion, not accepted implementation requirements or changes to the Rust stack.

- **API/representation:** retain standard associations and process/value descriptions. Any future additional exchange must identify discovery, relationship meanings, representation and unsupported cases explicitly; do not add ad hoc top-level provenance fields to claim interoperability.
- **Persistence:** distinguish reusable descriptions, the revision/context actually associated with a result, and server mutation evidence. Preserve remote references and known limitations. Do not require every raw input to be local or retained forever.
- **Rust implementation:** typed parsing/serialization and existing validation layers must not silently discard supported quality, role, version or protection information. An ontology's graph-shaped concepts do not dictate a graph store or RDF library; no new dependency was selected or benchmarked.
- **Security:** validate output as well as input; authorize referenced context; ensure alternate encodings/events do not bypass the same protection boundary. Unsupported transformations must be explicit instead of pretending to preserve labels or signatures.

**Prospective checks — not executed Glaux tests:**

| Check | Independently expected outcome |
|---|---|
| Direct observation, link traversal and genuinely absent history | Recover stated producer/method context; do not infer extra ancestry, creator or certainty |
| Two input results → derived output with recorded model revision | Preserve exact supplied identities; a shared sampling link or process diagram alone fails the exact-input question |
| Procedure/configuration change and corrected output | Earlier output retains its recorded context; server correction is distinguishable from original production |
| Identity score plus property uncertainty and unknown value | Preserve separate subjects, scales/units and meaning; no silent conversion, zero-fill or fusion |
| Quality structure positive/negative fixtures | Supported quality is meaningfully validated; malformed unknown data cannot acquire “supported quality” status through schema permissiveness |
| Mixed-context batch | Preserve individual identities and exceptions; common metadata applies only where true |
| JSON, supported SWE encodings, selected Part 3 publication | Reconstruct the same supported meaning or explicitly reject/limit an unsupported conversion; Part 5 only if later selected |
| Authorized versus unauthorized lineage consumer | No hidden identifiers, counts, filter effects, schema details, event contents or error leaks; no false completeness claim |
| Source binding followed by conversion/projection | Do not advertise verification of transformed output merely because source verification succeeded |
| Large or unavailable remote input set | Bounded processing and honest availability/retention status; no unrestricted dereferencing or invented input list |

Expected outcomes derive from meaning and visibility, not merely round-tripping through the same potentially lossy implementation. Peer differential testing is useful only after fixing an independent expected result; peer behavior is not the normative oracle.

### 7.2 Effort/Complexity Estimate

| Work item | Relative complexity | Estimate / assumptions |
|---|---|---|
| Focused Guide clarification | Low | Fits a later drafting iteration; no implementation time estimate |
| Revision/context preservation and semantic fixtures | Moderate | Builds on planned storage/validation; actual effort depends on supported forms |
| Cross-representation and protected-lineage verification | Moderate–high | Increases with representation and policy combinations; no benchmark or hours estimate |
| Optional public provenance profile | Not reliably estimable yet | Identity, discovery, payload and consumer contract remain unselected |
| IC/NATO adapter | Not estimable without applicable profile | No default commitment; external policy and validation expectations required |

## 8. Risks, Constraints, and Open Questions

### 8.1 Risks and Constraints

- A current URL can be mistaken for an exact historical revision; absent evidence must remain absent.
- Attribution/delegation assertions can be mistaken for verified accountability, or authentication for truth.
- Permissive schemas and lossy peer bindings can produce plausible but incomplete exchanges.
- Shared metadata can incorrectly flatten per-item differences; retained links may later be unavailable or restricted.
- Quality metrics can lose their subject, method or uncertainty interpretation during normalization.
- Provenance can disclose more sensitive information than the result itself; transformation can invalidate binding evidence.
- Accessible security overviews and historical binding examples cannot establish a deployment's actual obligations.

### 8.2 Open Questions

1. **Which consumer requires exact input/run/role exchange beyond existing representations?** A later sanitized example could justify a bounded profile; it is not a blocker for accepting this study or continuing the Guide.
2. **Which supported quality JSON interpretation will Glaux publish and test?** Carry the established artifact seam into Guide detail; do not silently declare a peer convention normative.
3. **How will a future public profile identify immutable inputs and incomplete/withheld views?** Not selected here; avoid inventing a standard history route while #149/#174/#201 remain unresolved.
4. **Does a deployment require a particular IC/NATO profile or granularity?** Obtain authoritative applicable material only if implementing that integration. No presumption in either direction.
5. **Will Part 5 be selected, and what can its binding preserve?** Existing pending discussion remains separate; this study adds semantic test cases, not an encoding decision.

## 9. Validation Against Plan Success Criteria

| Plan success criterion | Status | Evidence |
|---|---|---|
| Q1–Q6 answered with limits and consequences | Met | §§4.1–4.6, 8 |
| Standards roles, editions and alignments clear | Met | §§3, 4.1 |
| General cases, no verbatim/application-attributed requirements | Met | §§2, 4.4 |
| Production, server history, roles and unavailable evidence distinct | Met | §4.2, scenario/test tables |
| Quality meanings preserved; no universal conversion/fusion | Met | §4.3 |
| Grouping, revisions, references and scale assessed without new report/catalog | Met | §4.4 |
| Bounded label/binding/policy and disclosure assessment | Met within explicit access limits | §4.5 |
| Existing support, design detail, exchange gaps and unknowns separated | Met | §§3.3, 4.6; pinned peer paths; Appendix A |
| Practical alternatives, incremental implications and planning candidates | Met | §§5–8 |
| Template, reconciliation and separate synthesis/planning handoff | Met | §4.6, §10; completion checklist |

## 10. Next Steps and Handoff

1. **Report accepted September 18, 2026.** Owner: Glaux Project Lead. The `proceed` following publication records acceptance for downstream use, not implementation adoption.
2. **Separate synthesis addendum prepared.** Owner: Glaux research workflow under that authorization. [Addendum D](final-idr-research-report.md#addendum-d-provenance-and-related-metadata-interoperability) integrates the findings while preserving the original synthesis and Addenda A–C; the new addendum is prepared for review, not recorded as already accepted.
3. **Next, discuss provenance and the pending Part 5 implications for the Goal/Guide, then resume Guide drafting pass 2 after any agreed changes.** Owners: project lead and drafting workflow. Due: next user-directed discussion. Proposed clarifications can use the existing documents; no separate requirements framework or new approval phrase is needed.

The original research/report iteration did not edit the Goal, Guide, Roadmap, final synthesis or server code. The acceptance iteration adds the separate synthesis addendum and updates research tracking only; the Goal, Guide, Roadmap and server code remain unchanged.

## 11. References

Reference labels resolve directly to authoritative pages, pinned source files or existing project documents. Mutable public pages are dated in §3; repository artifacts use full commit pins.

- Standards: [CSAPI Part 1][part1], [Part 2][part2], [SensorML][sml], [SWE Common][swe], [OMS][oms], [OGC model/encoding overview][oms-overview], [2017 SOSA/SSN alignment][ssn], [2026 Working Draft][ssn-draft].
- Provenance/quality: [PROV-DM][prov-dm], [PROV-O][prov-o], [DQV][dqv], [NIST likelihood][likelihood], [NIST confidence][confidence], [VIM uncertainty][uncertainty], [coverage][coverage], [precision][precision], [resolution][resolution], [traceability][traceability], [GUM][gum], [ICD 203][icd203].
- Artifacts: [Observation][obs-schema], [DataStream][stream-schema], [DescribedObject][description-schema], [AbstractProcess][process-schema], [SimpleProcess][simple-schema], [AggregateProcess][aggregate-schema], [ResponsibleParty][party-schema], [Quantity][quantity-schema], [AbstractSimpleComponent][simple-component-schema].
- Practice: pinned peer paths in §3.2, [Testbed-20 report][testbed20], [Testbed-21 initiative][testbed21], and [history register][history].
- Protection: [ISM][ism], [IC-EDH][edh], [4774][nato4774], [4778][nato4778], [binding-profile index][nisp-profiles], [historical NCIA technical note][ncia].
- Project: [plan][plan], [overall plan][overall], [Goal][goal], [Guide][guide], and prior reports linked in §4.6.

## 12. Appendices

### Appendix A — Executed schema inspection and probe

**Observed on September 18, 2026:** five published `https://schemas.opengis.net/sweCommon/3.0/json/` file contents matched their pinned repository counterparts exactly as decoded UTF-8 text. SHA-256 values below hash that retrieved text, not HTTP framing or a release archive.

| File | SHA-256 |
|---|---|
| `Quantity.json` | `dc23d3496ae02a6d1de756d441aa12f248e74a1ff3feafda826640af04115b52` |
| `AbstractSimpleComponent.json` | `ed64ba08d5912ed73f1e398433cec9dde802594dcdc870570c1d1b406dc769f6` |
| `AbstractDataComponent.json` | `468e11b9fff5a712428628a4a373f7f550ff89a59f30e5af10017fb22e58e661` |
| `AbstractSweIdentifiable.json` | `22130e687acfa3efbddacc75b0107c99db0e72988acef33163457af767e284d9` |
| `basicTypes.json` | `13d915858e4f3ca4259c8c8c77a0eb4e9546f7678d0eed0584d020f102cdf2ff` |

The probe used installed PowerShell 7.6.6, `Test-Json` from `Microsoft.PowerShell.Utility` 7.0.0.0. It bundled these resources in memory, rebasing both document-relative and fragment-only references; it did not edit source files, install a validator or change schema constraints. An initial harness attempt omitted fragment rebasing; that invalid harness was corrected before the reported comparison.

| Input | Result | Interpretation |
|---|---|---|
| Quantity with temperature definition/label, `uom: {code: "Cel"}`, value `20.0` | `True` | Positive structure control |
| Same, plus `quality: "not-a-quality-component"` | `True` | Undeclared member is not meaningfully validated |
| Same baseline, but `uom: 17` | `False` | Negative control confirms active structural validation |

This is not proof that the arbitrary quality string conforms to SWE, that any proposed extension is interoperable, or that a production Rust validator behaves identically. The source inspection explains the limited result: the relevant inheritance chain does not declare `quality` and does not close the containing object against that member.

Reproduction using the pinned sources and already-installed PowerShell:

```powershell
$probePin = '8e03b236a049849f2ccc24b4fd9fdce5ff69bed2'
$probeDefs = [ordered]@{}
foreach ($probeName in @('Quantity', 'AbstractSimpleComponent',
    'AbstractDataComponent', 'AbstractSweIdentifiable', 'basicTypes')) {
    $probeUrl = 'https://raw.githubusercontent.com/opengeospatial/' +
        'ogcapi-connected-systems/' + $probePin +
        '/swecommon/schemas/json/' + $probeName + '.json'
    $probeText = (Invoke-WebRequest -UseBasicParsing -Uri $probeUrl).Content
    $probePrefix = '"$ref":"#/$defs/' + $probeName
    $probeText = [regex]::Replace($probeText, '"\$ref"\s*:\s*"#',
        [System.Text.RegularExpressions.MatchEvaluator]{param($m) $probePrefix})
    $probeText = [regex]::Replace($probeText,
        '"\$ref"\s*:\s*"(?:\./)?([^"/#]+)\.json(?:#([^"]*))?"',
        {param($m) '"$ref":"#/$defs/' + $m.Groups[1].Value + $m.Groups[2].Value + '"'})
    $probeDefs[$probeName] = ConvertFrom-Json -AsHashtable -InputObject $probeText
}
$probeSchema = ConvertTo-Json -Depth 100 -InputObject @{
    '$schema' = 'https://json-schema.org/draft/2020-12/schema'
    '$ref' = '#/$defs/Quantity'; '$defs' = $probeDefs
}
$probeValue = '{"type":"Quantity","definition":"https://example.org/property/temperature","label":"Temperature","uom":{"code":"Cel"},"value":20.0}'
Test-Json -Json $probeValue -Schema $probeSchema
Test-Json -Json ($probeValue.TrimEnd('}') + ',"quality":"not-a-quality-component"}') -Schema $probeSchema
Test-Json -Json ($probeValue.Replace('"uom":{"code":"Cel"}', '"uom":17')) -Schema $probeSchema -ErrorAction SilentlyContinue
```

### Appendix B — Evidence versus future tests

Executed: primary-source retrieval, pinned file inspection, official-history/ref checks, five-file content/hash comparison, and the three-input schema probe. Inspected but not executed: CS-Go/OSH handlers and tests, OGC demonstration reports. Proposed only: all Glaux semantic, access-control, integration, performance and cross-representation checks in §7. No interoperability certification, benchmark, policy validation or operational verification is claimed.

## Report Completion Checklist

- [x] Topic ID matches overall research plan index
- [x] Topic research plan is linked and aligned
- [x] Core research questions are covered or explicitly unresolved
- [x] Findings are evidence-backed with reproducible references
- [x] Normative and informative evidence are classified and not conflated
- [x] Mutable sources identify a version, release, tag, commit, or dated retrieval
- [x] Controlled, inaccessible, missing, or ambiguous evidence limitations are explicit
- [x] Source-backed findings, analyst inference, and project recommendations are distinguishable
- [x] Conflicts with accepted prior reports are reconciled or explicitly escalated
- [x] Executive summary is independently readable by the project lead, implementers, and later AI agents
- [x] Recommendations are explicit and actionable
- [x] Risks and open questions are documented
- [x] Success criteria validation is complete
- [x] Plan-owner acceptance and acceptance date are recorded before the topic is treated as complete downstream
- [x] Next steps are assigned

[plan]: ../IDR%20Plans/idr-srv-061-provenance-and-related-metadata-interoperability-study.md
[overall]: ../IDR%20Plans/overall-idr-research-plan.md
[history]: ../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md
[goal]: ../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md
[guide]: ../../../../../Plans/glaux-server/glaux-server-implementation-guide.md
[R019]: idr-srv-019-provenance-lineage-quality-and-trust-metadata-model-report.md
[R021]: idr-srv-021-sensorml-representation-strategy-report.md
[R022]: idr-srv-022-swe-common-data-component-strategy-report.md
[R023]: idr-srv-023-schema-and-encoding-validation-strategy-report.md
[R024]: idr-srv-024-units-observed-properties-and-semantic-binding-strategy-report.md
[R034]: idr-srv-034-datastream-observation-and-status-update-semantics-report.md
[R040]: idr-srv-040-policy-releasability-and-cross-boundary-access-constraints-report.md
[R041]: idr-srv-041-audit-logging-and-accountability-strategy-report.md
[R058]: idr-srv-058-draft-csapi-part-4-sampling-features-study-report.md
[R059]: idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study-report.md
[R060]: idr-srv-060-csapi-part-5-protobuf-first-implementation-study-report.md
[part1]: https://docs.ogc.org/is/23-001/23-001.html
[part2]: https://docs.ogc.org/is/23-002/23-002.html
[sml]: https://docs.ogc.org/is/23-000/23-000.html
[swe]: https://docs.ogc.org/is/24-014/24-014.html
[oms]: https://docs.ogc.org/as/20-082r4/20-082r4.html
[oms-overview]: https://www.ogc.org/standards/om/
[ssn]: https://www.w3.org/TR/2017/REC-vocab-ssn-20171019/#PROV_Alignment
[ssn-draft]: https://www.w3.org/TR/2026/WD-vocab-ssn-2023-20260914/
[prov-dm]: https://www.w3.org/TR/2013/REC-prov-dm-20130430/
[prov-o]: https://www.w3.org/TR/2013/REC-prov-o-20130430/
[dqv]: https://www.w3.org/TR/2016/NOTE-vocab-dqv-20161215/
[likelihood]: https://www.itl.nist.gov/div898/handbook/eda/section3/eda3652.htm
[confidence]: https://www.itl.nist.gov/div898/handbook/prc/section1/prc14.htm
[uncertainty]: https://jcgm.bipm.org/vim/en/2.26.html
[coverage]: https://jcgm.bipm.org/vim/en/2.37.html
[precision]: https://jcgm.bipm.org/vim/en/2.15.html
[resolution]: https://jcgm.bipm.org/vim/en/4.14.html
[traceability]: https://jcgm.bipm.org/vim/en/2.41.html
[gum]: https://www.bipm.org/documents/20126/2071204/JCGM_100_2008_E.pdf/cb0ef43f-baa5-11cf-3f85-4dcd86f77bd6
[icd203]: https://www.dni.gov/files/documents/ICD/ICD-203.pdf
[ism]: https://archive.dni.gov/index.php/who-we-are/organizations/ic-cio/ic-technical-specifications/information-security-marking-metadata
[edh]: https://archive.dni.gov/index.php/who-we-are/organizations/ic-cio/ic-technical-specifications/ic-enterprise-data-header
[nato4774]: https://nisp.nw3.dk/standard/nato-adatp-4774-ed.a-v1.html
[nato4778]: https://nisp.nw3.dk/standard/nato-adatp-4778-ed.a-v1.html
[nisp-profiles]: https://archive.nisp.nw3.dk/nisp-15.0/volume2/ch04.html
[ncia]: https://storage.nisp.nw3.dk/TN-1491_Edition2-Binding_Profiles_v1.0-Signed.pdf
[testbed20]: https://docs.ogc.org/per/24-036.html
[testbed21]: https://www.ogc.org/initiatives/ogc-testbed-21/
[ogc-tree]: https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2
[obs-schema]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/schemas/json/observation.json
[stream-schema]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/schemas/json/dataStream.json
[description-schema]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/sensorml/schemas/json/DescribedObject.json
[process-schema]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/sensorml/schemas/json/AbstractProcess.json
[simple-schema]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/sensorml/schemas/json/SimpleProcess.json
[aggregate-schema]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/sensorml/schemas/json/AggregateProcess.json
[party-schema]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/sensorml/schemas/json/ResponsibleParty.json
[quantity-schema]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/swecommon/schemas/json/Quantity.json
[simple-component-schema]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/swecommon/schemas/json/AbstractSimpleComponent.json
[csgo-obs]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/model/domains/observation.go
[csgo-format]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/model/formaters/json_formatters/observation_json.go
[csgo-tests]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/model/formaters/json_formatters/observation_json_test.go
[csgo-procedure]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/model/domains/procedure.go
[csgo-sml]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/model/formaters/sensorml_formatters/procedure_sensorml.go
[csgo-stream]: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/b1fd2e0e9bd69e222d05258d659a842ca24502cb/internal/model/domains/datastream.go
[osh-obs]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/obs/ObsBindingOmJson.java
[osh-interface]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/sensorhub-core/src/main/java/org/sensorhub/api/data/IObsData.java
[osh-stream]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/obs/DataStreamBindingJson.java
[osh-quality]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/lib-ogc/swe-common-core/src/test/java/org/vast/swe/test/TestSweJsonBindingsV20.java#L210
[osh-quality-fixture]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/lib-ogc/swe-common-core/src/test/resources/org/vast/swe/test/examples_v20/spec/quality.xml
[osh-dynamic-quality]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/lib-ogc/swe-common-core/src/test/resources/org/vast/swe/test/examples_v20/spec/datastream_with_quality.xml
[osh-sml]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/lib-ogc/sensorml-core/src/main/java/org/vast/sensorML/SMLJsonBindings.java#L372
[osh-sml-service]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/sensorml/SmlProcessBindingSmlJson.java#L54
[osh-batch]: https://github.com/opensensorhub/osh-core/blob/9a43f9ec42315e5a22e5e5d90a4ba79eef9cea08/sensorhub-service-consys/src/test/java/org/sensorhub/impl/service/consys/TestObservations.java#L73

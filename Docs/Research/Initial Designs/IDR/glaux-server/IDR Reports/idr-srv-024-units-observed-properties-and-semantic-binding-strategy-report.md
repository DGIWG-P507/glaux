# Section 024: Units, Observed Properties, and Semantic Binding Strategy - Research Report

**Topic ID:** IDR-SRV-024<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-024 Units, Observed Properties, and Semantic Binding Strategy](../IDR%20Plans/idr-srv-024-units-observed-properties-and-semantic-binding-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core questions, all detailed-question groups, all 6 methodology phases, and all 10 success criteria<br>
**Methodology Used:** Authority-ranked extraction from approved CSAPI Parts 1 and 2, SensorML 3.0, SWE Common 3.0, SSN/SOSA, OGC Naming Authority material, UCUM, QUDT, SKOS, accepted AEP and prior-IDR findings, implementation/interoperability evidence, and bounded official issue-history refresh; followed by concept, resource-family, source-preservation, normalization, validation, query, DDIL, security, and test mapping<br>
**Research Time:** Approximately 13 hours of AI-assisted execution on September 14, 2026<br>
**Official Standards Source Pin:** [`opengeospatial/ogcapi-connected-systems` `v1.0.0` at `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)<br>
**Mutable Upstream Recheck:** Official `master` remained at [`3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f) on September 14, 2026; issues [#178](https://github.com/opengeospatial/ogcapi-connected-systems/issues/178) and [#179](https://github.com/opengeospatial/ogcapi-connected-systems/issues/179) remained unresolved and were routed explicitly to this topic<br>
**Shared Register Baseline:** [OGC API - Connected Systems upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md), Version 1.10<br>
**Controlled AEP Source:** `AC/224(JCGISR)D(2026)0005`, April 27, 2026, SHA-256 `56dc757b6e677b3584e3152a957849f21a24b22854f562613ff283a8b599da8c`; used only through accepted project findings and not redistributed<br>
**Document Purpose:** Establish a decision-usable semantic and unit baseline for the Rust Glaux reference server without designing the database, choosing a public ontology profile, finalizing ingestion or command lifecycles, implementing draft Part 3, or implementing the server<br>
**Author:** OpenAI Codex<br>
**Accepted By:** TBD until Glaux Project Lead acceptance<br>
**Acceptance Date:** TBD until accepted<br>
**Date:** September 14, 2026<br>
**Last Updated:** September 14, 2026

---

## Reading Guide and Evidence Labels

| Label | Meaning |
|---|---|
| **N** | Normative or standards-derived finding from an approved applicable source |
| **A** | Project-controlling AEP/STANAG adoption or operational-context finding carried from an accepted report |
| **P** | Glaux project decision or recommendation proposed for acceptance here |
| **I** | Informative implementation, test, interoperability, or community evidence |
| **D** | Official draft, post-publication direction, or candidate vocabulary that is not an approved Glaux requirement |
| **X** | Published ambiguity, artifact conflict, evidence gap, or unresolved external decision |

An identifier, label, unit code, vocabulary mapping, and authorization decision are different facts. “Equivalent dimension” does not mean “same property,” “same property” does not mean “safe conversion,” and a semantic constraint does not grant permission to issue a command.

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Semantic Binding Extraction Methodology
5. Standards and Vocabulary Source Inventory
6. Semantic Concept Taxonomy
7. Unit Strategy Findings
8. Observed-Property Strategy Findings
9. Controlled-Property and Command-Parameter Semantic Findings
10. Resource-Family Semantic Mapping
11. SensorML and SWE Common Semantic Consistency Findings
12. Validation, Normalization, Vocabulary-Cache, Query, and DDIL Findings
13. Persistence, Metadata Storage, and Configuration Implications
14. Security, Policy, and Releasability Implications
15. Fixture, Golden-File, Conformance, and Interoperability Test Implications
16. Downstream Topic Handoff Matrix
17. Recommendations
18. Risks, Constraints, and Open Questions
19. Validation Against Plan Success Criteria
20. References

---

## 1. Executive Summary

Glaux should implement semantic binding as a **versioned, provenance-bearing graph beside exact source representations**, not as string cleanup and not as an RDF-only storage mandate. Every accepted data value needs an intelligible property definition; each semantic reference needs a well-defined resolution method; and every unit-bearing numeric component needs a valid unit declaration. The server should preserve what the publisher supplied, parse it without loss, bind it to a canonical concept or unit when evidence permits, materialize only approved query relationships, and generate each CSAPI/SensorML/SWE representation from those layers. **[N/P]**

The recommended unit baseline is deliberately constrained. SWE Common 3.0 recommends a UCUM code whenever the unit can be expressed in UCUM and otherwise permits a semantic `href`; when both are supplied they must denote the same unit. Glaux should validate the case-sensitive UCUM 2.1 baseline incorporated by SWE Common, preserve the full submitted `UnitReference`, and record a resolved unit identity, dimension, quantity kind, conversion metadata, and profile status separately. AEP-4789’s accepted project baseline prefers SI units unless an operationally necessary exception is specified. UCUM is therefore the primary machine-readable unit syntax, QUDT is a useful pinned enrichment source, and neither free-text symbols nor mutable online vocabulary responses are authoritative identity. **[N/A/P]**

Property roles are edges in context, not mutually exclusive classes. A CSAPI Property can be observable, controllable, or an asserted System property—and the same Property can have several roles. Glaux should bind each SWE leaf component to a property concept and an explicit role, while keeping feature attributes, characteristics, capabilities, status facts, command parameters, target selectors, and controlled properties distinct. A `DataStream` may carry several observed properties and a `ControlStream` several controlled properties. The server must never infer a property solely from a human label, field name, unit, dimension, or value position. **[N/P]**

Semantic query behavior should be conservative and explainable. Exact local ID or unique-identifier matching is the baseline. The CSAPI `baseProperty` derivation graph supports bounded, cycle-safe descendant expansion where the published filter semantics require it. SKOS or project mappings may support opt-in expansion, but `closeMatch`, shared labels, redirects, and dimensional compatibility must not become silent equality. Policy filtering applies before result counts, links, generated descriptions, vocabulary responses, and diagnostics so semantic discovery cannot become a capability oracle. **[N/P]**

Two published-contract seams remain open. Part 2 prose models generated `observedProperties`/`controlledProperties` as URI lists, while its tagged JSON schemas and examples use small semantic-description objects. Issue #178 has not resolved the source of generated DataStream summaries; issue #179 has not resolved how resource-specific property filters derive capabilities. Glaux should use an internal identity graph and a pinned 1.0 representation adapter, record provenance for every derived property assertion, and isolate the interim query derivation policy behind a versioned compatibility profile. **[X/P]**

No mandatory domain-property ontology is selected here. OGC definitions are authoritative for OGC-owned names, UCUM is recommended for unit codes, and QUDT/SKOS/RDF can enrich offline resolution and mappings. Mission or local vocabularies remain valid when they provide stable identifiers, definitions, authority, version, and an installable offline package. The resulting architecture supports DDIL operation, source fidelity, query, interoperability, and later policy enforcement without prematurely requiring a triple store, live dereferencing, or one universal ontology.

---

## 2. Scope and Plan Alignment

This report completes the bounded research authorized for `IDR-SRV-024`. It covers units, property roles, semantic references, vocabulary authority, source preservation, normalization, validation additions, query/index implications, offline resolution, disclosure risks, fixtures, interoperability, and downstream handoffs.

It does **not** select database tables or products; implement a vocabulary service; finalize ingestion, Observation, Command, feasibility, authorization, or retention workflows; create an AEP vocabulary; require RDF/OWL serialization; adopt the September 2026 SSN working draft; implement draft CSAPI Part 3; or implement the Glaux server. Those decisions remain with their routed topics.

### 2.1 Research Question Coverage Matrix

| Plan question | Short form | Status | Evidence location |
|---|---|---|---|
| Q1 | Required unit, property, and semantic concepts | Complete | Sections 5-9 |
| Q2 | Authority of normative, inherited, profile, implementation, optional, and unresolved concepts | Complete | Sections 3, 5, 6 |
| Q3 | Representation, validation, preservation, and normalization | Complete | Sections 7, 11-13 |
| Q4 | Discovery, query, ingestion, status, command, and interoperability support | Complete, with external ambiguities retained | Sections 8-12, 15, 18 |
| Q5 | Persistence, metadata, dynamic-data, command, security, fixture, and conformance handoffs | Complete | Sections 13-16 |

### 2.2 Accepted Baselines Consumed

- `IDR-SRV-015` supplies the encoding-neutral canonical resource graph.
- `IDR-SRV-016` requires typed identities, preservation of submitted URI lexical form, and purpose-specific comparison keys; it prohibits unsafe global URI normalization and identity merging from mere UID resemblance.
- `IDR-SRV-017` supplies explicit, directional, provenance-bearing relationship facts.
- `IDR-SRV-018` separates time axes and allows conversion only when frame, epoch, unit, and precision are defined.
- `IDR-SRV-019` supplies provenance, transformation, quality, uncertainty, and trust evidence.
- `IDR-SRV-020` separates operational status, availability, events, and command states.
- `IDR-SRV-021` supplies exact-source, parsed, canonical, generated, and evidence layers for SensorML and preserves `DerivedProperty.qualifiers` despite the CSAPI table gap.
- `IDR-SRV-022` supplies immutable SWE contracts, exact component paths and order, full component preservation, nil/optional/invalid distinctions, and capability-gated codecs.
- `IDR-SRV-023` supplies the evidence-producing validation pipeline, trusted contract selection, offline reference registry, strict public writes versus privileged quarantine, and safe stable diagnostics.

No conclusion below weakens those accepted decisions.

---

## 3. Evidence Base and Authority Classification

### 3.1 Primary Sources Reviewed

| Source | Version/status | Authority | Stable anchors used | Access date | Limitation |
|---|---|---|---|---|---|
| [OGC API - Connected Systems Part 1](https://docs.ogc.org/is/23-001/23-001.html) | OGC 23-001, Version 1.0, approved | Normative | Property resources; Systems, Procedures, Deployments, Sampling Features; filtering; Annex A | 2026-09-14 | Some conceptual/OAS/example mappings differ |
| [OGC API - Connected Systems Part 2](https://docs.ogc.org/is/23-002/23-002.html) | OGC 23-002, Version 1.0, approved | Normative | DataStreams, Observations, ControlStreams, Commands; filtering; Annex A | 2026-09-14 | Property-summary prose and JSON artifacts conflict |
| [Tagged CSAPI source](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2) | `v1.0.0`, commit `8e03b236` | Normative-source reproduction | `part1`, `part2`, `api`, schemas, examples, abstract tests | 2026-09-14 | Repository history does not amend the published standard |
| [SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html) | OGC 23-000, approved | Normative | Core model; identifiers/classifiers; inputs, outputs, parameters; ObservableProperty; DerivedProperty | 2026-09-14 | General model intentionally relies on external semantics/profiles |
| [SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html) | OGC 24-014, approved | Normative | Core Requirements 7-8; simple/aggregate components; UnitReference; encoding | 2026-09-14 | Does not choose a universal property vocabulary or security policy |
| [SSN/SOSA 2017 Recommendation](https://www.w3.org/TR/2017/REC-vocab-ssn-20171019/) | W3C Recommendation / OGC standard | Normative vocabulary baseline referenced by CSAPI | Property, ObservableProperty, ActuatableProperty, observation/actuation relationships | 2026-09-14 | CSAPI intentionally diverges in cardinality and resource representation |
| Controlled AEP package | `AC/224(JCGISR)D(2026)0005`, 2026-04-27 | Project-controlling adoption context | Accepted IDR-SRV-001/003 findings | 2026-09-14 | Controlled source; no text or artifact redistributed |
| [OGC Naming Authority](https://www.ogc.org/ogcna/) | OGC policies 09-048r6 and 09-046r6 | Authoritative for OGC names | URI/URN patterns, registers, aliases, version/code structure | 2026-09-14 | Not a universal phenomenon vocabulary |
| [UCUM](https://ucum.org/) | SWE normative reference: 2.1 (2017); current site: 2.2 (2024) | Normative unit-system reference through SWE; later version informative | syntax, case-sensitive variant, canonical terms, conversions | 2026-09-14 | Version 2.2 must not silently replace SWE’s pin |
| [SKOS Reference](https://www.w3.org/TR/skos-reference/) | W3C Recommendation, 2009 | Normative vocabulary-model reference | concepts, schemes, labels, notations, semantic and mapping relations | 2026-09-14 | Knowledge-organization relations are not automatic logical equivalence |

### 3.2 Candidate and Supporting Sources

| Source | Version/status | Evidence class | Use | Limitation |
|---|---|---|---|---|
| [QUDT](https://qudt.org/) | Catalog 3.5.1, generated 2026-08-29 | Candidate vocabulary **[D]** | Unit identity, quantity kind, dimension vector, labels, conversion enrichment | Not mandated by CSAPI/SensorML/SWE; unversioned endpoints are mutable |
| [OGC Definitions Server](https://defs.opengis.net/) | Live service checked 2026-09-14 | Authoritative service for OGC-owned concepts | Resolution and representation of registered OGC definitions | Online availability is unsuitable as a runtime dependency |
| [RDF 1.1 Concepts](https://www.w3.org/TR/rdf11-concepts/) and [OWL 2 Overview](https://www.w3.org/TR/owl2-overview/) | W3C Recommendations | Optional modeling technology | Vocabulary import, explicit relations, machine exchange | Neither is required as Glaux’s public wire format or storage engine |
| [SSN 2023 revision](https://www.w3.org/TR/vocab-ssn-2023/) | W3C Working Draft dated 2026-09-14 | Future compatibility signal **[D]** | Monitors movement toward generic `sosa:Property` and newer relationships | Not the approved CSAPI 1.0 baseline; must not trigger automatic rewrites |
| [Upstream issue #162](https://github.com/opengeospatial/ogcapi-connected-systems/issues/162) | Open | Post-publication gap **[D/X]** | Confirms `DerivedProperty.qualifiers` schema/table mismatch | No published Part 1 correction |
| [Upstream issue #178](https://github.com/opengeospatial/ogcapi-connected-systems/issues/178) | Open | Unresolved **[X]** | Generated/receivable DataStream fields and result-schema description | No adopted derivation algorithm |
| [Upstream issue #179](https://github.com/opengeospatial/ogcapi-connected-systems/issues/179) | Open, updated 2026-09-03 | Unresolved **[X]** | Resource-specific property-filter mapping | Working discussion is not an approved 1.0 rule |
| Accepted IDR-SRV-014A through 014G reports | Accepted implementation studies | Informative **[I]** | Parser-loss, inconsistent mapping, permissive identifiers, incomplete filters, client expectations | Implementation behavior cannot override approved standards |
| Accepted IDR-SRV-015 through 023 reports | Accepted project baselines | Project-controlling | Canonical graph, identity, provenance, SensorML/SWE, validation | Handoffs remain bounded by each report |

### 3.3 Evidence Hierarchy and Confidence

Obligations were taken first from approved standards and their incorporated artifacts, then from the project-controlled AEP adoption baseline, then from accepted Glaux reports. Official issue history explains defects or likely evolution but does not amend Version 1.0. Candidate vocabularies and implementations inform feasibility only.

Confidence is **high** for the layered representation model, UnitReference rules, property-resource roles, SWE definition requirements, and conservative mapping policy. Confidence is **moderate** for generated property-summary and cross-resource filter behavior because Parts 1/2 leave material derivation questions open. Those areas receive explicit adapters, provenance, versioning, and tests rather than invented claims of conformance.

---

## 4. Semantic Binding Extraction Methodology

The extraction unit was not a word such as “temperature.” It was a tuple:

`(subject identity, component path, semantic role, submitted identifier, expanded identifier, source layer, vocabulary snapshot, provenance, validation state, policy label)`.

For units it additionally captured exact `UnitReference`, UCUM version/code, semantic unit URI, dimension, quantity kind, scale/offset/function, SI/profile status, and conversion restrictions. This prevents a label, unit, property, field role, and transformation from being collapsed into one string.

Each extracted item was classified across five views:

1. **Exact source:** bytes and lexical values as received.
2. **Parsed source model:** standards-shaped fields and component paths without semantic rewriting.
3. **Canonical semantic graph:** typed concepts, assertions, roles, units, and mappings with provenance.
4. **Materialized validation/query view:** purpose-specific keys and bounded closures produced from a named profile and vocabulary package.
5. **Generated representation:** pinned CSAPI, SensorML, SWE, or optional extension output.

Normative requirements were compared across prose, schema, examples, and ATS. Conflicts were retained as separate claims. Accepted implementation studies were then used to identify practical loss and interoperability hazards. Finally, each finding was mapped to resource family, validation stage, offline behavior, disclosure impact, fixture, and downstream owner.

### 4.1 Classification Values

| Value | Meaning in this report |
|---|---|
| Normative | Required or defined by an approved applicable standard |
| Inherited | Required through an incorporated model, schema, or accepted Glaux baseline |
| Profile-specific | Required only when a named AEP, deployment, tenant, or conformance profile applies |
| Vocabulary-specific | True only under a named concept scheme and version |
| Implementation-support | Internal mechanism supporting standards behavior without becoming API law |
| Implementation-specific | Observed precedent, not a Glaux or standards requirement |
| Optional | Supported only when advertised or configured |
| Local | Authority-controlled mission or deployment term with explicit registration |
| Deferred | Mechanism assigned to a later topic |
| Unresolved | Evidence does not support one normative answer |

---

## 5. Standards and Vocabulary Source Inventory

| Source | What it governs | Glaux disposition |
|---|---|---|
| CSAPI Part 1 | Canonical Property resources; property identity and derivation fields; property filters across feature resources | Implement the published resource/API contract; isolate ambiguous derived filtering behind a compatibility profile |
| CSAPI Part 2 | Semantic summaries for DataStreams/ControlStreams and property filters on dynamic resources | Use canonical property identities internally and a Version 1.0 response adapter for the published object-shaped JSON artifact |
| SensorML 3.0 | Semantic descriptors for identifiers, classifiers, inputs, outputs, parameters, capabilities, characteristics, ObservableProperty, DerivedProperty | Preserve the full source model; bind component semantics without treating placement or label as sufficient proof of role |
| SWE Common 3.0 | `definition` on data components, `codeSpace`, units, value/constraint/encoding structures | Treat leaf `definition` and numeric unit as executable contract metadata; retain aggregate definitions separately |
| SSN/SOSA 2017 | Domain concepts and observation/actuation vocabulary referenced by CSAPI | Compatibility vocabulary, not a required RDF wire model; retain CSAPI’s stated divergences |
| OGC Naming Authority/Definitions | Authority-issued OGC URI/URN identifiers and aliases | Trust for OGC-owned names when installed from a pinned snapshot; do not generalize to all mission terms |
| UCUM 2.1 | Machine unit syntax and conversion semantics incorporated by SWE Common | Primary recommended code system; case-sensitive parser; immutable package and conformance corpus |
| QUDT 3.5.1 | Candidate units, quantity kinds, dimensions, and mappings | Optional pinned enrichment package; never override exact source or become mandatory by this report |
| SKOS | Vocabulary concepts, labels, notations, hierarchy, association, and cross-scheme mapping | Use mapping predicates with their actual strength and direction; never promote labels or `closeMatch` to equality |
| RDF/OWL | Optional interchange and formal modeling mechanisms | Permit vocabulary-package import/export; do not require triple storage or unrestricted reasoning |
| AEP-4789 | Adopted server context and profile expectations | Prefer SI; permit specified operational non-SI exceptions; later profile work may narrow accepted vocabularies |
| Local/mission schemes | Terms absent from public vocabularies or restricted by mission | Permit stable authority identifiers plus definitions, version, provenance, policy, and offline package |

### 5.1 Authority Is Scoped, Not Global

A vocabulary can be authoritative for its own namespace without being the authority for every semantic question. UCUM controls UCUM syntax but does not decide whether a quantity represents torque or energy. QUDT may supply a dimension vector but does not prove that two domain properties are interchangeable. OGC Naming Authority controls OGC names but not a coalition’s mission vocabulary. A local concept can be valid within its declared authority without being globally recognizable.

The active resolution profile therefore records `(scheme, version, digest, authority, resolver, mappings, policy scope)`. A bare prefix map, mutable URL, or cached label is insufficient.

---

## 6. Semantic Concept Taxonomy

### 6.1 Canonical Support Concepts

| Concept | Required content | Non-collapse rule |
|---|---|---|
| `SemanticReference` | Exact lexical value; optional expanded absolute URI; semantic role; scheme/version/digest; resolution method/state; source provenance; policy label | Expanded URI does not replace submitted text; unresolved is not invalid |
| `ConceptRecord` | Authority-issued identity; scheme; types; language-tagged labels; definition; notations; lifecycle; replacement; source version | Labels and redirects are not identity |
| `SemanticAssertion` | Subject/resource/component path; predicate/role; object concept; source layer; validity; derivation evidence | Assertion provenance is distinct from concept provenance |
| `ConceptMapping` | Directional relation (`exact`, `close`, broader, narrower, related, or named project relation); source; evidence; validity; profile | No mapping silently merges concept identities |
| `PropertyRoleBinding` | Property identity; observed/controlled/asserted/status/capability/characteristic/parameter role; exact scope | Role is an edge in context, not an exclusive Property subtype |
| `UnitBinding` | Exact UnitReference; resolved unit; code-system version; dimension; quantity kinds; transform metadata; profile result | Same dimension is not semantic or operational equivalence |
| `VocabularyPackage` | Immutable content; version/digest; provenance/license; resolver; prefix map; mappings; signature/trust; supported capabilities | Live dereference does not mutate an active package |

### 6.2 Property and Field Distinctions

- **Property resource:** reusable identified `sosa:Property` concept in CSAPI Part 1.
- **Observed property:** a Property role describing what an Observation result estimates. It is not the Feature of Interest and not its value.
- **Controlled property:** a Property role describing what a Command intends to change. It is not the command parameter object, target, authorization, feasibility result, or execution status.
- **Feature property:** an asserted characteristic of a feature; it may share the same underlying Property identity but retains a distinct assertion role.
- **Capability:** a declared potential range or function. It does not prove current availability, authorization, or feasibility.
- **Characteristic:** descriptive property intrinsic to a System/Procedure representation; it is not automatically an observed or controlled stream role.
- **Status property:** meaning assigned to status data; it remains distinct from current-state projection and command-status vocabulary.
- **SWE component definition:** semantic meaning of a data component at an exact component path.
- **Unit:** scale/reference for a numerical value; it does not define the phenomenon or field role.
- **Category code space:** authority/dictionary for categorical tokens; it is distinct from the component’s property `definition`.
- **Qualifier/statistic/object type:** refines a derived property. It does not replace the base property or role assertion.

### 6.3 Resolution States

Glaux should retain at least: `invalid-syntax`, `unresolved-unknown`, `known-snapshot-unavailable`, `resolved-pinned`, `deprecated`, `conflicting`, `denied-by-policy`, and `quarantined`. A disconnected resolver must not turn a previously valid pinned concept into “invalid,” and a syntactically valid URI must not be reported as resolved merely because it was accepted as text.

---

## 7. Unit Strategy Findings

### 7.1 Representation and Baseline

SWE Common numerical `Quantity` values require a `uom`; even dimensionless ratios require a scale expression such as unity or percent. `Count` is an integer component without a unit. `Time` requires a time-scale unit. The `UnitReference` can carry display label/symbol, machine `code`, semantic `href`, or the allowed combination. When `code` and `href` both occur, they must identify the same unit. **[N]**

Glaux should:

1. Preserve the complete submitted `UnitReference`, including lexical case and display fields.
2. Validate `code` using the case-sensitive UCUM 2.1 grammar incorporated by SWE Common; reject illegal whitespace and ambiguous case variants in strict active contracts.
3. Prefer `uom.code` when UCUM can represent the unit; otherwise require a resolvable `uom.href` under the active profile.
4. If both occur, resolve both through the pinned registry and require semantic agreement.
5. Treat `label` and `symbol` as display metadata. A mismatch produces a stable warning or profile error, never identity.
6. Record whether the unit is SI, accepted non-SI, or prohibited by the active AEP/deployment profile.

The current UCUM 2.2 site and QUDT 3.5.1 may be evaluated as explicitly installed compatibility packages, but conformance to SWE Common’s incorporated baseline must remain reproducible against UCUM 2.1. **[N/D/P]**

### 7.2 Normalization and Conversion

Normalization produces a separate binding, not a rewritten source document. A normalized unit record may include canonical UCUM term, dimension vector, quantity kind, scale, offset, non-linear function, SI status, and mappings to installed semantic unit URIs.

Conversion is permitted only when all gates pass:

- source and target unit identities are resolved under named package versions;
- dimensions are compatible;
- property/quantity-kind semantics are compatible—dimensional equality alone is insufficient, as torque and energy illustrate;
- affine offsets, logarithmic/custom scales, reference frames, and calendar/time semantics are explicitly supported;
- interval bounds, constraints, nil values, uncertainty, precision, rounding, overflow, and loss policy remain valid;
- the operation is authorized by a named profile or explicit API behavior; and
- transformation provenance records source value/unit, target value/unit, algorithm/version, rounding, and time.

Observation storage should retain the accepted value in the exact immutable stream-contract unit. A separately materialized converted view may be offered later with provenance. Commands must match the exact active ControlStream contract unit; Glaux must not silently convert command values. Any future unit-negotiating command extension requires explicit safety, policy, feasibility, audit, and interoperability approval. **[P]**

Categories and counts are not converted. Calendar durations, timestamps, coordinates, logarithmic measures, and domain-specific reference scales require specialized rules rather than generic dimensional conversion.

### 7.3 Missing, Invalid, and Conflicting Units

| Condition | Strict active-contract behavior | Privileged import behavior |
|---|---|---|
| Quantity/Time missing required unit | Reject with stable semantic/profile diagnostic | Preserve and quarantine; cannot activate or compile |
| Unknown valid `href` | Reject if the component must carry operational data and no installed resolver can define it | Preserve as `unresolved-unknown`; no conversion or equivalence indexing |
| Invalid UCUM code | Reject | Preserve exact text and quarantine |
| Both code and href disagree | Reject as conflict | Preserve both and quarantine |
| Label/symbol disagreement only | Warning or configured profile error | Preserve all fields; identity remains code/href |
| Non-SI unit | Accept only if active profile permits the declared exception | Preserve with profile-noncompliant status |
| Unit changes on an active stream | Create a new immutable contract revision; do not reinterpret existing values | Preserve historical revision and linkage |

---

## 8. Observed-Property Strategy Findings

### 8.1 Identity and Role

CSAPI Part 1’s Property resource requires a `uniqueIdentifier`, `name`, and `baseProperty`; it can also carry description, `objectType`, `statistic`, and—in the inherited SensorML schema—qualifiers. One Property may be observed, controlled, and asserted. Glaux must therefore use one canonical property identity with contextual role assertions rather than disjoint observed/controlled classes. **[N]**

SensorML `ObservableProperty` identifies something that can be observed or measured and intentionally omits unit, quality, and constraint fields because those describe a procedure or value representation. A SWE result component supplies the exact value contract: component `definition`, unit where applicable, constraints, nil values, and encoding path. The observed-property binding joins these facts without collapsing them. **[N/P]**

### 8.2 Multi-Property DataStreams

A DataStream may produce records, arrays, vectors, and nested components containing multiple observed properties. Glaux should compile an immutable semantic-role map keyed by exact SWE component path. Each leaf carrying an observation result must have one unambiguous property binding. Aggregate definitions may describe a composite but cannot replace missing leaf semantics for individual values. Phenomenon time, result time, Feature of Interest, quality, sampling geometry, identifiers, and ancillary metadata are excluded from the observed-property summary unless a standard/profile explicitly declares them result properties.

For a Version 1.0 DataStream response, `observedProperties` should be derived as the deterministic, de-duplicated union of property bindings actually present in accepted linked Observations; the active contract supplies the candidate set and path interpretation. Return `null` when no accepted Observation establishes a property. Internally retain candidate contract properties separately from actual-observation evidence. This follows Part 2’s server-generated/linked-Observation intent while acknowledging issue #178’s unresolved clarification request. **[N/X/P]**

Part 2 prose describes a list of URIs, while the tagged JSON schema/example uses objects containing semantic definition and human metadata. The canonical layer should store identities and localized descriptions independently. The Version 1.0 adapter should emit the pinned object shape; clients must not use its label as identity. A later corrected profile can change only the representation adapter, not historical semantic assertions. **[X/P]**

### 8.3 Property Derivation and Query

`baseProperty` represents derivation/generalization within the CSAPI Property graph. Glaux should validate it as a directed acyclic graph, preserve the exact asserted edge, and materialize bounded descendant closure for filters where the standard requires or recommends derived-property inclusion. `objectType`, `statistic`, and qualifiers refine a derived concept and remain queryable metadata when policy permits.

This graph is not automatically the same as SKOS `broader`, QUDT quantity-kind hierarchy, OWL subclassing, or an external synonym relation. Cross-scheme mapping requires a separate typed `ConceptMapping` with provenance and profile approval.

Exact local Property IDs and unique-identifier URIs are the baseline CSAPI filter inputs. CURIE acceptance applies only where the controlling field permits it and an advertised, versioned prefix map expands it unambiguously. Glaux should not add JSON-LD inference or arbitrary network lookup to ordinary property filtering.

### 8.4 Feature of Interest and Sampling Feature

Observed property describes **what** is estimated; Feature of Interest or sampled feature describes **whose property** is estimated. They must never be inferred from each other. SamplingFeature property discovery can be derived only through explicit, policy-visible associations to streams/observations and must retain the association and temporal scope used. External sampled-feature ambiguity from issue #165 remains a relationship/query handoff, not permission for fuzzy semantic matching.

---

## 9. Controlled-Property and Command-Parameter Semantic Findings

A controlled property is what execution intends to change. A command parameter is a value-bearing field used to express that intent; one controlled property can require several parameters, and some parameters may select a target, mode, frame, schedule, tolerance, or execution option rather than being controlled properties themselves.

Glaux should compile a ControlStream semantic-role map from the exact active SWE command schema. Each executable value path is classified as `controlled-property-value`, `target`, `context`, `mode`, `schedule`, `tolerance`, `constraint-input`, or another named role. Placement in SensorML `input`, `output`, or `parameter`, the component name, and the unit are useful evidence but never sufficient by themselves: SensorML allows ObservableProperty and SWE components in several positions. **[N/P]**

The Version 1.0 `controlledProperties` summary should be the deterministic set of controlled-property bindings in the valid active ControlStream contract. Return `null` until no valid active schema establishes one. Emit the pinned object-shaped JSON artifact through the adapter, while keeping identity separate from label/description. Unlike observed properties, this describes the stream’s command capability rather than claiming that a command has already executed. **[X/P]**

Command validation must check the submitted field against the exact contract revision, property role, unit, nil/optional state, shape, constraints, and target context. These checks do not prove authorization, feasibility, safe effect, or execution success. Capability declarations likewise do not establish current availability. Those independent decisions must consume the semantic evidence and produce their own auditable outcomes.

Controlled-property identifiers and parameter schemas may reveal command affordances. The same policy decision must govern filtered ControlStream discovery, generated SensorML, semantic summaries, schema endpoints, feasibility descriptions, errors, audit views, and vocabulary resolution. Returning an empty visible summary while leaving a count, link, label, or diagnostic that confirms a hidden capability is a disclosure defect.

`DerivedProperty.qualifiers` must be preserved and validated even though the Part 1 conceptual table omits it; the inherited published SensorML schema supports it, and issue #162 records the gap. Qualifiers can materially distinguish operational effects and may not be dropped during round trip. **[N/X/P]**

---

## 10. Resource-Family Semantic Mapping

### 10.1 Resource Behavior

| Resource family | Direct semantic source | Derived/query behavior | API/source rule |
|---|---|---|---|
| Property | CSAPI identity, base property, object type, statistic, qualifiers | Cycle-safe derivation closure | Preserve source and expose canonical Property representation |
| System | SensorML identifiers/classifiers, capabilities, characteristics, inputs/outputs; linked streams; subsystems | Versioned interim union of explicit description capability plus directly associated visible streams; recursively include visible subsystem evidence where required | Never conflate declared capability with current stream availability |
| Procedure | SensorML description and schemas; explicitly associated streams | Explicit declared capability plus streams tied to that Procedure | Do not infer from all Systems that reference the Procedure unless profile says so |
| Deployment | Deployed Systems and streams within deployment/valid-time context | Derive visible property evidence through explicit deployment relationships | Retain relationship and temporal provenance |
| SamplingFeature | Associated visible streams/Observations | Derive through exact association and scope | Keep property separate from feature identity/type |
| DataStream | Immutable result schema plus accepted linked Observation role maps | Candidate property set from contract; response summary from actual accepted results | Version 1.0 object adapter; `null` when no established observed property |
| Observation | Exact parent contract, values, times, feature, result metadata | Per-leaf observed assertions and optional proven converted views | Store exact value/unit under contract; no semantic inference from labels |
| ControlStream | Immutable command schema and semantic-role map | Controlled-property capability set from active executable contract | Version 1.0 object adapter; policy-filter as sensitive capability |
| Command | Exact parent contract and submitted values | Per-leaf intent assertions; no inference of success/effect | No silent unit conversion; auth/feasibility/status remain separate |
| CommandStatus | Published status vocabulary and linked command evidence | No controlled-property derivation merely from status | Status code is not a property or result value |
| System Event | Event type and affected resource | May reference semantic changes but does not define them | Event vocabulary remains distinct from property vocabulary |
| Status value/stream | Exact schema, component definitions, units | Bind declared status phenomena; derive current state only under IDR-SRV-020 | Do not collapse observed status datum and availability assessment |
| SensorML document | Exact document plus parsed/canonical layers | Supplies assertions, declarations, and provenance | Full round-trip preservation per IDR-SRV-021 |
| SWE contract | Exact schema/encoding and compiled component graph | Supplies executable component semantics and unit plan | Immutable fingerprint binding per IDR-SRV-022/023 |
| Vocabulary package | Signed/pinned terms, mappings, resolver rules | Produces resolution evidence and bounded query materialization | Never exposed or activated beyond policy/trust scope |

Issue #179 leaves the normative source of several feature-resource filters unclear. The System/Procedure rows above are therefore a **Glaux interim compatibility policy**, not a claim about Part 1 Version 1.0. Every returned match must be explainable by explicit assertions, subsystem edges, stream associations, component paths, vocabulary package, mapping rule, and temporal/policy scope. The policy must be versioned so an approved correction can replace it without corrupting stored assertions.

### 10.2 Semantic Binding Matrix

The following matrix is the controlling handoff inventory. “Exact” in the query column means identifier equality after field-specific safe expansion, not arbitrary URI rewriting.

| Concept type | Unit/property/semantic identifier | Source standard / vocabulary / anchor | Authority | Related resource | Representation | Normalization | Source preservation | Query/index | Validation | Vocabulary/cache | Security/policy | Test | Downstream | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Property identity | `uniqueIdentifier` URI/URN | CSAPI P1 Property model | N | Property, all role-bearing resources | Exact URI plus local canonical resource ID | Purpose-specific comparison key only | Required | Exact ID/URI | URI syntax, uniqueness, collision | Optional installed scheme | May reveal phenomenon/capability | lexical, alias, collision | 025, 028, 031, 050, 056 | Prefer stable authority ID |
| Base property | `baseProperty` URI/CURIE | CSAPI P1 Property/DerivedProperty | N/X | Property | Directed property edge | Safe CURIE expansion; closure separate | Required | Exact plus required descendant closure | Target validity, cycle, bounded depth | Pinned prefix/scheme | Hidden hierarchy can leak terms | cycle/depth/version | 025, 028, 034, 050 | Not automatically SKOS broader |
| Object type | `objectType` URI/CURIE | CSAPI P1 | N/X | Property | Refining semantic reference | Field-specific expansion | Required | Exact where advertised | Scheme/type compatibility | Offline resolver | May expose target class | prose/OAS `objectType` regression | 028, 034, 050 | Ignore erroneous example `object` parameter |
| Statistic | `statistic` URI/CURIE | CSAPI/SensorML DerivedProperty | N | Property, stream component | Refining semantic reference | Field-specific expansion | Required | Exact/profile mapping | Compatible with result meaning | Pinned vocabulary | Analytical method may be sensitive | mean/min/max/custom | 028, 034, 053 | Not inferred from aggregation alone |
| Qualifier | SWE simple component | SensorML DerivedProperty; issue #162 | N/X | Property | Ordered named qualifier components | Bind each definition/unit | Required | Profile-defined only | Full SWE validation | Relevant installed schemes | Can expose operational context | round-trip and omission | 028, 034, 036, 053 | Preserve despite Part 1 table omission |
| Observed role | Property identity + component path | CSAPI P1/P2; SensorML; SWE | N/P | DataStream, Observation, System, Procedure | `PropertyRoleBinding(observed)` | Deduplicate identity in summaries | Exact assertion/evidence | Exact and base-derived | One unambiguous leaf binding | Property scheme snapshot | Filter before counts/links | multi-property/nested | 027, 031, 034, 050, 056 | Role is contextual edge |
| Controlled role | Property identity + component path | CSAPI P1/P2; SensorML; SWE | N/P | ControlStream, Command, System, Procedure | `PropertyRoleBinding(controlled)` | Deduplicate identity in summaries | Exact assertion/evidence | Exact and base-derived | Executable role and contract match | Property scheme snapshot | High sensitivity | multi-parameter/role confusion | 031, 036-040, 055-056 | Not authorization or effect |
| Feature assertion | Property identity | CSAPI P1 | N | System/feature resources | Property assertion edge | None unless approved mapping | Required | Exact/profile | Subject, predicate, value consistency | Scheme snapshot | Attribute disclosure | same property/multiple roles | 025, 028, 040 | Not necessarily observed/controlled |
| SWE definition | Absolute URI or permitted resolvable locator | SWE Core Req. 7-8 | N | Every value component | Exact component `definition` | CURIE only when field/profile permits | Required | Path + expanded ID | required, resolvable method, type/role | Offline package | Semantic schema disclosure | missing/unknown/denied | 028, 031, 034, 036, 050 | Labels cannot substitute |
| Aggregate definition | Composite semantic URI | SWE aggregate components | N | Record/array/vector/matrix | Definition on aggregate node | Keep separate from leaves | Required | Optional | Cannot mask leaf-definition gaps | Offline resolver | Composite may reveal payload | aggregate-vs-leaf | 022, 028, 053 | Composite meaning only |
| Category code | Token + `codeSpace` | SWE Category | N | Observation/Command fields | Exact token under authority | Optional notation mapping | Required | Scheme+token | membership/profile, nil distinction | Offline code list | Codes may be classified | unknown/deprecated code | 028, 034, 036, 040, 053 | CodeSpace != property definition |
| Quantity unit | UCUM `code` and/or `href` | SWE UnitReference; UCUM 2.1 | N/P | Numeric SWE fields | Exact UnitReference + UnitBinding | Canonical term/dimension separately | Required | Unit ID optional; no fuzzy query | syntax, same-unit dual ref, profile | Pin UCUM and mappings | Unit may reveal measurement capability | UCUM cases/conversions | 027, 031, 034, 036, 050 | Code preferred where constructible |
| Unit URI | OGC/QUDT/local URI | SWE UnitReference | N/D/P | Numeric SWE fields | `href` semantic reference | Map only under installed package | Required | Exact/profile mapping | resolves to a unit compatible with code | Pin exact vocabulary version | Dereference/supply-chain risk | offline/conflict | 028, 047, 053 | QUDT optional, not mandatory |
| Count semantics | Component definition; no uom | SWE Count | N | Integer fields | Count component | No unit normalization | Required | By property definition | integer/value constraints | Property scheme only | Context dependent | reject spurious uom | 031, 034, 036, 053 | Not dimensionless Quantity |
| Time unit | UCUM/time-scale reference | SWE Time; IDR-SRV-018 | N/P | Temporal SWE fields | Exact unit plus temporal frame context | Only with epoch/frame/precision | Required | Usually component semantics | valid time scale and frame | Pinned unit/time registry | Timing sensitivity | offset/calendar/leap cases | 027, 034, 042, 053 | No generic unit-only conversion |
| Identifier type | Term `definition` and `codeSpace` | SensorML identifiers | N | System, Procedure, documents | Label/value plus authority/type | Comparison by accepted ID rules | Required | Typed exact lookup | designation and authority | Offline scheme if semantic | Identity disclosure | label/type/authority confusion | 025, 028, 039 | Definition describes ID type |
| Classifier | Term semantic type/value | SensorML classifiers | N | System, Procedure | Exact term | Optional approved mapping | Required | Exact/profile mapping | code-space membership | Offline vocabulary | Classification can be sensitive | multilingual/local scheme | 028, 039-040, 053 | Not identity unless specified |
| Capability | SWE/SensorML component definitions | SensorML | N/P | System, Procedure | Assertion with value/range/unit | Bind concepts; no availability collapse | Required | Policy-scoped | structural/semantic/profile | Offline schemes | Often highly sensitive | capability-vs-live | 028, 037-040, 055 | Potential, not authorization |
| Characteristic | SWE/SensorML component definitions | SensorML | N | System, Procedure | Descriptive assertion | Bind property/unit | Required | Policy-scoped | semantic/value consistency | Offline schemes | Attribute disclosure | role non-collapse | 028, 039-040, 053 | Not automatically stream property |
| Status property | Component definition/status vocabulary | CSAPI P2; IDR-SRV-020 | N/P | Status data, System | Observed semantic assertion | Profile mappings separately | Required | Exact/status-specific | vocabulary and state-model separation | Pin status scheme | Operational sensitivity | stale/unknown/status | 034, 039-042, 053 | Not CommandStatus |
| Concept label | Language-tagged text | SKOS/SensorML/CSAPI | N/D | All semantic records | Label set by language/source | Unicode/display normalization only | Required | Search aid, never identity | language/tag/size/safety | Package-local | Injection and inference risk | language/HTML-like input | 028, 039, 053 | Labels may collide |
| Concept mapping | SKOS/project mapping URI pair | SKOS/project profile | D/P | Vocabulary layer/query | Directional typed edge | No identity merge | Required with evidence | Opt-in except CSAPI base derivation | relation strength, cycles, scope | Pin source/target packages | Mapping may reveal equivalence | exact-vs-close/broader | 025, 028, 040, 043, 053 | `closeMatch` is not equality |
| Local mission term | Authority URI/URN | Local/profile registry | Local/P | Any semantic field | Stable ID + definition + version | Expand only advertised prefixes | Required | Exact by default | authority, definition, package trust | Install signed/offline package | Often restricted | missing package/DDIL | 028, 039-040, 042, 047 | Valid without public dereference |
| Vocabulary snapshot | Version/digest/package ID | Glaux support concept | P | Registry/configuration | Immutable signed package | None | Full package/provenance | Package selection index | integrity, compatibility, license | Local cache | Supply chain and policy | tamper/rollback/offline | 025, 028, 039, 042, 047 | Live fetch cannot mutate active profile |
| Query closure | Property/mapping edges + profile | CSAPI query + Glaux support | N/P/X | Collections | Materialized explainable index | Bounded transitive computation | Preserve rule/package evidence | Exact/base-derived; optional mapped | cycle, depth, tenant, version | Uses installed packages only | Apply policy before counts | false-positive/leakage | 025-026, 039-040, 050-051 | Resource derivation remains #179 seam |

---

## 11. SensorML and SWE Common Semantic Consistency Findings

SensorML describes Systems and Procedures and embeds or references SWE components for inputs, outputs, parameters, capabilities, and characteristics. SWE Common defines the data-component contract. CSAPI turns selected concepts into addressable resources, summaries, filters, and dynamic exchanges. Consistency therefore requires cross-layer checks; it does not permit one representation to overwrite another.

### 11.1 Required Consistency Rules

1. **Component identity and path:** every semantic binding used to interpret dynamic data identifies the exact immutable SWE contract fingerprint and component path. Names alone are insufficient.
2. **Definition coverage:** every value-bearing leaf has a clear property definition and well-defined resolver. An aggregate definition does not fill an undefined leaf.
3. **Role evidence:** observed/controlled/status/parameter classification is explicit in the compiled contract. SensorML placement provides evidence but does not mechanically decide role.
4. **Unit compatibility:** Quantity/Time units satisfy SWE and the active profile. A SensorML capability or characteristic referring to the same property can use another unit only when the difference is explicit and a safe comparison rule exists.
5. **Constraint alignment:** constraints, nil reasons, optionality, and quality remain attached to their exact components. They refine acceptable data but do not define property identity, authorization, or safety.
6. **Property identity:** a CSAPI Property reference and SWE `definition` either identify the same concept or carry an explicit approved mapping. Shared label or unit does not suffice.
7. **Derived property fidelity:** base property, object type, statistic, and all qualifiers are retained. Qualifier order, names, definitions, units, and values survive round trip.
8. **Categorical separation:** Category `codeSpace` governs tokens; component `definition` governs what the value means. Both are validated independently.
9. **Description versus operation:** declared capabilities and inputs/outputs remain description evidence. An active DataStream/ControlStream contract and current operational state remain separate.
10. **Generated representations:** emitted SensorML, SWE schemas, and CSAPI property summaries are deterministic projections from one canonical assertion set and named compatibility profile.

### 11.2 Consistency Outcomes

| Outcome | Meaning | Active-use disposition |
|---|---|---|
| Consistent | Exact identities or approved mapping; role/unit/constraint compatible | Compile and activate |
| Consistent with declared transform | Explicit safe unit transform and provenance plan | Activate only when the interaction permits transformation |
| Incomplete | Missing optional enrichment but executable meaning remains clear | Warn and preserve |
| Unresolved | Syntactically valid term lacks an installed definition/mapping | Do not compile affected operational component; import may quarantine |
| Conflicting | Sources assert incompatible identities, roles, units, or mappings | Reject active contract; preserve evidence in quarantine |
| Policy denied | Semantic data is valid but not permitted in the current security context | Do not expose or use in policy-crossing derivation |

Implementation studies found practical variations: parsers sometimes dropped semantic fields or flattened nested schemas; servers accepted diverse URI patterns; generated property fields and filters were incomplete or inconsistent. These are useful fixture targets **[I]**, but they do not justify weakening the published model. Glaux should test preservation first, then explicitly adapt known Version 1.0 representation seams.

---

## 12. Validation, Normalization, Vocabulary-Cache, Query, and DDIL Findings

### 12.1 Validation by Interaction Stage

Semantic validation extends, but does not replace, the `IDR-SRV-023` pipeline.

| Stage | Required semantic work | Outcome boundary |
|---|---|---|
| Registry/package installation | Verify digest/signature/trust, namespace ownership, version, license, prefix uniqueness, identifier syntax, mapping relations, cycles, resource limits | Install inactive, activate, or reject package |
| Resource registration/update | Validate identifiers, required definitions, property graph, role declarations, unit references, cross-document references, profile rules, policy markings | Strict write reject or privileged quarantine |
| Contract compilation | Resolve every executable leaf; compile unit/property/role maps and safe transforms against immutable package set | Contract is executable or non-activatable |
| Observation ingestion | Select exact parent contract; validate present leaves, definitions by path, units, nils, and semantic consistency | Accept exact record or reject/quarantine by authorized path |
| Command submission | Select exact ControlStream revision; validate controlled roles and exact units; produce evidence for separate auth/feasibility/safety stages | Semantic validity never authorizes execution |
| Query | Validate identifier type, safely expand allowed CURIE, select matching mode/profile, bound closure, apply temporal and policy scope | Exact explainable results or stable error |
| Response generation | Apply pinned representation adapter and policy filter; emit identities/labels/descriptions consistently | No hidden-field leakage or mutable dereference |
| CI/conformance | Rebuild packages, closures, generated summaries, fixtures, diagnostics, and round trips deterministically | Reproducible evidence bundle |

### 12.2 Identifier Normalization

The exact submitted lexical identifier is immutable evidence. A separate comparison key may perform only operations permitted for that field and identifier kind. Absolute URIs are not globally lowercased, percent-decoded, path-normalized, redirected, or stripped of versions. CURIE expansion occurs only where the standard/profile allows CURIEs and only through an advertised immutable prefix map. The expansion, prefix-map version, and result are recorded.

Concept replacement, deprecation, aliasing, `owl:sameAs`, SKOS mappings, HTTP redirects, and human synonyms are separate relations. None overwrites historical assertions. A response may choose an authority-preferred identifier under an advertised profile while retaining source identity and transformation evidence.

### 12.3 Vocabulary Package and Cache Model

Glaux should package semantic dependencies as immutable, content-addressed `VocabularyPackage` artifacts. At minimum each package contains:

- package identity, semantic version or source date, digest, producer, source URL, retrieval time, license, trust/signature evidence, and policy scope;
- authoritative namespaces and identifier patterns;
- term identities, types, definitions, language-tagged labels, notations, deprecation/replacement data, and permitted aliases;
- prefix bindings and deterministic expansion rules;
- unit records, property relations, and mappings supported by that package;
- parser/model version, validation result, limits, and compatibility declarations; and
- a resolver that works without network access.

Activation is explicit and atomic per deployment/tenant/profile. A background online refresh may stage a candidate package, but it cannot alter active resolution or query results until verified and promoted. Historical assertions retain the package/version/digest used. Eviction must respect records, contracts, audit evidence, and reproducibility holds.

DDIL operation uses installed packages and records `known-snapshot-unavailable` or `unresolved-unknown` rather than attempting uncontrolled network access. Public writes requiring executable semantics fail closed when no trusted package can resolve a required definition. Privileged import can preserve and quarantine unresolved content for later revalidation. Live vocabulary access is never on the critical Observation or Command path.

### 12.4 Query and Discovery Semantics

The default matching hierarchy is:

1. exact canonical local Property ID;
2. exact unique-identifier URI after safe field-specific parsing/expansion;
3. CSAPI base-property descendant expansion where applicable;
4. optional, explicitly advertised profile mappings; and
5. no fuzzy label, unit, dimension, redirect, or `closeMatch` matching by default.

Base-property traversal is cycle-safe, depth/node bounded, package/version scoped, and deterministic. Optional semantic mapping expansion must identify whether it uses exact, close, broader, narrower, or custom relations; only relations approved for the named query profile participate. Result evidence records why a resource matched.

For issue #179’s unresolved resource derivation, Glaux should version the following interim rule:

- DataStreams and ControlStreams match their direct compiled role bindings.
- SamplingFeatures match through explicit visible stream/Observation associations.
- Deployments match through deployed visible Systems/streams within the request’s temporal context.
- Systems match explicit SensorML capability assertions, directly associated streams, and—in the recursive behavior required by the applicable endpoint—visible subsystem evidence.
- Procedures match explicit description assertions and streams explicitly associated to that Procedure.

Declared capability matches and active-stream matches remain distinguishable internally and in audit/explanation evidence. Policy filtering occurs before closure-derived inclusion, counts, pagination, links, and facets. This is a project compatibility decision **[P/X]**, not a newly claimed Part 1 requirement.

### 12.5 API Exposure

Standards-required fields remain in standards representations. Optional semantic enrichment should be exposed only through an advertised extension/profile or vocabulary endpoint designed by later topics. Glaux must not inject unadvertised JSON-LD context, QUDT fields, internal trust scores, policy labels, mapping provenance, or conversion plans into normative representations.

---

## 13. Persistence, Metadata Storage, and Configuration Implications

This section identifies data responsibilities, not database schema or product choices.

### 13.1 Persisted Artifacts

- exact SensorML/SWE/CSAPI source representations and hashes;
- parsed standards-shaped documents and diagnostics;
- canonical Concept, SemanticReference, SemanticAssertion, PropertyRoleBinding, UnitBinding, and ConceptMapping records;
- immutable vocabulary packages, prefix maps, trust/license metadata, activation history, and resolution outcomes;
- property-derivation and approved mapping edges with effective/system time and provenance;
- immutable stream-contract fingerprints, component paths, semantic role maps, unit conversion plans, and activation states;
- actual Observation/Command semantic assertions tied to accepted records;
- materialized query closures with package/profile/policy/tenant/version identity;
- deterministic generated property summaries and representation-adapter version;
- validation, transformation, warning, quarantine, and policy-decision evidence.

Category E must decide physical ownership, transactional boundaries, indexing, bitemporality, retention, rebuilds, and cache eviction. Exact source and canonical semantic records must remain recoverable even if a materialized index or vocabulary cache is rebuilt.

### 13.2 Indexing Needs

Candidate indexes include exact property URI/local ID; role plus resource identity; component path plus contract fingerprint; base-property ancestor/descendant closure; scheme/version/term; unit code/URI/dimension/quantity kind/profile status; category scheme/token; relationship/temporal scope; and policy partition. Multilingual labels may support text discovery but never identity joins.

Indexes must be tenant/profile/version/policy aware. A global closure that mixes restricted mappings or incompatible vocabulary versions would produce false matches and disclosures. Rebuilds require deterministic manifests and comparison against accepted golden results.

### 13.3 Configuration Needs

Later configuration work should define trusted package sources and signing keys; active package pins; prefix maps; strict/quarantine rules; allowed local namespaces; AEP/deployment unit profiles; mapping relations permitted for query; closure limits; dereference/network policy; package size and graph limits; warning-to-error promotion; language/display preferences; and compatibility-adapter version.

Secrets belong only to package retrieval/signature services where necessary. Semantic records and configuration must not embed credentials or unrestricted fetch URLs.

---

## 14. Security, Policy, and Releasability Implications

Semantic metadata can reveal sensor modality, collection targets, measured phenomena, precision, range, platform type, processing methods, command effects, supported modes, and deployment capability even when data values are hidden. Vocabulary mapping can reveal that a restricted local term corresponds to a public capability. Unit and schema detail can fingerprint equipment. Property filters, counts, latency, validation diagnostics, links, generated SensorML, and vocabulary resolution can all become inference channels.

### 14.1 Required Controls for Later Topics

- Authorize the **semantic assertion and derivation path**, not only the terminal resource.
- Apply policy before semantic closure, counts, pagination, facets, links, alternate representations, generated descriptions, and diagnostics.
- Label source assertions, mappings, vocabulary packages, generated summaries, and materialized indexes with applicable policy scope.
- Ensure a public concept does not declassify the existence of a restricted binding or resource.
- Return stable, low-detail external diagnostics; retain detailed concept/path/package evidence only in protected audit channels.
- Block arbitrary server-side dereferencing to prevent SSRF, credential leakage, tracking, and cross-domain data exfiltration.
- Verify package provenance, signatures/digests, namespace authority, size, graph complexity, parsers, and licenses to prevent vocabulary poisoning and supply-chain compromise.
- Bound traversal, import recursion, language-tag sets, literal size, regular expressions, conversion functions, and reasoning to prevent resource exhaustion.
- Sanitize labels/definitions for their output context without changing preserved source bytes.
- Treat controlled-property, command-schema, feasibility, and capability semantics as particularly sensitive.
- Keep semantic validity, authorization, safety, feasibility, and releasability as separate evidence-producing decisions.

Cross-boundary export needs a deterministic policy-filtered projection and should not export hidden mappings, alternative labels, broader concepts, source provenance, or vocabulary-package details unless separately permitted. Redaction must not leave dangling identifiers or totals that expose the omitted concept.

---

## 15. Fixture, Golden-File, Conformance, and Interoperability Test Implications

### 15.1 Required Corpus Families

| Fixture family | Minimum cases | Principal assertions |
|---|---|---|
| UCUM | case-sensitive valid codes; whitespace; compound units; prefixes; unity/percent; Celsius affine conversion; invalid code; 2.1/2.2 delta | Exact preservation, version pin, canonical identity, conversion gates |
| UnitReference | code only, href only, both same, both conflict, display-label mismatch, non-SI profile exception | SWE rule and AEP profile behavior |
| Quantity/Count/Time | required unit, count without unit, time scale, calendar/ref-frame ambiguity | Component-specific validation |
| Property | URI/URN, local ID, base chain, cycle, diamond, object type, statistic, qualifiers, deprecated term | Identity, closure, full fidelity |
| Multiple roles | one Property observed, controlled, and asserted in distinct contexts | No disjoint-class assumption |
| Multi-property result | nested Record/Array/Vector, optional and nil leaves, ancillary fields | Exact component-path role map and summaries |
| Category | definition plus codeSpace, unknown/deprecated token, multilingual labels | Property/code-list separation |
| CURIE/URI | permitted expansion, unknown prefix, prefix-version change, misleading redirect, percent/case variants | Field-specific normalization only |
| Mapping | SKOS exact/close/broader/narrower/related, direction, cycles, version change | No silent identity merge; opt-in expansion |
| Local vocabulary | public and restricted mission packages, signed/tampered, unavailable, license metadata | Offline/trust/policy behavior |
| DataStream summary | no observations, partial optional leaves, several observations, schema revision, conflicting labels | Actual-evidence derivation and Version 1.0 adapter |
| ControlStream summary | no valid contract, multi-parameter command, target/context vs controlled value | Capability derivation and role distinction |
| SensorML round trip | identifiers, classifiers, inputs/outputs/parameters, capabilities, characteristics, derived qualifiers | Exact/parsed/canonical/generated separation |
| Query | exact, base descendant, optional mapped, System/subsystem, Procedure, Deployment, SamplingFeature | Explainable issue-#179 interim behavior |
| Policy | hidden property, hidden mapping, count/pagination/link/diagnostic side channels | Non-interference across all projections |
| DDIL | warm cache, cold unknown, missing referenced package, expired candidate, historical pin | Stable offline outcomes; no live critical dependency |
| Malicious content | huge graph, recursive import, label injection, hostile URI, conversion overflow | Resource limits and safe diagnostics |

### 15.2 Test Lanes

- **Unit tests:** URI parsing, CURIE expansion, UCUM grammar, mapping strength, graph cycle/depth, unit conversion gates, label handling.
- **Golden round trips:** source bytes to parsed/canonical/generated views and back where the standard permits; known lossy adapters must be declared.
- **Schema/semantic integration:** every active SWE contract compiles to stable role and unit maps and rejects unresolved executable leaves.
- **API conformance:** published filters, Property resources, summaries, null behavior, media representations, and required error responses.
- **Compatibility:** pinned Version 1.0 prose/schema conflicts, issue #162 qualifiers, issue #178 derivation variants, and issue #179 resource mapping profiles.
- **External clients:** CSAPI Explorer, generated clients, OS4CSAPI corpus, CS-Go/OSH/pygeoapi/SECD-derived examples; record client assumptions rather than altering standards silently.
- **Security/non-interference:** compare permitted versus denied views for resources, totals, ordering, links, diagnostics, caches, and latency-sensitive paths.
- **Offline/reproducibility:** rebuild with network disabled from exact vocabulary and schema package manifests.

Conformance assertions must identify the approved standard version and requirement/ATS anchor. Candidate QUDT, SSN draft, or project-extension behavior belongs in separate compatibility lanes and must never inflate standards-conformance claims.

---

## 16. Downstream Topic Handoff Matrix

| Topic(s) | Required handoff | Acceptance boundary |
|---|---|---|
| IDR-SRV-025 | Artifact ownership, semantic graph persistence, closure/index lifecycle, package activation transactions | Choose physical database/persistence architecture |
| IDR-SRV-026 | Spatial-feature plus property-filter planning and policy-safe query execution | Do not conflate property and Feature of Interest |
| IDR-SRV-027 | Exact-unit time-series storage, optional converted views, provenance, schema revision | Decide storage/materialization mechanics |
| IDR-SRV-028 | Exact documents, canonical semantic records, package/mapping metadata, multilingual text | Decide document/metadata representation and retention |
| IDR-SRV-029 | Atomic resource/contract/vocabulary changes and closure invalidation | Decide transaction, idempotency, concurrency rules |
| IDR-SRV-030 | Retention of source, package pins, historical mappings, assertions, derived indexes | Decide deletion/archive policy |
| IDR-SRV-031/032/033 | Publisher/simulator declaration requirements, strict vs quarantine, package availability | Finalize write and producer contracts |
| IDR-SRV-034 | Observation leaf bindings, actual-property summary algorithm, status semantics, conversion views | Finalize dynamic-data behavior; revisit issue #178 |
| IDR-SRV-035 | Carry property semantics into events without making draft Part 3 normative | Finalize pub/sub adoption profile only there |
| IDR-SRV-036 | ControlStream role map, exact command units, controlled summary, no silent conversion | Finalize Command lifecycle |
| IDR-SRV-037 | Capability/availability/feasibility distinctions and semantic constraint evidence | Finalize feasibility processing |
| IDR-SRV-038 | Semantic validity versus authorization/safety; controlled-property sensitivity | Finalize authorization, safety, audit behavior |
| IDR-SRV-039/039A/040/041 | Dereference threats, package supply chain, semantic inference, releasability, audit evidence | Finalize security/policy controls and profiles |
| IDR-SRV-042 | Installed package behavior, unresolved states, synchronization and revalidation | Finalize DDIL semantics |
| IDR-SRV-043 | Vocabulary/mapping/contract version conflicts and cross-node convergence | Finalize synchronization rules |
| IDR-SRV-047 | Package pins, prefix maps, trust anchors, limits, profiles, retrieval policy | Finalize configuration/secrets handling |
| IDR-SRV-050/051 | Requirement anchors and separate conformance/compatibility lanes | Build harness/traceability |
| IDR-SRV-053 | Corpus listed in Section 15 with licenses and source provenance | Build fixture/golden-file set |
| IDR-SRV-055 | Command-unit, controlled-property, disclosure, and semantic-confusion attacks | Build security/command test strategy |
| IDR-SRV-056 | Client behavior for URI/object summaries, multiple properties, local terms, units, filters | Build external interoperability matrix |
| IDR-SRV-057 | Carry accepted decisions and unresolved #178/#179 seams into synthesis | Do not mark this report accepted before lead review |

---

## 17. Recommendations

1. **Adopt the five-view semantic architecture.** Preserve exact source, parsed standards model, canonical semantic graph, materialized validation/query view, and generated representation independently. Priority: High.
2. **Model roles as contextual assertions.** One Property identity may be observed, controlled, asserted, status-related, or used in a capability/characteristic context. Priority: High.
3. **Require explicit executable leaf semantics.** Every operational SWE leaf needs a clear definition and resolver; every Quantity/Time needs its applicable unit. Priority: High.
4. **Use UCUM 2.1 as the reproducible recommended unit-code baseline.** Prefer `uom.code` when constructible; support `href`; require agreement when both occur. Priority: High.
5. **Apply the AEP SI preference as a profile rule.** Permit non-SI units only when specified as operationally necessary; preserve the submitted unit and exception evidence. Priority: High.
6. **Never silently convert Commands.** Require exact ControlStream contract units. Allow Observation conversion only as a separately provenance-bearing view after all compatibility gates pass. Priority: High.
7. **Treat QUDT 3.5.1 as optional pinned enrichment.** Use quantity kinds and dimensions for validation assistance, never to override source or impose a public wire vocabulary. Priority: Medium.
8. **Permit stable local/mission vocabularies.** Require authority, definition, version/digest, offline package, provenance, and policy scope; public dereference is not mandatory. Priority: High.
9. **Implement conservative, explainable semantic queries.** Exact matching and required CSAPI base-property derivation are baseline; optional mappings are named/profiled; fuzzy labels, same dimension, and `closeMatch` are off by default. Priority: High.
10. **Version the #178/#179 compatibility seams.** Keep canonical identity separate from Version 1.0 summary objects, and isolate interim cross-resource derivation behind a profile with match evidence. Priority: High.
11. **Build immutable offline vocabulary packages.** Pin version/digest, trust, license, prefix map, resolver, and mappings; stage rather than live-mutate active packages. Priority: High.
12. **Apply policy to semantic derivation before projection.** Protect concepts, mappings, counts, links, schemas, diagnostics, and generated descriptions consistently. Priority: High.
13. **Preserve `DerivedProperty.qualifiers`.** Treat the inherited SensorML schema as the source-backed representation and retain the Part 1 table omission as an interoperability test seam. Priority: High.
14. **Avoid unrestricted reasoning and dereferencing.** Use bounded, allowlisted resolution and materialization; a triple store or OWL reasoner is not required. Priority: High.
15. **Maintain distinct conformance and compatibility lanes.** Approved CSAPI/SensorML/SWE behavior, candidate vocabularies, draft SSN evolution, and Glaux extensions need separate claims. Priority: High.

---

## 18. Risks, Constraints, and Open Questions

### 18.1 Risks and Constraints

- Part 2 property-summary prose and JSON artifacts disagree; choosing one shape without a pinned adapter can break clients.
- Issue #178 leaves DataStream generated-field derivation incomplete; Observation absence, optional leaves, and schema revisions need later operational decisions.
- Issue #179 leaves feature-resource property-filter sources incomplete; the interim Glaux rule could differ from a future approved revision.
- Mutable QUDT and online OGC definition endpoints can change independently of stored data.
- UCUM version or case-mode drift can change parsing/conversion outcomes.
- Same-dimension conversion can be semantically wrong; affine, logarithmic, calendar, uncertainty, and bounded values create additional risk.
- Local vocabulary flexibility can undermine interoperability if definition, authority, package, and mapping evidence are optional.
- Overly rigid public-vocabulary enforcement can reject legitimate coalition or classified semantics.
- Semantic mappings and labels can disclose hidden capabilities even where source resources are access-controlled.
- Graph traversal, reasoning, dereferencing, and imported literals create denial-of-service, SSRF, injection, and supply-chain risks.
- Parser or generator field loss—especially qualifiers and nested component definitions—can corrupt meaning while remaining structurally valid.

### 18.2 Open Questions and Assigned Resolution

| Open question | Why unresolved | Resolution owner |
|---|---|---|
| Final DataStream `observedProperties` derivation across optional/no-data/schema-revision cases | Part 2 and issue #178 do not settle it | IDR-SRV-034, monitor upstream |
| Final resource-specific property-filter capability sources | Issue #179 working discussion is not approved | IDR-SRV-034/050/056 and upstream monitoring |
| Whether a future CSAPI profile emits URI lists or semantic-description objects | Published prose/schema conflict | IDR-SRV-010A/014/034 compatibility work |
| Which public or mission property vocabularies an AEP deployment mandates | Current accepted AEP baseline does not select one here | Future AEP/deployment profile governance |
| Whether UCUM 2.2 is accepted in addition to the SWE 2.1 pin | Requires compatibility corpus and profile decision | IDR-SRV-047/050/053 |
| Whether QUDT is shipped by default | License/package size/current-version and interoperability tradeoffs | IDR-SRV-025/028/047 |
| Public vocabulary/discovery endpoint shape | Not defined by this topic’s controlling standards | Later API/documentation/security topics |
| Exact cross-boundary treatment of restricted-to-public mappings | Policy architecture not yet complete | IDR-SRV-039A/040 |
| Unit-negotiating write or Command extension | Safety and interoperability implications exceed this topic | IDR-SRV-031/036/038; default remains prohibited |

No open question prevents this report from serving as the planned baseline because each ambiguity is isolated, given a conservative interim behavior where necessary, and routed to a responsible downstream topic.

---

## 19. Validation Against Plan Success Criteria

| Topic plan success criterion | Status | Evidence |
|---|---|---|
| Unit, observed/controlled property, identifier, vocabulary, and ontology-binding concepts have anchors | Met | Sections 3, 5-9 |
| Concepts mapped to canonical CSAPI resource families | Met | Section 10.1 and semantic binding matrix |
| Source, normalized, validation, query/index, and API fields distinguished | Met | Sections 4, 10.2, 12-13 |
| Candidate unit/vocabulary sources evaluated without premature mandate | Met | Sections 3.2, 5, 7, 17 |
| Validation, normalization, alias/mapping, versioning, DDIL, and query implications documented | Met | Section 12 |
| Security, policy, conformance, fixture, and interoperability implications documented | Met | Sections 14-15 |
| Implementation/community evidence incorporated as non-normative | Met | Sections 3.2 and 11.2 |
| Recommendations are decision-usable and bounded to Glaux Server | Met | Section 17; explicit scope in Section 2 |
| Downstream handoffs are explicit | Met | Section 16 |
| References are explicit and reproducible | Met | Section 20; immutable CSAPI pin and package versions recorded |

### 19.1 Report Completion Checklist

- [x] Topic ID matches the overall research plan.
- [x] Topic research plan is linked and aligned.
- [x] Core and detailed research questions are covered or explicitly unresolved.
- [x] Findings have reproducible evidence and authority labels.
- [x] Normative, project, candidate, implementation, and unresolved evidence are not conflated.
- [x] Mutable sources identify versions, commits, or dated retrieval.
- [x] Controlled-source handling is explicit and no controlled text is redistributed.
- [x] Conflicts with accepted prior reports are reconciled.
- [x] Executive summary is independently readable.
- [x] Recommendations, risks, open questions, and handoffs are explicit.
- [x] Plan-owner acceptance fields remain pending review.

---

## 20. References

### Project and Governance

- [IDR-SRV-024 research plan](../IDR%20Plans/idr-srv-024-units-observed-properties-and-semantic-binding-strategy.md).
- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md).
- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md).
- [Research Planning Approach](../../../../../Governance/research-planning-approach.md).
- [Research Report Template](../../../../../Governance/research-report-template.md).
- [OGC API - Connected Systems upstream-history evidence register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md), Version 1.10.
- Accepted `IDR-SRV-001` through `IDR-SRV-023` reports in this report directory, especially `IDR-SRV-014A` through `014G` and `IDR-SRV-015` through `023`.
- Project-controlled `AC/224(JCGISR)D(2026)0005`, April 27, 2026, SHA-256 `56dc757b6e677b3584e3152a957849f21a24b22854f562613ff283a8b599da8c`; accepted findings only, source not redistributed.

### Approved Standards and Published Artifacts

- Open Geospatial Consortium, [OGC 23-001, *OGC API - Connected Systems - Part 1: Feature Resources*, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html), published 2025-07-16.
- Open Geospatial Consortium, [OGC 23-002, *OGC API - Connected Systems - Part 2: Dynamic Data*, Version 1.0](https://docs.ogc.org/is/23-002/23-002.html), published 2025-07-16.
- Open Geospatial Consortium, [tagged CSAPI Version 1.0 source and API artifacts, commit `8e03b236`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2).
- Open Geospatial Consortium, [OGC 23-000, *OGC SensorML Encoding Standard*, Version 3.0](https://docs.ogc.org/is/23-000/23-000.html), published 2025-07-16.
- Open Geospatial Consortium, [OGC 24-014, *OGC SWE Common Data Model Encoding Standard*, Version 3.0.0](https://docs.ogc.org/is/24-014/24-014.html), published 2025-07-16.
- W3C and OGC, [*Semantic Sensor Network Ontology*, 2017 Recommendation](https://www.w3.org/TR/2017/REC-vocab-ssn-20171019/).
- Open Geospatial Consortium, [OGC API - Features - Part 1: Core](https://docs.ogc.org/is/17-069r4/17-069r4.html).
- Open Geospatial Consortium, [OGC Naming Authority](https://www.ogc.org/ogcna/), including OGC 09-048r6 and OGC 09-046r6.
- Open Geospatial Consortium, [OGC Definitions Server](https://defs.opengis.net/).
- Regenstrief Institute, [Unified Code for Units of Measure](https://ucum.org/), SWE-incorporated Version 2.1 baseline and current Version 2.2 context.
- W3C, [*SKOS Simple Knowledge Organization System Reference*](https://www.w3.org/TR/skos-reference/), Recommendation, 2009.
- W3C, [*RDF 1.1 Concepts and Abstract Syntax*](https://www.w3.org/TR/rdf11-concepts/).
- W3C, [*OWL 2 Web Ontology Language Document Overview*](https://www.w3.org/TR/owl2-overview/).

### Candidate, Mutable, and Compatibility Evidence

- QUDT, [QUDT.org](https://qudt.org/), catalog Version 3.5.1, generated 2026-08-29; candidate enrichment source, not a mandated Glaux vocabulary.
- W3C, [*Semantic Sensor Network Ontology (2023 Edition)*](https://www.w3.org/TR/vocab-ssn-2023/), Working Draft dated 2026-09-14; future compatibility signal only.
- OGC API - Connected Systems [issue #40](https://github.com/opengeospatial/ogcapi-connected-systems/issues/40), [#73](https://github.com/opengeospatial/ogcapi-connected-systems/issues/73), [#74](https://github.com/opengeospatial/ogcapi-connected-systems/issues/74), and merged [PR #94](https://github.com/opengeospatial/ogcapi-connected-systems/pull/94), UnitReference history reflected in the published baseline.
- OGC API - Connected Systems [issue #162](https://github.com/opengeospatial/ogcapi-connected-systems/issues/162), open `DerivedProperty.qualifiers` mapping gap.
- OGC API - Connected Systems [issue #165](https://github.com/opengeospatial/ogcapi-connected-systems/issues/165), open sampling-feature/query ambiguity.
- OGC API - Connected Systems [issue #178](https://github.com/opengeospatial/ogcapi-connected-systems/issues/178), open generated/receivable DataStream-field clarification.
- OGC API - Connected Systems [issue #179](https://github.com/opengeospatial/ogcapi-connected-systems/issues/179), open property-filter/resource-derivation clarification, checked 2026-09-14.
- Official CSAPI `master` snapshot [`3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f), checked 2026-09-14.

---

**Review gate:** This report is complete and in review. It becomes an accepted downstream baseline only after the Glaux Project Lead records acceptance in this report, the topic plan, and the overall plan.

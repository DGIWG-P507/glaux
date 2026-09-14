# Section 023: Schema and Encoding Validation Strategy - Research Report

**Topic ID:** IDR-SRV-023<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-023 Schema and Encoding Validation Strategy](../IDR%20Plans/idr-srv-023-schema-and-encoding-validation-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core questions, all detailed-question groups, all 6 methodology phases, and all 11 success criteria<br>
**Methodology Used:** Authority-ranked extraction from approved CSAPI Parts 1 and 2, OGC API - Features, SensorML 3.0, SWE Common 3.0, OpenAPI 3.1.2, JSON Schema 2020-12, GeoJSON, HTTP, Problem Details, tagged schemas/OpenAPI artifacts, abstract tests, accepted AEP and prior-IDR findings, implementation/interoperability evidence, and bounded official issue-history refresh; followed by resource-family, interaction-stage, outcome, tooling, security, and test mapping<br>
**Research Time:** Approximately 13 hours of AI-assisted execution on September 14, 2026<br>
**Official Standards Source Pin:** [`opengeospatial/ogcapi-connected-systems` `v1.0.0` at `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)<br>
**Mutable Upstream Recheck:** Official `master` remained at [`3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f) on September 14, 2026; no material shared-register change was required<br>
**Shared Register Baseline:** [OGC API - Connected Systems upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md), Version 1.9<br>
**Controlled AEP Source:** `AC/224(JCGISR)D(2026)0005`, April 27, 2026, SHA-256 `56dc757b6e677b3584e3152a957849f21a24b22854f562613ff283a8b599da8c`; used only through accepted project findings and not redistributed<br>
**Document Purpose:** Establish the reusable validation architecture for the Rust Glaux reference server without selecting final crates, persistence products, authorization policy, semantic registries, command lifecycle, or conformance-harness implementation<br>
**Author:** OpenAI Codex<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** September 14, 2026<br>
**Date:** September 14, 2026<br>
**Last Updated:** September 14, 2026

---

## Reading Guide and Evidence Labels

| Label | Meaning |
|---|---|
| **N** | Normative or standards-derived finding from an approved applicable source or incorporated artifact |
| **A** | Project-controlling AEP/STANAG adoption or operational-context finding carried from an accepted report |
| **P** | Glaux project decision or recommendation proposed for acceptance here |
| **I** | Informative implementation, test, interoperability, or community evidence |
| **D** | Official draft or post-publication direction that is not an approved requirement |
| **X** | Published ambiguity, artifact conflict, evidence gap, or unresolved project decision |

“Schema-valid,” “standards-conformant,” “safe to execute,” “authorized,” and “interoperable” are different claims. This report uses **validator** for one bounded check and **validation pipeline** for the ordered combination of checks, evidence, and outcomes.

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Validation Extraction Methodology
5. Schema, OpenAPI, Media Type, and Encoding Inventory
6. Validation Type Taxonomy
7. Resource-Family Validation Strategy
8. Interaction-Stage Validation Strategy
9. SensorML and SWE Common Validation Findings
10. Dynamic-Data, Observation, Status, Event, Command, and Feasibility Validation Findings
11. Error, Warning, Normalization, Rejection, Quarantine, and Audit Findings
12. Runtime, CI, Conformance, Fixture, and Golden-File Validation Findings
13. Tooling, Schema-Cache, `$ref`, and Offline/DDIL Validation Implications
14. Security, Policy, and Releasability Validation Implications
15. Downstream Topic Handoff Matrix
16. Recommendations
17. Risks, Constraints, and Open Questions
18. Validation Against Plan Success Criteria
19. References

---

## 1. Executive Summary

Glaux should implement validation as a **versioned, evidence-producing pipeline**, not as a JSON Schema middleware switch. Every accepted mutation or ingestion unit passes an ordered set of syntax, media/encoding, structural, profile, semantic, relationship, temporal, operational, and security/policy gates appropriate to its resource family and interaction stage. Interoperability and conformance checks surround that runtime core but do not replace it. [N,A,P]

The reusable architecture has four controls:

1. a provenance-bearing **contract registry** containing exact vendor artifacts, Glaux-owned overlays, compiled SWE contracts, media/encoding profiles, reference closures, digests, capability state, and validation-policy versions;
2. a **validation orchestrator** that selects a contract from the route, method, negotiated representation, parent resource revision, profile, tenant, and operation—not from untrusted payload claims alone;
3. typed validators/codecs for rules that schemas cannot express, including relationship integrity, temporal containment, stream-schema binding, SWE value decoding, units and semantic seams, command feasibility/safety seams, and information-flow policy; and
4. immutable, disclosure-filtered **validation evidence** recording the input digest, exact contract fingerprint, stage, validator versions, normalized changes, findings, outcome, and correlation to provenance and audit events. [P; IDR-SRV-015–022]

OpenAPI remains a generated and continuously checked description of the implementation, not the runtime source of truth. Glaux should generate one complete deployment-specific OpenAPI 3.1.2 definition from the same typed contract registry that drives routes, extractors, schemas, media negotiation, conformance declarations, and tests. The approved CSAPI tagged OpenAPI artifacts are valuable standards evidence and negative/reference fixtures, but they are incomplete as a server contract: the Part 2 navigation surface omits required routes, examples remain unresolved in the release bundles, and the artifacts declare no operation identifiers or security schemes. [N,X,P; IDR-SRV-014]

JSON Schema Draft 2020-12 is the project dialect for Glaux-owned JSON schemas and the dialect used by the 69 inspected tagged CSAPI/SensorML/SWE JSON schemas; those tagged schemas declare `$schema` but no stable `$id`. Glaux must vendor the exact approved artifacts, assign identities in an external manifest rather than rewriting them, close all `$ref` graphs offline, preserve recursion, and compile them under bounded depth, memory, time, alias, and regex limits. Network retrieval is disabled by default. `format`, OpenAPI `contentMediaType`, `contentEncoding`, `contentSchema`, and `readOnly`/`writeOnly` are annotations unless Glaux explicitly implements and tests their assertion or directional semantics. [N,P]

For SensorML and SWE Common, structural schema validation is only an early gate. Glaux preserves exact source, parses the complete published model, produces a canonical projection, generates authorized views, and stores validation/transformation evidence. A DataStream or ControlStream revision binds to an immutable SWE contract fingerprint. Observation results and command parameters validate against that exact parent revision; ordinary `application/json` validates the outer CSAPI object and interprets `resultSchema`/`parametersSchema` as SWE component descriptions, not JSON Schema. SWE media types validate the whole record using `recordSchema` plus its `encoding`. [N,P; IDR-SRV-021–022]

Strict public writes either become fully valid active state or fail atomically. They are not partially accepted. A separately authorized administrative import workflow may preserve invalid, partial, legacy, or profile-divergent source in quarantine with findings, but quarantined material cannot become active, satisfy references, drive code generation, advertise conformance, or execute commands until explicit promotion revalidates it. Normalization must be deterministic and recorded; it may canonicalize representation but never silently repair meaning. [P]

Client errors use RFC 9457 problem details and the accepted IDR-SRV-013 status taxonomy. Malformed syntax and CSAPI-mandated parent-schema failures are `400`; unsupported request media type or content coding is `415`; unacceptable response negotiation is `406`; a well-formed, structurally valid but semantically unprocessable representation may be `422` only when no controlling `400` or `409` rule applies; current-state conflicts are `409`; precondition failures are `412`; missing required preconditions are `428`. A response that fails Glaux’s own contract is a server defect and returns `500`, never `422`. Diagnostics expose stable public codes and safe locations, not internal schema paths, authorization predicates, filesystem paths, secrets, or hidden resource existence. [N,P; IDR-SRV-013]

Initial runtime support should be deliberately capability-gated: full JSON structural and semantic validation; full SWE JSON value validation for supported component trees; SWE Text only after streaming codec parity; and SWE Binary only after complete bounded layout, datatype, byte-order, reference, length, compression/encryption, and WKB evidence. Unsupported encodings fail explicitly and are never advertised. GeoJSON wrong polygon winding should be diagnosed and canonically generated according to the right-hand rule, but not rejected solely for winding because RFC 7946 requires parsers to retain backward compatibility. [N,P]

Acceptance of this report establishes the taxonomy, pipeline, contract registry, resource/stage matrix, outcome and diagnostic rules, offline resolution posture, test lanes, security boundary, and downstream handoffs. It does not accept IDR-SRV-024, choose final Rust dependencies, authorize draft Part 3 implementation, or begin server implementation.

---

## 2. Scope and Plan Alignment

### 2.1 Completed Scope

This report:

- identifies authoritative and supporting schema, OpenAPI, media-type, encoding, profile, HTTP, and conformance sources;
- reconciles material published-artifact issues with the pinned `v1.0.0` release and September 14, 2026 upstream state;
- defines structural, semantic, profile, operational, security/policy, conformance, and interoperability validation as separate types;
- maps validation by resource family and by registration, update, import, ingestion, normalization, command, feasibility, response, CI, conformance, fixture, and golden-file stage;
- applies the accepted canonical graph, identity, relationship, temporal, provenance, status/event, SensorML, and SWE contract baselines;
- specifies strict-write, quarantine, normalization, diagnostic, audit, schema-cache, `$ref`, and DDIL behavior;
- distinguishes ordinary JSON from SWE JSON/Text/CSV/Binary validation and identifies phased capability gates;
- carries implementation, smoke-test, interoperability, and community evidence only as non-normative design/test input; and
- gives explicit handoffs to Categories E, F, G, and I without finalizing their decisions.

### 2.2 Deliberate Boundaries

This report does not:

- choose or pin a Rust JSON Schema, OpenAPI, GeoJSON, SensorML, SWE, XML, or binary-codec library;
- define the database schema, transaction implementation, queue product, or dead-letter mechanism;
- finalize units, observed properties, controlled properties, code-space governance, or semantic equivalence (IDR-SRV-024);
- finalize ingestion batching/backpressure, dynamic-data semantics, command lifecycle, feasibility algorithms, authorization, releasability, cross-boundary policy, or audit retention;
- publish a new public Glaux profile or repair an OGC artifact in place;
- claim executable SensorML or Connected Systems certification beyond available approved tests; or
- adopt or implement draft OGC API - Connected Systems Part 3.

### 2.3 Research Question Coverage Matrix

| Plan question | Coverage | Evidence location |
|---|---|---|
| CQ1 — validation responsibilities | Complete | Sections 5–14 |
| CQ2 — validation taxonomy | Complete | Section 6 |
| CQ3 — authority by resource and interaction | Complete | Sections 3, 5, 7, and 8 |
| CQ4 — staged application | Complete | Sections 8, 11, and 12 |
| CQ5 — downstream implications | Complete | Sections 13–16 |
| Detailed — CSAPI Part 1/Part 2, OAPIF, SensorML, SWE | Complete | Sections 5, 7, 9, and 10 |
| Detailed — OpenAPI, JSON Schema, GeoJSON, HTTP/media | Complete | Sections 5, 11, 12, and 13 |
| Detailed — outcomes, diagnostics, tools, DDIL, security | Complete | Sections 11–14 |
| Detailed — implementation and test lessons | Complete | Sections 12 and 17 |

---

## 3. Evidence Base and Authority Classification

### 3.1 Authority Order

No single linear order resolves every artifact conflict. Glaux applies this decision procedure:

1. identify the exact applicable approved standard, conformance class, requirement, and incorporated external standard;
2. identify the exact versioned schema or other artifact the approved standard makes applicable;
3. determine whether prose, conceptual model, artifact, and abstract test agree;
4. apply accepted AEP/STANAG profile constraints and accepted Glaux decisions without representing them as base-standard obligations;
5. when approved sources conflict, preserve the conflict, vendor artifacts unchanged, select a documented Glaux interpretation for runtime behavior, test both sides, disclose conformance impact, and monitor the upstream issue; and
6. use examples, executable tests, implementations, issue discussion, and community experience to understand or test the contract, never to invent a normative obligation.

| Class | Examples | Permitted use |
|---|---|---|
| Normative approved | OGC 23-001, 23-002, 17-069r4, 23-000, 24-014; RFC 7946, 9110, 9457; OAS 3.1.2; JSON Schema 2020-12 | Derive obligations and semantic meaning |
| Incorporated/versioned artifact | Tagged schemas and OAS files associated with CSAPI 1.0 | Validate the representation within the scope assigned by the standard; preserve exact provenance |
| Profile-controlling | Accepted AEP/STANAG findings | Narrow or add Glaux deployment requirements; label separately |
| Project-controlling | Accepted IDR decisions | Control Glaux architecture unless explicitly superseded |
| Conformance evidence | Abstract tests and an applicable official ETS | Test claims; does not override the governing requirement |
| Draft/post-publication | `master`, open issues/PRs, SensorML ETS development, Part 3 working draft | Compatibility and monitoring only |
| Informative | Examples, OSH, CS-Go, pygeoapi, SECD, client smoke tests, discussions | Risk, fixture, and interoperability input only |

### 3.2 Primary Evidence Inventory

| Source | Pin/status | Authority and use | Limitation |
|---|---|---|---|
| OGC API - Connected Systems Part 1, OGC 23-001 | Approved 1.0 | Part 1 resource, representation, operation, and conformance requirements | Published artifacts have recorded gaps/conflicts |
| OGC API - Connected Systems Part 2, OGC 23-002 | Approved 1.0 | Dynamic data, schema binding, status, event, control, command, feasibility requirements | OAS navigation coverage is incomplete |
| Official CSAPI repository | `v1.0.0`, commit `8e03b236…` | Reproducible schemas, modular OAS, examples, and tests | Repository metadata/history is not an erratum |
| OGC API - Features Part 1, OGC 17-069r4 | Approved 1.0 | Inherited feature, collection, GeoJSON, filtering, and conformance behavior | Only applicable inherited conformance classes control |
| SensorML 3.0, OGC 23-000 | Approved 3.0 | Sensor/process/deployment document semantics and JSON representation | Schema issue #183 remains unresolved |
| SWE Common 3.0, OGC 24-014 | Approved 3.0 | Component, constraint, quality, nil, JSON/Text/Binary encoding semantics | Published prose/schema discrepancies require seams |
| OpenAPI Specification | 3.1.2, published September 19, 2025 | Glaux API description model and Schema Object semantics | OAS validation cannot prove application conformance |
| JSON Schema Core and Validation | Draft 2020-12 | Schema dialect, identifiers, references, vocabularies, annotation/assertion behavior | Format assertion is vocabulary/configuration dependent |
| GeoJSON, RFC 7946 | Published standard | Geometry/object structural and semantic rules | Right-hand-rule compatibility rule precludes winding-only rejection |
| HTTP Semantics, RFC 9110 | Published standard | Representation metadata, negotiation, and status meanings | Glaux’s deterministic choices remain project profile decisions |
| Problem Details, RFC 9457 | Published standard | Safe machine-readable error envelope | Domain codes and disclosure remain Glaux-owned |
| Controlled AEP source | `AC/224(JCGISR)D(2026)0005`, April 27, 2026; digest above | Operational/profile context through accepted findings | Controlled; not quoted or redistributed here |

### 3.3 Reproducibility and Evidence Quality

- The official release baseline is the tag `v1.0.0`; later `master` remained `3fd86c73…` at the bounded recheck. [D]
- The shared history register remains Version 1.9 because no tracked material state changed during this topic. [I]
- Mechanical prior-IDR inventory found 69 tagged JSON Schema files across the applicable CSAPI/SensorML/SWE set; all declare Draft 2020-12 and none declares `$id`. The external manifest therefore supplies stable local identity without modifying vendor bytes. [I,P]
- The accepted OpenAPI study measured 20 Part 1 paths/39 operations and 23 Part 2 paths/48 operations. Published modular packages normalize to their tagged counterparts, while released bundles retain unresolved relative example/schema references and cannot serve as a reference-closed deployment OAD. [I; IDR-SRV-014]
- Official issue state was refreshed September 14, 2026 for #18, #43, #71, #87, #172, #174, #181, #182, #183, and #200. Open/closed status is evidence, not authority.
- No approved executable Connected Systems ETS was identified. CSAPI abstract tests remain the conformance source to translate and trace; OGC API - Features has an established official validator/ETS lane. SensorML ETS repository material is useful draft implementation support but not the sole authority for a SensorML 3.0 claim. [X]
- Current tool documentation informs feasibility only. Exact dependencies and versions remain for implementation planning and architecture decision records. [I]

### 3.4 Material Conflict Register

| Item | Published/release fact | Later state | Glaux validation disposition |
|---|---|---|---|
| JSON Schema dialect, issue #87 | Tagged schemas use 2020-12 | Issue closed after dialect selection | Compile as 2020-12; preserve as regression evidence |
| Regex, issue #18 | ECMA-262-compatible regex intended | Issue closed | Enforce supported ECMA-262 subset/behavior, bound execution, test anchoring explicitly |
| SWE Boolean flags, issue #71 | Prose reverses `false`/`true` object-versus-array meaning relative to schema/UML/examples | Issue closed with intended schema/UML direction | Use `false`/default = objects, `true` = arrays; retain contradiction fixture and interpretation record |
| Relative Quaternion, issue #43 | Published Pose schema follows intended unit-quaternion shape | Issue remains open against UML type | Validate published representation; monitor conceptual correction; do not invent alternative JSON |
| Feature `featureType`, issue #172 | Base feature requires it and specialized schemas narrow it | Open issue is not a demonstrated release defect | Keep requirement; add regression fixture against accidental removal |
| Procedure `validTime`, issue #174 / PR #199 | Tagged `procedure.json` omits SensorML `validTime` | Approved/unmerged repair direction; not in `master` | Glaux-owned wrapper/profile permits the normative optional property; vendor file remains exact; test both artifacts |
| Ordinary observation schema, issue #181 | Outer Observation JSON plus parent DataStream `resultSchema` | Open clarification | Treat `resultSchema` as SWE component description, not arbitrary JSON Schema; custom `+json` needs an explicit extension contract |
| `validTime` open bounds, issue #182 | Source permits date-time or `now`; prose speaks of two datetimes; `..` is an OAPIF query token | Open, with missing published target noted | Do not transfer `..` into representation values; apply accepted temporal adapter and explicit interpretation tests |
| `timeInstantOrNow`, issue #183 | Published SensorML closure omits referenced artifact | Open; ETS copy uses a narrower time type | Supply a manifest-controlled Glaux overlay only where required; preserve gap; do not call ETS copy normative repair |
| ReDoc, issue #200 | Published online rendering is broken | Open | Validate/download pinned source directly; never treat documentation rendering success as schema validity |

---

## 4. Validation Extraction Methodology

### 4.1 Extraction Unit

Each rule was reduced to a validation unit with: resource family; interaction stage; representation/media type; source and stable anchor; authority; validation type; selected contract; assertion; outcome; diagnostic exposure; normalization permission; security/policy seam; runtime/CI/conformance placement; fixture; and downstream owner.

### 4.2 Procedure

1. Pin approved documents, tagged artifacts, mutable evidence, and controlled-source digest.
2. Carry forward accepted IDR-SRV-001–022 decisions and identify non-negotiable invariants.
3. Build schema/OAS/media/encoding inventories without assuming artifact completeness.
4. Reconcile known issues to the published tag and current official repository state.
5. Map every resource family and interaction stage to the seven-type taxonomy in Section 6.
6. Separate checks that can be expressed in JSON Schema from typed/cross-resource/policy checks.
7. Select strict-write, quarantine, normalization, response, and evidence outcomes.
8. Cross-check runtime rules against CI, abstract-test, fixture, golden-file, and interoperability needs.
9. Record unresolved decisions at the narrowest downstream topic rather than preempting them.

### 4.3 Contract Selection Rule

The server selects a validation contract from trusted context:

`operation + route family + HTTP method + direction + negotiated media type/content coding + profile + parent revision/fingerprint + tenant/policy version`

Payload fields may participate only after the outer contract validates them. A client cannot switch validators, load a remote schema, weaken format checks, select an older stream contract, or gain a capability merely by naming it in the body. [P]

### 4.4 Evidence Record

Every state-changing validation decision should be reproducible from:

- exact input/source digest and byte length;
- selected schema/contract/profile identifiers and digests;
- compiled validator/codec version and options;
- route, operation, principal/policy reference, parent revision, and interaction stage;
- ordered findings with internal and disclosure-safe representations;
- deterministic normalization/transformation record;
- accept/reject/quarantine outcome and atomic unit; and
- correlation identifiers to provenance, transaction, audit, and generated response.

The evidence record is operational metadata, not a substitute for retaining exact imported source where the accepted source-fidelity policy requires it. [P; IDR-SRV-019,021–022]

---

## 5. Schema, OpenAPI, Media Type, and Encoding Inventory

### 5.1 Inventory Summary

| Domain | Applicable source | Runtime role | CI/conformance role | Key boundary |
|---|---|---|---|---|
| CSAPI Part 1 JSON/GeoJSON | OGC 23-001 + pinned schemas | Outer resource structure plus typed semantic validators | Schema examples, abstract tests, route/response probes | OAS examples are not the server contract |
| CSAPI Part 2 JSON/GeoJSON | OGC 23-002 + pinned schemas | Dynamic-resource envelopes, parent-schema rules | Schema/negative tests and translated ATS | Parent stream contract supplies value meaning |
| OGC API - Features | OGC 17-069r4 + RFC 7946 | Collection/item/GeoJSON rules where inherited | Official OAPIF ETS plus Glaux regression tests | Foreign members and winding need standards-aware handling |
| SensorML JSON | OGC 23-000 + pinned schemas | Import/write early gate, generated-view check | Closure, examples, round-trip, abstract tests | Schema alone cannot verify inheritance or CSAPI mapping |
| SWE component descriptions | OGC 24-014 + pinned schemas | Contract registration and compilation | Full component/constraint/nil/quality fixtures | Conceptual/schema gaps need overlays and typed checks |
| SWE JSON values | SWE 3.0 + parent contract | Compiled streaming/DOM value validator | Differential and round-trip corpus | JSON shape depends on flags and component order |
| SWE Text/CSV values | SWE 3.0 + TextEncoding | Streaming tokenizer/codec | Golden octets and negative boundary corpus | Separators, escaping, optional fields, blocks are contract-bearing |
| SWE Binary values | SWE 3.0 + BinaryEncoding | Disabled until capability proof, then bounded codec | Layout/endian/reference/length/WKB corpus | Declarations do not prove safe executability |
| OpenAPI | OAS 3.1.2 + generated Glaux registry | Documentation/discovery output; not sole middleware authority | Meta-schema, lint, reference closure, semantic parity, client generation | Schema annotations need explicit enforcement |
| Problem responses | RFC 9457 + IDR-SRV-013 | Error envelope and safe diagnostics | Contract, redaction, precedence tests | `status` matches wire; type URI identifies problem |
| AEP profile | Controlled accepted findings | Additional profile/policy gate | Controlled conformance/acceptance lane | Never disclose controlled text through diagnostics |

### 5.2 OpenAPI Position

Glaux uses OpenAPI 3.1.2 for its canonical generated OAD. OAS 3.1 patch releases are intended to remain tooling-compatible within 3.1, while the exact generated artifact and dialect remain pinned and regression tested. External JSON Schemas follow their own `$schema`; inline Schema Objects use the OAS base dialect unless a different dialect is deliberately selected. [N,P]

The accepted registry-first hybrid from IDR-SRV-014 remains controlling:

- typed operation/representation contracts generate Rust routing and extraction metadata, the OAD, conformance declarations, documentation examples, and parity tests;
- one complete deployment-specific OAD is published;
- source may remain modular, but the consumer artifact is reference-closed;
- exact OGC schemas remain vendor artifacts with provenance and digests;
- Glaux-owned schemas declare 2020-12 with stable absolute `$id`; and
- OpenAPI 3.0 may be generated only as a separately tested compatibility view, never by weakening the canonical contract silently.

OpenAPI meta-schema validation is necessary but insufficient. The official OAS schemas do not capture every specification violation, and specification text controls their interpretation. CI must additionally check unique/stable operation IDs, route-method parity, parameter serialization, status/media coverage, security declarations, conformance links, reference closure, examples, and generated-client smoke behavior. [N,P]

### 5.3 JSON Schema Position

Glaux-owned JSON schemas use Draft 2020-12. The external contract manifest records each vendor path, logical URI, media role, digest, source release, allowed dependents, and local resolver key. Vendor bytes are immutable. [P]

Required validator policy:

- validate schemas against the correct meta-schema before compiling instances;
- enable assertion behavior for a tested allowlist of formats used by the contract, including `uri`, `uri-reference`, and `date-time`, rather than assuming annotations reject input;
- implement cross-field and domain rules outside JSON Schema;
- preserve `$dynamicRef`, recursive references, and original resource identities when bundling;
- prohibit uncontrolled network/file resolution and schema-supplied executable code;
- bound recursion, instance depth, node count, string length, array size, regex work, errors collected, and validation time;
- recognize that `pattern` is not implicitly anchored and must behave compatibly with ECMA-262 expectations; and
- produce stable internal findings independent of library-specific prose.

### 5.4 Media Types, Content Codings, and Character Encoding

Media negotiation is validation, not a string equality check. Parse type/subtype and parameters case-insensitively where HTTP permits; compare parameter values according to their grammar; reject duplicate/conflicting parameters; enforce representation-specific required/allowed parameters; and retain the original header for audit. Unknown request `Content-Type` or unsupported `Content-Encoding` returns `415`; no acceptable response representation follows accepted IDR-SRV-012 behavior and returns `406`. [N,P]

For JSON-based media types, require a supported Unicode/JSON representation and reject malformed bytes, duplicate-member ambiguity under the project JSON parser policy, non-finite JSON numbers, excessive nesting, and trailing non-whitespace data before schema validation. A `+json` suffix says that the representation uses JSON semantics; it does not make an embedded `schema` value a JSON Schema or authorize arbitrary custom payload semantics. [N,P]

Content coding (for example compression) is distinct from SWE’s internal value encoding. Decode HTTP content codings under expansion and resource limits before representation validation; retain digest evidence for received and decoded forms. Encryption or compression declared inside SWE BinaryEncoding is part of the stream contract and needs separate capability/policy approval. [P]

### 5.5 GeoJSON Position

GeoJSON validation combines JSON structure, RFC 7946 object/member rules, finite coordinate values, minimum coordinate arity, geometry-specific nesting, bbox dimensional consistency, longitude/latitude range policy, CSAPI feature specialization, and application spatial-policy checks. Foreign members are not automatically invalid. Polygon rings should be generated in right-hand-rule orientation and wrong winding should produce a diagnostic/canonicalization opportunity, not rejection solely on winding, because RFC 7946 directs parsers not to reject such polygons for backward compatibility. [N,P]

CRS, precision, antimeridian, empty geometry, topology, and operational-area decisions that exceed RFC 7946 must be explicitly profiled. Geometry libraries cannot silently repair, snap, reproject, or discard coordinates without a recorded transformation. [P]

### 5.6 Artifact Completeness Findings

The tagged OAS files describe useful standard examples but are not complete deployment definitions. Accepted IDR-SRV-014 findings remain the baseline: Part 1 has 20 paths/39 operations; Part 2 has 23 paths/48 operations; required Part 2 Deployment-scoped streams, stream-to-Sampling-Feature/feature-of-interest, Feasibility, and canonical SystemEvent-item routes are absent while removed history paths remain. The roots identify OAS 3.1.0 and `info.version` 0.0.1, include demo/local servers, and lack security schemes and operation IDs. Glaux therefore validates them as pinned evidence and fixtures, never republishes them as its OAD. [I,X,P]

The release Part 1 bundle retains 32 unresolved relative example references; Part 2 retains 51 unresolved references (45 example, 6 schema). Redocly CLI 2.43.2 probes from the accepted study reported structural/style findings and recursion failures. Those counts are tool observations, not normative defect counts; they justify bounded multi-tool CI and reference-closure tests. [I]

---

## 6. Validation Type Taxonomy

| Type | Question answered | Examples | Mechanism | Failure meaning |
|---|---|---|---|---|
| Syntax/decoding | Can bounded bytes be decoded as the selected representation? | JSON grammar, UTF handling, HTTP coding, SWE tokenization | Bounded parser/decoder | Representation malformed |
| Structural | Does the decoded form match its declared shape? | required members, types, enum/const, component cardinality | JSON Schema plus typed parser | Shape violates selected contract |
| Profile | Does valid base-standard content satisfy Glaux/AEP deployment constraints? | supported classes, required identifiers, approved encoding subset | Versioned profile validator | Base-valid but profile-divergent |
| Semantic | Do values and references mean what the contract says? | unit/property binding, nil reason, temporal relation, feature type | Typed domain validator and registries | Meaning is inconsistent/unknown |
| Operational | Is the action valid against current state and capabilities? | parent revision, lifecycle, idempotency, command state, feasibility window | Transactional service/domain checks | Action cannot be applied now |
| Security/policy | May this principal cause or observe this effect? | authorization, classification, releasability, URI fetch, command safety | Policy decision/enforcement points | Denied or sanitized independent of syntax |
| Conformance | Does an implementation satisfy an applicable claimed requirement/class? | CSAPI ATS, OAPIF ETS | Traceable harness | Claim failure, not automatically one request failure |
| Interoperability | Can independent parties exchange complete meaning? | external clients, servers, round trips, negotiation | Scenario tests and semantic comparison | Compatibility gap with assigned owner |

Validation types compose but never collapse. In particular:

- schema validity is not semantic validity;
- semantic validity is not authorization or safety approval;
- feasibility is not command acceptance or success;
- a fresh HTTP response is not fresh domain data;
- conformance-test success is not production security assurance; and
- one implementation’s tolerance is not a standard requirement. [P; IDR-SRV-018,020,022]

### 6.1 Finding Severity and Disposition

| Severity | Meaning | Default disposition |
|---|---|---|
| Fatal | Cannot safely decode/select a contract or resource limits exceeded | Reject; do not continue dependent checks |
| Error | Applicable contract violated | Reject strict write; quarantine eligible import; fail CI/conformance |
| Warning | Interoperability, quality, deprecation, or non-controlling ambiguity | Record; strict write may accept only where policy explicitly allows |
| Information | Deterministic normalization or advisory observation | Record when material; no semantic repair |
| Internal defect | Generated response/artifact violates Glaux’s own contract | Abort/500, alert, preserve safe evidence |

Severity is versioned policy attached to a rule identifier. A library warning does not automatically become a Glaux warning, and a normative `MUST` violation cannot be downgraded merely to accommodate an implementation. [P]

---

## 7. Resource-Family Validation Strategy

The following matrix contains the plan-required fields. “Runtime” includes request, import, ingestion, and response stages as stated; “CI/ATS” includes contract and conformance lanes. Downstream IDs own unresolved detailed policy.

| Resource family | Interaction stage | Payload / representation | Schema / encoding source | Validation type | Authority | Runtime / CI / conformance role | Error / warning behavior | Security / policy | Tooling | Test implication | Downstream | Notes / unresolved |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Landing/API/Conformance | Generate/respond | JSON/HTML links and declarations | CSAPI P1/P2, OAPIF, generated OAD registry | Structural, semantic, conformance | N/P | Runtime generated-view invariants; route/OAD/claim parity in CI | Internal mismatch = 500; claim mismatch fails release | Filter links/claims by policy without false conformance | Typed serializers, registry diff | Link relation, media, auth-view goldens | 044, 050, 051, 056 | Do not advertise unimplemented encoding/Part 3 |
| OpenAPI definition | Generate/respond | OAS 3.1.2 JSON/YAML | OAS 3.1.2 + Glaux registry | Structural, profile, conformance, interoperability | N/P | Meta-schema + semantic CI; safe response checks | Generation mismatch blocks release; runtime invalid = 500 | Deployment servers/security and hidden routes filtered consistently | OAS model, meta-validator, linter, bundler | Reference closure, client generation, route parity | 044, 050–053, 056 | Canonical 3.1.2; optional tested 3.0 view |
| Collections | Write/respond | JSON | OAPIF/CSAPI schemas and typed model | Structural, semantic, profile, policy | N/A/P | Full write and generated response validation | 400 structural; 422 eligible semantic; 500 response | Ownership/classification/releasability separate | JSON Schema + domain validator | Negative filters, links, extent fixtures | 024–030, 039–041 | Extents derived with provenance, not blindly trusted |
| Systems | Register/update/import/respond | GeoJSON + SensorML-derived fields | CSAPI P1, SensorML 3.0, RFC 7946 | All runtime types | N/A/P | Strict public write; quarantine import; response invariant | Reject errors; warn/quarantine bounded legacy/profile divergence | Capabilities, position, contacts, classifications can be sensitive | Schema, GeoJSON, SensorML parser, graph validator | Exact/parsed/canonical/generated round trip | 024–030, 039–041, 053 | Preserve five layers; do not collapse status into description |
| Procedures | Register/update/import/respond | GeoJSON + SensorML | CSAPI P1, SensorML 3.0, Glaux overlay | Structural, semantic, profile | N/P/X | Same as Systems; explicit overlay lane | Tagged-schema-only result is not final; diagnose conflict | Procedure capability disclosure filtered | Vendor schema + wrapper/profile | `validTime` omission regression | 024, 028, 044, 050, 053 | Vendor `procedure.json` omits optional `validTime` |
| Deployments | Write/import/respond | GeoJSON/SensorML Deployment | CSAPI P1, SensorML, GeoJSON | Structural, relationship, temporal, profile | N/P | Validate membership, deployed systems, interval, links | 400 shape; 422 semantic; 409 current graph conflict | Membership/link visibility and operational context | Schema + graph/temporal validators | Deployment-scoped route and interval fixtures | 025–030, 039–043, 050 | OAS route omissions require normative overlay |
| Sampling features / FOI | Write/link/respond | GeoJSON feature | CSAPI/OAPIF/RFC 7946 | Structural, spatial semantic, relationship, policy | N/P | Validate geometry and typed relationships | Winding warning/canonical output; invalid coordinates reject | Location may be sensitive; spatial generalization after validation | GeoJSON parser + spatial checks | Foreign member, bbox, winding, antimeridian | 024–030, 039–043, 053 | Preserve exact source before authorized transformation |
| Sampling feature properties | Write/respond | JSON/linked vocabulary | CSAPI P1 + semantic source | Structural, semantic, relationship | N/P | URI syntax now; registry meaning deferred | Unknown meaning may reject/profile-warn by later policy | URI dereference denied by default | Schema + typed URI/reference resolver | Known/unknown/offline vocabulary fixtures | 024, 027–029, 042–043 | Semantic binding owned by 024 |
| DataStreams | Register/update/respond | JSON/GeoJSON + schema link | CSAPI P2 + SWE contract | Structural, semantic, relationship, temporal, profile | N/P | Compile immutable parent schema fingerprint before activation | Unsupported codec/error rejects; version conflict 409 | Schema disclosure and source access independently controlled | Schema validator + SWE compiler + graph validator | Revision, capability, link, negotiation fixtures | 024–034, 039–044, 053 | `live` not overall availability; revisions never silently rebind history |
| Observation schema resources | Register/respond | JSON schema wrapper, SWE component + encoding | CSAPI P2, SWE 3.0 | Structural, semantic, encoding, profile | N/P | Validate wrapper then component closure then compile codec | Invalid/unsupported = reject; import may quarantine | Treat component URIs/data URIs as untrusted | Vendor schemas + component compiler | Every component/flag/nil/encoding combination | 024, 028, 031–034, 044, 053 | Ordinary JSON `resultSchema` is SWE, not JSON Schema |
| Observations | Ingest/query/respond | JSON/GeoJSON; SWE JSON/Text/CSV/Binary | CSAPI P2 + exact DataStream contract | Syntax through policy | N/A/P | Per-record or declared atomic batch; streaming bounded value validation | Parent-schema violations = 400; no silent discard; response defect = 500 | Data values and location/classification filtered independently | Streaming parser/codec + domain validators | Boundary, malformed, stale schema, batch atomicity | 029, 031–035, 039–043, 050–056 | Binary disabled until evidence gate; exact contract fingerprint retained |
| Status / dynamic properties | Ingest/respond | Observation-like values/current projection | CSAPI P2 + DataStream contract + IDR-020 | Structural, semantic, temporal, operational | N/P | Validate evidence then derive current projection transactionally | Invalid evidence rejected/quarantined; stale valid evidence not “invalid” | Operational state can be highly sensitive | SWE validator + temporal/projection rules | out-of-order, stale, unknown, degraded fixtures | 029, 031–035, 039–043, 053 | Do not collapse validity, freshness, trust, and availability |
| System Events | Create/derive/respond | JSON/GeoJSON event record | CSAPI P2 + IDR-020 event model | Structural, semantic, temporal, relationship, policy | N/P/X | Validate durable fact and affected resource/revision | Unknown type/profile divergence explicit; never fabricate history | Events may reveal hidden changes/resources | Schema + event vocabulary/graph validator | event matrix, ordering, deletion, visibility | 028–030, 034–035, 039–043, 050–053 | Published item/schema gaps remain adapter seams |
| ControlStreams | Register/update/respond | JSON/GeoJSON + command schema | CSAPI P2 + SWE contract | Structural, semantic, relationship, operational, policy | N/A/P | Compile exact schema/capability before activation | Unsupported encoding/unsafe profile rejects | Existence and capabilities require tighter disclosure | SWE compiler + policy seam | revision, lifecycle, capability matrices | 024–030, 036–041, 044, 053–055 | Constraints are not authorization or safety |
| Command schema resources | Register/respond | JSON wrapper, SWE component + encoding | CSAPI P2 + SWE 3.0 | Structural, semantic, encoding, profile, security seam | N/P | Validate parameters/results/feasibility descriptions; compile | Reject invalid executable contract; quarantine import only | Component range reveals control affordance | Component compiler + capability registry | nested choices, optional/nil, result/feasibility schemas | 024, 028, 036–041, 044, 053–055 | Full preservation may exceed executable codec support |
| Commands | Submit/update/respond | JSON; SWE JSON/Text/CSV/Binary | CSAPI P2 + exact ControlStream contract | All runtime types | N/A/P | Validate atomically before durable acceptance/dispatch | Parent-schema violation = 400; conflict/state errors per 013/036 | Auth, safety, policy remain independent gates with non-oracular errors | Streaming codec + transaction/state/policy validators | auth/schema/state precedence and replay corpus | 029, 036–041, 044, 053–056 | Schema-valid never implies feasible, authorized, or safe |
| Command status/results | Ingest/respond | JSON and contract-bound result values | CSAPI P2 + command revision + IDR-020 | Structural, semantic, lifecycle, temporal | N/P | Validate allowed transition/source and exact result contract | Illegal transition conflict; invalid device result quarantined/alerted by later policy | Source authenticity and result releasability | State machine + SWE validator | duplicate, out-of-order, terminal, malformed result | 029, 036–041, 043, 053–056 | Never rewrite invalid device evidence as successful result |
| Feasibility | Request/evaluate/respond | JSON parameters/result | CSAPI P2 + ControlStream feasibility contract | Structural, semantic, temporal, operational, policy | N/A/P | Validate request contract then separately evaluate | Schema failure = 400; negative feasibility is a valid domain result | May reveal capability, policy, resource availability | SWE validator + async task/policy seam | deterministic/async, expired, denied, indeterminate | 029, 036–041, 053–056 | Feasible is not authorized, accepted, or executed |
| Links/references | All stages | URI/link objects | CSAPI/OAPIF/OAS + typed relationship model | Structural, semantic, relationship, security | N/P | Syntax local; controlled resolution; graph integrity transactional | Unresolved external may be represented, not dereferenced blindly | SSRF, existence leakage, cross-boundary traversal | Typed URI/link parser + allowlisted resolver | cycles, aliases, offline, hidden targets | 016–019, 026–030, 039–043 | Exact unresolved/deferred/denied states remain distinct |
| Source/document imports | Administrative import/promotion | JSON, GeoJSON, SensorML/SWE, legacy/external | Applicable pinned contract + import profile | All types | N/A/P | Preserve exact source, parse, validate, quarantine, explicit promotion | Findings retained; never active until all promotion gates pass | Separate ingest privilege, malware/content limits, classification | Sandboxed parsers, registry, evidence store | malformed, partial, controlled, unsupported, repair lineage | 028–031, 039–043, 053 | Quarantine is not public partial success |

---

## 8. Interaction-Stage Validation Strategy

### 8.1 Ordered Pipeline

The default stage order is:

1. **transport envelope:** request-line/header size, transfer/content coding, media parsing, authentication context, rate/work budget;
2. **bounded decoding:** syntax, charset/JSON handling, decompression ratio, token/element limits;
3. **contract selection:** route/method/direction/profile/parent fingerprint/capability;
4. **structural validation:** JSON Schema or exact encoding grammar;
5. **profile and semantic validation:** required Glaux/AEP rules, SWE meaning, URI/unit/property seams;
6. **relationship and temporal validation:** typed parents, identifiers, revision, interval/freshness evidence;
7. **operational and policy decisions:** lifecycle, preconditions, authorization, releasability, command/feasibility/safety seam;
8. **deterministic normalization:** canonical internal representation with recorded changes;
9. **transactional commit or quarantine:** atomic state/evidence/provenance/audit behavior;
10. **response projection and validation:** authorized representation, negotiated media, invariant check before emission.

Some early transport/security rejection may precede full parsing to protect resources and avoid existence disclosure. Public error precedence is deterministic but need not reveal every internal check performed. [P]

### 8.2 Stage Matrix

| Stage | Required checks | Allowed outcomes | Atomicity/evidence |
|---|---|---|---|
| Registration/create | Full pipeline against current route/profile and referenced parents | Accept or reject | Resource, relationship facts, schema binding, provenance, validation evidence commit together |
| Update/replace/patch | Full affected document plus patch/media/precondition rules and unchanged invariants | Accept, reject, conflict, precondition failure | Never validate only changed tokens when whole-resource invariant can change |
| Privileged import | Preserve exact bytes, malware/size gate, best-effort bounded parse and all applicable checks | Quarantine, reject transport threat, or promote through separate action | Source and findings immutable; quarantine cannot satisfy active references |
| Promotion | Revalidate exact quarantined revision under pinned/current approved policy chosen explicitly | Activate or remain quarantined | Record policy migration and new evidence; never mutate original source |
| Ingestion | Bound stream/batch, select exact parent contract, validate values/relationships/time/policy | Atomic accept/reject per declared unit; authorized quarantine/dead-letter only | No silent row loss; record schema fingerprint per accepted value unit |
| Normalization | Only deterministic representation-to-canonical conversions | Canonical output plus transformation evidence, or error | No inferred semantic repair; source preserved when policy requires |
| Command submission | Decode and schema-validate, then transactional state, authorization, safety, and dispatch eligibility | Reject, accept durable command, or defined async result | Never dispatch before durable accepted state/audit |
| Feasibility | Validate request parameters then evaluate separately | Valid positive/negative/indeterminate feasibility result or validation error | Negative feasibility is not a validation error |
| Response generation | Project policy-authorized view, serialize, check invariants/contract before bytes are committed | Emit or internal failure | Invalid response = 500/alert; do not blame client |
| CI/build | Schema meta-validation, reference closure, generated parity, examples, negative/differential/property tests | Pass/fail release gate | Produce pinned machine-readable reports |
| Conformance | Execute only applicable claimed classes with requirement traceability | Claim pass/fail/Not Applicable with evidence | Keep tool defects and implementation defects distinguishable |
| Fixture/golden generation | Validate source fixture and expected finding/outcome; deterministic serialization | Admit/reject fixture | Fixture metadata pins authority, contract, expected diagnostics, sensitivity |

### 8.3 Validation Policy Evolution

Validation policy is versioned. A policy change does not retroactively rewrite historical acceptance. Glaux records whether stored data was valid under its ingestion policy, whether it validates under a newer policy, and whether revalidation is required before a new use. Revalidation produces new evidence; it does not change exact source or earlier evidence. [P; IDR-SRV-019]

Schema/contract changes use immutable revisions. Existing observations and commands remain bound to the exact fingerprint used when accepted. New writes to a newer parent use the new contract. Migration is an explicit transformation with provenance and compatibility checks, never silent rebinding. [P; IDR-SRV-022]

---

## 9. SensorML and SWE Common Validation Findings

### 9.1 SensorML Five-Layer Validation

The accepted IDR-SRV-021 architecture remains controlling:

| Layer | Validation concern | Prohibited collapse |
|---|---|---|
| Exact source | bytes, media, digest, provenance, handling | Do not “fix” vendor/user source in place |
| Parsed document graph | syntax, schema, references, extension preservation | Schema-valid does not mean canonical/domain-valid |
| Canonical projection | typed identity, relationships, time, status/capability separation | Do not discard unmapped source or infer unsupported meaning |
| Generated view | deterministic mapping, audience/policy, schema and semantic invariants | Generated view is not the original source |
| Validation/transformation evidence | contracts, versions, findings, mappings, outcome | Evidence does not replace source or domain state |

Strict public SensorML/CSAPI writes must be structurally and semantically valid for the selected supported profile. Administrative import can quarantine partial, invalid, legacy, or extension-rich material. Generation validates after projection and after serialization so mapping defects cannot escape as client errors. [P]

SensorML validation beyond JSON Schema includes globally meaningful `uniqueId`, local ID/name scoping, type/class consistency, valid-time interpretation, inheritance resolution, typed relations, inputs/outputs/parameters, modes/configuration, deployment membership, reference frames, positions, contacts, security/legal constraints, and lossless unknown-extension preservation. Not every concept becomes an indexed canonical field. [N,P]

### 9.2 SWE Contract Compilation

A valid active SWE contract contains:

- exact source and immutable digest;
- parsed complete component tree and separate encoding descriptor;
- canonical typed component paths, names, order, definitions, units, constraints, nil mappings, quality, frames/axes, optional/updatable flags, associations, and extensions;
- parent DataStream/ControlStream resource revision;
- compiled bounded validator/codec plan;
- supported operation/direction/media capability flags; and
- validation/profile/tool version evidence. [P; IDR-SRV-022]

Compilation rejects duplicate field/item names, invalid component graphs, impossible cardinalities, unresolved required references, unsupported executable combinations, illegal constraints, ambiguous nil mappings, layout overflow, unsafe regex, and encoding/component mismatches. A definition may still be preserved in quarantine or as non-executable source, but it cannot advertise or drive unsupported runtime operations. [P]

### 9.3 Component and Value Rules

Glaux preserves and structurally validates the complete published component family: Boolean, Text, Category, Count, Quantity, Time; CategoryRange, CountRange, QuantityRange, TimeRange; DataRecord, Vector, DataChoice, DataArray, Matrix, Geometry; and top-level SWE DataStream descriptors. Runtime codec support is capability-gated by family, encoding, direction, and operation. [N,P]

Key typed checks include:

- scalar types, finite/non-finite conventions, and exact two-member range arrays;
- component constraints, enumeration/code-space shape, numeric bounds, significant bits/digits, and regular expressions;
- units and definitions being syntactically valid now while equivalence/governance remains with IDR-SRV-024;
- unique names and stable ordered `ComponentPath` addressing for DataRecord, DataChoice, Vector, arrays, and matrices;
- choice selection, array/matrix element counts, optional fields, and encoding flags;
- reference-frame and axis consistency;
- nil sentinel uniqueness and declared nil-reason URI;
- quality structure preservation despite the published simple-component JSON schema gap; and
- extensions preserved but never executed without an approved handler/profile. [N,P,X]

Five value states remain distinct: omitted optional component; present declared nil sentinel; present ordinary value; structurally invalid value; operationally rejected value. JSON `null` is not automatically a SWE nil. SWE’s `NaN`, `-Infinity`, and `+Infinity` string conventions apply only where the applicable numeric component encoding permits them; JSON numbers themselves remain finite. [N,P]

Constraints validate possible component values; they do not authorize commands, certify safe operation, prove feasibility, or represent current capability/status. Those later checks consume, but do not collapse into, contract validation. [P]

### 9.4 Encoding Rules

| Encoding | Initial posture | Validation requirements | Activation gate |
|---|---|---|---|
| SWE JSON | First-class runtime | Full tree, object/array flags, compact order, optional/nil/choice/array/matrix/geometry rules | Complete positive/negative/round-trip corpus |
| SWE Text / CSV profile | Second phase | Separator grammar, escaping, decimal/quote rules, block boundaries, optional fields, counts, WKT, exact octets | Streaming encoder/decoder parity and resource bounds |
| SWE Binary | Preserve but do not advertise initially | datatype, bit/byte length, byte order, block layout, references, base64/raw transport, WKB, compression/encryption declarations | Evidence-complete bounded codec and policy approval |
| XML/legacy | Import/preservation only unless separately profiled | Secure XML parsing, namespace/schema pin, entity/network disabled, mapping provenance | Separate supported profile; not SWE Common 3.0 claim by default |

The closed issue #71 contradiction is an explicit regression seam: `false` or omitted record/vector array flags mean object representation; `true` means arrays, following the published schema/UML/examples and documented intent. The issue #43 quaternion UML mismatch does not authorize a new JSON form. [X,P]

### 9.5 Parent Binding

For `application/json` Observation schema resources, `resultSchema` and optional `parametersSchema` are SWE component descriptions. The Observation outer object validates against CSAPI, and its `result`/`parameters` values validate through the compiled component contract. They are not arbitrary JSON Schemas. The same rule applies to JSON Command `parametersSchema`, `resultSchema`, and `feasibilityResultSchema`. [N,D,P]

For `application/swe+json`, `application/swe+text`, `application/swe+csv`, or `application/swe+binary`, `recordSchema` describes the complete value record and `encoding` controls the serialization. Format selection and encoding must agree. A custom `+json` format requires a versioned extension profile that defines the embedded contract, validation, security, and interoperability behavior; suffix semantics alone are insufficient. [N,P]

---

## 10. Dynamic-Data, Observation, Status, Event, Command, and Feasibility Validation Findings

### 10.1 Observation and Dynamic Data

Observation ingestion validates, in order: request representation; outer CSAPI Observation shape; typed parent DataStream relationship; exact immutable schema fingerprint; phenomenon/result time and accepted temporal rules; optional feature-of-interest relationship; parameters and result against the compiled contract; profile/semantic references; authorization/releasability; and transactional uniqueness/idempotency. [N,A,P]

CSAPI Part 2 explicitly assigns `400` to observation payloads violating the parent DataStream schema. This specific rule controls over a generic preference for `422`. A stale contract fingerprint, missing/changed parent, or idempotency collision may instead be an operational `409` or precondition error when the payload is otherwise valid; IDR-SRV-029/034 will finalize concurrency details. [N,P]

For batches, Glaux must publish the atomic unit. The default recommendation is atomic rejection for one request unless a later endpoint explicitly defines per-record outcomes. High-rate ingestion compiles contracts once, uses bounded streaming validation, and never silently drops invalid records. A dead-letter/quarantine path is only valid when explicitly authorized and observable; it is not a substitute for a successful client response. [P]

### 10.2 Status and Dynamic Properties

A valid status value is evidence for a separately derived current-state projection. Structural validity, event time, ingestion time, freshness, trust, availability, and current selection remain separate dimensions. Late or stale evidence may be valid and retained without becoming the current projection. Unknown, stale, degraded, unavailable, and invalid are not interchangeable. [P; IDR-SRV-018–020]

DataStream/ControlStream `live` validates its narrow stream-state meaning; it is not a server-wide or system-wide availability assertion. Projection updates validate causal input, source authority, temporal ordering, and policy before an atomic replacement. [N,P]

### 10.3 System Events

System Event validation covers event identity, type vocabulary, affected resource and exact revision, event/ingestion time, actor/source/provenance, typed relationship, payload contract, and disclosure policy. Derived events must identify the generating rule and evidence; imported events retain source fidelity. An event does not become history merely because a state changed internally unless the accepted event-generation matrix says so. [P; IDR-SRV-019–020]

The published missing/inconsistent SystemEvent item artifacts remain a documented adapter seam. Glaux builds from approved requirements plus a Glaux-owned schema/profile and marks the difference in tests and its OAD rather than silently filling the vendor tree. [X,P]

### 10.4 Commands

Command processing uses separate gates:

1. syntax/media and outer command structure;
2. exact ControlStream revision and compiled parameter contract;
3. semantic references, units, controlled properties, modes, and time windows;
4. current resource/lifecycle and concurrency/precondition state;
5. authorization, releasability, command policy, and safety controls;
6. durable command/audit transaction; and
7. dispatch eligibility and adapter validation. [P]

Failure at one gate does not prove or disclose the result of later gates. A schema-valid command can still be infeasible, unauthorized, unsafe, conflicting, expired, or unsupported by a device adapter. Conversely, a principal’s authority cannot make an invalid value conform to its parent schema. [P]

Part 2 explicitly assigns `400` to command parent-schema violations. Command status transitions and results validate against their own lifecycle and exact result contract; malformed external/device evidence is preserved through an authorized invalid-evidence path, not rewritten to success. IDR-SRV-036–038 own the detailed lifecycle, asynchronous behavior, safety, and audit design. [N,P]

### 10.5 Feasibility

Feasibility input first validates against the applicable ControlStream feasibility contract. Evaluation then returns a valid positive, negative, or defined indeterminate domain result. A negative feasibility result is not a validation error. Feasibility does not grant authorization, reserve capacity, guarantee later state, accept a command, or prove execution. [N,P]

Async feasibility tasks validate task identity, input fingerprint, contract revision, expiration, allowed state transition, result provenance, and access policy. Detailed timing and task semantics remain with IDR-SRV-037. The missing feasibility paths in tagged Part 2 OAS are covered by the Glaux normative route overlay and traceable tests. [X,P]

---

## 11. Error, Warning, Normalization, Rejection, Quarantine, and Audit Findings

### 11.1 Outcome Model

| Outcome | Meaning | Public strict write | Administrative import | Active-state effect |
|---|---|---|---|---|
| Accept | All applicable gates pass | Yes | Promotion result | Atomic active commit |
| Accept with recorded normalization | Only approved deterministic representation normalization occurred | Yes, with canonical response where applicable | Yes on promotion | Commit canonical state plus source/transformation evidence |
| Warning | Non-fatal quality/interoperability/deprecation finding | Only where versioned policy permits | Retain with quarantine/promotion evidence | Never waives an error |
| Reject | Request cannot be accepted under selected contract/policy | Standard problem response | Transport/security rejection possible | No partial active state |
| Quarantine | Source retained but not trusted/active | Not exposed as ordinary successful write | Authorized import only | Cannot satisfy refs, advertise capability, generate public views, or execute |
| Defer | Resolution/evaluation intentionally pending | Only via an endpoint whose contract defines async behavior | Possible analysis state | Not equivalent to validity |
| Audit | Immutable record of decision/effect | Always for material state/security decisions | Always | Evidence only; not a validation outcome by itself |

### 11.2 HTTP and Problem Details

Application-generated body-capable 4xx/5xx responses use `application/problem+json` and RFC 9457. The problem `type` URI is the stable primary identifier; `status` is advisory and must match the wire status. Extensions may carry a stable Glaux code, correlation ID, and bounded same-type error array. [N,P]

| Condition | Status | Notes |
|---|---|---|
| Malformed JSON/SWE syntax, invalid/undeclared query value, CSAPI-required parent-schema failure | 400 | Use safe source location; do not relabel parent-schema failure 422 |
| Authentication required/failed | 401 | Per authentication scheme; avoid schema oracle behavior |
| Authenticated but forbidden | 403 | May use concealment behavior where policy requires |
| Resource not found or intentionally concealed | 404 | Do not reveal hidden existence through validation details |
| No acceptable response representation | 406 | Accepted deterministic Glaux negotiation decision |
| Current-state/revision/idempotency conflict | 409 | Not a generic validation bucket |
| Supplied precondition failed | 412 | Preserve selected representation/current validator semantics |
| Unsupported request media type, parameters, or content coding | 415 | Different from syntactically malformed supported representation |
| Well-formed and structurally valid but semantically unprocessable | 422 | Only when no controlling 400/409 rule applies |
| Missing required precondition | 428 | Accepted concurrency baseline; detailed policy deferred |
| Request/work budget exceeded | 413/429 or applicable defined response | Choose by actual condition; no validator crash/leak |
| Generated response violates Glaux contract | 500 | Server defect; never blame client with 422 |

### 11.3 Stable Diagnostic Model

Recommended public codes include `invalid-syntax`, `schema-violation`, `profile-violation`, `semantic-violation`, `unsupported-media-type`, `not-acceptable`, `reference-unresolved`, `relationship-violation`, `temporal-violation`, `stream-schema-violation`, `schema-version-conflict`, `precondition-required`, `precondition-failed`, `policy-denied`, `command-rejected`, and `internal-response-invalid`. The final catalog remains a versioned API artifact. [P]

Each finding has an internal rule ID, public code, type URI, severity, stage, resource family, safe source location (`query`, `path`, `header`, or body JSON Pointer/component path), safe message, and correlation ID. Internal evidence can additionally retain schema URI/digest, keyword/component rule, actual/expected summaries, validator version, and policy decision reference. [P]

Do not expose filesystem paths, Rust/SQL details, stack traces, internal schema/cache URIs, secrets, sensitive values, hidden parent identity, authorization predicates, controlled vocabulary contents, classification labels not visible to the caller, or exhaustive alternatives that create an oracle. Multiple public errors may be returned only when bounded and of the same problem type; unrelated failures retain deterministic precedence. [P; IDR-SRV-013]

### 11.4 Normalization Rules

Allowed normalization is deterministic, semantics-preserving, idempotent, documented by profile, and testable. Examples include canonical media parameter ordering internally, URI parsing without changing identity, canonical generated JSON member ordering for goldens, and polygon orientation in generated GeoJSON where source preservation and transformation evidence remain intact. [P]

Normalization must not silently:

- invent required values, identifiers, units, nil reasons, relationships, or times;
- change a client-provided command value to fit a constraint;
- reinterpret unknown URI meaning or code-space equivalence;
- discard unknown extensions or source precision;
- rebind a value to a newer stream schema;
- make quarantined content active; or
- conceal a normative/profile violation.

### 11.5 Audit Minimum

Audit material includes actor/service identity, operation, target and parent revisions, input/output digests where permitted, selected contract and policy versions, outcome, public/internal code set, normalization/transformation IDs, transaction/correlation ID, and command/security decision references. Data minimization, classification, redaction, retention, and access controls apply to audit itself. IDR-SRV-041 owns the final audit schema and retention. [A,P]

---

## 12. Runtime, CI, Conformance, Fixture, and Golden-File Validation Findings

### 12.1 Validation Placement

| Lane | What must run | What must not be inferred |
|---|---|---|
| Runtime request | Bounded decode, selected structural/profile/semantic/relationship/temporal/operational/policy gates | Passing means only that this operation may proceed |
| Runtime response | Typed construction and invariants always; full schema validation in test/debug and configurable production canary/high-risk paths | A response validator cannot repair a bad response after bytes are sent |
| CI contract | Schema meta-validation, `$ref` closure, OAS structural/semantic parity, format tests, generated code/docs/client smoke | One linter count is not conformance |
| CI behavior | Unit, integration, property, fuzz, mutation, differential, negative, round-trip, resource-budget tests | Different validators agreeing does not prove standard meaning |
| Conformance | Requirement-to-test traced abstract tests; official OAPIF ETS where applicable; controlled profile lane | Tool failure and implementation failure remain distinct |
| Interoperability | Independent clients/servers, representation and negotiation matrix, semantic round-trip | Tolerating a peer quirk does not change canonical behavior |
| Release | All advertised operation/media/capability combinations proven; OAD/claims match executable state | Preserved-but-disabled encodings are not advertised |

Production response validation should combine cheap typed invariants on every response with full validation for high-risk/generated documents, pre-release runs, debug/test, and sampled canary operation. Any full-validator bypass must be measurable and cannot bypass typed construction or policy projection. Streaming responses validate headers/contract before commit and validate each item before emission; protocols must define how a late stream failure terminates without emitting fabricated success. [P]

### 12.2 Required CI Gates

1. Vendor-artifact digest and manifest identity check.
2. JSON Schema meta-schema and reference-closure compilation under no-network resolver.
3. Glaux schema `$id` uniqueness and vocabulary/dialect policy check.
4. Generated OpenAPI 3.1.2 meta-schema plus semantic lint.
5. Registry-to-route/extractor/serializer/status/media/security/conformance parity.
6. Every normative example as reference or expected-negative fixture after authority classification.
7. Boundary and negative corpus for every rule and public diagnostic.
8. SWE component/encoding cross-product capability tests, with unsupported combinations asserted absent from discovery.
9. Independent JSON Schema oracle comparison for a curated semantics corpus.
10. GeoJSON semantic/property tests and canonical generation goldens.
11. Fuzz/resource-exhaustion tests for JSON, URI, regex, recursion, SWE Text/Binary, compression, and multipart/streaming boundaries where supported.
12. Generated client compilation and smoke scenarios against the actual server contract.
13. Abstract-test requirement traceability and applicable OAPIF ETS.
14. Deterministic offline/DDIL build and runtime tests with network blocked.

### 12.3 Fixture and Golden-File Model

Every fixture records: stable ID; source/authority and license/handling; exact bytes/digest; resource family; stage; media/encoding; schema/contract/profile fingerprints; expected outcome; ordered stable rule IDs; public redacted finding; normalization/golden output if applicable; sensitivity; and provenance. [P]

Required fixture classes include:

- official positive examples that actually validate;
- official contradictory or broken artifacts as expected-negative/reference fixtures;
- minimum/maximum, missing/extra, wrong-type, invalid-format, and recursive cases;
- every GeoJSON geometry, bbox dimension, winding, foreign member, range, and antimeridian case;
- every SensorML class/mapping/source-layer round trip;
- every SWE component family, flags, nil/optional state, constraint boundary, aggregate nesting, and encoding;
- stale/unknown/revision-mismatch/out-of-order dynamic data;
- command schema/state/auth/safety precedence and feasibility separation;
- hidden-resource/redaction/error-oracle cases;
- offline resolver/cache corruption and schema-cycle cases; and
- external client/server interoperability scenarios with ownership-classified differences.

Golden files apply to deterministic generated representations, OpenAPI, problem details, normalization records, compiled-contract summaries, and validation evidence. They must avoid brittle library-specific error wording; assert stable Glaux codes, paths, order, and semantic content. [P]

### 12.4 Implementation and Community Lessons

The accepted IDR-SRV-014A–014G studies show recurring non-normative risks: incomplete route descriptions; implementation-specific permissiveness; mismatched media negotiation; silent empty results; schema/resource discovery failures; generated-artifact drift; partial dynamic/tasking coverage; and client assumptions that hide missing links or representations. These findings support registry parity, explicit capability discovery, ownership-classified interop failures, negative fixtures, and end-to-end semantic scenarios. They do not authorize copying any implementation’s validator behavior. [I,P]

---

## 13. Tooling, Schema-Cache, `$ref`, and Offline/DDIL Validation Implications

### 13.1 Tooling Architecture

Glaux needs categories, not one universal validator:

| Category | Required capability | Selection evidence for later ADR/IDR |
|---|---|---|
| Rust JSON Schema engine | Draft 2020-12, custom formats/keywords, structured findings, meta-validation, offline retriever, bounded compilation/execution | Corpus compatibility, safety, performance, maintenance, license |
| Independent CI oracle | Same targeted JSON Schema semantics through a separate implementation | Differential corpus and explainable differences |
| OpenAPI model/validator/linter | OAS 3.1.2 parsing, reference handling, semantic checks | Recursive/vendor corpus and route parity integration |
| SensorML/SWE parser/compiler | Complete preservation, typed graph, JSON/Text/Binary capability matrix | Full component/encoding fixtures and source round trip |
| GeoJSON/spatial validator | RFC 7946 structure plus bounded semantic checks | Precision/topology behavior is explicit and non-repairing |
| HTTP/media parser | Correct negotiation, media parameters, content codings | RFC edge corpus and smuggling/ambiguity resistance |
| Fuzz/property framework | Structured generation, shrinking, budget checks | Reproducibility and CI integration |
| Conformance harness | Requirement IDs, applicability, evidence, external ETS adapter | Traceability and offline execution |

The Rust `jsonschema` project is a plausible candidate because current documentation advertises Draft 2020-12, custom formats/keywords, structured output, meta-schema validation, and custom retrieval. It is not selected here. IDR-SRV-044 and implementation ADRs must pin and probe the exact release; default remote retrieval must not be enabled. [I,P]

### 13.2 Schema Registry and Cache

The registry separates:

- immutable vendor artifacts keyed by manifest logical URI, path, release, and digest;
- immutable Glaux-owned schemas/overlays with stable absolute `$id` and version;
- reference-closed compound bundles that preserve embedded resource identities;
- compiled validators/codecs keyed by contract fingerprint, dialect, engine version, options, and profile;
- capability metadata by resource, operation, direction, and media/encoding; and
- signed/verified distribution package metadata for offline nodes. [P]

Cache entries are content-addressed and immutable. A logical alias resolves through a versioned manifest; it never overwrites a historical artifact. Activation is atomic only after digest, signature/provenance, dialect, meta-schema, reference closure, recursion/resource, and test checks pass. Cache corruption fails closed for new validation while already compiled immutable entries remain usable according to operational policy. [P]

### 13.3 `$ref` Resolution

JSON Schema `$id` establishes resource identity and is not a command to fetch over the network. Resolution uses a prebuilt allowlisted registry. HTTP/file/data URI dereferencing is denied unless a separately approved import process retrieves and pins the content under size, type, redirect, address, certificate, digest, and provenance controls. [N,P]

Bundling must retain base URIs, anchors, dynamic scope, recursion, and output locations. Naive full dereferencing is prohibited because cycles and expansion can exhaust resources or change semantics. Resolver aliases, redirect chains, duplicate identifiers, case/percent-encoding tricks, and reference-depth limits require adversarial tests. [N,P]

### 13.4 Offline and DDIL Behavior

The build and runtime validation package contains every active contract and its closed dependencies. Startup verifies its manifest and precompiles or deterministically lazy-compiles contracts without network access. A missing dependency is a deployment/configuration failure, not a reason to fetch from the public Internet or skip validation. [A,P]

Disconnected operation retains exact contract fingerprints and policy versions. Synchronization later exchanges content-addressed artifacts and validation evidence; conflicting logical aliases never overwrite content silently. New schemas received while disconnected remain inactive until trust, provenance, compatibility, and local policy gates pass. IDR-SRV-042/043 own degraded and synchronization behavior. [A,P]

---

## 14. Security, Policy, and Releasability Validation Implications

Validation is an attack surface and an information-flow control seam. Inputs can trigger recursion, pathological regex, huge numeric/string/array structures, decompression bombs, binary length overflow, URI resolution, external references, expensive geometry, misleading media metadata, or parser differential behavior. All stages use explicit work budgets and fail without panics, partial commits, uncontrolled fetches, or unbounded diagnostics. [P]

### 14.1 Required Security Controls

- Authenticate and establish request budget before expensive validation where protocol permits.
- Select contracts from trusted route/parent context, never arbitrary payload URI.
- Disable schema/XML external entities, file access, network retrieval, executable extension hooks, and unsafe regex behavior by default.
- Bound bytes, decoded expansion, depth, nodes, strings, arrays, errors, references, compilation time, execution time, and cache growth.
- Treat schemas, examples, SensorML/SWE source, vocabulary documents, and generated code inputs as untrusted supply-chain content.
- Separate validation from authorization, command safety, classification, releasability, tenant isolation, and cross-boundary release decisions.
- Apply policy before exposing schema resources, capability matrices, errors, links, counts, timing differences, caches, logs, exact source, or generated views.
- Validate the policy-filtered response, not only the unfiltered internal object.
- Sign or otherwise integrity-protect deployable offline contract packages and record provenance.
- Avoid algorithm/key acceptance merely because SWE BinaryEncoding names compression or encryption.

### 14.2 Non-Oracle Error Behavior

The outward failure can be less specific than the internal finding. Hidden parent/schema existence, alternative permitted command values, classification constraints, and authorization branch decisions must not leak through status, message, error count, response timing, or cache behavior. The security policy owns concealment; validation still records an internal evidence result with appropriately restricted access. [P]

### 14.3 Releasability Layers

Policy applies independently to:

1. exact imported source;
2. parsed and canonical fields/indexes;
3. schema/component definitions and capability metadata;
4. dynamic values and command parameters/results;
5. validation findings and audit/provenance evidence;
6. generated OpenAPI, conformance declarations, links, and examples; and
7. caches, metrics, traces, and operational alerts.

Validation success at one layer does not authorize release at another. A generated redacted view must remain structurally and semantically valid after filtering; if mandatory visible structure cannot be satisfied without disclosure, omit/conceal the representation according to policy rather than emit invalid content. [A,P]

---

## 15. Downstream Topic Handoff Matrix

| Topic(s) | Binding handoff from IDR-SRV-023 | Decision deliberately left open |
|---|---|---|
| IDR-SRV-024 | Semantic validators are separate from structure; component `definition`, units, code spaces, observed/controlled properties need versioned registries and offline states | URI governance, equivalence, unit conversion, ambiguity policy |
| IDR-SRV-025–028 | Persist exact artifacts, manifests, immutable schema/contract revisions, fingerprints, validation/transformation evidence, quarantine state, and policy metadata | Database/product and physical layout |
| IDR-SRV-029–030 | Commit active resource, relationships, schema binding, provenance, evidence, and audit atomically; validate full affected invariants and preconditions | Transaction/isolation/idempotency/lifecycle mechanisms |
| IDR-SRV-031–033 | Use precompiled bounded streaming validators/codecs; define batch atomicity and explicit invalid-record handling; never silently discard | Write API, buffering, backpressure, time-series architecture |
| IDR-SRV-034–035 | Bind observations/status to exact stream revision; keep validity/freshness/current projection distinct; advertise only proven streaming/media capabilities | Detailed dynamic semantics and final Part 3 adoption/profile |
| IDR-SRV-036–038 | Parent-schema `400`; schema, feasibility, authorization, safety, acceptance, dispatch, lifecycle, result, and audit gates remain separate | State machine, async protocol, safety and command policy |
| IDR-SRV-039–041 | Contract selection cannot be client-controlled; validation is bounded/non-oracular; policy applies to source, schema, values, findings, and generated views | AuthN/Z, cross-boundary rules, audit schema/retention |
| IDR-SRV-042–043 | Complete signed/content-addressed offline contract package; no runtime fetch; synchronize immutable artifacts/evidence without alias overwrite | DDIL service semantics, trust exchange, conflict resolution |
| IDR-SRV-044 | Evaluate and pin Rust JSON Schema/OAS/HTTP/GeoJSON/SWE tooling against this corpus; keep independent CI oracle | Exact crates, versions, wrappers, performance choices |
| IDR-SRV-050–052 | Translate applicable ATS requirements; official OAPIF ETS lane; distinguish conformance from runtime/unit/security/interop validation | Harness implementation and Rust test architecture |
| IDR-SRV-053–055 | Fixture metadata/golden rules and mandatory negative/security/command cross-products are defined here | Corpus storage, generators, final scenario allocation |
| IDR-SRV-056 | Test external clients/servers by resource, stage, media/encoding, links, errors, and semantic completeness; assign defect ownership | Partner matrix and execution schedule |
| IDR-SRV-057 | Carry validation taxonomy, registry, pipeline, conflicts, outcomes, and risk gates into synthesis | Overall prioritization and roadmap |

---

## 16. Recommendations

1. **Adopt the versioned validation-pipeline architecture.**
   - Use trusted contract selection, ordered validator types, explicit outcomes, and immutable evidence.
   - Priority: High.

2. **Make a provenance-bearing contract registry the shared source for runtime validation, generated OpenAPI, discovery, conformance declarations, and tests.**
   - Prevent route/OAD/schema/capability drift while preserving exact vendor artifacts.
   - Priority: High.

3. **Use JSON Schema Draft 2020-12 with an explicit Glaux assertion and resource-limit profile.**
   - Enable only tested formats, add typed semantic checks, and treat annotations as annotations unless explicitly enforced.
   - Priority: High.

4. **Vendor and verify exact standards artifacts; add Glaux-owned wrappers/overlays rather than editing them.**
   - Maintain digest, source, conflict, and conformance provenance for Procedure `validTime`, missing SensorML closure, SystemEvent, OAS, and SWE gaps.
   - Priority: High.

5. **Disable uncontrolled `$ref`/URI/network resolution and ship a reference-closed offline package.**
   - Preserve resource identity and recursion; use content-addressed immutable caches and bounded compilation.
   - Priority: High.

6. **Bind every Observation and Command value to the exact immutable parent stream contract fingerprint.**
   - Never infer JSON Schema from ordinary `resultSchema`, custom `+json`, or payload-selected URIs; never silently rebind history.
   - Priority: High.

7. **Capability-gate encodings and operations.**
   - Begin with full JSON/SWE JSON evidence, add Text/CSV after streaming parity, and advertise Binary only after complete layout/security tests.
   - Priority: High.

8. **Adopt atomic strict writes and a separate privileged quarantine/promotion workflow.**
   - Quarantine preserves evidence but cannot become active or executable until explicit revalidation and promotion.
   - Priority: High.

9. **Adopt RFC 9457 diagnostics and the IDR-SRV-013 status/precedence rules.**
   - Stable public codes/locations, bounded same-type errors, restricted internal detail, and 500 for invalid server responses.
   - Priority: High.

10. **Validate at runtime, CI, conformance, and interoperability layers with different claims.**
    - Use meta-schema, semantic OAS parity, independent oracle, property/fuzz/differential tests, ATS traceability, and external scenarios.
    - Priority: High.

11. **Record normalization and policy filtering as transformations and validate the resulting representation.**
    - Never use normalization to repair meaning or policy to emit structurally invalid fragments.
    - Priority: High.

12. **Keep units/semantic binding, persistence, ingestion, command lifecycle, security, DDIL, and harness mechanisms with their named downstream topics.**
    - This report supplies their validation invariants without preselecting solutions.
    - Priority: High.

---

## 17. Risks, Constraints, and Open Questions

### 17.1 Risks and Mitigations

| Risk | Consequence | Required mitigation |
|---|---|---|
| Schema-as-conformance fallacy | Semantically invalid or unsafe state accepted | Seven-type taxonomy and typed validators |
| Standards artifact conflict | False rejection or false claim | Immutable vendor copy, interpretation record, overlay, dual fixtures |
| Validator/library differential | Environment-dependent acceptance | Exact pin/options, Glaux stable codes, independent curated oracle |
| Recursive/ref/regex/compression attack | CPU/memory/network exhaustion | Offline allowlist and explicit budgets/fuzzing |
| Over-strict profile | Rejects interoperable base-standard data | Separate base/profile result and versioned policy |
| Over-permissive import | Bad source becomes active | Quarantine and explicit promotion transaction |
| Silent normalization | Loss of meaning/provenance | Allowed-transform registry and evidence |
| Stream schema drift | Historical values reinterpret incorrectly | Immutable fingerprint binding |
| Diagnostic oracle | Hidden resource, capability, policy, or data leak | Public/internal separation and deterministic concealment |
| Capability overclaim | Clients select unimplemented encoding/path | Registry-driven discovery/OAD/release gates |
| Incomplete official ETS | False confidence in conformance | Translate ATS, trace requirements, distinguish tool status |
| Golden brittleness | Tool upgrades create noise or hide meaning | Assert stable project semantics, not library prose |

### 17.2 Constraints

- Approved CSAPI/SensorML/SWE artifacts contain known omissions and contradictions; Glaux cannot erase that evidence.
- The controlled AEP source constrains what can be reproduced publicly.
- Final tooling depends on Rust ecosystem probes and implementation performance evidence not performed here.
- Full SWE Binary and custom encoding support is unsafe to claim before evidence-complete codecs.
- Authorization and releasability can intentionally reduce diagnostics and representations after internal validation.
- Offline/DDIL operation forbids depending on live public schema or vocabulary retrieval.

### 17.3 Open Questions Assigned Downstream

1. Which unit/property registries, equivalence rules, and offline semantic states become normative for the Glaux profile? — IDR-SRV-024.
2. Which exact artifacts/evidence are stored inline, object-addressed, or regenerated? — IDR-SRV-025–028.
3. What are endpoint-specific batch atomicity, idempotency, and precondition policies? — IDR-SRV-029/031/034.
4. Which production responses receive full schema validation versus typed invariants plus sampling? — IDR-SRV-044/052, based on benchmarks and risk.
5. Which SWE Text/Binary combinations enter the first implementation milestone? — IDR-SRV-044 and roadmap synthesis.
6. What precise command validation precedence is externally observable under concealment policy? — IDR-SRV-036–041.
7. Which official SensorML/CSAPI executable tests mature before implementation and how are tool defects adjudicated? — IDR-SRV-050/051 refresh.
8. Does a later approved CSAPI maintenance release resolve issues #43, #174, #181–183, or #200? — shared-register monitoring; never silently roll forward.

No open question blocks acceptance of the architecture baseline.

---

## 18. Validation Against Plan Success Criteria

| Topic plan success criterion | Status | Evidence |
|---|---|---|
| Authoritative schemas, OAS, media types, encodings, sources identified | Met | Sections 3 and 5 |
| Official issue/repair history reconciled to release and authority | Met | Sections 3.3–3.4 and 5.6 |
| Validation types distinguished | Met | Section 6 |
| Resource-family and interaction-stage responsibilities mapped | Met | Sections 7 and 8 |
| CSAPI, OAPIF, SensorML, SWE, GeoJSON, JSON Schema, OAS, media/encoding implications | Met | Sections 5, 7, 9, and 10 |
| Runtime, CI, conformance, fixture, golden, interoperability boundaries | Met | Sections 8 and 12 |
| Error, warning, normalization, rejection, quarantine, audit behavior | Met | Section 11 |
| Implementation/community lessons incorporated as non-normative | Met | Sections 3 and 12.4 |
| Recommendations decision-usable and server-bounded | Met | Sections 1 and 16 |
| Downstream handoffs explicit | Met | Section 15 |
| References explicit and reproducible | Met | Section 19 and pinned metadata |

### Report Completion Checklist

- [x] Topic ID matches overall research plan index
- [x] Topic research plan is linked and aligned
- [x] Core and detailed research questions are covered or explicitly assigned
- [x] Findings are evidence-backed with reproducible references
- [x] Normative, profile, project, draft, implementation, and gap evidence are distinguished
- [x] Mutable sources identify release/tag/commit and access date
- [x] Controlled and ambiguous evidence limitations are explicit
- [x] Source findings, interpretation, and recommendations are distinguishable
- [x] Conflicts with accepted reports are reconciled
- [x] Executive summary is independently readable
- [x] Recommendations, risks, open questions, and handoffs are explicit
- [x] All plan success criteria are mapped
- [x] Plan-owner acceptance and date recorded

---

## 19. References

### 19.1 Project Governance and Accepted Baselines

- [IDR-SRV-023 research plan](../IDR%20Plans/idr-srv-023-schema-and-encoding-validation-strategy.md)
- [Glaux Server overall IDR research plan](../IDR%20Plans/overall-idr-research-plan.md)
- [Glaux Server goal and definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Research report template](../../../../../Governance/research-report-template.md)
- [OGC API - Connected Systems upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md)
- Accepted IDR-SRV-001 through IDR-SRV-022 reports in [`IDR Reports/`](./)
- Controlled AEP source `AC/224(JCGISR)D(2026)0005`, April 27, 2026, SHA-256 `56dc757b6e677b3584e3152a957849f21a24b22854f562613ff283a8b599da8c` (project-controlled access; not redistributed)

### 19.2 Approved Standards and Registries

- [OGC API - Connected Systems landing page](https://ogcapi.ogc.org/connectedsystems/)
- [OGC API - Connected Systems Part 1: Feature Resources, OGC 23-001](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems Part 2: Dynamic Data, OGC 23-002](https://docs.ogc.org/is/23-002/23-002.html)
- [OGC API - Features Part 1, OGC 17-069r4](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [OGC SensorML Encoding Standard 3.0, OGC 23-000](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC SWE Common Data Model Encoding Standard 3.0, OGC 24-014](https://docs.ogc.org/is/24-014/24-014.html)
- [OpenAPI Specification 3.1.2](https://spec.openapis.org/oas/v3.1.2.html)
- [OpenAPI official schemas](https://spec.openapis.org/oas/3.1/schema/)
- [JSON Schema Draft 2020-12 Core](https://json-schema.org/draft/2020-12/json-schema-core)
- [JSON Schema Draft 2020-12 Validation](https://json-schema.org/draft/2020-12/json-schema-validation)
- [GeoJSON, RFC 7946](https://www.rfc-editor.org/rfc/rfc7946)
- [HTTP Semantics, RFC 9110](https://www.rfc-editor.org/rfc/rfc9110)
- [HTTP Caching, RFC 9111](https://www.rfc-editor.org/rfc/rfc9111)
- [Problem Details for HTTP APIs, RFC 9457](https://www.rfc-editor.org/rfc/rfc9457)
- [IANA Media Types Registry](https://www.iana.org/assignments/media-types/media-types.xhtml)
- [Semantic Sensor Network Ontology](https://www.w3.org/TR/vocab-ssn/)

### 19.3 Versioned Artifacts, Tests, and Maintenance Evidence

- [Official CSAPI repository at `v1.0.0` / `8e03b236…`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)
- [Official CSAPI `v1.0.0` release](https://github.com/opengeospatial/ogcapi-connected-systems/releases/tag/v1.0.0)
- [Official CSAPI `master` snapshot `3fd86c73…`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f)
- [Official CSAPI issues](https://github.com/opengeospatial/ogcapi-connected-systems/issues)
- [Issue #18 — regular expressions](https://github.com/opengeospatial/ogcapi-connected-systems/issues/18)
- [Issue #43 — Relative Quaternion UML type](https://github.com/opengeospatial/ogcapi-connected-systems/issues/43)
- [Issue #71 — SWE array flags](https://github.com/opengeospatial/ogcapi-connected-systems/issues/71)
- [Issue #87 — JSON Schema 2020-12 migration](https://github.com/opengeospatial/ogcapi-connected-systems/issues/87)
- [Issue #172 — `featureType`](https://github.com/opengeospatial/ogcapi-connected-systems/issues/172)
- [Issue #174 — Procedure `validTime`](https://github.com/opengeospatial/ogcapi-connected-systems/issues/174)
- [Issue #181 — observation schema interpretation](https://github.com/opengeospatial/ogcapi-connected-systems/issues/181)
- [Issue #182 — `validTime` open-bound representation](https://github.com/opengeospatial/ogcapi-connected-systems/issues/182)
- [Issue #183 — `timeInstantOrNow` schema closure](https://github.com/opengeospatial/ogcapi-connected-systems/issues/183)
- [Issue #200 — ReDoc rendering](https://github.com/opengeospatial/ogcapi-connected-systems/issues/200)
- [SensorML 3.0 executable-test repository](https://github.com/opengeospatial/ets-sensorml30)
- [OGC API - Features executable-test repository](https://github.com/opengeospatial/ets-ogcapi-features10)

### 19.4 Informative Implementation and Tool Evidence

- Accepted IDR-SRV-014A through IDR-SRV-014G implementation, smoke-test, interoperability, and community reports in [`IDR Reports/`](./)
- [OS4CSAPI organization](https://github.com/OS4CSAPI)
- [OS4CSAPI client repository](https://github.com/OS4CSAPI/ogc-client-CSAPI_2)
- [SECD interoperability repository](https://github.com/Sam-Bolling/csapi-server-interop-secd)
- [CSAPI Explorer](https://ogc-csapi-explorer.pages.dev/)
- [Rust `jsonschema` documentation](https://docs.rs/jsonschema/latest/jsonschema/)

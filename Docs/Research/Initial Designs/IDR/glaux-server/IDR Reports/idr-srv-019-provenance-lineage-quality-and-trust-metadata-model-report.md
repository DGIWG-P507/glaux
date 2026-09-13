# Section 019: Provenance, Lineage, Quality, and Trust Metadata Model - Research Report

**Topic ID:** IDR-SRV-019<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-019 Provenance, Lineage, Quality, and Trust Metadata Model](../IDR%20Plans/idr-srv-019-provenance-lineage-quality-and-trust-metadata-model.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core questions, all detailed-question groups, all 6 methodology phases, and all success criteria<br>
**Methodology Used:** Authority-ranked synthesis of accepted IDR-SRV-001 through IDR-SRV-018; direct extraction from approved CSAPI 1.0 prose, requirements, schemas, examples, and tests; review of SensorML 3.0, SWE Common 3.0, W3C PROV-DM/PROV-O/PROV Constraints, W3C SSN/SOSA, W3C Data Quality Vocabulary, Data on the Web Best Practices, RFC 9110, RFC 9530, and current NIST hash guidance; and resource, activity, lineage, fidelity, quality, trust, exposure, persistence, and fixture analysis<br>
**Research Time:** Approximately 8 hours of AI-assisted execution on September 13, 2026<br>
**Official Standards Source Pin:** [`opengeospatial/ogcapi-connected-systems` `v1.0.0` at `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)<br>
**Shared Register Baseline:** OGC API - Connected Systems upstream-history register version 1.9; no tracked approved-standard or draft Part 3 state required a register change during this topic<br>
**Document Purpose:** Establish the encoding-neutral provenance, lineage, source-fidelity, quality-assertion, uncertainty, trust-evidence, exposure, persistence, and validation baseline that the Rust Glaux reference server must preserve<br>
**Author:** OpenAI Codex<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** September 13, 2026<br>
**Date:** September 13, 2026<br>
**Last Updated:** September 13, 2026

---

## Reading Guide and Evidence Labels

| Label | Meaning |
|---|---|
| **N** | Normative or standards-derived finding from an approved applicable source or incorporated artifact |
| **A** | Project-controlling AEP/STANAG adoption or operational-context finding carried from an accepted report |
| **P** | Glaux project decision or recommendation proposed for acceptance here |
| **I** | Informative implementation, test, interoperability, or community evidence |
| **D** | Official draft evidence useful for a future seam but not an approved requirement |
| **X** | Published ambiguity, evidence gap, or unresolved project decision |

The verbs **records**, **preserves**, and **exposes** are intentionally different. Glaux can retain evidence internally without placing it in a normal CSAPI representation, and it can expose a redacted projection without exposing the complete evidence graph.

---

## Table of Contents

1. Executive Summary and Decisions Requested
2. Research-Question Coverage
3. Primary-Source Inventory
4. Standards and Profile Requirement Map
5. Terminology and Boundaries
6. Resource and Artifact Coverage Matrix
7. Provenance-Producing Activity Matrix
8. Conceptual Provenance and Lineage Model
9. Source Fidelity and Transformation Record
10. Quality Assertion and Uncertainty Model
11. Multidimensional Trust Evidence and Decision Boundary
12. State-Change Provenance Rules
13. Exposure, Query, and Redaction Matrix
14. Persistence and Downstream Implications
15. Graph, Temporal, and Referential Invariants
16. Fixture and Test Corpus
17. Recommendations, Rejected Options, Open Questions, and Review Triggers
18. Validation Against Plan Success Criteria

---

## 1. Executive Summary and Decisions Requested

Glaux must be able to answer four different questions without substituting one for another:

1. **Provenance:** who or what influenced a resource or artifact, through which activity, from which evidence?
2. **Lineage:** how did the current or exported entity derive from earlier entities and transformations?
3. **Quality and uncertainty:** what bounded assertion or measurement describes fitness, error, completeness, or conformance for a stated subject and purpose?
4. **Trust evidence:** what is known, unknown, verified, failed, stale, or inapplicable about identity, authority, integrity, validation, provenance, freshness, and policy context?

The planning baseline is:

> **Every authoritative Glaux mutation produces an atomic, append-only provenance activity that identifies the exact input and output entities, observed and asserted actors, method and version, outcome, governing clocks, and evidence restrictions. Quality and trust remain scoped, attributable, time-aware evidence; they never collapse into an unexplained truth claim or scalar trust score.** [N,A,P]

The internal model should use W3C PROV's Entity–Activity–Agent grammar and relation meanings, while remaining storage- and encoding-neutral. A canonical resource revision, raw source artifact, normalized representation, schema revision, validation report, quality assertion, provenance bundle, and redacted export are entities. Registration, import, validation, transformation, enrichment, merge, correction, synchronization, redaction, archival, and deletion are activities. Human principals, organizations, authenticated clients, publishers, simulators, federated nodes, adapters, server components, migrations, and administrators are agents in roles. [N,P]

This is an internal canonical and optional-export model, not a claim that CSAPI Parts 1 or 2 define a generic provenance endpoint. CSAPI already carries useful domain context—UIDs, SensorML process descriptions, Observation procedure and sampling-feature associations, command sender, generated identifiers and times, and System Events—but it does not provide a complete operational lineage, validation, transformation, trust, or audit vocabulary. Tagged source also retains removed System History material that is useful design evidence but is not an approved 1.0 conformance class. Glaux must not invent core fields or link relations. Any public provenance profile must be separately capability-advertised and authorized. [N,X,P]

Quality assertions must identify the exact subject and revision, dimension, metric, method/ruleset, typed value and unit, evaluator, evaluation time, input evidence, coverage, limitations, validity/expiry, origin, and handling restrictions. Multiple or contradictory assertions coexist. SWE Common quality objects remain close to measurement values and schemas; metadata-quality assessments may map to W3C DQV concepts. Neither vocabulary supplies a universal list of quality dimensions or proves fitness for every use. [N,P]

Trust must remain multidimensional. At minimum Glaux records separate evidence for authenticated identity, asserted identity, authority scope, transport protection, content integrity, signature verification, structural validation, semantic validation, provenance completeness, freshness, source availability, operational health, quality/uncertainty, and policy context. Each dimension has an outcome such as `verified`, `failed`, `unknown`, or `not_applicable`, along with method, evaluator, time, evidence, limitations, and expiry. A later policy engine may consume those records and persist its policy version and rationale, but authentication, authorization, classification, releasability, command safety, and trust decisions remain owned by IDR-SRV-039 through 041 and tasking topics. [P]

Raw-source fidelity is risk-based. Glaux retains exact bytes when required for legal/accountability obligations, loss-prone sources, nondeterministic or noncanonical transformations, disputed content, quarantine, synchronization conflict, or reproducibility. Otherwise a strong digest plus an immutable, durable, access-controlled reference and sufficient acquisition metadata can be adequate. If the referenced bytes may disappear, a digest and URL are not sufficient for reproduction. Hashes are computed over explicitly named byte domains before transformations; every transformed artifact receives its own identity and digest. A digest can show equality to the bytes originally digested, but does not authenticate the source or prove semantic truth. [N,P]

The project lead is asked to accept these decisions:

1. adopt the canonical PROV-compatible internal model and relation semantics in §8;
2. require atomic resource-revision and essential-provenance persistence, with failed/rejected attempts routed to the audit boundary;
3. adopt the fidelity and transformation rules in §9;
4. adopt the structured quality assertion and multidimensional trust-evidence records in §§10–11;
5. adopt the public/administrative/restricted exposure boundary in §13;
6. adopt the invariants and fixture corpus in §§15–16; and
7. authorize only IDR-SRV-020 after acceptance; this report does not authorize server implementation, public extension design, database selection, retention periods, or draft Part 3 implementation.

---

## 2. Research-Question Coverage

### 2.1 Scope Completed

- Defined provenance, lineage, fidelity, quality, uncertainty, integrity, trust-evidence, trust-decision, authorization, and audit boundaries.
- Mapped every accepted canonical resource family and supporting artifact family.
- Defined actor distinctions and all requested provenance-producing activities.
- Defined derivation, revision, specialization, alternate, invalidation, replacement, supersession, aggregation, merge, redaction, and synchronization behavior.
- Defined source-fidelity, reproducibility, quality, trust, exposure, transaction, retention-class, DDIL, and test contracts.
- Reconciled the model with accepted identity, relationship, and temporal decisions.

### 2.2 Explicitly Out of Scope

- Physical database schema or product: IDR-SRV-025 through 029.
- Retention durations and legal schedules: IDR-SRV-030.
- Write endpoint and mutation protocol design: IDR-SRV-031 through 034.
- Streaming transport and draft Part 3 binding: IDR-SRV-035.
- Command, feasibility, tasking, and safety state machines: IDR-SRV-036 through 038.
- Authentication, authorization, classification, releasability, policy evaluation, and security-audit design: IDR-SRV-039 through 041.
- DDIL synchronization conflict policy and wire exchange: IDR-SRV-042/043.
- Exact public provenance media type, endpoint, profile, or link relation: later API/representation topics.

### 2.3 Core Question Coverage

| Question | Short form | Status | Evidence |
|---|---|---|---|
| CQ1 | Concepts required by resource family and processing stage | Complete | §§5–10 |
| CQ2 | Standards/profile obligations versus Glaux decisions | Complete within available evidence | §§3–4 |
| CQ3 | Entities, actors, activities, relations, validation, and transformation | Complete | §§7–9, 12 |
| CQ4 | Scoped quality and trust evidence without overstatement | Complete | §§10–11 |
| CQ5 | Persistence, API, security, lifecycle, sync, and testing contracts | Complete | §§13–17 |

### 2.4 Detailed-Question Disposition

| Group | Result |
|---|---|
| Terminology and authority | Defined and authority-labeled in §§3–5 |
| Resources and artifacts | Full matrix in §6 |
| Actors, sources, and activities | Actor and capture models in §§7–8 |
| Lineage and versions | Relation rules in §§8 and 12; invariants in §15 |
| Source fidelity and transformations | Decision algorithm and record in §9 |
| Quality and uncertainty | Structured models and coexistence rules in §10 |
| Trust boundaries | Separate dimensions and decision boundary in §11 |
| Registration and state change | Required evidence and edge cases in §12 |
| Exposure, query, and security | Layered projection and traversal rules in §13 |
| Persistence, retention, synchronization | Logical contracts and handoffs in §14 |
| Validation and testing | Invariants and fixture corpus in §§15–16 |

---

## 3. Primary-Source Inventory

### 3.1 Standards, Protocols, and Official Artifacts

| Source | Version/status | Authority | Stable anchors used | Access | Limits |
|---|---|---|---|---|---|
| [CSAPI Part 1](https://docs.ogc.org/is/23-001/23-001.html) | OGC 23-001, 1.0, approved 2025-06-02 | N | §7 Common/IDs/UIDs; §9 Systems; §11 Deployments; §13 Procedures; §14 Sampling Features; §15 Properties; CRUD and §19 encodings | 2026-09-13 | No generic provenance or quality API |
| [CSAPI Part 2](https://docs.ogc.org/is/23-002/23-002.html) | OGC 23-002, 1.0, approved 2025-06-02 | N | §§9–12 DataStreams/Observations, ControlStreams/Commands/Status/Results, Feasibility, System Events; CRUD/query and §16 encodings | 2026-09-13 | Domain fields are not a complete operational provenance model |
| [Official CSAPI source](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2) | tags `v1.0.0` and `v1.0-schemabundle`, commit `8e03b236...` | N/artifact | OAS, JSON schemas, examples, ATS | 2026-09-13 | Artifacts cannot silently create absent prose requirements |
| [SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html) | OGC 23-000, Version 3.0, approved | N | §7 core process concepts; §§8.2.2–8.2.7 metadata/constraints/references; §9.1.4 JSON metadata | 2026-09-13 | Describes processes and lineage context, not server transaction history |
| [SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html) | OGC 24-014, Version 3.0, approved | N | §§8.2.7–8.2.16 component semantics, units, time/frames, Quality union, nil values; JSON encodings | 2026-09-13 | Quality representation is intentionally general; no universal metric vocabulary |
| [W3C PROV-DM](https://www.w3.org/TR/prov-dm/) | W3C Recommendation, 2013-04-30 | N/semantic | §§5.1–5.6 Entity, Activity, Agent, generation, usage, derivation, revision, attribution, delegation, invalidation, bundle, specialization, alternate, collection | 2026-09-13 | Does not prescribe Glaux storage or public API |
| [W3C PROV-O](https://www.w3.org/TR/prov-o/) | W3C Recommendation, 2013-04-30 | N/semantic | Starting-point and expanded classes/properties | 2026-09-13 | RDF mapping is optional for Glaux |
| [W3C PROV Constraints](https://www.w3.org/TR/prov-constraints/) | W3C Recommendation, 2013-04-30 | N/semantic | uniqueness, ordering, disjointness, specialization, alternate constraints | 2026-09-13 | Glaux adds stricter operational invariants where labeled P |
| [W3C SSN/SOSA](https://www.w3.org/TR/vocab-ssn/) | W3C Recommendation, 2017-10-19; joint W3C/OGC work | N/semantic | Observation, result, procedure, sensor, feature of interest, observed property, times, stimulus | 2026-09-13 | Domain semantics, not security audit or source-authentication proof |
| [W3C DQV](https://www.w3.org/TR/vocab-dqv/) | W3C Working Group Note, 2016-12-15 | I/semantic | §§4.1–4.10 measurements/metrics/dimensions/annotations; §§6.2–6.4 provenance patterns | 2026-09-13 | Not a normative or complete definition/list of quality |
| [Data on the Web Best Practices](https://www.w3.org/TR/dwbp/) | W3C Recommendation, 2017-01-31 | N/best practice | §§8.4–8.6 provenance, quality, and versioning; original-source practices | 2026-09-13 | Web-data guidance, not CSAPI conformance |
| [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110) | Internet Standard, 2022 | N | §8.8 validators and conditional-request semantics | 2026-09-13 | ETag identity is scoped to a resource representation; not a general content identity |
| [RFC 9530](https://www.rfc-editor.org/rfc/rfc9530) | Standards Track, 2024 | N/protocol | §§2–3 `Content-Digest`/`Repr-Digest`; §§5–6 algorithms and security | 2026-09-13 | Digest fields do not authenticate an author or establish truth |
| [NIST FIPS 180-4](https://csrc.nist.gov/pubs/fips/180-4/upd1/final) and [NIST hash policy](https://csrc.nist.gov/projects/hash-functions) | Final 2015; revision planned; SHA-2/SHA-3 current | N/security guidance | approved SHA families and digest purpose | 2026-09-13 | Algorithm policy can change; recheck at implementation freeze |

### 3.2 Project and Controlled Inputs

The controlled AEP/STANAG baseline remains `AC/224(JCGISR)D(2026)0005`, dated 27 April 2026, SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`. It was not redistributed or newly quoted. Accepted IDR-SRV-001 through 003 establish the Part 1, Part 2, SensorML 3.0, and SWE Common 3.0 adoption boundary and an operational need for discoverable, understandable, linked, trustworthy, and secure context. That evidence supports retaining source, derivation, quality, uncertainty, and handling context; it does not authorize invented wire fields or establish a universal trust algorithm. [A]

Accepted IDR-SRV-015 supplies the canonical resource inventory. IDR-SRV-016 controls ResourceId, UID, canonical address, revision, replacement, deletion, and tombstone semantics. IDR-SRV-017 controls typed authoritative relationship facts and policy-aware traversal. IDR-SRV-018 controls valid/effective time, transaction time, domain clocks, pipeline clocks, request evaluation time, freshness, current/as-of behavior, and schema-revision retention. This report does not reopen those decisions. [P]

### 3.3 Evidence Limits and Authority Rules

1. Approved OGC and incorporated standards control conformance behavior.
2. W3C PROV controls the meaning of reused PROV terms; Glaux-specific restrictions are labeled project decisions.
3. W3C DQV is an informative interoperability vocabulary, not a mandatory quality taxonomy.
4. SensorML process history and SWE quality are domain metadata, not substitutes for server transaction provenance.
5. An example, implementation, discussion, signature, schema pass, authenticated connection, or TLS session cannot be elevated into proof of semantic truth.
6. The controlled profile is used only through findings already accepted or directly available under its handling rules.
7. Mutable cryptographic policy and CSAPI artifacts must be rechecked before implementation or conformance freeze.

---

## 4. Standards and Profile Requirement Map

| Source concept | What it establishes | Glaux consequence | Classification |
|---|---|---|---|
| CSAPI Resource IDs and UIDs | Server-local resource identity and globally meaningful real-world identity are distinct | Provenance subjects exact ResourceId/revision; external-source identity remains source-qualified | N,P |
| Tagged, removed System History material | The source repository preserves a history design, but accepted IDR-SRV-015/018 found it is not an approved 1.0 conformance class | Preserve internal historical revisions; do not claim the removed route/class as core conformance | X,P |
| CSAPI Observation associations | Observation binds stream, feature of interest, procedure/method, result, phenomenon/result time | Preserve these as domain facts and lineage inputs; do not reinterpret them as authenticated actor provenance | N |
| CSAPI Command `sender` | Identifier of person/entity submitting a command; server-read-only in tagged schema | Bind to observed authenticated/request context under later policy; do not trust a payload-supplied value or publish sensitive principal identifiers by default | N,P |
| CSAPI server-generated fields | IDs, derived stream extents, current status, and certain times are server controlled | Record the generating/defaulting activity and source-versus-server origin | N,P |
| SensorML core process model | Processes have inputs, outputs, parameters, methodology; descriptions support lineage and qualification | Preserve referenced process/method/version as domain lineage context | N |
| SensorML descriptive metadata | Identification, classification, validity, security/legal constraints, characteristics, capabilities, contacts, documentation, and history | Keep source values and restrictions; distinguish process history from Glaux ingest/edit history | N,P |
| SWE Common Quality union | Quality can be quantity, range, category, or text, defined and unit-bearing as appropriate | Preserve quality next to value/schema and retain vocabulary/definition URI; do not coerce all quality to a number | N,P |
| SWE nil values and frames | Missing/unknown reasons, units, and reference frames affect interpretation | Treat semantic binding and conversion as explicit transformation inputs; `unknown` is data, not absence | N,P |
| PROV Entity/Activity/Agent | Fixes the canonical provenance grammar | Adopt an encoding-neutral subset with optional PROV-O export | N,P |
| PROV generation/use/derivation | Connects outputs, inputs, and producing activities | Require exact revision/artifact endpoints and qualifying activity for material transforms | N,P |
| PROV revision | Revision is a derivation retaining substantial original content | Use only for same-identity revised versions; replacement is not revision merely because chronology exists | N,P |
| PROV primary source | Contextual relation, not a type of entity | Record who asserts the relation and domain convention; avoid universal “original source” flags | N,P |
| PROV invalidation | Start of destruction, cessation, or expiry of an entity | Apply to a revision/artifact availability state; keep ResourceId lifecycle/tombstone semantics distinct | N,P |
| PROV specialization/alternate | More-specific aspect versus alternate aspect of the same thing | Use deliberately for projections/representations; never infer causal derivation from alternateness alone | N,P |
| PROV bundle/collection | Named provenance sets can themselves have provenance; collections can have members | Support batch/bundle evidence without silently applying mixed context to every item | N,P |
| PROV Constraints | Entity/activity disjointness, ordering, uniqueness, and relation implications | Validate canonical graph plus stricter project invariants in §15 | N,P |
| SSN/SOSA | Observation/result/procedure/sensor/FOI/property/stimulus semantic graph | Use for domain semantic mapping where useful; do not treat it as transaction, actor, or authorization provenance | N |
| DQV | Scoped measurements, metrics, dimensions, annotations, and measurement provenance | Permit mapping of Glaux quality assertions; require explicit project/profile vocabularies | I,P |
| DWBP | Provide origins, changes, quality, source, and version history | Supports accessible provenance projections but does not mandate a CSAPI endpoint | N/best practice,P |
| RFC 9110 validators | Strong validators distinguish representations of one resource over time | Store ETag separately from artifact content identity; do not compare ETags across resources as hashes | N,P |
| RFC 9530 digest fields | Digests can cover message content or representation data | Record digest domain/algorithm; treat received protocol digest as evidence requiring verification | N,P |
| FIPS/NIST hash guidance | Approved SHA-2/SHA-3 families; SHA-1 transition | Default new Glaux artifact digests to SHA-256 or stronger policy-approved algorithms with agility | N,P |
| Accepted AEP context | Operational data must remain interpretable, traceable, and handled securely | Retain restrictions and uncertainty; no new public field is inferred | A,P |

### 4.1 Conformance Boundary

Glaux can claim CSAPI conformance only for actual CSAPI requirements and adopted profiles. The internal provenance model, DQV mapping, administrative lineage query, trust evidence, and redaction record are Glaux capabilities unless a later accepted profile assigns stronger authority. A future public extension must use an identified media type/profile, documented schema, advertised conformance or capability, and access-control policy; it must not alter core JSON members in a way that falsely appears standardized. [P]

---

## 5. Terminology and Boundaries

| Term | Glaux meaning | Must not be confused with |
|---|---|---|
| Provenance | Evidence about entities, activities, and agents that influenced an entity or assertion | A narrative log, truth, or authorization |
| Lineage | The derivation/revision/source path among exact entity versions and transformations | General CSAPI resource relationships |
| Source artifact | Exact received or acquired representation before Glaux semantic transformation | Publisher, agent, or canonical resource |
| Source | A contextual origin claim: artifact, endpoint, organization, system, publisher, or earlier entity, qualified by kind | Authenticated principal or primary source in every context |
| Entity | A revision, artifact, assertion, bundle, schema, vocabulary, representation, or other thing with fixed aspects in a provenance account | Mutable ResourceId abstraction or activity |
| Activity | Bounded process that uses, generates, validates, transforms, invalidates, or communicates entities | Resulting entity or business status |
| Agent | Something bearing responsibility for an entity/activity, with a role and possibly delegation | Unverified payload author string |
| Attribution | Responsibility relationship between an entity and agent | Proof of authorship or authority |
| Association | Responsibility relationship between an activity and agent, optionally with role/plan | Resource ownership |
| Derivation | Output depends materially on an input through transformation, update, or construction | Mere temporal succession, reference, or co-membership |
| Revision | Derivation whose output is a revised version retaining substantial content from the original | Replacement with different ResourceId |
| Specialization | A more specific entity sharing all aspects of a more general entity and adding specificity | Revision or arbitrary subset |
| Alternate | Two entities presenting aspects of the same thing | Equality, equivalence of bytes, or causation |
| Invalidation | Start of destruction, cessation, or expiry of an entity's availability/use | Resource deletion, valid-time end, or policy concealment in all cases |
| Quality assertion | Attributed, scoped claim or measurement about a named quality dimension/metric | Objective universal quality |
| Measurement uncertainty | Information about dispersion/error/interval for a measured or estimated result | Metadata quality or evaluator confidence |
| Confidence | A method-defined measure attached to a specific assertion/inference | Trust, accuracy, probability, or authorization by default |
| Integrity evidence | Evidence that bytes/state match a referenced digest, signature, validator, or protected transaction | Truth, authorship, authority, or freshness |
| Trust evidence | Structured observations relevant to a later purpose-specific trust/policy decision | A single trusted/untrusted status |
| Trust decision | A policy-versioned conclusion for a subject, purpose, evaluation time, and evidence set | Authentication fact or permanent source reputation |
| Audit record | Security/accountability record of attempted or completed action, including failed reads/writes | Data lineage; audit and provenance can cross-reference |
| Canonical resource | Stable Glaux identity defined by IDR-SRV-015/016 | Any particular revision or representation |
| Provenance bundle | Named set of provenance descriptions that is itself an entity | A guarantee that every member shares identical provenance |

### 5.1 Required Actor Distinctions

Glaux records separately:

- **authenticated principal:** identity established by the request/security mechanism;
- **asserted actor:** identity claimed in submitted content;
- **registered publisher/source:** entity authorized by configuration to submit within a scope;
- **responsible organization/principal:** party on whose behalf an actor operated;
- **software agent:** adapter, validator, normalizer, simulator, migration, federation client, command gateway, or server component with version/build identity;
- **source endpoint/device:** acquisition point or represented system, not automatically an accountable agent; and
- **operator/administrator:** human or service identity observed by Glaux, protected according to policy.

One value must never overwrite another. A payload's self-declared author cannot replace the server-observed authenticated principal. Delegation and roles are activity-scoped. [N,P]

### 5.2 Provenance Versus Domain Relationships

IDR-SRV-017's `implements`, `deployedSystem`, `featureOfInterest`, `procedure`, stream membership, command/result, and other domain relationships say what resources mean and how they are navigated. PROV derivation says how one exact entity was produced from another. A normalized Observation can therefore retain its CSAPI `procedure@link` while its revision was derived from a source message by a decoder activity. Neither relation implies the other. [P]

---

## 6. Resource and Artifact Coverage Matrix

### 6.1 Canonical Resource Families

| Family | Essential subject/evidence | Typical producing activities | Quality/uncertainty notes |
|---|---|---|---|
| System | exact description revision, UID/aliases, source document, relationships, actor/method, valid and transaction time | register, import, normalize, edit, correct, sync, retire/delete | description completeness, identifier consistency, positional/time uncertainty; SensorML characteristics are domain claims |
| Procedure | exact procedure revision, method/configuration source, referenced documents/schemas/vocabularies | register, import, normalize, version, correct, sync | calibration/performance assertions stay scoped to conditions and revision |
| Deployment | revision, participating systems/platform, domain interval, geometry, source evidence | register, update, correct, merge, sync | completeness and spatial/temporal accuracy; do not equate valid interval with provenance activity interval |
| Sampling Feature | revision, sampled/host relation, geometry source and transformation | register, derive, transform coordinates, correct, sync | positional accuracy, CRS, transformation method and uncertainty |
| Property definition | revision, definition URI, source vocabulary/version, semantic mapping | import vocabulary, bind, map, deprecate, replace | semantic consistency/completeness; mapping confidence separately attributed |
| DataStream | revision, System/FOI/Procedure/Properties, schema revision, formats, derived extents | register, infer defaults, update schema, aggregate extents | schema coverage, stream completeness, cadence/freshness as time-aware quality evidence |
| Observation | exact record, raw message/batch, stream/schema revision, FOI/procedure, source and pipeline clocks | receive, decode, validate, normalize, convert, enrich, insert, correct, replay, sync | SWE per-result quality/uncertainty; separate validation and metadata-quality assertions |
| Dynamic feature property/status record | exact record, target revision, schema, source clock and pipeline clocks | receive, validate, project current state, correct, replay | freshness/coverage and state-confidence evidence; vocabulary finalized in IDR-SRV-020 |
| System Event | exact event revision, affected resource links, occurrence time, reporter/source | receive, create, correlate, correct, publish | event confidence and completeness; not the mutation activity itself |
| ControlStream | revision, System/FOI/Procedure, command schema revision, capabilities/limits | register, update schema/capability, sync | capability quality is conditional; constraints and validity preserved |
| Command | exact command, stream/schema revision, target, parameters, observed sender, gateway | submit, authorize boundary, validate, dispatch, retry/replay | syntactic/semantic/safety validation are distinct; execution outcome is not input quality |
| CommandStatus | exact status report, command, reporter, report/receipt/commit times | receive, validate transition, persist, correct, publish | report authenticity, ordering, completeness; current status is a projection |
| CommandResult | exact result, command/status, result schema, source artifact | receive, validate, normalize, link/inline, correct | result quality/uncertainty handled like observations where applicable |
| Feasibility record | request, evaluated constraints and versions, result, evaluator, evaluation time | submit, evaluate, expire, supersede, correct | explicitly purpose-, rule-, state-, and time-scoped; never a permanent guarantee |
| Collection membership | collection revision or membership fact, member revision/range, source/actor | create, add/remove, import batch, derive view | completeness can be assessed; membership is not derivation |
| Relationship fact | exact typed fact, endpoints/revisions, validity, source/actor | assert, infer, validate inverse, correct, expire/delete | confidence/validation applies to the fact, not automatically its endpoint resources |
| External reference/cache entry | source-qualified identifier/URI, validators/digests, retrieval metadata, cached artifact | resolve, retrieve, revalidate, map, cache, fail, expire | availability/freshness/integrity distinct; remote attribution retained |

### 6.2 Supporting Artifacts and Administrative Evidence

| Artifact | Required provenance behavior | Retention dependency |
|---|---|---|
| Raw request/message/document | exact byte-domain identity, acquisition context, media type/encoding, digest verification result, restrictions | Retain bytes when §9.2 requires; otherwise retain digest plus durable reference and metadata |
| Decompressed/decoded/parsed form | new artifact entity derived through named activity | Retain when needed to explain/reproduce later normalized state |
| Canonical normalized representation | exact revision/representation, transformation inputs and software/rules | Must remain traceable to source and schema/vocabulary revisions |
| Schema revision | immutable version/digest and authority/source | Must outlive any retained record decoded or validated with it |
| Vocabulary/profile/ruleset snapshot | immutable identifier/version/digest and source | Must outlive dependent semantic mappings, quality, trust, or policy decisions |
| OpenAPI/conformance document | generator/source version, build inputs, output digest | Retain reproducible release artifacts; not every runtime response |
| Validation report | subject revision/artifact, validator/ruleset version, findings and severity | Retain with dependent decision or according to audit/quality policy |
| Quality assertion | exact subject/scope, metric, evaluator, evidence, time/status | Preserve contradictory/superseded assertions as policy allows |
| Trust evidence/decision | subject/purpose, dimension, method/policy version, evaluator, time/expiry | Sensitive; retain according to security/accountability requirements |
| Quarantine record | rejected artifact reference, reasons, observed source, handling restrictions | Raw payload may need shorter/restricted retention; audit pointer can outlive it |
| Migration record | input/output revisions, migration and schema versions, batch and exceptions | Must support rollback/explanation and cross-version interpretation |
| Export/redacted projection | source entities, policy/projection version, omitted/generalized fields, output digest | Retain enough to reproduce exactly what was released |
| Tombstone/deletion attestation | subject identity, last revision pointer where permitted, deletion activity, authority/reason class | Must prevent sync resurrection and explain absence without exposing deleted content |
| Provenance bundle/batch envelope | member identities, common acquisition/activity context, exceptions | Shared facts retained once only if each member has an unambiguous binding |

### 6.3 Inheritance Rules for Batches and Streams

A batch or stream can hold shared source, acquisition, transport, actor, schema, method, and activity context only when all bound items actually share it. Each accepted item still needs an exact membership/binding and its own validation outcome, identity, and exceptional fields. Mixed transformations, partial rejection, per-item signatures, different source clocks, or different handling restrictions prohibit blind inheritance. Absence of an item-level override means “use the explicitly linked shared value,” not “guess from the nearest parent.” [P]

---

## 7. Provenance-Producing Activity Matrix

| Activity | Uses | Generates/affects | Minimum additional evidence | Failure behavior |
|---|---|---|---|---|
| Acquire/receive | transport/session/source context | raw artifact or receipt evidence | received time, endpoint/channel, media/encoding, byte length, digest, observed principal/source | Audit rejected/failed acquisition; quarantine bytes only under policy |
| Register/create | submitted artifact and request context | new ResourceId and initial revision | observed/asserted actors, reason, source kind, schema/profile, commit transaction | No generated authoritative entity on rollback |
| Import/bulk load | source bundle, manifest, schema/vocabulary | member revisions and batch report | batch ID, item binding, common context, per-item disposition | Record partial results and exact rejected members |
| Decode/decompress/parse | exact source artifact | intermediate artifact | codec/format/version, parameters, byte domains, warnings | Preserve failure location and input identity |
| Structural validation | artifact/revision and schema | validation assertion/report | schema ID/digest, validator/build, outcome/findings, evaluation time | Failure does not prove maliciousness or false semantics |
| Semantic validation/binding | parsed content, vocabularies, domain rules | validation/binding assertion | ruleset/vocabulary versions, field scope, evidence and limitations | `unknown` distinct from pass/fail |
| Normalize | parsed entity | canonical revision/representation | mapping/ruleset/software versions and field-level source/server/derived origin | Source is never overwritten or reattributed |
| Transform/convert | input entity, parameters, frames/units | derived entity | algorithm/version, parameters, deterministic flag, environment/seed if material, uncertainty propagation | Failed output is not generated authoritative state |
| Enrich/infer | source facts, rules/model | derived assertions/relationships | inference rule/model/version, confidence definition, inputs | Mark inferred origin; do not present as supplied fact |
| Aggregate/project | member revisions and snapshot | extent, latest/current view, summary | operator, snapshot/watermark, authorization view, evaluation time | Derived cache can be rebuilt; provenance records basis |
| Update/patch/replace | prior revision, submitted change | new same-identity revision | before/after revision IDs, patch/request artifact, concurrency token, reason | Conflict/duplicate/rejection is audit evidence, not a revision |
| Correct | incorrect revision and corrective evidence | corrected same-identity revision | correction reason/category, affected scope, effective/transaction time, attribution | Earlier revision remains addressable internally per policy |
| Merge | all material inputs and conflict decisions | merged entity/revision | selected/rejected claims, conflict rationale, precedence policy/version | Do not hide losing claims; partial merge explicit |
| Replay/idempotent retry | previously seen artifact/request | reference to prior outcome or new attempt record | idempotency/dedup key, original activity, receipt time, outcome | Must not create duplicate generation or domain record |
| Synchronize | remote bundle/revisions/tombstones | local imported revisions, conflict branches, mappings | origin node, remote IDs/sequence/version/hash, hops, local receipt/commit, dedup/conflict decision | Never rewrite origin as local authorship |
| Redact/project for release | protected entity and policy | release-specific derived representation | projection/policy version, release context, omitted/generalized categories, output digest | Do not leak protected values in reason, count, ID, timing, or graph shape |
| Archive/restore | entity and lifecycle policy | archive artifact/state or restored revision | storage tier/reference, integrity check, authority/reason, times | Restore is new activity; no silent history rewrite |
| Delete/invalidate | resource/artifact and authorization/policy | tombstone/invalidation/deletion attestation | target revision, scope, authority, reason class, dependency result | Physical erasure may be delayed/restricted; lineage references remain safe |
| Migrate/reindex/rebuild | old schema/store and revisions | migrated representation/index/cache | migration build, before/after checks, batch exceptions | Rebuild of derived index does not claim new domain fact |
| Publish/export | exact revision/projection | representation/message artifact | media/profile, authorization projection, publication time, digest, destination class | Publication failure does not invalidate committed domain state |

### 7.1 Minimal Successful Mutation Envelope

Every successful authoritative mutation must bind, in one logical transaction:

- immutable `activity_id` and activity kind;
- request/correlation and idempotency identifiers where applicable;
- observed authenticated principal, registered source, asserted actor, software agent, and delegation/roles as distinct optional references;
- start/end or occurrence time as available plus received, committed, and published clocks under IDR-SRV-018;
- exact used input artifact/entity/revision references;
- exact generated revision/entity references and any invalidated entity;
- method, software build, schema, vocabulary, ruleset, and policy versions materially used;
- outcome and diagnostics reference;
- reason or reason class where required;
- origin classification: `source_asserted`, `server_observed`, `server_defaulted`, `derived`, `inferred`, or `migrated`; and
- classification, releasability, retention class, and integrity metadata.

Fields that do not apply are absent or explicitly `not_applicable`; fields that should exist but are unavailable are `unknown` with reason. The two states are not interchangeable. [P]

---

## 8. Conceptual Provenance and Lineage Model

### 8.1 Canonical Node Types

| Node | Identity and purpose |
|---|---|
| `ProvenanceEntity` | Immutable entity ID plus type, exact subject/revision/artifact locator, content identity where applicable, validity/transaction context, origin, and restrictions |
| `ProvenanceActivity` | Immutable service-wide activity ID, kind, times, outcome, correlation, method/version, reason, and restrictions |
| `ProvenanceAgent` | Stable source-qualified agent identity and kind; sensitive observed identities can use opaque internal IDs |
| `AgentRole` | Activity/relation-scoped role such as submitter, publisher, operator, validator, normalizer, simulator, gateway, synchronizer, administrator, or responsible organization |
| `ProvenanceRelation` | Typed edge with exact endpoints, optional activity/role/time, source assertion, and restrictions |
| `ProvenanceBundle` | Named set/version of statements with attribution and its own provenance |

The implementation can store these as typed relational tables, append records, or another indexed form. PROV-O/RDF export is optional; a graph database is not required. [P]

### 8.2 Core Relation Mapping

| Glaux relation | PROV correspondence | Use rule |
|---|---|---|
| generated by | `wasGeneratedBy` | Output entity/revision to the activity that fixed its aspects |
| used | `used` | Activity to every material input entity |
| derived from | `wasDerivedFrom` | Output to input when transformation/update/construction materially depends on it |
| revision of | qualified `wasDerivedFrom` with `prov:Revision` | Same canonical identity and substantial retained content only |
| attributed to | `wasAttributedTo` | Entity responsibility claim; source and confidence retained |
| associated with | `wasAssociatedWith` | Activity-agent participation with role/plan |
| acted on behalf of | `actedOnBehalfOf` | Activity-scoped delegation; preserves both delegate and principal responsibility |
| invalidated by | `wasInvalidatedBy` | Entity ceased to be available/usable; separate from domain validity/lifecycle |
| specialization of | `specializationOf` | Exact revision/projection fixes additional aspects of a more general entity |
| alternate of | `alternateOf` | Different aspects/representations of same thing; never causal by itself |
| primary source | qualified derivation `prov:PrimarySource` | Contextual, asserted relation with domain convention/evaluator |
| had member | `hadMember` | Batch/bundle/collection membership; no derivation implied |
| informed by | `wasInformedBy` | Activity communication where required, including cross-process handoff |

### 8.3 Canonical Resource, Revision, and Representation

The stable ResourceId is the identity anchor from IDR-SRV-016; each committed state is a distinct immutable provenance entity. A representation generated from revision `r7` is another entity specialized to the requested media type/profile/projection and derived from `r7` through a serialization or redaction activity when that detail matters. Different JSON, SensorML, GeoJSON, SWE, compact binary, and redacted forms can be alternates or specializations without being byte-equal. [N,P]

`revisionOf(r7, r6)` requires a real revision relationship. Chronology alone does not justify it. If resource B supersedes or replaces resource A with a different ResourceId, the domain relationship and replacement reason remain explicit; the project must not misuse PROV revision to collapse their identities. [P]

### 8.4 Complete, Partial, and Disputed Lineage

Lineage completeness is recorded as an assertion over a stated boundary: for example, “complete from Glaux receipt to revision r7,” not “complete provenance.” Unknown remote preprocessing is represented as an explicit frontier or unresolved source, never filled with inferred edges. Competing bundles or source claims may coexist. A merge activity can derive a new bundle from them without rewriting either account. [N,P]

### 8.5 Provenance of Provenance

Quality measurements, validation reports, trust evidence, redacted exports, and provenance bundles are entities with their own generating activity and attribution. Corrections to provenance create a new assertion/bundle revision and invalidate or supersede the prior assertion as appropriate; they do not rewrite history silently. [N,P]

---

## 9. Source Fidelity and Transformation Record

### 9.1 Artifact Identity

Each retained or referenced artifact records:

- artifact ID and artifact kind;
- exact byte-domain description (`received-content`, decompressed payload, decoded frame, canonical form, representation data, or another named domain);
- media type, content coding, character encoding, schema/profile, byte length;
- digest algorithm identifier and digest bytes;
- digest source (`computed_by_glaux`, `received_content_digest`, signature manifest, remote metadata) and verification outcome;
- acquisition method, source-qualified URI/endpoint, remote validator/version where available;
- received and source times with precision/uncertainty under IDR-SRV-018;
- storage/durable-reference locator, availability state, and last verification;
- classification, releasability, legal/retention class, encryption/key-reference metadata; and
- source signature object and verification evidence when present, never just a Boolean `signed` flag.

Glaux should use SHA-256 at minimum for new interoperable artifact digests unless current project/NIST policy selects a stronger SHA-2 or SHA-3 algorithm. The algorithm is always stored, and migration supports multiple digests. SHA-1 is not used for new collision-resistant identities. [N,P]

### 9.2 When Bytes Must Be Preserved

Retain exact source bytes when any of the following applies:

- an accountability, evidentiary, contractual, legal, controlled-data, or profile rule requires them;
- the external source is mutable, ephemeral, offline, deletion-prone, or not under an enforceable preservation commitment;
- parsing, canonicalization, transcoding, decompression, coordinate/unit conversion, semantic mapping, or schema migration may lose information;
- the transform is nondeterministic, environment-dependent, model-dependent, or cannot be reproduced from retained inputs;
- validation failed, content is disputed, the record is quarantined, or correction/merge/conflict review needs the original;
- a multi-hop DDIL/synchronization process needs non-repudiable comparison of what a node actually received; or
- the original is required to verify a signature whose byte serialization matters.

A digest plus reference is sufficient only when the bytes are durably immutable and retrievable under Glaux's access/retention requirements, the digest covers the exact intended byte domain, the transformation is reproducible from retained dependencies, and no stronger obligation applies. A URL plus hash to content that can disappear is integrity metadata, not reproducibility. [P]

### 9.3 Transformation Record

| Field | Rule |
|---|---|
| `transformation_id` | Stable activity ID; unique and immutable |
| `kind` | decompress, decode, parse, canonicalize, normalize, map, convert, filter, redact, aggregate, enrich, infer, migrate, serialize |
| `inputs` / `outputs` | Exact entity/artifact IDs; all material inputs and outputs |
| `method` | Stable algorithm/process URI or controlled identifier |
| `implementation` | software component, semantic version, build/commit, and relevant dependency/model identity |
| `parameters` | Canonical parameter set, including unit/CRS/frame, rounding, locale, precision, filtering, and redaction profile |
| `semantic_dependencies` | exact schema, vocabulary, registry, profile, calibration, mapping, and ruleset revisions |
| `determinism` | deterministic, deterministic-with-environment, seeded, nondeterministic, or unknown |
| `environment` | only reproducibility-relevant runtime/platform/library values; avoid secrets |
| `field_origins` | source-provided, observed, defaulted, normalized, converted, derived, inferred, redacted; field map or compact shared rules plus exceptions |
| `times` | activity start/end and transaction clocks; source/domain clocks remain on their facts |
| `agent/role` | responsible software and configured/observed human or organization roles |
| `outcome` | success, partial, failed, rejected, quarantined, rolled_back, duplicate; diagnostics reference |
| `integrity` | input/output digest records and verification results |
| `limitations` | known loss, uncertainty propagation, unsupported values, approximation, or unreproducible factors |
| `restrictions` | classification/releasability/retention and policy version |

### 9.4 Required Transformation Semantics

- Hash exact received bytes before decompression or normalization; hash each retained output independently.
- Canonicalization is an activity and never changes what bytes were received.
- Unit and coordinate transformations identify source/target unit/frame, algorithm/grid/calibration, precision, rounding, and propagated uncertainty.
- Semantic mappings preserve source term, target term, vocabulary revisions, mapping relation, method, and confidence/limitations.
- Redaction produces a new derived entity; the protected relationship remains internal.
- Serialization can be omitted from persistent lineage for routine ephemeral responses unless release accountability, signed export, caching, or exact reconstruction requires it.
- A cryptographic signature records signed-byte domain, algorithm, key/certificate identifier, verifier, verification time, certificate/revocation/time evidence, and outcome. A valid signature binds bytes to a key under those conditions; it does not establish the claimant's authority or the content's truth. [P]

### 9.5 Fidelity Tests

Required tests perform exact-byte recovery where promised; source-to-normalized-to-source semantic round trips where lexical recovery is not promised; digest mismatch detection; decompression/canonicalization domain separation; signature invalidation after byte change; missing external artifact behavior; retained schema/vocabulary replay; and comparison of deterministic output digests across supported environments. [P]

---

## 10. Quality Assertion and Uncertainty Model

### 10.1 Quality Assertion Record

| Field | Required meaning |
|---|---|
| `assertion_id` | Stable immutable ID |
| `subject` | Exact resource revision, artifact, field/component path, relationship fact, collection/batch scope, or time-bounded population |
| `origin` | source-asserted, server-measured, inherited-by-explicit-binding, derived, inferred, or human-reviewed |
| `dimension` | Versioned URI/controlled identifier such as completeness, conformance, consistency, timeliness, positional accuracy, temporal accuracy, provenance completeness |
| `metric` | Versioned metric/rule definition; never only a display label |
| `value` | Typed Boolean, number, range, category, structured value, or text; include unit/vocabulary/data type |
| `method` | Procedure/ruleset/software/model and version used to calculate or assert it |
| `evaluator` | Agent and role; asserted versus observed identity preserved |
| `evaluation_time` | Trusted evaluation instant and transaction snapshot; source-asserted time if different |
| `validity/expiry` | Applicability interval, reevaluation trigger, or expiration where meaningful |
| `evidence` | Input entity/artifact/validation references and their revisions |
| `coverage` | Field, sample, population, count, spatial/temporal extent, and exclusions |
| `uncertainty/confidence` | Method-defined structure and interpretation, not an unlabeled number |
| `status` | current, superseded, invalidated, disputed, or withdrawn within the assertion lifecycle |
| `limitations` | Missing inputs, assumptions, known bias, applicability constraints, or incomplete lineage |
| `restrictions` | classification, releasability, retention, and exposure class |

This record can map to DQV `QualityMeasurement`, `Metric`, `Dimension`, or `QualityAnnotation` for an optional semantic export, but DQV's Note status and open taxonomy mean Glaux must version the actual metrics and vocabularies it adopts. [I,P]

### 10.2 Initial Dimension Families

| Dimension | Example metric or assertion | Important boundary |
|---|---|---|
| Structural conformance | JSON Schema/encoding/profile pass with rule set and findings | Pass does not prove semantic correctness |
| Semantic consistency | definition/unit/frame/relationship compatibility | Depends on vocabulary and rule versions |
| Completeness | required/expected fields, members, samples, lineage stages | Denominator and expected population must be explicit |
| Timeliness/freshness | evidence age against named versioned policy at evaluation time | Not HTTP cache freshness or truth |
| Positional accuracy | error, tolerance, covariance, or category under stated frame/method | Preserve frame, unit, confidence meaning, and conditions |
| Temporal accuracy | clock precision, synchronization error, interval uncertainty | Separate source clock from server receipt/commit clocks |
| Measurement quality | accuracy, variance, tolerance, probability, category, or operator note | Prefer SWE value/schema-local quality representation |
| Provenance completeness | covered stages/sources relative to a stated boundary | Never claim universal completeness |
| Referential integrity | resolved/unknown/broken endpoints at snapshot | Authorization can make a reference intentionally opaque |
| Availability | source/artifact retrievability observed at a time | Not content quality or resource lifecycle |
| Fitness for use | evaluator- and purpose-specific conclusion | Must name purpose/policy/evidence; cannot become global truth |

### 10.3 Measurement Uncertainty

SWE Common's Quality union supports quantity, range, category, or text and lets quality vary per measurement in the data stream. Glaux preserves that structure close to the applicable data component or schema, including definition URI, unit, reference frame, conditions, confidence interpretation, and nil-value reasons. “Precision,” “accuracy,” “probability,” “variance,” “tolerance,” and “confidence” are not interchangeable. [N,P]

Metadata-quality assertions about a dataset or revision use the canonical assertion record. When an ingestion transform changes units, frames, precision, or sampling, the transformation record states how uncertainty was propagated or that it could not be. An unknown uncertainty is represented as unknown, not zero. [P]

### 10.4 Coexistence and Aging

Assertions from publisher, Glaux validator, administrator, federation partner, or consumer can coexist even when contradictory. Glaux does not overwrite them into one value. A consumer or later policy selects relevant assertions using subject, scope, authority, evaluator, method, time, validity, and purpose. Changes to the subject revision, schema, vocabulary, calibration, ruleset, source authority, or governing policy can expire or trigger reevaluation; they do not retroactively erase the earlier assertion. [P]

### 10.5 Public Quality Representation Boundary

Normal CSAPI representations expose only quality/uncertainty already defined by the applicable standard/encoding or an accepted profile. Additional Glaux quality metadata requires an advertised optional profile or linked representation. Compact flags may be derived only when each flag has a stable vocabulary definition, scope, evaluator, evaluation time, and link/reference to detail. An unlabeled `valid: true` is rejected because it cannot distinguish syntax, schema, semantics, freshness, authority, or fitness. [P]

---

## 11. Multidimensional Trust Evidence and Decision Boundary

### 11.1 Trust Evidence Record

Each record contains `evidence_id`, exact subject and scope, dimension, evidence/claim type, outcome, observed/asserted origin, evaluator/issuer agent, collection activity, effective/evaluation/expiry times, method and version, evidence artifact references, confidence definition if applicable, limitations, revocation/supersession state, restrictions, and transaction metadata. Allowed outcomes begin with:

- `verified`: the named check succeeded under the recorded method and time;
- `failed`: the named check failed;
- `unknown`: relevant evidence is unavailable or indeterminate;
- `not_applicable`: dimension does not apply to this subject/purpose; and
- `not_evaluated`: evidence may exist but the named check was not performed.

`unknown`, `not_evaluated`, and `failed` are materially different. [P]

### 11.2 Required Independent Dimensions

| Dimension | Evidence examples | What it does not prove |
|---|---|---|
| Authenticated identity | credential subject, mechanism, issuer, assurance context, session observation | That payload assertions are true or authorized for every scope |
| Asserted identity | document author, device/source ID, command sender claim | Authentication or accountability |
| Authority scope | publisher registration, delegated role, permitted resource/operation/interval | Correctness of a particular statement |
| Transport protection | TLS/session/channel protection and peer verification | Origin or integrity before the protected hop |
| Content integrity | computed/received digest, immutable validator, transaction checksum | Authorship, authority, semantics, freshness |
| Signature verification | signed-byte match, key/certificate/path/revocation/time evidence | Claim truth or authorization beyond key/policy binding |
| Structural validation | parser/schema/profile results | Semantic validity or operational safety |
| Semantic validation | units, vocabularies, relationships, domain rules | Source honesty or fitness for every use |
| Provenance completeness | covered stages and known frontier | Correctness of represented lineage |
| Freshness/timeliness | named policy evaluation and evidence age | Accuracy or availability |
| Availability/health | reachability, service/device health observation | Freshness or truth of stored historical data |
| Quality/uncertainty | scoped assertions from §10 | Authorization or a global trust result |
| Policy context | classification, releasability, purpose, mission, tenant, decision policy version | Objective or permanent trustworthiness |
| Behavior/conflict history | prior validation failures, conflicts, revocation, anomaly evidence | Automatic guilt or authority to deny without policy |

### 11.3 Trust Decision Boundary

A trust decision is downstream and purpose-specific. If persisted, it records subject/scope, requested purpose/action, evaluation time/snapshot, decision (`allow`, `deny`, `indeterminate`, `defer`, or domain-specific result), policy identifier/version, evidence IDs considered, obligations/redactions, rationale code, decision agent, and expiry/recheck trigger. IDR-SRV-039/040 owns policy and authorization semantics; IDR-SRV-041 owns audit/accountability; IDR-SRV-036 through 038 own command-safety decisions. [P]

No trust-evidence writer grants access. No quality score becomes an authorization predicate except through an explicit versioned policy. No global source reputation silently changes past facts. Revocation or policy change produces new evidence/decision state at a new evaluation time while retaining the historical context under policy. [P]

### 11.4 Cryptographic and Validation Non-Overstatement Rules

- A hash comparison means that the checked bytes match the recorded digest under the named algorithm; the digest's own trusted origin remains separate.
- A valid signature means the signed bytes verify under a key and recorded verification context; identity binding, key status, signing time, authority, and semantic truth remain separate.
- Authentication means the mechanism established control of a credential/identity for a session; it does not validate self-asserted payload fields.
- TLS protects a transport hop; it does not establish the pre-hop origin of existing content.
- Schema conformance means the artifact met the identified structural rules; it does not establish correct measurements, honest assertions, safe commands, or fitness for use.
- Fresh evidence can be wrong, stale evidence can remain historically correct, and complete lineage can describe a low-quality result. [N,P]

---

## 12. State-Change Provenance Rules

### 12.1 Registration, Create, and Import

A first revision is generated from the submitted/source artifact through registration/import and validation/normalization activities. ResourceId allocation, UID/alias decisions, defaults, server-generated links/times, schema/vocabulary bindings, and actor distinctions are recorded. A bulk import uses a batch entity plus item bindings and per-item outcomes. A rolled-back create produces no authoritative revision but can produce restricted audit/quarantine evidence. [P]

### 12.2 Update, Patch, and Replace

An accepted update or patch creates a new immutable revision under the same ResourceId and derives it from both the prior revision and submitted change artifact. It records concurrency token, changed field origins, validation, reason, valid/effective time, and transaction time. Full HTTP replacement of a representation is still normally a new same-identity revision; it is not necessarily a new canonical resource. A domain replacement under IDR-SRV-016 uses a different ResourceId and explicit replacement/supersession relationship. [P]

### 12.3 Correction and Supersession

A correction never erases the earlier committed value. It produces a new revision, records corrective evidence and reason category, and identifies whether valid/effective time is retroactive while transaction time remains later. The earlier revision may be superseded for current projection or invalidated for use, but remains available internally subject to policy. “Supersedes” must state whether it is a domain relationship, assertion lifecycle, representation obsolescence, or provenance invalidation. [P]

### 12.4 Merge and Conflict

A merge activity uses every material input revision or assertion, generates a new revision/bundle, and records the conflict set, selection/rejection decisions, precedence/policy version, human/software agents, and unresolved fields. Losing claims remain traceable. A result can be a revision of one input only when PROV revision semantics genuinely apply; it is derived from all material inputs. A no-op merge can record a decision without generating a new domain revision. [N,P]

### 12.5 Duplicate Submission and Replay

An idempotent replay or duplicate receipt records a new attempt/receipt and links to the original activity/outcome. It does not generate the same authoritative entity a second time or advance domain state. A changed payload under the same idempotency key is a conflict and must not be silently linked as equivalent. Replayed source event time, local received time, remote sequence, and commit time remain distinct. [P]

### 12.6 Deletion, Archival, and Restore

Logical deletion changes ResourceId lifecycle and generates a tombstone/deletion attestation; an activity may invalidate the exact active revision or artifact. Physical erasure follows IDR-SRV-030 and security/legal policy. Essential identity, origin, deletion authority/reason class, time, and anti-resurrection sync evidence can outlive content when permitted. References to erased evidence resolve to a policy-safe tombstone or explicit unavailable state, never a dangling guess. Restore is a new activity and revision, not removal of the deletion event. [P]

### 12.7 Redaction

Redaction is a transformation producing a derived projection with its own digest, policy/profile version, release context, and generation activity. Internally, the link to protected input remains. Externally, the service can disclose only that a policy-governed projection was produced; reason text, hidden identifiers, exact timestamps, counts, graph degree, and stable correlations must not reveal protected facts. [P]

### 12.8 Failed, Rejected, Partial, Quarantined, and Rolled-Back Activities

These attempts belong primarily to security/operational audit and ingestion diagnostics. They may create restricted raw artifacts, validation reports, quarantine records, or an attempt activity, but do not generate an authoritative resource revision. Partial batch success names accepted and rejected members. A rollback record references the attempted transaction and cleanup outcome; it cannot imply that rolled-back state was publicly committed. [P]

---

## 13. Exposure, Query, and Redaction Matrix

### 13.1 Exposure Classes

| Evidence | Normal standards representation | Optional authorized public/profile view | Administrative view | Restricted internal evidence |
|---|---|---|---|---|
| CSAPI/SensorML/SWE domain source, procedure, FOI, quality | As defined by standard/profile | Additional summaries if profile-defined | Full source mapping | Protected source correlation |
| Resource revision identifier/history | Only where standard/profile defines it | Opaque version/provenance link if accepted later | Full revision graph | Transaction internals |
| Attribution | Public contact/publisher where source permits | Generalized organization/role | Source and configured actors | Authenticated principal/session identifiers |
| Transformation summary | None in core unless encoded as domain process | Method category/version and limitations | Full inputs, parameters, builds, findings | Sensitive topology, secrets, exploit details |
| Integrity | HTTP validators/digests only under protocol rules | Artifact digest/signature summary if policy permits | Full verification evidence | Keys, credential material, protected certificate/policy data |
| Validation/quality | Standard quality fields | Scoped flag/measurement with metric/evaluator/time | Full reports/findings | Rejected payloads and sensitive rules |
| Trust evidence/decision | None in core | Coarse purpose-safe result only if profile/policy permits | Authorized dimensions/rationale codes | Identity assurance details, source reputation, policy internals |
| Lineage graph | None in core | Bounded redacted projection via later profile | Authorized graph/query | Hidden nodes/edges, cross-tenant/source topology |
| Audit | None | None by default | Authorized accountability view | Security logs, failed access, raw identities/network data |
| Deletion/tombstone | Standard lifecycle response where defined | Opaque tombstone/reason class | Full authorized lifecycle evidence | Legally erased/protected content |

### 13.2 Query Boundary

This report does not add a core CSAPI filter. A later administrative or optional-profile API can support subject, source, agent, activity kind, time, validation outcome, quality dimension, assertion status, and lineage direction. It must enforce:

- authorization before graph expansion, count, existence test, aggregation, or pagination;
- maximum depth, nodes, edges, wall time, result bytes, and fan-out;
- stable snapshot/evaluation time and cursor-bound authorization context under IDR-SRV-018;
- deterministic order and continuation without disclosing hidden-node positions;
- per-node and per-edge policy checks, with opaque terminal nodes only where safe;
- projection-specific ETag/digest/cache behavior; and
- no negative inference from hidden counts, timestamps, gap patterns, or response timing.

Provenance search is not a reason to reveal raw authentication principals or protected endpoint topology. [P]

### 13.3 Redaction Propagation

Restrictions attach to nodes, edges, attributes, and artifacts and propagate conservatively through derived outputs until a versioned policy explicitly declassifies/generalizes them. A public projection has a different entity identity, content digest, and validator from its protected source. If a lineage path crosses hidden material, the response may end at an opaque boundary, generalize the source category, or omit the path; it must not splice visible nodes together and falsely imply direct derivation. [P]

### 13.4 Export Contract

An export binds exact subject revisions, provenance bundle version, serialization/profile, policy projection, evaluation time, authorization context or safe equivalent, output digest, generator version, classification/releasability markings, and destination class. PROV-O can be supported as an optional semantic representation, but compact JSON is acceptable for operational use if relation meanings remain unambiguous and round-trip mappings are tested. [P]

---

## 14. Persistence and Downstream Implications

### 14.1 Logical Persistence Contract

- Provenance entities, activities, relations, assertions, and bundles are immutable or append-corrected; current summaries and traversal indexes are derived.
- A successful authoritative revision and its minimal provenance envelope commit atomically. The outbox record needed for later publication is in the same boundary.
- Stable IDs are service-wide and opaque; UUIDv7 is suitable for allocation but never semantic or causal ordering.
- Transaction sequence/watermark controls commit ordering; source, activity, domain, receipt, publication, and sync clocks remain distinct.
- Exact revision/artifact references and source-qualified remote IDs prevent ambiguity across nodes.
- Relational adjacency indexes are sufficient initially; no graph database or distributed ledger is required.
- Derived indexes/caches can be rebuilt from authoritative records and preserve the authorization projection boundary.

Suggested logical indexes include subject-to-entity, entity-to-generation/activity, input/output derivation, agent/role/activity, source-qualified remote identity, correlation/idempotency, activity/transaction time, quality subject/dimension/status, evidence expiry, digest/algorithm, retention class, and restriction labels. Physical forms belong to IDR-SRV-025–029. [P]

### 14.2 Transaction and Consistency

IDR-SRV-029 must ensure exactly one committed generation record for each new authoritative revision; validate referenced inputs; prevent an activity from partially committing only its resource or only its provenance; update reconstructable current/aggregate indexes consistently; and link idempotent retries to the original outcome. A failed publish after commit leaves authoritative state committed and records publication failure separately for retry. [P]

### 14.3 Retention Dependencies

IDR-SRV-030 sets durations, but must recognize these classes:

1. essential lineage needed to interpret a retained resource/revision;
2. schemas, vocabularies, process/calibration/ruleset and software identities needed to decode or reproduce it;
3. raw source artifacts required by §9.2;
4. validation and quality evidence supporting a retained decision;
5. sensitive trust, quarantine, and security evidence subject to minimization;
6. tombstone/deletion/synchronization evidence preventing resurrection;
7. derived caches and summaries that are reconstructable; and
8. exports/releases requiring proof of what was disclosed.

Deletion dependency checks must not erase the only schema, vocabulary, transformation specification, or source artifact required by retained content. Conversely, keeping provenance is not blanket authority to retain unrestricted personal, credential, raw, or classified data indefinitely. [P]

### 14.4 Audit and Security

Provenance explains data state; audit supports accountability for attempted and completed access/action. A failed unauthorized read is audit, not data derivation. A successful correction can have both records joined by correlation/activity ID. Security controls protect integrity and confidentiality of provenance itself, including tamper-evident storage options, least privilege, purpose limitation, log injection resistance, encryption, key separation, and redacted administration. Exact controls belong to IDR-SRV-039–041. [P]

### 14.5 DDIL and Synchronization

Cross-node lineage retains origin node, remote entity/activity/bundle ID, remote source sequence/version, remote digest/signature evidence, local receipt/commit/publication time, hop/forwarder information where available, dedup key, and local validation/transformation. Import creates a local account of a remote entity; it never reattributes origin to the receiving node. Conflicts create parallel branches or assertions until a documented merge/resolution activity acts on all material inputs. Tombstones carry anti-resurrection identity/version evidence. [P]

Clock comparison follows IDR-SRV-018: source clocks can be delayed, skewed, uncertain, or noncomparable; local monotonic transaction order cannot rewrite remote occurrence time. Disconnected nodes may retain last-known evidence with freshness and completeness limitations. [A,P]

### 14.6 Draft Part 3 Seam

IDR-SRV-014H remains controlling draft evidence. Future event publication can carry opaque resource/event IDs and preserve occurrence, commit, publication, delivery, replay, offset, and dedup context. The outbox can point to the provenance activity and exact projection without placing the full protected graph in a message. Topic 035 chooses the transport/binding only after rechecking the draft. [D,P]

### 14.7 Downstream Handoff Matrix

| Topic | Required handoff |
|---|---|
| IDR-SRV-020 | status/event facts versus provenance activities; availability/freshness evidence; source/quality uncertainty; projection vocabulary |
| IDR-SRV-021–024 | SensorML/SWE/JSON mapping, field-origin preservation, quality encoding, vocabulary/schema versioning, transformation round trips |
| IDR-SRV-025–029 | immutable revision/activity/entity/edge/assertion model, indexes, artifact store seam, atomic mutation+provenance+outbox, idempotency |
| IDR-SRV-030 | retention classes/dependencies, raw-byte decision, tombstones, evidence minimization, legal erasure |
| IDR-SRV-031–034 | capture envelopes for create/import/update/observation/status; validation, quarantine, corrections, replay, aggregation |
| IDR-SRV-035 | outbox/provenance correlation and occurrence/commit/publication/delivery/replay clocks; no full graph leakage |
| IDR-SRV-036–038 | observed sender, gateway, authorization/safety validations, feasibility inputs/policy versions, status/result lineage |
| IDR-SRV-039/040 | identity/authority/policy decision model, restrictions, per-edge authorization, inference-resistant projection |
| IDR-SRV-041 | audit/provenance overlap and correlation, failed attempts, tamper evidence, administrative accountability |
| IDR-SRV-042/043 | source-qualified bundle/entity IDs, dedup, branches, merge, tombstone, multi-hop, remote/local clock evidence |
| IDR-SRV-045/046 | optional profile/link design, projection validators/digests/cache, bounded traversal and cursor contract |
| IDR-SRV-050/051 | distinguish CSAPI conformance, semantic mapping, project invariants, and evidence traceability |
| IDR-SRV-053/054 | golden artifacts, graphs, access cases, replay/sync storms, traversal limits, fidelity and reconstruction tests |
| IDR-SRV-056 | interoperability exports with exact profile, revisions, source versions, authorization projection, and known limits |

---

## 15. Graph, Temporal, and Referential Invariants

### 15.1 Graph Invariants

1. Entity and Activity identities are disjoint; Agent overlap follows PROV rules but uses distinct operational role records. [N,P]
2. Every committed canonical revision has exactly one Glaux logical generation event; multiple representations of it are distinct entities. This is a stricter project invariant than PROV's allowance for simultaneous generation descriptions. [P]
3. Material derivation names the generated entity, used entity, and activity; a mere `used` plus unrelated output does not automatically assert derivation. [N,P]
4. Entity-revision derivation is acyclic. Resource-domain relationships are not subject to this blanket rule and may form valid cycles by type. [P]
5. `revisionOf` endpoints share the same canonical ResourceId and satisfy substantial-content semantics; replacement does not. [N,P]
6. `specializationOf` is irreflexive and transitive in the asserted specific-to-general graph, with compatible inherited aspects. [N,P]
7. `alternateOf` is symmetric/equivalence-like and is excluded from causal cycle checks. [N]
8. Merge output derives from every material selected input; rejected/conflicting inputs remain referenced by the merge decision record. [P]
9. A lineage projection never joins visible nodes across hidden edges as if direct derivation existed. [P]
10. Collection membership does not imply derivation, common attribution, or identical restrictions. [N,P]

### 15.2 Temporal Invariants

1. Activity start does not follow end; a use/generation attributed to an activity falls within its bounds when comparable.
2. Generation precedes usage and invalidation of that entity; derivation input use precedes output generation where comparable. [N]
3. An entity is not used after its invalidation in the same provenance account. [N]
4. Specialization generation/invalidation order is compatible with the general entity's lifetime. [N]
5. Source/domain times can precede local receipt/commit by arbitrary delay; they are not rewritten to make order convenient. [P]
6. Transaction sequence, not wall clock or UUIDv7, decides local commit order. [P]
7. A correction can have retroactive valid time but later transaction time; historical as-known answers remain reproducible. [P]
8. Trust/quality evaluations name evaluation time, evidence snapshot, and expiry/recheck triggers. [P]
9. Cross-clock ordering is asserted only with a declared mapping and uncertainty; otherwise it remains unknown. [P]

### 15.3 Referential and Integrity Invariants

1. Every local entity, activity, agent, relation, assertion, and bundle reference resolves or is explicitly external/source-qualified, redacted, erased-by-policy, or unresolved with reason.
2. A retained record binds to immutable schema/vocabulary/process revisions needed for interpretation.
3. Artifact digests include algorithm and byte-domain identity; ETag is never silently substituted for a cross-resource content hash.
4. Server-added/defaulted/derived/inferred fields retain origin and generating method.
5. `unknown` is never normalized to `verified`, `not_applicable`, zero uncertainty, complete lineage, or absent restriction.
6. A verified signature/digest/validation fact cannot directly set semantic truth, authority, fitness, or authorization.
7. A deletion/tombstone never leaves retained lineage pointing ambiguously to a reused identity.
8. Batch inheritance is valid only through an explicit member-to-context binding with per-item exceptions.
9. Quality and trust assertions target exact revision/scope; a new subject revision does not inherit them unless an explicit rule generates a new assertion.
10. Public digests, counts, IDs, errors, timing, and graph boundaries follow the same authorization/redaction policy as their source facts.

---

## 16. Fixture and Test Corpus

### 16.1 Positive Fixtures

| ID | Scenario | Expected evidence |
|---|---|---|
| P19-01 | SensorML System registration | raw bytes, digest, parsed artifact, schema validation, normalized initial revision, observed and asserted agents separated |
| P19-02 | Observation batch ingestion | shared envelope plus exact member bindings, per-item validation/outcome, schema revision, source/result/receipt/commit clocks |
| P19-03 | Unit/CRS normalization | exact inputs/outputs, method/version/parameters, uncertainty propagation, distinct digests |
| P19-04 | Same-identity metadata correction | new revision, prior revision retained, retroactive valid time and later transaction time, reason/evidence |
| P19-05 | Three-source merge | output derives from all material inputs; selected/rejected claims and policy version preserved |
| P19-06 | Idempotent replay | second receipt/attempt links to first outcome; no duplicate resource generation |
| P19-07 | Federated synchronization | remote origin and IDs retained, local validation/commit separate, dedup and hop context |
| P19-08 | Redacted export | derived projection, policy/profile/evaluation time, different digest/ETag, protected link internal only |
| P19-09 | SWE variable quality | per-measurement Quantity/Range/Category/Text remains attached to component with definition/unit |
| P19-10 | Contradictory quality claims | source and server assertions coexist with distinct evaluator/method/scope/time |
| P19-11 | Signature verification | signed domain, key/path/status/time and verification outcome recorded separately from authority decision |
| P19-12 | Delete and restore | tombstone/invalidation plus anti-resurrection evidence; restore creates new activity/revision |

### 16.2 Negative Fixtures

| ID | Scenario | Required result |
|---|---|---|
| N19-01 | Missing material input/actor without allowed unknown reason | reject or quarantine provenance record; no authoritative revision |
| N19-02 | Entity and activity reuse same forbidden identity | graph invalid |
| N19-03 | Derivation/revision cycle | graph invalid; resource-domain cycles tested separately |
| N19-04 | `revisionOf` across different ResourceIds | reject; require replacement/domain relation or plain derivation |
| N19-05 | Use after invalidation or output before comparable input use | invalid temporal graph |
| N19-06 | Digest verified over wrong/unnamed byte domain | invalid integrity evidence |
| N19-07 | Payload actor overwrites authenticated principal | reject mapping; retain both identities and discrepancy |
| N19-08 | `trusted: true` or unlabeled scalar score | reject canonical trust record |
| N19-09 | `valid: true` without dimension/metric/scope | reject canonical quality assertion |
| N19-10 | Schema pass promoted to semantic truth/authorization | policy/type validation failure |
| N19-11 | Public traversal reveals hidden source by count/ID/timing | security regression |
| N19-12 | Deleted artifact leaves ambiguous reusable reference | referential-integrity failure |

### 16.3 Boundary Fixtures

- Unknown versus failed versus not-evaluated versus not-applicable trust evidence.
- Exact zero uncertainty versus unknown uncertainty.
- Source URL remains reachable but validator/content changes; immutable promise broken.
- Same bytes under different media/semantic context; same ETag across different resource URLs; multiple digest algorithms during migration.
- Deterministic transformation with changed dependency version and nondeterministic transformation with seed/environment captured or absent.
- Batch with one different source, one invalid signature, one quarantined item, and one stricter classification.
- Provenance complete only from receipt forward, with opaque remote frontier.
- Alternate representations that are semantically equivalent but not byte-equivalent.
- Revision retaining substantial content versus different-identity replacement.
- Redaction that would leak hidden source through stable IDs, exact time, error text, graph degree, or response length.
- Quality assertion expires after schema/calibration/policy revision; historical evaluation remains reproducible.
- Credential/authority revoked after original ingest; current decision changes without rewriting historical evidence.

### 16.4 Replay and Synchronization Fixtures

- Identical artifact and idempotency key, identical artifact with new key, changed artifact with reused key, and repeated remote sequence.
- Delayed/out-of-order remote revisions, parallel offline edits, tombstone versus update, and merge after reconnect.
- Multi-hop A→B→C lineage that preserves A as origin, B as forwarder/transformer, and C's local receipt/validation.
- Same external UID from two source namespaces; source-qualified identities remain distinct until an explicit equivalence decision.
- Remote provenance bundle corrected after local import; both accounts and local reconciliation activity remain traceable.
- Publication failure after atomic commit, retry, duplicate delivery, and consumer replay without duplicate domain mutation.

### 16.5 Acceptance Oracles

The harness distinguishes approved CSAPI requirements, W3C semantic mapping tests, Glaux project invariants, profile tests, and security-policy tests. A passing PROV mapping does not imply CSAPI conformance, and a passing CSAPI schema does not imply complete provenance. Golden fixtures include exact raw artifacts where distributable, normalized records, expected entity/activity/agent graphs, quality/trust assertions, redacted projections, digests, and negative diagnostic classes without operational secrets. [P]

---

## 17. Recommendations, Rejected Options, Open Questions, and Review Triggers

### 17.1 Recommendations

| ID | Recommendation | Priority | Basis |
|---|---|---|---|
| R-019-01 | Adopt an encoding-neutral W3C PROV-compatible Entity–Activity–Agent model with qualified relations and optional PROV-O export. | Critical | PROV [N,P] |
| R-019-02 | Treat every canonical resource revision and material artifact/assertion as an immutable provenance entity. | Critical | PROV/IDR-016 [N,P] |
| R-019-03 | Atomically commit each authoritative revision with its essential provenance activity and outbox reference. | Critical | consistency/accountability [P] |
| R-019-04 | Preserve authenticated principal, asserted actor, registered source, responsible principal, and software agent as distinct role-qualified facts. | Critical | PROV/security [N,P] |
| R-019-05 | Record source/server/defaulted/derived/inferred/migrated origin for material values and never erase raw-source evidence during normalization. | Critical | fidelity [P] |
| R-019-06 | Retain exact bytes under §9.2; otherwise require digest, immutable durable reference, acquisition metadata, and reproducible dependencies. | Critical | fidelity/integrity [P] |
| R-019-07 | Use SHA-256 or stronger current policy-approved SHA-2/SHA-3 with algorithm agility and explicit byte domains; no SHA-1 for new content identity. | High | NIST/RFC 9530 [N,P] |
| R-019-08 | Record every material transform with inputs/outputs, method/build, parameters, dependencies, determinism, limitations, and restrictions. | Critical | PROV/SensorML [N,P] |
| R-019-09 | Represent quality as scoped, attributed, versioned assertions; preserve contradictory claims and unknowns. | Critical | SWE/DQV [N,I,P] |
| R-019-10 | Keep SWE measurement uncertainty/value-local quality separate from metadata-quality and evaluator confidence. | Critical | SWE/DQV [N,I,P] |
| R-019-11 | Represent trust as independent evidence dimensions with method, outcome, time, expiry, and limitations; prohibit a canonical scalar trust score. | Critical | boundary analysis [P] |
| R-019-12 | Keep trust/policy decisions downstream and persist policy/evidence context when decisions matter. | Critical | security boundary [P] |
| R-019-13 | Preserve earlier revisions and corrective/merge evidence; do not silently overwrite, flatten conflicts, or misuse revision for replacement. | Critical | PROV/IDR-016 [N,P] |
| R-019-14 | Treat redaction and release as accountable derivations with different entity identities/digests and inference-resistant projections. | Critical | security/exposure [P] |
| R-019-15 | Authorize before traversal/count/aggregate and bound lineage depth, fan-out, time, and bytes. | Critical | security/performance [P] |
| R-019-16 | Keep provenance and audit distinct but correlated; failed/rejected attempts must not create authoritative domain lineage. | High | accountability [P] |
| R-019-17 | Preserve origin, remote IDs, hops, clocks, branches, conflict/merge, and tombstone evidence across DDIL synchronization. | Critical | IDR-018/DDIL [A,P] |
| R-019-18 | Validate the graph, temporal, referential, non-overstatement, and exposure invariants with the §16 corpus. | High | PROV/project [N,P] |
| R-019-19 | Do not invent a core CSAPI provenance endpoint/member; design any public capability later as an advertised, access-controlled profile. | Critical | CSAPI boundary [N,P] |
| R-019-20 | Recheck CSAPI artifacts, profile requirements, NIST algorithm policy, and selected vocabularies before implementation freeze. | High | mutable evidence [P] |

### 17.2 Rejected Options

| Option | Reason rejected |
|---|---|
| One mutable `source`/`updatedBy` field | Cannot distinguish artifact, actor, principal, transport, transformation, or history |
| One scalar trust/reputation score | Hides purpose, evidence dimensions, uncertainty, policy, age, and contradictions |
| Successful authentication equals trusted data | Credential control does not establish authority or truth |
| Valid signature equals authoritative/correct claim | Signature, key identity, authority, revocation, content meaning, and truth are separate |
| Schema-valid equals semantically valid | Structural constraints cannot prove units, relationships, measurement correctness, or safety |
| Hash every canonical JSON object and discard source | Canonicalization can lose evidence; external bytes can disappear; digest does not reproduce source |
| Retain every payload forever | Violates minimization, classification, legal, operational, and cost boundaries |
| Store current state plus free-text log | Cannot enforce graph/revision invariants or reproduce transforms and decisions |
| Treat every newer resource as PROV revision | Replacement, alternate, specialization, and derivation have different identity semantics |
| Inherit batch provenance implicitly | Mixed source, validation, transformation, and restriction cases become false |
| Expose the full graph to authenticated users | Authentication alone does not authorize hidden nodes, actors, topology, or inference |
| Require graph database or blockchain | Unnecessary to satisfy semantics, indexing, append history, or tamper controls |
| Put all audit events in provenance | Failed access and security attempts do not explain generated data entities |

### 17.3 Open Questions Routed Downstream

- Which accepted profile, media type, schema, link relation, and capability advertisement will expose optional provenance/quality views? IDR-SRV-021/045/046.
- Which metric/dimension vocabularies will be project-default, profile-required, or deployment-configured? IDR-SRV-022–024/040.
- Which artifacts require exact-byte retention and for how long under each deployment's legal/classification regime? IDR-SRV-028/030/039.
- Which identities, signature systems, credential assurance levels, authority scopes, and revocation sources are supported? IDR-SRV-039/040.
- What trust-decision and audit schemas, tamper evidence, and administrative disclosure are required? IDR-SRV-040/041.
- How are transaction retries, partial bulk writes, cross-resource atomicity, and provenance correction implemented physically? IDR-SRV-029/031.
- What synchronization bundle/profile exchanges provenance and tombstones, and how are cross-node conflicts resolved? IDR-SRV-042/043.

None blocks acceptance of the canonical evidence model; each has a named downstream owner.

### 17.4 Review Triggers

Revisit this report when:

- an adopted AEP/STANAG revision adds provenance, quality, source, integrity, or handling requirements;
- CSAPI gains an approved provenance/quality extension or changes relevant Part 1/2 fields;
- draft Part 3 or another event standard fixes message provenance/replay semantics;
- the project selects public semantic export, signature, content-addressing, identity, or trust-policy standards;
- NIST transitions approved hash algorithms or project cryptographic policy changes;
- retention/law/classification rules require different raw-artifact handling;
- synchronization introduces multi-master, cross-domain, or cross-classification exchange; or
- performance evidence shows current graph/index/projection bounds insufficient.

---

## 18. Validation Against Plan Success Criteria

| Plan criterion | Result | Evidence |
|---|---|---|
| Every core/detailed question answered or bounded | Satisfied | §2 and referenced sections |
| Standards/profile claims use exact sources/status | Satisfied | §§3–4, references |
| Provenance, lineage, quality, uncertainty, integrity, trust, authorization, policy distinguished | Satisfied | §§5, 10–11 |
| Every resource/artifact family mapped | Satisfied | §6 |
| Required activity capture points defined | Satisfied | §§7, 12 |
| Raw fidelity and transformations testable | Satisfied | §9 |
| Trust multidimensional, not scalar | Satisfied | §11 |
| Exposure and redaction rules defined | Satisfied | §13 |
| Graph/version/temporal/referential invariants defined | Satisfied | §15 |
| Positive, negative, boundary, replay, sync fixtures | Satisfied | §16 |
| Persistence, transaction, retention, audit, security, DDIL, sync, API handoffs | Satisfied | §§13–14 |
| Mutable sources pinned and evidence gaps stated | Satisfied | §3 |
| Recommendations bounded and no invented standards obligation | Satisfied | §§4.1, 17 |
| Polished, recommendation-first, self-contained | Satisfied | entire report |

### 18.1 Methodology Phase Completion

| Phase | Output | Status |
|---|---|---|
| 1. Source freeze and terminology | inventory, authority rules, definitions | Complete |
| 2. Resource and activity coverage | resource/artifact and activity matrices | Complete |
| 3. Lineage, fidelity, and quality | canonical graph, fidelity decision, transformation and quality records | Complete |
| 4. Trust, exposure, lifecycle | multidimensional evidence, decision boundary, projections, handoffs | Complete |
| 5. Fixtures and implementation | invariants, fixture corpus, logical persistence/index requirements | Complete |
| 6. Synthesis | decisions, recommendations, rejected options, review triggers | Complete |

### 18.2 Acceptance Boundary

The Glaux Project Lead accepted this report on September 13, 2026. The accepted baseline establishes the provenance, lineage, fidelity, quality, uncertainty, trust-evidence, exposure, persistence, and testing model and authorizes only IDR-SRV-020. Acceptance specifically confirms:

1. the PROV-compatible Entity–Activity–Agent model and qualified relation semantics;
2. atomic authoritative-revision and essential-provenance persistence;
3. risk-based raw-byte retention and reproducible transformation records;
4. scoped quality assertions and multidimensional trust evidence without a scalar trust score;
5. controlled, inference-resistant provenance projections; and
6. the downstream ownership boundaries in §14.

---

## References

### Project Sources

- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)
- [IDR-SRV-019 Research Plan](../IDR%20Plans/idr-srv-019-provenance-lineage-quality-and-trust-metadata-model.md)
- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Research Report Template](../../../../../Governance/research-report-template.md)
- [IDR-SRV-015 Canonical Resource Model](idr-srv-015-canonical-glaux-server-resource-model-report.md)
- [IDR-SRV-016 Identifier, URI, and Lifecycle Strategy](idr-srv-016-identifier-uri-and-resource-lifecycle-strategy-report.md)
- [IDR-SRV-017 Relationship and Linkage Model](idr-srv-017-relationship-and-linkage-model-report.md)
- [IDR-SRV-018 Temporal, Validity, and Freshness Model](idr-srv-018-temporal-validity-and-freshness-model-report.md)
- [CSAPI upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md), version 1.9

### Standards and Official Artifacts

- [OGC API - Connected Systems - Part 1: Feature Resources, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2: Dynamic Data, Version 1.0](https://docs.ogc.org/is/23-002/23-002.html)
- [Official CSAPI `v1.0.0` source pin](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)
- [Tagged Part 2 Observation schema](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/schemas/json/observation.json)
- [Tagged Part 2 Command schema](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/schemas/json/command.json)
- [OGC SensorML Encoding Standard 3.0](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC SWE Common Data Model Encoding Standard 3.0](https://docs.ogc.org/is/24-014/24-014.html)
- [W3C PROV-DM](https://www.w3.org/TR/prov-dm/)
- [W3C PROV-O](https://www.w3.org/TR/prov-o/)
- [W3C PROV Constraints](https://www.w3.org/TR/prov-constraints/)
- [W3C/OGC Semantic Sensor Network Ontology](https://www.w3.org/TR/vocab-ssn/)
- [W3C Data Quality Vocabulary](https://www.w3.org/TR/vocab-dqv/)
- [W3C Data on the Web Best Practices](https://www.w3.org/TR/dwbp/)

### Protocol and Integrity Sources

- [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)
- [RFC 9530: Digest Fields](https://www.rfc-editor.org/rfc/rfc9530)
- [NIST FIPS 180-4: Secure Hash Standard](https://csrc.nist.gov/pubs/fips/180-4/upd1/final)
- [NIST Hash Functions and Current Policy](https://csrc.nist.gov/projects/hash-functions)

---

**End of IDR-SRV-019 Research Report**

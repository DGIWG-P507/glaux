# Section 028: Metadata and Document Storage Strategy - Research Report

**Topic ID:** IDR-SRV-028<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-028 Metadata and Document Storage Strategy](../IDR%20Plans/idr-srv-028-metadata-and-document-storage-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core questions, all detailed question groups, all 6 methodology phases, all 10 success criteria, and the required 14-field metadata/document category matrix<br>
**Methodology Used:** Authority-ranked extraction from approved OGC standards, controlled-source findings, and accepted IDR-SRV-001 through IDR-SRV-027; direct review of current primary technology documentation; category and lifecycle classification; storage-option comparison; and bounded synthesis into a source-preserving relational/artifact strategy<br>
**Research Time:** Approximately 14 hours of AI-assisted execution on September 14, 2026<br>
**Official Standards Source Pin:** [`opengeospatial/ogcapi-connected-systems` `v1.0.0` at `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)<br>
**Shared Register Baseline:** [OGC API - Connected Systems upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md), Version 1.10; upstream `master` remained `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f` on September 14, 2026<br>
**Technology Documentation Snapshot:** PostgreSQL 18, OpenAPI 3.1.2 and 3.2.1, JSON Schema Draft 2020-12, SQLite JSON1, DuckDB JSON, Rust `object_store` 0.14.1, `jsonschema` 0.52.0, SQLx 0.9.0, and `serde_json` 1.0.151; checked September 14, 2026 and treated as mutable implementation evidence<br>
**Controlled AEP Source:** `AC/224(JCGISR)D(2026)0005`, April 27, 2026, SHA-256 `56dc757b6e677b3584e3152a957849f21a24b22854f562613ff283a8b599da8c`; used only through accepted project findings and not redistributed<br>
**Document Purpose:** Establish the Glaux Server source-document, normalized-metadata, immutable-artifact, validation-evidence, offline-package, indexing, lifecycle, policy, DDIL, and test baseline without defining final DDL, implementing draft Part 3, or implementing the server<br>
**Author:** OpenAI Codex<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** September 14, 2026<br>
**Date:** September 14, 2026<br>
**Last Updated:** September 14, 2026

---

## Reading Guide and Evidence Labels

| Label | Meaning |
|---|---|
| **N** | Normative or standards-derived behavior |
| **A** | Project-controlling AEP/STANAG adoption or operational-context finding |
| **P** | Glaux architecture direction or recommendation proposed for acceptance here |
| **I** | Informative technology, implementation, test, or community evidence |
| **D** | Detailed design deferred to the named downstream topic |
| **X** | Evidence gap or unresolved decision requiring profile input, prototype, or benchmark |

The word **document** in this report covers exact received bytes, parsed trees, canonical domain state, generated representations, schema/profile/vocabulary packages, validation evidence, fixtures, and caches. Those roles are deliberately distinct. A SHA-256 digest identifies bytes; it does not by itself establish resource identity, authority, trust, ownership, or releasability.

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Metadata/Document Requirement Extraction Methodology
5. Metadata and Document Category Inventory
6. Source Fidelity and Preservation Findings
7. Normalization and Indexed Metadata Findings
8. Versioning, Lifecycle, Retention, and Archival Findings
9. Validation Artifact and Diagnostic Storage Findings
10. Schema, Profile, Vocabulary, and Semantic Artifact Cache Findings
11. Metadata/Document Storage Option Evaluation
12. Provenance, Source Authority, Federation, and DDIL Implications
13. Security, Policy, Releasability, Redaction, and Audit Implications
14. Fixture, Golden-File, Conformance, and Interoperability Test Implications
15. Downstream Topic Handoff Matrix
16. Recommendations
17. Risks, Constraints, and Open Questions
18. Validation Against This Plan's Success Criteria
19. References

---

## 1. Executive Summary

Glaux should use a **source-preserving relational/artifact hybrid** inside the PostgreSQL/PostGIS authoritative core selected by IDR-SRV-025. PostgreSQL owns resource identity, revisions, typed relationships, normalized discovery metadata, artifact catalog entries, provenance, lifecycle, validation summaries, policy facts, and transaction coordination. Immutable exact bytes are stored by an artifact adapter: PostgreSQL `bytea`/TOAST is the initial low-complexity baseline for control documents and small-to-medium artifacts, while filesystem or S3-compatible content storage remains a measured option for larger objects. **[P]**

The logical design has six non-collapsible roles: **exact source bytes; artifact catalog and provenance; parsed document graph; canonical normalized resource graph; authorized generated representations; and immutable validation/transformation evidence plus derived indexes**. SensorML, SWE Common, imported schemas, profile material, vocabulary packages, fixtures, signed documents, and rejected/quarantined payloads retain exact bytes when policy permits. JSONB is useful for validated heterogeneous subtrees and derived projections, but it is not an exact-source store: PostgreSQL documents that `jsonb` discards insignificant whitespace, object-key order, and duplicate keys.[^1] **[I/P]**

Content addressing uses an algorithm-qualified digest such as `sha256:<hex>`. The digest verifies and deduplicates bytes, but the catalog separately records logical document identity, resource identity, revision, source URI, source authority, acquisition activity, policy marking, validation state, storage locator/version, and dependencies. Identical bytes acquired from two authorities remain two provenance events and may carry different policy. Raw digest namespaces are not public object identifiers. **[P]**

Stable, security-relevant query fields belong in typed relational columns and typed edge tables: IDs and aliases, resource family and lifecycle, profile and conformance claims, source authority, semantic/property/unit identifiers, contract fingerprints, temporal and spatial links, validation state, and policy labels. Selected validated JSONB fields may receive expression or GIN indexes. PostgreSQL full-text search may support an approved title/description/keyword discovery surface with a pinned text-search configuration; unrestricted JSONPath, generic document search, or a dedicated search engine is not an initial requirement. **[I/P]**

Documents and packages are immutable versions linked by `supersedes`, `derivedFrom`, `validates`, or other typed relations. Domain validity, receipt, ingest, commit, publication, validation, activation, and retirement are separate clocks. Deleting a resource does not automatically delete its source, validation, provenance, audit, fixture, or package evidence. IDR-SRV-030 must set retention schedules, holds, archival formats, restore guarantees, and garbage-collection rules; collection should be catalog-rooted mark-and-sweep rather than naive reference counting. **[P/D]**

Schema, OpenAPI, profile, and vocabulary dependencies are packaged for offline operation as immutable, reference-closed sets with manifests, logical-URI-to-digest mappings, versions, dialects, licenses, trust and signature status, dependency closure, and resolver policy. Runtime validation and response generation perform no uncontrolled network dereference. Vendor artifacts remain byte-exact; Glaux wrappers, overlays, and resolver mappings are separate artifacts. Every validation or generation result binds the exact package set, tool version, options, policy profile, and input digest. **[P]**

The accepted OpenAPI baseline remains 3.1.2. OpenAPI 3.2.1 was published on September 10, 2026, but is only a monitored migration input; no automatic baseline change follows from publication.[^2] Official CSAPI `v1.0.0` OpenAPI bundles remain provenance and compatibility fixtures where their external references prevent standalone deployment. IDR-SRV-014's generated, self-hosted, reference-closed Glaux contract remains controlling. **[I/P]**

Security policy applies before indexing, link generation, diagnostics, search, caching, and document rendering. If field removal would invalidate an advertised schema, Glaux must serve a named safe projection/profile or suppress the representation; it must not label structurally altered content as the unchanged schema. Exact-source access is a separate privilege from access to a generated public view. Imported documents are untrusted inputs subject to size, recursion, decompression, resolver, parser, malware, path, and rendering limits. **[P]**

The principal remaining decisions are measurable or profile-owned: `bytea` versus external-object thresholds, final DDL, retention durations, archive tiers, digital-signature policy, public source-artifact administration, AEP package composition, package signing, search workload, and an eventual OpenAPI 3.2.x migration. They are handed to IDR-SRV-029/030 and the named configuration, ingestion, security, deployment, fixture, conformance, performance, and interoperability topics. **[D/X]**

---

## 2. Scope and Plan Alignment

This report executes `IDR-SRV-028`, the fourth Category E topic. It inventories metadata and document categories; separates source, normalized, generated, cached, test, provenance, and validation roles; defines preservation and indexing rules; evaluates storage patterns; and establishes bounded handoffs.

It does **not** define final SQL DDL, transaction algorithms, retention durations, delete authority, configuration-secret storage, ingestion endpoints, authorization policy, audit schemas, deployment topology, or product procurement. It does not adopt or implement draft OGC API - Connected Systems Part 3 Publish/Subscribe and does not implement the server. `IDR-SRV-029` and later topics remain unauthorized during this iteration.

### 2.1 Research Question Coverage Matrix

| Plan question | Short form | Status | Evidence location |
|---|---|---|---|
| Q1 | Which metadata and document categories must be stored, preserved, versioned, indexed, validated, exposed, or derived? | Complete | Sections 5-10 |
| Q2 | Which categories are source, normalized, generated, validation, package, cache, fixture, or provenance/audit records? | Complete | Sections 5-6 |
| Q3 | Which storage patterns support fidelity, query, validation, lifecycle, provenance, and policy? | Complete; physical thresholds deferred | Sections 6-11 |
| Q4 | How does document storage interact with SensorML, SWE Common, validation, semantics, spatial/time-series state, and CSAPI resources? | Complete | Sections 5-10, 12-14 |
| Q5 | What downstream implications follow? | Complete | Sections 12-17 |

### 2.2 Accepted-Baseline Reconciliation

- IDR-SRV-015's encoding-neutral canonical graph remains the resource authority; a source document is evidence for that graph, not the graph itself.
- IDR-SRV-016 identities remain distinct: resource ID/UID, canonical URL, external ID/alias, revision ID, event ID, logical document ID, artifact ID/digest, schema `$id`, and storage locator are not interchangeable.
- IDR-SRV-017 typed relationships carry link semantics; embedded source links are preserved but normalized only through validated, typed edges.
- IDR-SRV-018 and IDR-SRV-020 keep domain validity, transaction history, lifecycle state, status, and event facts distinct.
- IDR-SRV-019 requires source/transformation evidence, quality, uncertainty, and trust rather than a single provenance string.
- IDR-SRV-021 through IDR-SRV-024 require exact source, parsed, canonical, generated, contract/package, and validation views; this report supplies their common artifact substrate without collapsing them.
- IDR-SRV-025 through IDR-SRV-027 keep PostgreSQL/PostGIS authoritative, exact artifacts immutable, caches disposable, and spatial/time-series state in their specialized stores rather than duplicated inside documents.

---

## 3. Evidence Base and Authority Classification

### 3.1 Controlling and Approved Sources

| Source | Pin/status | Authority | Relevant anchors | Availability/limit |
|---|---|---|---|---|
| OGC API - Connected Systems Part 1, OGC 23-001 | Approved 1.0; source tag `v1.0.0`, commit `8e03b236...` | Normative | feature resources, collections, links, SensorML/GeoJSON representations, conformance | Public and locally inspected |
| OGC API - Connected Systems Part 2, OGC 23-002 | Approved 1.0; same pin | Normative | dynamic-resource schemas, SWE encodings, OpenAPI/JSON Schema artifacts | Public and locally inspected |
| OGC API - Features Part 1, OGC 17-069r4 | Approved corrigendum | Normative dependency | collections, items, links, extents, API definition | Public |
| SensorML 3.0, OGC 23-000 | Approved | Normative representation | process descriptions, identifiers, classifiers, contacts, characteristics, capabilities, inputs/outputs, links, history | Public; IDR-SRV-021 controls architecture |
| SWE Common 3.0, OGC 24-014 | Approved | Normative data model | component/aggregate/stream structures, constraints, nil/quality, encodings | Public; IDR-SRV-022 controls contracts |
| JSON Schema Draft 2020-12 | Published 2022-06-16 | Normative technology standard | dialects, vocabularies, identifiers, references, compound documents | Public |
| OpenAPI 3.1.2 | Published 2025-09-19 | Project-selected contract baseline | multi-document descriptions, Schema Objects, references, security and Markdown risks | Public |
| Controlled AEP source | `AC/224(JCGISR)D(2026)0005`, 2026-04-27, recorded digest | Project-controlled profile evidence | accepted server/profile/package findings only | Content not reproduced; conclusions narrowed to accepted findings |
| Accepted IDR-SRV-001 through 027 reports | Accepted through 2026-09-14 | Project-controlling architecture | requirements, resource/API models, representations, validation, semantics, persistence, spatial/time-series | Repository-local and linked in References |

### 3.2 Mutable Primary Technology Evidence

| Source | Version/retrieval | Evidence used | Authority limit |
|---|---|---|---|
| PostgreSQL documentation | 18, checked 2026-09-14 | `json`/`jsonb`, GIN/expression indexes, TOAST, large objects, full-text search | Implementation capability, not CSAPI obligation |
| Rust `object_store` | 0.14.1, checked 2026-09-14 | common local/S3/Azure/GCS adapter, conditional/atomic and multipart APIs | API abstraction; backend behavior still needs contract tests |
| Rust `jsonschema` | 0.52.0, checked 2026-09-14 | Draft 2020-12, structured output, meta-validation, bundling, custom retrieval | Candidate library, not selected implementation |
| SQLx PostgreSQL types | 0.9.0, checked 2026-09-14 | mappings for typed JSON values and raw JSON values | Candidate Rust persistence evidence |
| `serde_json` | 1.0.151, checked 2026-09-14 | typed/untyped JSON parsing/serialization | Parsing capability does not promise byte round trip |
| SQLite JSON1 | current page, checked 2026-09-14 | text JSON and SQLite-private JSONB characteristics | Reduced-profile evidence only |
| DuckDB JSON | current page, checked 2026-09-14 | JSON logical type/import/analysis behavior | Analytical/fixture-tool evidence only |
| Amazon S3 documentation | checked 2026-09-14 | conditional create and delete-marker/versioning semantics | S3-compatible semantic example, not a cloud mandate |

### 3.3 Implementation and Community Evidence

Accepted IDR-SRV-014A through 014G studies of OpenSensorHub, Connected Systems Go, pygeoapi, SECD, smoke tests, interoperability results, and OS4CSAPI discussions were reused as informative evidence. They reinforce practical needs for representation preservation, local schemas, deterministic generated documents, tolerant import with explicit diagnostics, and golden interoperability artifacts. No implementation behavior is elevated into a standards requirement.

The official `v1.0.0` CSAPI repository artifacts were inspected at the recorded tag. The shared register records that the Part 1 release bundle retains 32 relative example references and the Part 2 bundle retains 51 relative references; both are valuable provenance/negative fixtures, not reference-closed Glaux deployment contracts. Upstream `master` was unchanged from the register baseline during this iteration.

### 3.4 Evidence Quality and Limitations

Normative standards outrank project decisions; accepted project reports constrain this topic; controlled AEP findings refine project behavior; implementation and technology evidence informs feasibility only. The controlled document was not redistributed and no new requirement was inferred from unavailable content. Exact AEP package composition, retention policy, public source-document access, legacy XML launch scope, and digital-signature requirements remain profile or downstream decisions. Technology versions are dated because their APIs and support can change.

---

## 4. Metadata/Document Requirement Extraction Methodology

Each requirement was reduced to four linked records:

1. **Source anchor** — standard clause/artifact, accepted IDR decision, controlled finding, implementation observation, or technology capability.
2. **Information role** — source bytes, parsed graph, normalized fact, generated representation, package dependency, validation/transformation evidence, fixture, cache/index, provenance, policy, or audit-adjacent record.
3. **Persistence behavior** — authority, fidelity, logical identity, byte identity, version clocks, lifecycle, query/index need, retention root, validation state, and exposure policy.
4. **Handoff** — the later topic that owns detailed transaction, retention, ingestion, policy, deployment, or test behavior.

### 4.1 Evaluation Criteria

Storage options were assessed against: byte fidelity; structural preservation; typed query/index capability; referential and transaction integrity; immutable versioning; artifact/package closure; validation evidence; policy filtering; Rust maturity; local/container/DDIL reproducibility; backup/restore; corruption detection; testability; operational complexity; portability; and measured scale.

### 4.2 Decision Rules

- Preserve first, parse second, normalize third; a failure at a later layer must not silently mutate an earlier layer.
- Give each fact one authoritative home. Documents may project relational/spatial/time-series state but do not become a competing authority.
- Bind derived output to all material inputs: exact source digest, resource revision, contract/package set, tool and configuration, policy view, and transformation activity.
- Treat a document's logical identity, revision, bytes, provenance event, external URI, and resource representation as separate identifiers.
- Require offline reference closure and explicit resolver policy for runtime dependencies.
- Put policy before index, search, diagnostics, links, rendering, caching, and download.
- Defer physical thresholds and detailed DDL where evidence requires benchmark or workload data.

---

## 5. Metadata and Document Category Inventory

The matrix is the minimum conceptual catalog. “CAS” means immutable content-addressed bytes through the selected artifact adapter. “PG” means authoritative PostgreSQL relational state. “JSONB” means a validated parsed or canonical subtree, never assumed byte-exact.

| Document/metadata category | Related resource family | Source topic / source anchor | Source/generated/normalized/cache/test classification | Source fidelity requirement | Normalized field requirement | Versioning/lifecycle requirement | Validation artifact requirement | Query/index needs | Retention/archival needs | Security/policy needs | Candidate storage pattern(s) | Downstream topic handoff | Notes/unresolved issues |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Imported SensorML description | System, Procedure, Deployment, Sampling Feature | SensorML 3.0; CSAPI Part 1; IDR-021 | Source + parsed + normalized | Exact bytes and media/encoding; preserve known/unknown structure | IDs, type, names, classifiers, contacts, validity, relationships, capability/property summaries | Immutable acquisition/version; supersession and valid/transaction clocks | Structural, profile, semantic, policy, normalization findings | Typed discovery fields; approved JSONB paths only | Long-lived evidence; policy/hold aware | Source may disclose topology, contacts, location, commands; separate source permission | CAS + artifact catalog + parsed JSONB + relational graph | 029-031, 039-043, 053, 056 | XML launch scope/profile decision remains open |
| Generated SensorML representation | Same feature resources | IDR-021; CSAPI representations | Generated view/cache | Deterministic output; not misrepresented as source | Build inputs and representation profile | Regenerable by input fingerprint; retain release/golden outputs as needed | Transformation and output-validation record | Retrieve by resource rev/media/profile/policy/language | Cacheable; release evidence retained | Policy-shaped before generation; safe profile if redacted | Generated CAS/bytea + build manifest/cache | 031-033, 039-040, 048, 053-056 | Exact byte equality required only for pinned golden output |
| SWE Common data-contract source | DataStream, Observation, ControlStream, Command | SWE Common 3.0; CSAPI Part 2; IDR-022 | Source + parsed + canonical contract | Exact component tree and source bytes | Component paths, types, definitions, units, nil/quality, constraints, encoding refs | Immutable contract revision/fingerprint; never reinterpret stored values under a new contract | Full validation/compile record | Contract fingerprint, property/unit IDs, active state | Retain while any sample/command/evidence references it | Constraints and controlled-property details may be sensitive | CAS + registry rows + parsed JSONB + compiled cache | 029, 031, 034-038, 053-056 | Codec launch set remains capability-gated |
| Observation/result or command payload artifact | Observation, Command, Status/Event | CSAPI Part 2; IDR-022/027 | Source evidence + parsed fact | Preserve exact payload when needed for replay/dispute; canonical facts authoritative | Contract binding, identity, clocks, result/status fields, admission hash | Immutable event/version; correction/supersession explicit | Decode/contract/policy admission evidence | Typed time/identity indexes in specialized stores | Family policy; archive manifests; not document TTL | High disclosure and command-safety sensitivity | CAS only when payload preservation is required + typed temporal tables | 029-030, 034-043 | IDR-027 owns time-series authority |
| Official/vendor JSON Schema | All encoded resources | CSAPI repo; JSON Schema 2020-12; IDR-023 | External source/package | Byte-exact vendor artifact; no in-place fixes | Logical URI, `$id`, dialect, digest, dependencies, trust/license | Immutable package member; staged/active/deprecated/retired/quarantine | Meta-schema, reference-closure, conflict, signature evidence | URI, digest, dialect, package, lifecycle | Retain all used validation versions | Untrusted refs; no runtime network; supply-chain controls | CAS + package manifest/registry + compiled cache | 029-030, 047-050, 053-056 | Official release artifacts can be non-closed negative fixtures |
| Glaux schema/wrapper/overlay | All | IDR-023; project source | Generated/project-authored package | Exact released bytes; separate from vendor source | Stable `$id`, version, compatibility and vendor dependency | Immutable semantic version/release | CI/meta/compatibility evidence | `$id`, version, digest, target profile | Retain supported and referenced releases | Code-review/signing/release policy | Git source + release CAS + registry | 045-051, 053-056 | Never silently edit vendor artifact |
| OpenAPI description and modules | Service/endpoint contract | IDR-014; OAS 3.1.2 | Project source model + generated document + release package | Modular source reproducible; generated bytes deterministic per tool version | Operations, capabilities, schemas, security, conformance, policy profile | Versioned release; 3.2.x migration gated | Lint, bundle closure, runtime parity, client/render evidence | Version/profile/media/digest | Retain releases and evidence; caches replaceable | Avoid hidden operations/schemas/examples; sanitize Markdown | Typed registries + Git + generated CAS + offline bundle | 032-033, 045, 048, 050-056 | 3.2.1 monitored; 3.1.2 remains baseline |
| Conformance declaration | Service/API | CSAPI/OAF conformance resources; IDR-006/014 | Canonical facts + generated representation | Generated response must match enabled behavior | URI, implementation/profile, enabled status, evidence release | Versioned with deployment/capability snapshot | Route/test evidence backing every claim | URI/status/deployment release | Retain deployment evidence | Do not claim hidden/disabled behavior | PG capability registry + generated response cache | 032-033, 044-045, 050-051 | No static claim independent of runtime capability |
| Collection/resource discovery metadata | All feature/dynamic collections | CSAPI Parts 1/2; OAF; IDR-015-020 | Normalized authority + generated views | Source links retained; normalized graph controls output | IDs, title/description, type, extents, classifiers, semantics, relationships, lifecycle, policy | Bitemporal revisions and typed events | Source/normalization and representation evidence | Relational, PostGIS, temporal and selected FTS indexes | Resource policy plus provenance dependencies | Field/link/extent leakage; policy-first projections | PG typed columns/edges + PostGIS + selected JSONB | 029-033, 039-043 | Avoid duplicating spatial/time-series authorities in JSON |
| Profile/AEP/STANAG material | Deployment and conformance profile | Controlled material; IDR-001/023/024 | Controlled external source/package | Preserve permitted exact artifact; respect handling constraints | Title/version/status/digest/authority/applicability; not controlled content | Immutable version; activation history | Package approval, applicability and closure evidence | Metadata only as authorized | Policy/records schedule | Classification, distribution, export and releasability controls | Controlled repository/CAS adapter + limited catalog | 039-041, 044-047, 050-051 | Exact package composition remains profile-owned |
| Vocabulary/ontology/unit package | Properties, units, classifiers | SSN/SOSA; UCUM; QUDT; IDR-024 | External source + immutable offline package + cache | Exact licensed source/package and mapping artifacts | URI, version, digest, prefixes, concepts, unit codes, mappings, trust/license | Immutable set; atomic activation; deprecate, do not reinterpret history | Parse, closure, collision, mapping and semantic tests | URI/code/label/mapping indexes; explainable query | Retain every set referenced by data/evidence | License and inference/disclosure policy | CAS + registry/relational mappings + compiled lookup cache | 029-030, 034-035, 040, 047-050, 053-056 | UCUM 2.1 center; QUDT 3.5.1 optional enrichment |
| Validation run and findings | Any source/resource/package | IDR-023 | Immutable evidence + public/admin projections | Preserve exact structured outcome and tool inputs; public projection may redact | Input/target, package digests, tool/version/options, policy, outcome, codes, pointers, times | Append new run; never overwrite prior result | Self-identifying evidence record | Target, digest, outcome, code, time, validator, policy | Retention by operational/test/audit class | Findings can echo secrets/hidden values; stable safe public form | PG run/finding tables + optional CAS report | 029-031, 039-041, 049-053 | Production and CI evidence use separate namespaces/schedules |
| Rejected/quarantined document | Any import/write | IDR-021/023 | Source + quarantine evidence | Exact bytes only when authorized; otherwise secure digest/metadata | Source, reason class, size/media, policy, correlation, validation refs | Staged/quarantined/promoted/rejected/expired | Complete rejection evidence; promotion is new activity | Restricted operational lookup | Short/controlled retention; holds possible | Highest-risk untrusted content; deny normal retrieval | Encrypted restricted CAS + catalog + validation rows | 029-031, 039-041, 047-049 | Retention and administrator access deferred |
| Transformation/normalization manifest | Imported/generated resources | IDR-019/021-024 | Provenance evidence | Canonical structured record; input/output artifacts immutable | activity, inputs/outputs, rules/tool/config/package, actor, clocks, mapping notes | Append-only per transformation | Link validation before/after | By target/input/tool/outcome/time | Retain while outputs or audit obligations exist | May reveal source details and policy decisions | PG provenance/activity tables + CAS for large reports | 029-031, 041, 051-053 | Must distinguish lossless, lossy, redacted, inferred steps |
| Federation/synchronization manifest | External/local resources and artifacts | IDR-019; DDIL plans | Source manifest + normalized sync evidence | Exact signed manifest where used | origin, logical IDs, digests, versions, clocks, dependencies, policy, conflict state | Immutable exchange round; acknowledgment history | Integrity/signature/conflict evidence | Origin/digest/state/time | Until reconciliation and records policy permit disposal | Cross-domain markings and metadata minimization | CAS manifest + PG sync ledger | 029-030, 042-043 | Logical-URI conflicts are not last-write-wins |
| Fixture/golden/corpus artifact | All | IDR-014E/F, 021-024, 050-056 | Test-only source/generated/expected evidence | Exact bytes, filenames, media, expected digest; intentionally malformed bytes preserved safely | Fixture ID, scenario, license/source, expected outcome, requirement links | Immutable versioned corpus | Expected validation/transformation/interoperability results | Scenario/tag/requirement/tool | Retain releases; license-aware | Never serve as production content; sanitize malicious fixtures | Git LFS or test CAS + manifest + CI results | 050-056 | Separate licensed/public/controlled corpora |
| Audit-adjacent metadata | All write/read/admin actions | IDR-019/020; later IDR-041 | Provenance/security event | Tamper-evident facts; payload content minimized | actor/subject, action, target, result, policy decision, correlation, clocks | Append-only; corrections are linked events | Chain/ingest integrity checks | Actor/target/action/time/correlation | Records/legal schedule; immutable archive | Extremely restricted; avoid storing secrets/content unnecessarily | Dedicated PG event family + archive manifests | 029-030, 039-043 | IDR-041 owns audit schema and guarantees |
| Generated indexes, previews, search documents, compiled validators | Any | This report; PostgreSQL/tool docs | Derived cache/index | No source-fidelity claim; reproducible from named inputs | Build key, policy scope, version, watermark, dependencies | Disposable/invalidate on any material input change | Build and parity check | Purpose-specific | Short/rebuildable; release evidence exception | Partition by policy scope; prevent cross-tenant reuse | PG indexes/materialized data + local/object cache | 029-030, 039-040, 048-049, 052-055 | Cache key must include policy and package/tool versions |
| Configuration-adjacent metadata and secret handles | Deployment/package registry | IDR-047 boundary | Normalized metadata, never secret value | Preserve approved config source separately as policy allows | setting name, schema version, source layer, secret handle, effective hash | Deployment revision/history | Config-schema and startup evidence | Admin-only | Operational schedule | Secrets excluded; logs/docs must not echo values | PG/config files/secret manager references | 044-049 | Detailed configuration and secrets explicitly deferred |

### 5.1 Cross-Category Invariants

One source document may derive many resources, and one resource revision may incorporate many documents. The relation is many-to-many and provenance-bearing. The same bytes may have multiple acquisitions, authorities, policies, and logical roles. Conversely, two byte-distinct documents may be semantically equivalent without being interchangeable evidence.

Artifact metadata should conceptually include `artifact_id`, algorithm-qualified digest, byte length, media type and parameters, declared/detected encoding, storage backend/key/version/checksum, logical document/revision IDs, source URI and authority, acquisition activity/time, policy/marking, trust/signature and validation state, dependency edges, and target resource revisions. Final names and decomposition are DDL decisions for later work.

---

## 6. Source Fidelity and Preservation Findings

### 6.1 Preservation Classes

| Class | Required behavior | Examples |
|---|---|---|
| Byte-exact source | Store received bytes before semantic parsing, after bounded transport/security admission; verify digest on retrieval | imported SensorML/SWE, vendor schemas, profile/vocabulary packages, signed manifests, fixtures, policy-permitted rejected inputs |
| Structurally preserved parsed graph | Preserve modeled and unknown members, order where semantically relevant, lexical/value distinctions, reference context, and parse diagnostics | SensorML/SWE trees, JSON Schema resources, OAS modules |
| Canonical normalized state | Store typed facts under Glaux domain invariants; record mapping from source | resource graph, typed relationships, semantic bindings, geometry/time references |
| Deterministically generated view | Regenerate from a complete build key and validate output; do not call it original source | SensorML/GeoJSON/JSON responses, OAS, conformance documents |
| Derived cache/index | Rebuildable and authority-free; invalidated on source, policy, schema, tool, or config change | compiled validators, FTS documents, response cache, previews |

### 6.2 Exact-Byte Rules

PostgreSQL `json` preserves the original JSON text's whitespace, key order, and duplicate keys, while `jsonb` decomposes the value, drops whitespace and order, and retains only the last duplicate key.[^1] Even `json` is not sufficient evidence of the originally received HTTP entity because media-type parameters, charset/BOM, compression, transfer framing, and invalid byte sequences sit outside the JSON value. Exact evidence therefore uses bytes plus transport/acquisition metadata; parsed JSON/JSONB is a separate artifact.

Source bytes must be immutable after admission. “Fixing” an imported document creates a new Glaux-authored artifact linked by an explicit transformation; it never overwrites the vendor/source object. A source digest is recalculated while streaming, compared with any declared checksum/signature under a named policy, and verified on retrieval or audit sampling. Hash success establishes integrity against the recorded bytes, not semantic validity or source trust.

### 6.3 Round-Trip and Representation Rules

Round trip has three distinct tests:

- **byte round trip:** returned source download is exactly the admitted bytes;
- **structural round trip:** parse/serialize preserves all semantically and extension-relevant structure but may change lexical form;
- **semantic round trip:** generated CSAPI representation expresses the same canonical facts under a named profile.

Only the first supports signature verification or forensic equality. Generated SensorML, SWE/JSON, GeoJSON, or OpenAPI output normally promises deterministic, profile-valid semantic representation, not equality with imported bytes. Unknown extensions remain in the parsed/source layer and must not be invented into the canonical graph.

### 6.4 Admission and Quarantine

Strict client writes fail atomically when required structural, semantic, profile, authorization, or safety checks fail. Privileged bulk/import workflows may quarantine exact bytes plus safe metadata when policy allows. Staging first enforces request size, decompression ratio, archive entry, nesting/recursion, reference count/depth, parse time, memory, URI scheme/host, and malware policies. Promotion creates a new immutable validation and transformation activity; it does not erase the rejection history.

---

## 7. Normalization and Indexed Metadata Findings

### 7.1 Typed Normalization Boundary

Normalize fields that carry identity, authorization, stable discovery, relationship, validation, lifecycle, spatial, temporal, or semantic meaning. At minimum this includes resource/document/revision IDs; external IDs and aliases; family/type; lifecycle and validity; typed edges; profile/conformance identifiers; source authority; property/unit/vocabulary identifiers; key classifiers and keywords; contract/package fingerprints; validation status; provenance activity; policy labels; geometry references; and observation/status time links.

Keep validated heterogeneous subtrees in JSONB where their structure is standards-driven but not worth brittle table expansion. Promote a JSONB field into a typed column or expression index only when a named API/filter/policy/test workload needs it, its semantics and type are stable, and write/invalidation cost is understood. Do not expose a generic JSONPath query surface merely because PostgreSQL supports it.

### 7.2 Index Strategy

| Need | Initial pattern | Guardrail |
|---|---|---|
| Identity and lifecycle lookup | B-tree unique/non-unique indexes on typed columns | Respect IDR-016 identity domains and bitemporal rules |
| Relationship traversal | typed relational edge indexes by subject/predicate/object and validity | Do not infer authority from an embedded link alone |
| Spatial/temporal discovery | PostGIS and typed time indexes from IDR-026/027 | Documents do not duplicate authoritative geometry/sample facts |
| Profile, contract, package, validation | B-tree indexes on immutable fingerprints, state, and target | Activation snapshot must be transactionally coherent |
| Approved heterogeneous metadata | narrow JSONB expression or GIN indexes | Validate shape; benchmark write amplification; document supported paths |
| Text discovery | policy-approved `tsvector` over title/description/keywords with pinned configuration | No source body, hidden field, or cross-policy token leakage |
| Artifact lookup | digest, logical document/revision, source authority, state | Digest route is administrative/internal, not public authority |

PostgreSQL GIN can efficiently search JSONB keys/key-value pairs, but indexing every document indiscriminately increases storage and write cost.[^1] Full-text behavior depends on parser/configuration/dictionaries, so the configuration and generated `tsvector` build version must be explicit.[^3] Search results must be built from an authorized projection or secured index partition; filtering unauthorized hits after ranking can leak existence, counts, snippets, or timing.

An external search engine is not selected. It may be reconsidered only if IDR-SRV-054 demonstrates that policy-safe PostgreSQL search cannot meet a concrete workload and the team accepts another derived-store consistency and operations burden.

---

## 8. Versioning, Lifecycle, Retention, and Archival Findings

### 8.1 Identity and Clocks

A logical document owns immutable revisions; a revision references one exact-byte artifact and zero or more parsed/canonical/generated artifacts. Schema `$id`, OpenAPI URL, source URL, artifact digest, storage object version, resource canonical URL, and revision ID remain separate. Supersession is an explicit typed edge; neither URI reuse nor equal digest silently replaces history.

Record separately where applicable: domain valid time; source creation/issue time; receipt time; ingest start/end; authoritative commit time; publication time; validation time; package activation/deactivation time; and archival/deletion time. Revalidation creates a new evidence record and does not rewrite the source revision or prior finding.

### 8.2 Lifecycle States

Package and source workflows need explicit states such as `staged`, `validated`, `active`, `deprecated`, `retired`, `quarantined`, `rejected`, and `archived`. State names and allowed transitions are finalized later, but state transitions must be events with actor/activity, reason, policy, and transaction time. “Deprecated” does not mean unavailable; “retired” does not mean deletable; “archived” must state whether online query, retrieval, and restoration remain supported.

### 8.3 Retention and Collection

Retention follows artifact role, mission/profile, tenant, provenance/audit/legal holds, policy marking, validation reproducibility, release support, synchronization acknowledgments, and downstream references. Resource deletion alone cannot remove evidence used by another resource, package, validation result, fixture release, audit record, or archive manifest.

IDR-SRV-030 should use catalog-rooted reachability/mark-and-sweep with grace periods, staging leases, legal/policy holds, package-release roots, quarantine rules, and archive manifests. Naive reference counts are insufficient because references can be bitemporal, policy-hidden, externally archived, or temporarily inconsistent during repair. Destruction must record authorization and verifiable outcome without retaining prohibited content.

Archive manifests should include logical IDs, digests, sizes, media types, package/contract versions, provenance, policy, time bounds, dependency closure, object locations/versions, encryption/key references, restore procedure/version, verification status, and whether query is online, manifest-only, or restore-required.

---

## 9. Validation Artifact and Diagnostic Storage Findings

Every validation run should bind:

- immutable run ID, target logical document/resource revision, and exact input digest;
- contract/package snapshot and every material schema/profile/vocabulary digest;
- parser/validator/compiler/generator name, version, build, options, limits, and resolver policy;
- caller/workflow, authorization and policy profile, correlation/trace identifiers, and relevant clocks;
- outcome, stage, stable internal finding codes, severity, safe pointer/location, and protected detail;
- normalized/transformed output digest, mapping/provenance activity, and any lossy/unknown-field note.

Validation evidence is append-only. A new validator or vocabulary package produces a new run and may change the current eligibility projection, but historical decisions stay reproducible. The public RFC 9457 problem response is a stable, minimized projection; internal details, rejected values, filesystem paths, schema internals, policy facts, and stack traces remain restricted.

Operational admission evidence, asynchronous revalidation evidence, CI schema checks, conformance-test results, interoperability runs, and fixture expectations have different authorities and retention schedules. They may reuse one evidence model but require separate namespaces, actors, environments, and policy. A passing CI artifact does not prove that a production payload was validated, and a production rejection is not automatically a conformance failure.

Large reports, traces, or third-party output may be stored as immutable report artifacts with normalized run/finding summaries in PostgreSQL. The report digest and parser version bind the opaque report to searchable safe findings. Validation of validation records—required fields, target existence, package closure, and code namespace—belongs in the write transaction design.

---

## 10. Schema, Profile, Vocabulary, and Semantic Artifact Cache Findings

### 10.1 Immutable Offline Package Model

Glaux should package runtime dependencies into immutable, reference-closed sets. A package manifest records package ID/version/digest, member logical URI and digest, media type, dialect/vocabulary, dependency edges, origin/authority, license/handling, trust/signature status, applicability profile, build tool/configuration, and activation state. JSON Schema Draft 2020-12 explicitly supports compound schema documents and distinguishes dialect/vocabulary concepts, but Glaux still needs a controlled logical-URI resolver and collision policy.[^4]

Vendor members remain byte-exact. If an official artifact is incomplete, uses an unstable URI, lacks `$id`, or contains a problematic reference, Glaux adds a separately versioned wrapper, overlay, resolver mapping, or generated bundle. It does not patch the vendor artifact in place. Official CSAPI release bundles with residual relative references remain negative/reference fixtures; a Glaux release package must prove closure.

### 10.2 Resolver and Activation Rules

- Runtime validation, rendering, and request handling do not fetch arbitrary network resources.
- Supported URI schemes, authorities, redirects, recursion, cycles, total bytes, depth, and retrieval time are explicitly bounded.
- A logical URI resolves within the active package snapshot to a recorded digest; ambiguous mappings fail closed.
- Package staging validates member digests, dialects, IDs, reference closure, license/handling, compatibility, and signatures where required.
- Activation is atomic. In-flight work remains bound to its captured snapshot; a package update never changes an existing Observation/Command contract interpretation.
- Missing, denied, stale, quarantined, and conflicting dependencies are distinguishable outcomes.

The Rust `jsonschema` 0.52.0 crate supports Draft 2020-12, meta-validation, structured output, bundling, and custom retrieval, making it a plausible implementation candidate; any network/file retrieval features must be disabled or replaced by the Glaux resolver.[^5] Compiled validators, vocabulary maps, prefix tables, and OAS render models are caches keyed by exact package digest set, tool version, options, target architecture where material, and policy profile.

### 10.3 OpenAPI and Generated Documentation

IDR-SRV-014 remains controlling: typed capability/route/parameter/representation/schema/error/security/conformance registries generate modular source, canonical JSON `service-desc`, YAML alternate, self-hosted HTML `service-doc`, a reference-closed bundle, and an offline archive. The build key includes server release/capability snapshot, named policy profile, schema/package set, generator/toolchain, configuration, media/language, and template/static-asset digests.

OpenAPI 3.1.2 remains the canonical baseline. The newly published 3.2.1 specification is monitored through a separately reviewed migration gate addressing toolchain support, generated clients, schema behavior, security, rendering, parity, and fixtures.[^2] Publication alone does not change stored contracts or advertised conformance.

---

## 11. Metadata/Document Storage Option Evaluation

| Option | Fidelity | Query/transaction behavior | Local/DDIL and operations | Decision |
|---|---|---|---|---|
| Typed PostgreSQL relational tables | Excellent for canonical fields, edges, provenance, lifecycle, policy and atomic changes; not natural exact-byte model alone | Strong constraints, joins, indexes, bitemporal history and transaction coordination | Existing selected authority; mature backup/restore | **Select** for catalog and normalized authority |
| PostgreSQL JSONB | Structural JSON, not lexical/byte fidelity | Good selective expression/GIN query; schema validation remains application/constraint work; broad indexes cost writes | Simple within authority | **Select conditionally** for validated heterogeneous parsed/canonical subtrees |
| PostgreSQL `bytea` with TOAST | Byte-exact; TOAST transparently compresses/moves large values out of row[^6] | Same database transaction as catalog/resource; streaming and very large-object workloads need measurement | Lowest initial complexity and strong backup coherence | **Select initial baseline** for control/small-medium artifacts; benchmark thresholds |
| PostgreSQL Large Objects | Byte-exact and streamable | Separate large-object API/system table; lifecycle and privilege handling add seams[^7] | Backup supported but orphan/ownership operations are more complex | **Do not select by default**; reconsider only for proven streaming need |
| Local filesystem content-addressed store | Byte-exact | Atomic rename/conditional-create patterns possible; catalog/object transaction requires staging and repair | Strong offline simplicity; shared/multi-node and backup semantics require design | **Keep** as adapter backend and development/edge candidate |
| S3-compatible object storage | Byte-exact; provider checksum/version features vary | Conditional create can prevent overwrite; catalog transaction is cross-store; versioning/delete markers are not destruction[^8] | Scalable and portable through adapter, but adds service/credentials/repair | **Keep conditionally** for large-scale deployments after contract/ops tests |
| Dedicated document database | Natural flexible documents; exact bytes still separate | Creates second authority/transaction/migration/policy surface without a demonstrated query gap | Higher operations and DDIL burden | **Reject initially**; evidence gate required |
| External search engine | Derived index only | Powerful text/facets; consistency, policy, deletion and leakage complexity | Additional service and rebuild/monitoring burden | **Reject initially**; performance/security evidence gate required |
| SQLite JSON/reduced profile | Text JSON available; SQLite JSONB is private-format BLOB and not PostgreSQL-compatible[^9] | Adequate bounded local tests/single-node reduced deployments; semantics differ from PostgreSQL | Excellent embedding/offline portability | **Reduced profile only**; never assume behavioral parity |
| DuckDB JSON/Parquet analysis | Useful import/analysis, not authority | Strong offline analytical inspection and corpus/archive verification | Reproducible tool; no serving authority | **Select as optional tool**, not server truth |

### 11.1 Selected Hybrid

The selected conceptual pattern is:

1. **PostgreSQL catalog/graph:** logical identities, revisions, normalized metadata, edges, provenance, policy, package manifests, validation summaries, storage locations, outbox, and lifecycle.
2. **Immutable byte adapter:** PostgreSQL `bytea` initially, with filesystem/S3-compatible backends behind one measured contract where size/throughput/backup evidence warrants.
3. **Derived structures:** parsed/canonical JSONB, compiled packages, generated representations, approved indexes, previews, and caches bound to exact inputs.

For an external byte backend, the safe conceptual write is: receive into bounded staging; stream hash/scan; conditionally create the content key; verify checksum/readability; commit the PostgreSQL catalog/resource/provenance/outbox transaction; then finalize or retain the immutable object. A janitor reconciles expired staging and unreferenced candidates. PostgreSQL remains authority because no distributed transaction is assumed. IDR-SRV-029 must prove concurrent writers, crash points, retries, idempotency, orphan repair, and retrieval verification.

The `object_store` 0.14.1 Rust API provides a common interface for local files and major object APIs and exposes conditional and multipart operations, but backend guarantees are not presumed identical; Glaux needs a storage conformance suite for every supported backend.[^10]

No universal size threshold is set here. Benchmark representative SensorML/SWE, schema/vocabulary packages, OAS bundles, validation reports, fixtures, quarantine payloads, backup/restore, replication, streaming download, garbage collection, and fault recovery. Prefer the simplest `bytea` deployment until measured benefit justifies an external object service.

---

## 12. Provenance, Source Authority, Federation, and DDIL Implications

Provenance is modeled as activities and typed input/output/attribution relations, not mutable columns such as `source = external`. Each acquisition records origin node/URI, asserted author/authority, transport/auth context as permitted, receipt activity and clocks, source-declared identifiers/version/checksum/signature, computed digest, validation policy/outcome, and policy marking. Trust is an evaluated, versioned conclusion separate from origin and byte integrity.

Duplicate bytes are deduplicated physically only when security, encryption, tenant, and backend policy permit. Catalog records remain distinct so provenance and policy are never merged. Conflicting logical URIs or external identifiers create an explicit conflict set; they are not resolved by last arrival, URI string, hash order, or silent overwrite.

DDIL nodes carry immutable package/artifact manifests locally and perform no request-time external schema/vocabulary retrieval. Synchronization exchanges manifests and missing digests, then transmits allowed content; digest-based transfer savings do not erase separate provenance. Resolution returns explicit `available`, `missing`, `denied`, `stale`, `quarantined`, or `conflict` states.

Merge decisions must consider logical/resource identity, origin authority, causal/transaction clocks, domain validity, revision lineage, policy, signatures, package compatibility, and typed conflict rules. IDR-SRV-042/043 own disconnected semantics and conflict algorithms. This report requires only that the catalog retain enough evidence to make those decisions and never mutate historical artifacts during reconciliation.

Archive and synchronization manifests may be signed, but signing algorithm, key distribution, revocation, and cross-domain recognition are unresolved security/profile decisions. A content digest alone is not a signature.

---

## 13. Security, Policy, Releasability, Redaction, and Audit Implications

Metadata can disclose more than payloads: exact locations, capabilities, observed/controlled properties, command affordances, topology, contacts, affiliations, source organizations, profile membership, validation weaknesses, schema names, hidden resource existence, and timing. Policy therefore runs before every derived surface, including search tokens, counts/facets, extents, links, errors, OAS modules, schemas/examples, ETags, caches, previews, and source downloads.

### 13.1 Projection and Redaction Rules

- Exact source and generated public representation are separately authorized resources.
- Policy derives a named projection/profile. If removal breaks the advertised schema, generate a valid safe profile or suppress the document/resource; never claim the original schema still applies.
- Generated cache keys include tenant/security domain, named policy profile, resource revision, package set, generator/configuration, language/media, and authorization-relevant snapshot. Per-user OAS generation is avoided in favor of a small set of reviewed policy profiles.
- Diagnostics use stable public codes/pointers and correlation IDs; protected values, hidden fields, schema internals, policies, paths, SQL, and stack traces stay internal.
- Digests, object keys, existence checks, `ETag`s, timing, and dedup behavior are treated as possible cross-tenant side channels.

### 13.2 Untrusted-Content Controls

Required controls include bounded bytes and nesting; decompression/archive entry and ratio limits; JSON/YAML/XML entity/alias/recursion limits; `$ref` scheme/host/depth/count/size controls; URI normalization; path traversal prevention; SSRF denial; parser/validator time and memory budgets; duplicate-key policy; signature and media sniffing policy; malware scanning where required; and isolation for malicious fixtures. Rendered Markdown/HTML and schema/example descriptions must be sanitized to prevent active content in documentation tools.

Encryption at rest/in transit, backend keys, object credentials, signed URLs, key rotation, tenant isolation, and backup encryption are deployment/security decisions. Secrets are not document artifacts. Configuration metadata may record a secret handle and effective configuration hash, never the secret value.

### 13.3 Audit Boundary

Audit records reference artifact/document/resource IDs and policy decisions while minimizing copied content. Access to exact sources, quarantine, controlled packages, administrative downloads, package activation, provenance changes, redaction profiles, retention holds, export, and destruction are auditable actions. IDR-SRV-041 decides tamper evidence, event schema, retention, authorized viewers, and cross-boundary export.

---

## 14. Fixture, Golden-File, Conformance, and Interoperability Test Implications

### 14.1 Required Corpus Families

| Corpus family | Minimum cases | Assertions |
|---|---|---|
| Exact-source fidelity | whitespace/key-order variants, duplicate JSON keys, BOM/charset, invalid UTF-8, compressed input, signed artifact | byte digest/download equality; parsed differences explicit; no JSONB substitution |
| SensorML/SWE preservation | known/unknown extensions, nested components, nil/quality, constraints, references, legacy/profile-divergent documents | structural preservation, canonical mapping, loss notes, deterministic generation |
| Schema/OAS package | nested relative refs, `$id` collisions, cycles, missing/denied refs, mixed dialects, official residual-reference bundles | offline closure, fail-closed resolution, unchanged vendor bytes, stable diagnostics |
| Vocabulary/semantic package | URI aliases, version changes, deprecated concepts, unit mappings, ambiguous mappings | exact package binding, conservative query, no historical reinterpretation |
| Validation evidence | accept/reject/quarantine, revalidation under new package/tool, internal/public diagnostics | immutable runs, reproducibility, safe projection, current-state derivation |
| Policy/redaction | hidden fields/links/extents/schemas/examples, structurally required field, cross-tenant identical bytes | valid named safe view or suppression; no search/cache/digest leakage |
| Version/lifecycle | supersede/deprecate/retire/archive/restore/delete hold, resource removed while evidence shared | explicit transitions/clocks; dependency-aware retention; restore verification |
| Transaction/backend | concurrent same/different provenance upload, crash before/after object/catalog commit, corrupt object, orphan staging, backend conditional-write races | idempotency, repair, digest verification, no authority split |
| DDIL/federation | missing package, digest-only sync, same URI/different bytes, same bytes/different policy, divergent revisions | explicit states/conflicts; provenance separation; no last-write-wins |
| Generated documents | SensorML, GeoJSON, OAS JSON/YAML/HTML/offline archive, conformance docs | deterministic build keys, runtime parity, reference closure, client/render safety |

### 14.2 Test-Lane Separation

Unit/property tests cover identifier domains, manifest canonicalization, digest verification, resolver bounds, lifecycle transitions, cache keys, and safe diagnostic projection. Integration tests exercise PostgreSQL transactions, TOAST/streaming behavior, storage adapters, crash recovery, backup/restore, and policy-index isolation. Conformance tests map normative requirements to generated representations. Interoperability tests use CSAPI Explorer and external clients/tools against both generated and exact-source endpoints, comparing semantic equality unless exact-byte retrieval is the stated contract.

Golden files carry fixture ID, origin/license, exact digest, media/encoding, expected outcome, requirement links, tool/package pin, and update rationale. Intentionally malicious or controlled fixtures remain isolated and are never shipped or served inadvertently. Current official bundles with unresolved relative references are valuable negative closure fixtures.

Backend parity tests must prove conditional create, checksum verification, range/stream reads if supported, concurrency, crash recovery, listing/repair, deletion/version behavior, credentials, and backup/restore for PostgreSQL `bytea`, local filesystem, and each claimed S3-compatible backend. Passing against one provider does not certify another.

---

## 15. Downstream Topic Handoff Matrix

| Topic(s) | Required handoff from IDR-SRV-028 | Acceptance/evidence gate |
|---|---|---|
| IDR-SRV-029 | catalog/object staging transaction, global idempotency keys, conditional insert, concurrent package activation, outbox, crash/orphan repair | Prove atomic authority and repeatable recovery for every backend |
| IDR-SRV-030 | role-based retention, holds, archive manifests, reachability collection, quarantine expiry, restore/query/destruction guarantees | No deletion schedule may rely on resource age or naive refcount alone |
| IDR-SRV-031 | preserve-before-parse pipeline, strict versus quarantine import, transformation manifests, source/canonical many-to-many relations | Admission must bind exact bytes and validation package before publication |
| IDR-SRV-032/033 | generated representation/service-desc behavior, source-download boundary, explicit filters and safe text discovery | Runtime/OAS parity and policy-before-generation/indexing |
| IDR-SRV-034/035 | immutable SWE contract/package binding for dynamic data and any separately authorized subscription work | Replay/catch-up never changes payload interpretation; Part 3 not pre-authorized here |
| IDR-SRV-036-038 | command/control document fidelity, validation evidence, safety/policy views, audit references | No command execution from unvalidated or reinterpreted document state |
| IDR-SRV-039-041 | threat model, exact-source permission, policy projections, safe diagnostics/search, audit events, controlled artifacts | Test hidden-data leakage across every derived surface |
| IDR-SRV-042/043 | DDIL manifests, missing/denied/stale states, logical-URI conflicts, provenance-preserving synchronization | No uncontrolled dereference or last-write-wins conflict collapse |
| IDR-SRV-044-046 | backend topology, migrations, backup/restore, observability for staging/repair/package activation | Deployment claims require recovery and integrity drills |
| IDR-SRV-047 | package/resolver/backend metadata, configuration schemas, secret handles | Never persist secret values in document/catalog/log/diagnostic artifacts |
| IDR-SRV-048/049 | reproducible reference-closed build packages; bounded parsers/validators; dependency/license/signature management | Release manifest pins every material artifact/tool and passes supply-chain checks |
| IDR-SRV-050/051 | package closure, generated-document parity, validation evidence, requirement-to-fixture traceability | Every advertised conformance claim has pinned executable evidence |
| IDR-SRV-052/053 | property/fuzz/model tests and exact corpus/golden organization | Include all Section 14 source, security, lifecycle, crash and DDIL cases |
| IDR-SRV-054/055 | representative artifact/search benchmarks and benchmark-gated `bytea`/object/search thresholds | No external store/search adoption without measured benefit and failure/ops evidence |
| IDR-SRV-056 | external client/tool round trips, generated versus source semantics, OAS 3.1.2 bundle use | Record exact client/tool/package versions and distinguish byte/structural/semantic equality |

---

## 16. Recommendations

1. **Adopt the six-role source/parsed/canonical/generated/evidence/cache architecture. [High]** Never collapse exact bytes, parsed structure, canonical facts, generated views, validation evidence, or disposable indexes.
2. **Keep PostgreSQL/PostGIS as the authoritative catalog and normalized graph. [High]** Store identity, revisions, typed metadata/relationships, provenance, lifecycle, policy, packages, validation summaries, and transaction coordination there.
3. **Use immutable algorithm-qualified content addressing through a backend adapter. [High]** Treat digest as byte integrity/location, not resource identity, trust, authorization, or public URL.
4. **Start with PostgreSQL `bytea`/TOAST for control and small-to-medium artifacts. [High]** Add local/S3-compatible object storage only after representative benchmark, recovery, backup, and operations evidence defines a threshold.
5. **Preserve exact imported bytes before parsing when admission policy permits. [High]** Record media/encoding/size/source/acquisition/policy metadata, and make any repair a new linked artifact.
6. **Normalize stable semantic and policy fields into typed structures. [High]** Use validated JSONB selectively; prohibit generic public JSONPath and evidence-free blanket GIN indexing.
7. **Use policy-built PostgreSQL text discovery initially. [Medium]** Pin search configuration and index only approved generated fields; require a workload/security gate before external search.
8. **Version documents, packages, transformations, and validation results immutably. [High]** Preserve distinct validity, receipt, commit, publication, validation, activation, and retirement clocks.
9. **Create immutable reference-closed schema/profile/vocabulary packages. [High]** Preserve vendor bytes, store wrappers/overlays separately, map logical URIs to digests, activate atomically, and forbid uncontrolled runtime network retrieval.
10. **Bind every derived or validated artifact to complete inputs. [High]** Include source/resource revision, contract/package digests, tool/options/configuration, policy profile, language/media, and transformation activity.
11. **Apply policy before every derived surface. [High]** Exact source is separately authorized; redaction produces a valid named safe projection or suppression, never a mislabeled invalid document.
12. **Store validation as immutable provenance-bearing evidence with safe projections. [High]** Separate production, CI, conformance, and interoperability namespaces and retention.
13. **Make retention dependency- and policy-aware. [High]** IDR-SRV-030 should use catalog-rooted reachability, holds, grace periods, archive manifests, and verified destruction/restore rather than document TTL or naive refcount.
14. **Require backend, parser/resolver, fidelity, policy, crash, DDIL, and interop fixture suites. [High]** Test malicious and conflicting inputs as first-class cases.
15. **Retain OpenAPI 3.1.2 until a gated migration. [Medium]** Monitor 3.2.1, but require tool/client/security/parity evidence and an explicit project decision before changing the baseline.

---

## 17. Risks, Constraints, and Open Questions

### 17.1 Risks and Mitigations

| Risk | Consequence | Current mitigation/owner |
|---|---|---|
| JSONB used as original document | Lost lexical/duplicate-key/signature evidence | Exact bytes + separate parsed/canonical layers; 031/053 tests |
| Digest conflated with identity/trust | provenance/policy collision and side-channel exposure | Separate identity domains/catalog records; 039/040 controls |
| External object store creates split authority | orphan objects, missing catalog rows, inconsistent retry | staged conditional write, PG authority, reconciliation; 029 proof |
| Over-normalized standard documents | brittle schema and lost extensions | typed stable core + preserved parsed/source subtree |
| Opaque-only storage | weak discovery, policy and validation | deliberate typed normalization/index rubric |
| Runtime network dereference | SSRF, nondeterminism, DDIL failure, supply-chain risk | immutable offline packages and controlled resolver |
| Redaction breaks schema | misleading interoperability and leakage | named safe profile or suppression |
| Search/cache crosses policy boundaries | existence/content disclosure | policy-built documents, scoped keys/indexes, leakage tests |
| Retention removes shared evidence | irreproducible validation/audit/history | dependency-aware roots/holds/manifests; 030 |
| Provider claims assumed portable | data loss or failed concurrency/recovery | per-backend contract and recovery suite |
| Controlled/licensed artifact mishandling | policy/legal violation | metadata minimization, controlled adapter/catalog/access/audit |
| New specification version silently adopted | tool/client incompatibility and changing contract | explicit OpenAPI migration gate |

### 17.2 Open Questions and Decision Owners

- What measured artifact sizes, throughput, replication, backup and restore conditions trigger filesystem/S3-compatible storage instead of `bytea`? **Owners:** 029, 044-046, 054-055.
- Does the initial server profile accept legacy/XML SensorML and SWE sources, and which byte/structural round-trip promises apply? **Owners:** 031-034, project profile.
- Which exact source documents receive client-visible download endpoints versus restricted administrator access? **Owners:** 032-033, 039-040.
- What are retention durations, quarantine expiry, legal/mission holds, archive formats/tiers, restore service levels, and destruction proofs? **Owner:** 030 with policy/operations.
- What exact AEP/STANAG schema/profile/vocabulary package members and handling rules are deployment-approved? **Owner:** project/profile authority; 040/047/050.
- Are digital signatures required for imported documents, packages, manifests, or releases; which trust anchors and revocation rules apply? **Owners:** 039, 045, 048.
- Which title/description/keyword languages and stemming configurations are supported, and does workload justify external search? **Owners:** 032/033/040/054.
- When does OpenAPI 3.2.x provide enough benefit and ecosystem support to migrate from 3.1.2? **Owners:** 014/045/050/056 migration review.
- May physical deduplication cross tenant/security domains, or must encryption/key boundaries defeat global dedup? **Owners:** 039/040/044.

---

## 18. Validation Against This Plan's Success Criteria

| Topic Plan Success Criterion | Validation Status | Evidence |
|---|---|---|
| Metadata and document categories are identified with source anchors | Met | Section 5 required-field matrix |
| Source, normalized, generated, validation, schema/profile, vocabulary, cache, fixture, and audit/provenance roles are distinguished | Met | Sections 4-6 and matrix classification column |
| Fidelity, round-trip, normalization, versioning, lifecycle, and validation implications are documented | Met | Sections 6-9 |
| Query/index, policy, retention, archival, DDIL, and package-cache implications are documented | Met | Sections 7-8, 10, 12-13 |
| Candidate storage options are evaluated against explicit criteria | Met | Sections 4.1 and 11 |
| Security, policy, fixture, conformance, and interoperability implications are documented | Met | Sections 13-14 |
| Implementation-study and community findings are informative, not normative | Met | Sections 3.3-3.4; evidence labels |
| Recommendations are decision-usable and bounded to Glaux Server | Met | Sections 2.2 and 16 |
| Downstream handoffs are explicit | Met | Section 15 and open-question owners in Section 17 |
| References are explicit and reproducible | Met | Header pins, Section 3, footnotes, and Section 19 |

The report is complete as research and is **In Review**. It is not accepted for downstream use until the Glaux Project Lead records acceptance in this report, its topic plan, and the overall plan.

---

## 19. References

### 19.1 Standards and Project Sources

- [OGC API - Connected Systems - Part 1: Feature Resources, OGC 23-001](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2: Dynamic Data, OGC 23-002](https://docs.ogc.org/is/23-002/23-002.html)
- [Official OGC API - Connected Systems repository, approved `v1.0.0` source pin](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)
- [OGC API - Features - Part 1: Core, OGC 17-069r4](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [OGC SensorML Encoding Standard 3.0, OGC 23-000](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC SWE Common Data Model Encoding Standard 3.0, OGC 24-014](https://docs.ogc.org/is/24-014/24-014.html)
- [W3C/OGC Semantic Sensor Network Ontology](https://www.w3.org/TR/vocab-ssn/)
- [OpenAPI Specification 3.1.2](https://spec.openapis.org/oas/v3.1.2.html)
- [OpenAPI Specification 3.2.1](https://spec.openapis.org/oas/v3.2.1.html)
- [JSON Schema Draft 2020-12](https://json-schema.org/draft/2020-12)
- [JSON Schema Core, Draft 2020-12](https://json-schema.org/draft/2020-12/json-schema-core)
- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)
- [IDR-SRV-028 Research Plan](../IDR%20Plans/idr-srv-028-metadata-and-document-storage-strategy.md)
- [OGC API - Connected Systems Upstream-History Register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md)
- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Research Report Template](../../../../../Governance/research-report-template.md)
- Accepted [IDR-SRV-014](idr-srv-014-openapi-description-and-api-documentation-strategy-report.md), [IDR-SRV-015](idr-srv-015-canonical-glaux-server-resource-model-report.md), [IDR-SRV-016](idr-srv-016-identifier-uri-and-resource-lifecycle-strategy-report.md), [IDR-SRV-017](idr-srv-017-relationship-and-linkage-model-report.md), [IDR-SRV-018](idr-srv-018-temporal-validity-and-freshness-model-report.md), [IDR-SRV-019](idr-srv-019-provenance-lineage-quality-and-trust-metadata-model-report.md), and [IDR-SRV-020](idr-srv-020-status-availability-and-system-event-model-report.md)
- Accepted [IDR-SRV-021](idr-srv-021-sensorml-representation-strategy-report.md), [IDR-SRV-022](idr-srv-022-swe-common-data-component-strategy-report.md), [IDR-SRV-023](idr-srv-023-schema-and-encoding-validation-strategy-report.md), and [IDR-SRV-024](idr-srv-024-units-observed-properties-and-semantic-binding-strategy-report.md)
- Accepted [IDR-SRV-025](idr-srv-025-database-and-persistence-architecture-options-report.md), [IDR-SRV-026](idr-srv-026-geospatial-storage-and-query-strategy-report.md), and [IDR-SRV-027](idr-srv-027-time-series-observation-storage-strategy-report.md)
- Controlled project source: `AC/224(JCGISR)D(2026)0005`, April 27, 2026, SHA-256 `56dc757b6e677b3584e3152a957849f21a24b22854f562613ff283a8b599da8c` (not redistributed; used only through accepted findings)

### 19.2 Primary Technology Sources

- [PostgreSQL 18: JSON Types](https://www.postgresql.org/docs/18/datatype-json.html)
- [PostgreSQL 18: TOAST](https://www.postgresql.org/docs/18/storage-toast.html)
- [PostgreSQL 18: Large Objects](https://www.postgresql.org/docs/18/lo-intro.html)
- [PostgreSQL 18: Full-Text Search Controls](https://www.postgresql.org/docs/18/textsearch-controls.html)
- [PostgreSQL 18: Tables and Indexes for Full-Text Search](https://www.postgresql.org/docs/18/textsearch-tables.html)
- [Rust `object_store` 0.14.1](https://docs.rs/object_store/0.14.1/object_store/)
- [Rust `jsonschema` 0.52.0](https://docs.rs/jsonschema/0.52.0/jsonschema/)
- [SQLx 0.9.0 PostgreSQL Type Mappings](https://docs.rs/sqlx/0.9.0/sqlx/postgres/types/)
- [`serde_json` 1.0.151](https://docs.rs/serde_json/1.0.151/serde_json/)
- [SQLite JSON Functions](https://www.sqlite.org/json1.html)
- [DuckDB JSON Type](https://duckdb.org/docs/stable/data/json/json_type)
- [Amazon S3 Conditional Writes](https://docs.aws.amazon.com/AmazonS3/latest/userguide/conditional-writes.html)
- [Amazon S3 Delete Markers](https://docs.aws.amazon.com/AmazonS3/latest/userguide/DeleteMarker.html)

### 19.3 Footnotes

[^1]: PostgreSQL Global Development Group, [PostgreSQL 18 JSON Types](https://www.postgresql.org/docs/18/datatype-json.html), checked September 14, 2026. The source documents the preservation differences between `json` and `jsonb` and JSONB indexing options.
[^2]: OpenAPI Initiative, [OpenAPI Specification 3.1.2](https://spec.openapis.org/oas/v3.1.2.html), published September 19, 2025; and [OpenAPI Specification 3.2.1](https://spec.openapis.org/oas/v3.2.1.html), published September 10, 2026. Checked September 14, 2026.
[^3]: PostgreSQL Global Development Group, [Controlling Text Search](https://www.postgresql.org/docs/18/textsearch-controls.html) and [Tables and Indexes](https://www.postgresql.org/docs/18/textsearch-tables.html), PostgreSQL 18, checked September 14, 2026.
[^4]: JSON Schema project, [Draft 2020-12](https://json-schema.org/draft/2020-12), [Core specification](https://json-schema.org/draft/2020-12/json-schema-core), and [release notes](https://json-schema.org/draft/2020-12/release-notes), checked September 14, 2026.
[^5]: `jsonschema` maintainers, [`jsonschema` 0.52.0 documentation](https://docs.rs/jsonschema/0.52.0/jsonschema/), checked September 14, 2026.
[^6]: PostgreSQL Global Development Group, [PostgreSQL 18 TOAST](https://www.postgresql.org/docs/18/storage-toast.html), checked September 14, 2026.
[^7]: PostgreSQL Global Development Group, [PostgreSQL 18 Large Objects](https://www.postgresql.org/docs/18/lo-intro.html), checked September 14, 2026.
[^8]: Amazon Web Services, [S3 conditional writes](https://docs.aws.amazon.com/AmazonS3/latest/userguide/conditional-writes.html) and [delete markers](https://docs.aws.amazon.com/AmazonS3/latest/userguide/DeleteMarker.html), checked September 14, 2026. Used only as an example of S3 semantics; Glaux does not require AWS.
[^9]: SQLite project, [JSON Functions and Operators](https://www.sqlite.org/json1.html), checked September 14, 2026.
[^10]: Apache Software Foundation contributors, [`object_store` 0.14.1 documentation](https://docs.rs/object_store/0.14.1/object_store/), checked September 14, 2026.

---

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
- [x] Executive summary is independently readable
- [x] Recommendations are explicit and actionable
- [x] Risks and open questions are documented
- [x] Success criteria validation is complete
- [x] Plan-owner acceptance and acceptance date are recorded
- [x] Next steps and owners are identified

---

**Acceptance record:** Accepted by the Glaux Project Lead on September 14, 2026. IDR-SRV-029 was authorized as the next bounded single-topic iteration; no later topic, draft Part 3 implementation, or server implementation was authorized.

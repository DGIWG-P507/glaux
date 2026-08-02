# Section 011: Query, Filtering, Sorting, Pagination, and Selection Semantics - Research Report

**Topic ID:** IDR-SRV-011<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-011 Query, Filtering, Sorting, Pagination, and Selection Semantics](../IDR%20Plans/idr-srv-011-query-filtering-sorting-pagination-and-selection-semantics.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core and 55 top-level detailed questions; all six methodology phases, ten success criteria, nineteen required content areas, and fourteen minimum query-matrix fields are validated<br>
**Methodology Used:** Reconciliation of accepted IDR-SRV-006 through IDR-SRV-010A with approved OGC standards; mechanical review of every tagged CSAPI Part 1 and Part 2 query parameter, path binding, schema, and relevant abstract test; bounded review of current official issues, pull requests, release/tag history, OGC query building blocks, implementations, and clients; and three independent read-only evidence audits<br>
**Research Time:** Approximately 6 hours of AI-assisted elapsed execution time, including three parallel independent read-only audits, on August 1, 2026<br>
**Primary Sources:** OGC 23-001, OGC 23-002, OGC 17-069r4, the official CSAPI `v1.0.0` tag, OGC 19-079r2, OGC 21-065r2, OGC 20-004r1, OGC 23-058r2, RFC 9110, and RFC 6585<br>
**Supporting Resources:** Accepted IDR-SRV-001 through IDR-SRV-010A reports; the shared upstream-history register; tagged OpenAPI, JSON Schema, and ATS source; current official issue/PR state; Unicode 17.0.0 text algorithms; OWASP API Security 2023 guidance; pinned connected-systems-go, OpenSensorHub, OS4CSAPI client, and CSAPI Explorer source; and bounded non-reproducible live implementation illustrations<br>
**Document Purpose:** Establish a human-readable and implementation-usable query contract baseline for the Rust Glaux reference server without confusing standards obligations, official artifact defects, implementation precedent, or future extensions<br>
**Author:** OpenAI Codex, with independent read-only Part 1/Features, Part 2/dynamic-data, and optional-extension/interoperability audits<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** August 2, 2026<br>
**Date:** August 1, 2026<br>
**Last Updated:** August 2, 2026

---

## 1. Executive Decision Summary

Glaux can have a strong, predictable query API without creating a new query language. The initial stable Rust server should implement the complete OGC API - Connected Systems (CSAPI) Part 1 and Part 2 Advanced Filtering classes selected by the accepted Glaux conformance profile. It should also implement the inherited `limit`, `bbox`, `datetime`, response, and paging behavior wherever the approved standards actually incorporate it. The same typed query registry should drive Rust parsing, endpoint behavior, OpenAPI, documentation, authorization, indexes, and tests.

The approved CSAPI standards define a substantial filter surface: IDs and UIDs, keyword search, WKT geometry, hierarchy, procedures, features of interest, observed and controlled properties, deployments and Systems, stream times, Observation times, Command state and sender, and System Event type. Different query parameters combine with logical **AND**; alternatives inside one list-valued parameter use logical **OR**. Valid identifiers that match nothing produce an empty result. Malformed or undeclared parameters produce `400`, except a valid `limit` above the advertised maximum is clamped.

The standards do **not** define client-controlled sorting, `offset`, cursor syntax, field projection, embedded-resource expansion, or generic CQL2 support for CSAPI. Glaux should therefore publish none of those as baseline CSAPI parameters. It should still guarantee deterministic paging internally: a documented total order, opaque confidential/integrity-protected `next` links, stable ID tie-breakers, caller-visible counts only, and snapshot-bound continuation when a repeatable mission query requires it. A future sorting extension should align with OGC `sortby` plus Sortables rather than invent `sort` and `order`; a future CQL2 profile should be separately advertised and justified by a demonstrated use case.

The official tagged OpenAPI files are useful but are not a conformance contract. They omit required parameters, advertise several filters that have no approved requirement, model temporal intervals incompatibly with the approved slash grammar, and retain removed history routes. Glaux must generate its contract from the approved requirements plus an explicit compatibility ledger—not copy those files literally.

### 1.1 Proposed Glaux Baseline

1. **Implement and eventually claim both CSAPI Advanced Filtering classes completely.** Claim them only after every applicable canonical, nested, deployment-scoped, feasibility-related, and typed collection endpoint passes the full matrix.
2. **Use one typed query registry.** Each entry records route/family applicability, wire name, type, serialization, predicate, defaults, cost limits, ordering, standards source, OAS disposition, authorization rule, and test anchors.
3. **Adopt the tagged `limit` profile initially:** minimum `1`, default `10`, maximum `10,000`; publish those values. Clamp larger valid integers, reject zero/negative/non-integers, and treat any later default or maximum change as a contract change.
4. **Use approved temporal grammar.** Accept RFC 3339 instants and slash-delimited closed or half-bounded intervals. Permit `latest` only for Observation `resultTime`. Do not baseline date-only values, `now`, `earliest`, or generalized `latest` merely because the tagged schema lists some of them.
5. **Keep filter semantics exact.** One comma-separated occurrence for list parameters; OR within the list; AND across parameters; `q` uses Unicode 17.0.0 `toNFKC_Casefold` followed by code-point substring matching over at least `name` and `description`; no undocumented lemmatization in the first stable contract.
6. **Preserve the spatial distinction.** `bbox` includes otherwise matching features without geometry as inherited from Features; `geom` is WKT intersection and excludes geometry-less features.
7. **Apply authorization first.** Filtering, ordering, totals, extents, page boundaries, and links operate only on the caller-visible relation. Continuations bind a pseudonymous principal/security context and policy version without exposing raw sensitive identifiers.
8. **Page by opaque links, not a public offset contract.** Require `next` whenever Glaux knows more visible results exist; use keyset continuation and a stable unique-ID tie-breaker; include snapshot/watermark state where repeatability is promised.
9. **Return full schema-conformant resources.** Do not baseline `select`, `fields`, `properties`, `expand`, `embed`, or summary projections. Representation selection belongs to IDR-SRV-012.
10. **Do not baseline client-controlled sorting.** Freeze an internal/publicly documented default order per resource family, but expose no sort parameter until an optional standards-aligned extension is approved.
11. **Treat OAS-only filters individually.** Repair missing normative parameters; reject ambiguous or deliberately omitted ones; use a bounded compatibility alias only where semantics are unambiguous and the accepted compatibility policy permits it.
12. **Budget every query dimension.** Limit list cardinality, text/WKT complexity, hierarchy traversal, temporal breadth, scanned rows, execution time, memory, response bytes, concurrency, and rate without silently changing filter meaning.

### 1.2 Plain-Language Processing Model

For every collection request, Glaux should produce the observable result of one consistent logical sequence. An implementation may safely combine or reorder physical predicates and index operations only when the returned resources, metadata, errors, timing behavior, and cost controls cannot disclose an unauthorized row:

1. authenticate the caller and logically derive the permitted resource relation;
2. establish the route scope, including collection membership, parent resource, and `recursive` behavior;
3. parse and validate all declared parameters and cost limits;
4. apply each filter with AND semantics and list alternatives with OR semantics;
5. resolve relationship and hierarchy matches cycle-safely and deduplicate by canonical resource identity;
6. apply the family’s stable total order and immutable ID tie-breaker;
7. establish the page snapshot or watermark when the selected consistency level requires it;
8. return at most the effective `limit`, exact `numberReturned`, optional exact visible `numberMatched`, and an opaque `next` link when more visible results are known; and
9. encode complete resources in the representation selected under IDR-SRV-012.

This ordering is both a design rule and a test oracle. It prevents a page token, count, or timing shortcut from bypassing authorization or changing query meaning.

---

## 2. Scope and Plan Alignment

### 2.1 In Scope

- Approved and inherited query obligations for all selected Part 1 and Part 2 resource families.
- Tagged OpenAPI parameter and path bindings, schema constraints, ATS coverage, and their conflicts with approved text.
- Filter grammar, combination, hierarchy, relationship, geospatial, temporal, status, and tasking semantics.
- Deterministic order, link-driven paging, counts, changing datasets, and authorization interaction.
- Classification of sorting, projection, CQL2, schema discovery, asynchronous query, and implementation-specific options.
- Direct handoffs to persistence, indexing, validation, errors, OpenAPI, security, performance, and testing topics.

### 2.2 Out of Scope

- Database engine, schema, index implementation, query optimizer, or Rust framework selection.
- Final media-type and content-negotiation rules, owned by IDR-SRV-012.
- Final error document schema and machine error codes, owned by IDR-SRV-013.
- Final resource/relationship/temporal model details, owned by IDR-SRV-015 through IDR-SRV-020.
- Authentication protocol, releasability rules, or policy language, owned by IDR-SRV-039/039A/040.
- Implementing or beginning any later research plan.

### 2.3 Dependencies and Accepted Inputs

IDR-SRV-006 and IDR-SRV-007 provide the accepted requirement inventories. IDR-SRV-008 selects all 25 CSAPI conformance classes as the target end state, making both Advanced Filtering classes part of the Glaux goal while still requiring truthful staged declarations. IDR-SRV-009 establishes one capability registry and generated API surface. IDR-SRV-010 establishes canonical, nested, collection, relationship, and paging-link identity. IDR-SRV-010A requires stable defaults and query semantics, explicit artifact adapters, immutable evidence pins, and prior-client replay.

### 2.4 Core Research Question Coverage

| Core question | Status | Direct answer / location |
|---|---|---|
| Q1. What behavior is required or expected? | Complete | §§1, 5–9 and Appendix B |
| Q2. What is inherited versus CSAPI-specific? | Complete | §§3.4, 5.1–5.4 and matrix classification |
| Q3. Which families differ? | Complete | §6 and Appendix B |
| Q4. How do spatial, temporal, identifier, relationship, status, and dynamic-data queries work? | Complete | §§7.2–7.8 |
| Q5. What downstream implications follow? | Complete | §§10–15 |

All 55 detailed questions are answered in Appendix A. All six methodology phases and ten success criteria are validated in §18 and Appendix D.

---

## 3. Evidence Base and Authority Classification

### 3.1 Primary Sources Reviewed

| Source | Version / pin | Authority | Anchors used | Access | Limitations |
|---|---|---|---|---|---|
| [OGC 23-001, CSAPI Part 1](https://docs.ogc.org/is/23-001/23-001.html) | Version 1.0; approved; published 2025-07-16 | Controlling normative CSAPI source | §§7.7–7.8, 8.7, 9–16; Reqs 3, 6, 10–12, 15, 20–22, 26, 30, 35, 38–59; Annex A | 2026-08-01 | Contains overview/requirement, route, ATS, and artifact conflicts |
| [OGC 23-002, CSAPI Part 2](https://docs.ogc.org/is/23-002/23-002.html) | Version 1.0; approved; published 2025-07-16 | Controlling normative CSAPI source | §§8–13; Reqs 2, 7, 12, 20, 24, 31, 34, 41, 45–62; Annex A | 2026-08-01 | Contains copy/paste targets, missing property bindings, and ATS/path conflicts |
| [OGC 17-069r4, OGC API - Features Part 1](https://docs.ogc.org/is/17-069r4/17-069r4.html) | Version 1.0.1 corrigendum | Normative where CSAPI incorporates it | §§7.5, 7.15.2–7.15.8; Reqs 8–9, 21–32 | 2026-08-01 | CSAPI adapts feature wording for non-feature resources |
| [Official CSAPI repository release tag](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2) | `v1.0.0`, commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2` | Official fixed supporting artifact | Part 1/2 OAS, JSON Schemas, source requirements, ATS, examples | 2026-08-01 | Artifacts cannot override approved normative text |
| [OGC API - Features Part 3: Filtering](https://docs.ogc.org/is/19-079r2/19-079r2.html) | Version 1.0 | Optional standards context | Queryables, `filter`, `filter-lang`, features-filter binding | 2026-08-01 | Not inherited by CSAPI merely because it exists |
| [CQL2](https://docs.ogc.org/is/21-065r2/21-065r2.html) | Version 1.0 | Optional predicate-language context | Basic, text/JSON and operator classes | 2026-08-01 | Does not bind itself to CSAPI endpoints |
| [OGC API - Records Part 1](https://docs.ogc.org/is/20-004r1/20-004r1.html) | Version 1.0 | Comparative OGC sorting pattern | Sorting Reqs 43–47, `sortby`, Sortables | 2026-08-01 | Records-specific; not CSAPI inheritance |
| [OGC API - Features Part 5 / Common Part 3](https://docs.ogc.org/is/23-058r2/23-058r2.html) | Version 1.0; approved current publication | Comparative schema/queryable/sortable discovery | Queryables, Sortables, Returnables/Receivables | 2026-08-01 | Sortables advertise keys; Features Part 8 sorting behavior is not yet an approved dependency |
| [OGC API - Environmental Data Retrieval](https://docs.ogc.org/is/19-086r6/19-086r6.html) | Version 1.1 | Comparative parameter-selection pattern | `parameter-name` | 2026-08-01 | Domain-specific; not generic CSAPI field projection |
| [OGC API - Processes Part 1](https://docs.ogc.org/is/18-062r2/18-062r2.html) | Version 1.0 | Comparative asynchronous execution pattern | Job/execution behavior | 2026-08-01 | Process jobs are not ordinary CSAPI collection queries |
| [Unicode Standard 17.0.0, Chapter 3](https://www.unicode.org/versions/Unicode17.0.0/core-spec/chapter-3/) and [UAX #15 Revision 57](https://www.unicode.org/reports/tr15/tr15-57.html) | Unicode 17.0.0 | Formal text-normalization/caseless-matching source for Glaux policy | `toNFKC_Casefold`, normalization conformance | 2026-08-01 | Project policy; CSAPI does not select a Unicode algorithm |
| [OWASP API Security Top 10 2023: API1](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/), [API3](https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/), and [API4](https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/) | 2023 edition | Non-normative security-practice evidence | Object/property authorization and bounded resource consumption | 2026-08-01 | Awareness guidance; not an OGC or Glaux conformance obligation |
| [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html) | Internet Standard, June 2022 | Normative HTTP semantics | 400, 404 concealment, 406, caching context | 2026-08-01 | Does not define CSAPI query grammar |
| [RFC 6585](https://www.rfc-editor.org/rfc/rfc6585.html#section-4) | Standards Track, April 2012 | Normative HTTP extension | 429 and `Retry-After` | 2026-08-01 | Rate policy remains a project decision |

### 3.2 Official Artifact and History Evidence

| Source | Pin / state | Finding and use | Limitation |
|---|---|---|---|
| [Part 1 modular OAS](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/openapi-connectedsystems-1.yaml) | Raw tagged-byte SHA-256 `b40a3a86dda90824b204fb9882e4238f533ea2a66f1d0540051c5d386660b546` | Mechanical inventory of paths and parameter bindings | Example OAS `info.version` is `0.0.1`; incomplete/defective |
| [Part 2 modular OAS](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/openapi-connectedsystems-2.yaml) | Raw tagged-byte SHA-256 `b1708f904711dc6268dcda4ccbeed70b9170ab78b32fa1325e602bdd8992f366` | Mechanical inventory of dynamic-resource parameters and stale paths | Omits normative surface and retains removed System History |
| [Part 1 advanced-filter source](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/standard/sections/clause_15_requirements_class_advanced_filtering.adoc) | Raw tagged-byte SHA-256 `2fd6e0a9d0167aeb839b3455abf94a1a5b4feb8f05a4ddeb7907d91f16aa1ba7` | Reproducible source/ATS/OAS inclusion review | Source clause number differs from published HTML clause 16 |
| [Part 2 advanced-filter source](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/clause_14_requirements_class_advanced_filtering.adoc) | Raw tagged-byte SHA-256 `49b670119bb6d9553b29c28678fb0b96d6e8cfad9645e9992b3a0641be68e021` | Reproducible dynamic filter extraction | Source clause number differs from published HTML clause 13 |
| [Part 1 ATS](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/standard/sections/annex-abstract-test-suite.adoc) / [Part 2 ATS](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/annex-abstract-test-suite.adoc) | Raw tagged-byte SHA-256 `00de697cc76282944556c8eb1714bf8179f555afde7c0ab1ab2505f23a87da36` / `9d98dd59f88e0d84797e68c6956725e2991abbfaf06f38ef17be1ecbd4d37d86` | Exact positive-test inventory and defect ledger in §14.3 | Incomplete coverage and copied-target defects require supplemental tests |
| [Upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md) | Version 1.4 at research start; Version 1.5 after this topic's refresh | Bounded issue/PR/release dispositions and routing | Mutable history never silently amends Version 1.0 |
| [Current official branch](https://github.com/opengeospatial/ogcapi-connected-systems/commit/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f) | `master` at `3fd86c73...`, dated 2026-04-20 | No query-relevant tagged-source repair; only one example differs from tag | Mutable retrieval-date pin |
| [Issue #64](https://github.com/opengeospatial/ogcapi-connected-systems/issues/64) and [PR #88](https://github.com/opengeospatial/ogcapi-connected-systems/pull/88) | Issue closed; PR closed unmerged | `earliest` was not adopted; comments changed direction | History, not approved behavior |
| [Issue #165](https://github.com/opengeospatial/ogcapi-connected-systems/issues/165) | Open; updated 2026-07-21 | FOI/link/sampling-chain matching still needs SWG resolution | Current recommendation remains unapproved |
| [Issue #175](https://github.com/opengeospatial/ogcapi-connected-systems/issues/175) | Open; updated 2026-07-23 | Confirms no CSAPI sorting and discusses standards-aligned future options | Proposals only |
| [Issue #179](https://github.com/opengeospatial/ogcapi-connected-systems/issues/179) | Open; updated 2026-07-21 | Property-filter source and OAS binding remain partly unclear | Targeted future Parts 1/2 work, not current obligation |
| [Issue #169](https://github.com/opengeospatial/ogcapi-connected-systems/issues/169) and [PR #196](https://github.com/opengeospatial/ogcapi-connected-systems/pull/196) | Issue open; PR approved but unmerged on 2026-08-01 | Tagged `/deployments` OAS omits required `recursive` | Approved PR still does not alter release tag |
| [Issue #182](https://github.com/opengeospatial/ogcapi-connected-systems/issues/182) | Open; updated 2026-07-21 | Resource `validTime` open-bound/schema issue affects later temporal modeling | Routed primarily to IDR-SRV-018/023 |

### 3.3 Supporting Implementation and Client Evidence

| Source | Pin | Relevance | Limitation |
|---|---|---|---|
| [connected-systems-go query model](https://github.com/OS4CSAPI/connected-systems-go/blob/e900da88738cca92872038b703c4ad537fc0c8fd/internal/model/query_params/query_params.go#L34-L101) and [collection formatter](https://github.com/OS4CSAPI/connected-systems-go/blob/e900da88738cca92872038b703c4ad537fc0c8fd/internal/model/formaters/formatter.go#L170-L192) | `e900da88738cca92872038b703c4ad537fc0c8fd` | Public nonstandard offset links plus exact standard counts | Informative Go implementation, not authority; this pin does not implement cursors |
| [OpenSensorHub listing](https://github.com/OS4CSAPI/osh-core/blob/b2badae59aaa78455c5638ad73b452ccdee40207/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/resource/BaseResourceHandler.java#L240-L285), [links](https://github.com/OS4CSAPI/osh-core/blob/b2badae59aaa78455c5638ad73b452ccdee40207/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/BaseHandler.java#L427-L459), [envelope](https://github.com/OS4CSAPI/osh-core/blob/b2badae59aaa78455c5638ad73b452ccdee40207/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/resource/ResourceBindingJson.java#L153-L210), [separate count route](https://github.com/OS4CSAPI/osh-core/blob/b2badae59aaa78455c5638ad73b452ccdee40207/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/resource/BaseResourceHandler.java#L101-L107), and [count computation](https://github.com/OS4CSAPI/osh-core/blob/b2badae59aaa78455c5638ad73b452ccdee40207/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/resource/BaseResourceHandler.java#L470-L482) | `b2badae59aaa78455c5638ad73b452ccdee40207` | Offset/limit, `limit+1`, collection-envelope, and count-subresource precedent | Envelope omits standard counts, but implementation has a separate count subresource; informative Java only |
| [OS4CSAPI client query model](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/6fdf39669e8b11fe1e9c0c8914278c5f11a3b57f/src/ogc-api/csapi/model.ts#L140-L160), [serializer](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/6fdf39669e8b11fe1e9c0c8914278c5f11a3b57f/src/ogc-api/csapi/url_builder.ts#L280-L329), and [response parser](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/6fdf39669e8b11fe1e9c0c8914278c5f11a3b57f/src/ogc-api/csapi/formats/response.ts#L28-L129) | `6fdf39669e8b11fe1e9c0c8914278c5f11a3b57f` | Models `limit`, `offset`, `cursor`, separate `uid`; tolerates absent counts and preserves links | Modeling/serialization cannot prove server support or create obligations |
| [CSAPI Explorer count/link logic](https://github.com/OS4CSAPI/ogc-csapi-explorer/blob/00f1c188e05738ee03390fd95f09d351e073a9c3/demo/src/components/ResourceList.vue#L164-L238), [link extraction](https://github.com/OS4CSAPI/ogc-csapi-explorer/blob/00f1c188e05738ee03390fd95f09d351e073a9c3/demo/src/components/ResourceList.vue#L367-L387), and [supplied-link traversal](https://github.com/OS4CSAPI/ogc-csapi-explorer/blob/00f1c188e05738ee03390fd95f09d351e073a9c3/demo/src/components/ResourceList.vue#L565-L580) | `00f1c188e05738ee03390fd95f09d351e073a9c3` | Follows supplied server links; uses a `limit=1000` item-count fallback when totals are absent | Link may contain offset; fallback is capped/racy and is not exact-count evidence |
| Bounded live implementation checks | Observed 2026-08-01 | One OpenSensorHub deployment used offset links without counts; one CS GO deployment advertised advanced filtering and returned counts | Non-reproducible illustrations only: exact endpoint URLs and response captures were not retained; excluded from requirements and decision support |

### 3.4 Authority and Classification Rules

The report classifies each material matrix entry on three separate axes so a conditional obligation is not mistaken for an optional suggestion, and a project mapping does not erase the underlying standard:

- **Authority (`AUTH`):** `N` direct normative CSAPI text; `I` inherited normative OGC behavior; `A` tagged artifact evidence; `H` maintenance/history evidence; `P` Glaux-only proposal.
- **Kind (`KIND`):** `R` required; `C` conditional once the named class, route, or capability is selected; `O` standards-defined optional/recommended; `X` implementation-specific; `F` future candidate; `Ø` unsupported/out of the initial profile.
- **Disposition (`DISP`):** `B` implement in the baseline; `M` implement through a documented mapping/repair; `D` defer; `J` reject; `S` hand off to its owning topic; `T` track upstream.
- A trailing `?` marks an unresolved mapping without erasing its authority or kind. Authority may be compound only when one behavior genuinely has multiple controlling layers, such as `N+I`; every row otherwise has exactly one `KIND` and one `DISP`. When a standards rule and a Glaux choice differ, the matrix uses separate rows. Thus `N|C|B` is a normative obligation when its class applies, while `P|X|M` is a defined Glaux adapter—not an OGC requirement.

Approved normative requirements control over OAS, schemas not incorporated by a requirement, examples, ATS mistakes, implementations, clients, and issue discussions. A later official publication may change the baseline only through the accepted IDR-SRV-010A delta/adoption process.

### 3.5 Evidence Limitations

- No complete CSAPI executable conformance suite exists; the normative ATS has significant positive-test bias and copied-target defects.
- This topic did not run an exhaustive implementation benchmark or the later IDR-SRV-014A–014G studies.
- Standards do not define stable default order, cursor contents, snapshot consistency, authorization-filtered metadata, or exact cost budgets.
- Issue #165, #175, #179, and #182 recommendations remain pending SWG decisions.
- The tagged OAS is an illustrative modular document, not a complete release contract, and its schemas contain unresolved references and response-model omissions.
- Persistence feasibility for snapshot-bound paging must be confirmed by IDR-SRV-025–029 before implementation commitment.

---

## 4. Query Behavior Extraction Methodology

### 4.1 Phase Execution

1. **Framework and prerequisites:** verified the accepted reports, selected conformance profile, goal, all named governance/templates, and exact deliverable; reviewed the required OS4CSAPI research-plan corpus and its blueprint, inventory/sourcing, and synthesis exemplars for expected depth and traceability without treating their technical conclusions as Glaux evidence.
2. **Standards extraction:** traced every applicable Part 1/Part 2 requirement through inherited Features clauses and recorded parameter grammar, selection effect, response rule, and ATS anchor.
3. **Resource-family mapping:** mapped common and family-specific filters to canonical, nested, deployment-scoped, feasibility-related, and typed collection routes.
4. **Sorting/paging/selection analysis:** compared the absence of CSAPI facilities with Features paging, Records sorting, current Sortables, Features Part 3/CQL2, EDR, Processes, OAS, and client precedent.
5. **Security/performance/error analysis:** applied authorization-before-query invariants, information-leak controls, query budgets, index needs, and negative-test implications without designing a database or final error schema.
6. **Synthesis and validation:** reconciled three independent read-only audits, refreshed relevant history, produced matrices and proposed decisions, and checked every question, phase, criterion, and deliverable item.

### 4.2 Mechanical Artifact Review

The tagged repository's complete `api` tree contains 424 YAML/JSON files; 315 are under Part 1 and Part 2. No active query parameter named `sortby`, `sortBy`, `sortOrder`, `select`, `fields`, `offset`, `cursor`, `filter`, or `filter-lang` was found anywhere in that 424-file corpus. Commented `select` references are not active contract evidence. Every parameter reference in every Part 1 and Part 2 path file was inventoried; the principal discrepancies appear in §13.2.

### 4.3 Decision Test

For each apparent behavior, the research asked:

1. Is there an applicable approved `SHALL`, incorporated requirement, or precise recommendation?
2. Does the requirement apply to this class, resource family, and route variant?
3. Does the OAS/ATS faithfully express it, omit it, or add something else?
4. Is current history an adopted release change or only design rationale/proposal?
5. What minimum deterministic Glaux policy is necessary without crossing into a later topic?
6. What must be proven by tests or handed to another owner?

---

## 5. Standards-Based Query Parameter and Behavior Inventory

### 5.1 Inherited Collection Behavior

| Behavior | Classification | Exact baseline |
|---|---|---|
| `limit` | I | Optional integer; initial Glaux min/default/max `1/10/10000`; never return more than effective limit; count only first-level resources; may return fewer; over-maximum clamps |
| `bbox` | I on Part 1 resource endpoints importing Features §7.15.3 | Four or six numbers; CRS84/CRS84h unless an applicable CRS extension is selected; intersect geometry; geometry-less features also match; coordinates must be within CRS extent |
| `datetime` | I plus N Part 1 mapping | RFC 3339 instant or slash interval; intersection; Part 1 evaluates `validTime` and includes absent/null validity; Part 2 adds it only to named classes |
| Filter combination | I/N | `bbox`, `datetime`, declared simple properties, and CSAPI advanced filters combine with AND |
| Response | I/N | HTTP 200 and only selected resources; `self` and alternate links where imported; precise media handling deferred to IDR-SRV-012 |
| `next` | I recommendation elevated to P Glaux rule | Server-authored link whenever Glaux knows more visible results exist; additional not-yet-returned members; opaque implementation |
| `prev` | I permission | Optional; not required for first stable Glaux contract |
| `timeStamp` | I conditional elevated to P | If present, response-generation time; Glaux includes it in JSON collection envelopes where the selected schema permits it |
| `numberMatched` | I conditional | If present, exact count of the complete filtered caller-visible selection; may be omitted if unknown/difficult |
| `numberReturned` | I conditional elevated to P | If present, exact current-page count; Glaux should always include it in JSON collection envelopes |
| Unknown/invalid query | I | Undeclared parameter or invalid value produces 400; declared extension parameters must appear in the API definition; final body/code belongs to IDR-SRV-013 |

The apparent Features contradiction between `limit` rules is resolved by the numbered requirement: a valid integer above the maximum is clamped, while non-integers, values below the minimum, and other invalid syntax produce `400`.

### 5.2 Part 1 Common and Family-Specific Filters

When Part 1 Advanced Filtering is claimed, `id` and `q` apply to every CSAPI resource collection GET offered by the server; `geom` applies to geometry-bearing feature endpoints; and family-specific filters apply to every resources endpoint for that family.

| Scope | Parameters | Required selection meaning |
|---|---|---|
| All CS resource families | `id`, `q` | ID/UID alternatives and UID trailing-`*` prefix; at least `name`/`description` keyword content |
| Geometry-bearing feature families | `geom` | WKT intersection; geometry-less resources excluded |
| Systems | `parent`, `procedure`, `foi`, `observedProperty`, `controlledProperty` | Direct parent; implemented Procedure; sampling/domain FOI; observable/controllable capability; FOI/property matching includes recursive subsystems |
| Deployments | `parent`, `system`, `foi`, `observedProperty`, `controlledProperty` | Direct parent; deployed System; FOI/property associated through deployed Systems |
| Procedures | `observedProperty`, `controlledProperty` | Procedure can observe/control requested Property |
| Sampling Features | `foi`, `observedProperty`, `controlledProperty` | Associated FOI; properties derived through associated DataStreams/ControlStreams per ATS interpretation |
| Properties | `baseProperty`, `objectType` | Direct/transitive derivation from base Property; matching object type |

The `ID_List` is a nonempty homogeneous list of local IDs or URI-form UIDs. The tagged OAS serializes it as one comma-separated `form`, `explode=false` occurrence. Alternatives are OR. `q` is also a comma-separated array of 1–50-character terms and matches at least one term. Different parameter names are AND.

### 5.3 Part 2 Dynamic-Data Filters

Part 2 Advanced Filtering depends on Part 1 Advanced Filtering, so common `id` and `q` remain applicable.

| Family | Base parameters | Advanced parameters | Selection meaning |
|---|---|---|---|
| DataStream | `limit`, `datetime` | `id`, `q`, `phenomenonTime`, `resultTime`, `observedProperty`, `foi` | `datetime`→`validTime`; stream temporal extents/properties/FOI |
| Observation | `limit` | `id`, `q`, `phenomenonTime`, `resultTime`, `foi` | Exact property time intersection; `resultTime=latest` additionally required |
| ControlStream | `limit`, `datetime` | `id`, `q`, `issueTime`, `executionTime`, `controlledProperty`, `foi` | `datetime` intended for `validTime`; stream temporal extents/properties/FOI |
| Command | `limit` | `id`, `q`, `issueTime`, `executionTime`, `statusCode`, `sender`, `foi` | Command property times; current status; issuing sender; FOI |
| CommandStatus | `limit`, `datetime` | `id`, `q`, `statusCode` | Status-code match; `datetime` is interpreted against `reportTime` |
| CommandResult | `limit` | `id`, `q` | Common identity/text filters only |
| SystemEvent | `limit`, `datetime` | `id`, `q`, `eventType` | `datetime` maps to tagged JSON `time`/prose `eventTime`; type maps to tagged `definition`/prose `type` |
| Feasibility | `limit`; conditional Command `id`, `q`, `issueTime`, `executionTime`, `statusCode`, `sender`, `foi` | Requirement 36 makes the nested Feasibility route a Command resources endpoint; Requirement 39C applies the Command query contract to typed Feasibility collections | The contract is normative on nested and advertised typed endpoints; only applying it to the accepted project root `/feasibility` requires a Glaux adapter because the overview supplies no parallel numbered root-list requirement |

`statusCode` accepts exactly `PENDING`, `ACCEPTED`, `REJECTED`, `SCHEDULED`, `UPDATED`, `CANCELED`, `EXECUTING`, `FAILED`, and `COMPLETED`. `sender` is a list of strings. `eventType` is a list of exact type identifiers; although the resource model describes URIs, its parameter schema does not enforce URI format.

### 5.4 Optional OGC Building Blocks

| Capability | Current relationship to CSAPI | Glaux disposition |
|---|---|---|
| Simple property equality | Features/CSAPI recommendation | Support only an explicit schema-derived allowlist after IDR-SRV-015; never accept arbitrary JSON property names silently |
| Features Part 3 Queryables/Filter | Optional separate OGC classes | Preserve architectural room; do not claim in the first CSAPI profile |
| CQL2 | Optional language classes used with a binding | Defer until a demonstrated use case justifies parser/planner/security cost; advertise separately if adopted |
| Records `sortby` | Records-specific optional sorting pattern | Preferred lexical model for a future Glaux/CSAPI extension, not current baseline |
| Common Part 3 / Features Part 5 Sortables | Advertises sortable properties, not yet a CSAPI sorting binding | Re-evaluate with issue #175 and Features Part 8; no first-release claim |
| EDR `parameter-name` | Domain-specific observed-parameter selection | Not generic field projection; possible later Observation-result profile only |
| Processes async execution | Job/process model | Keep outside ordinary CSAPI collection GET; possible future analytics service |

---

## 6. Resource-Family Query Semantics Matrix

The matrix includes every field required by the research plan. Compact codes are defined in §3.4. `400` means the final Problem Details body/code is deferred to IDR-SRV-013. `QReg` means the proposed typed query registry; `OAD` means OpenAPI Description; `RID` means canonical local resource ID.

| Resource family | Parameter / behavior | Source anchor | Requirement summary | Class | Type / allowed | Default | Response effect | Errors | Security | Persistence / index | Test | Handoff | Notes / unresolved |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Every applicable collection | `limit` semantics | [Features Reqs 21–22](https://docs.ogc.org/is/17-069r4/17-069r4.html#req_core_fc-limit-definition); CSAPI endpoint imports | Cap first-level returned members; declare server min/default/max; clamp a valid above-max integer | I\|R\|B | Integer within declared server range | Declared server default | At most effective value; may be fewer | 400 below min/noninteger; above max clamps | Bound response/cost after auth | Limit pushdown; limit+1 lookahead | Grammar, declared bounds, clamp, page size | 013/014/025/054 | Normative behavior; exact Glaux values are separate below |
| Every applicable collection | Glaux `limit` profile | P-011-02; pinned [tagged parameter](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/parameters/limit.yaml) | Freeze minimum/default/maximum at `1/10/10000` | A+P\|X\|B | Integer 1–10000 | 10 | At most 10,000 first-level members | 400 below 1/noninteger; above 10,000 clamps | Project cost/response bound | Same implementation as normative limit | Exact 1/10/10000 and compatibility change | 013/014/025/054 | Project profile; values must match OAD and change through IDR-SRV-010A |
| Every Part 1 resources endpoint | `bbox` semantics | [Features Reqs 23–24](https://docs.ogc.org/is/17-069r4/17-069r4.html#req_core_fc-bbox-definition); Part 1 endpoint imports | Spatial intersection against a server-designated geometry set; no-geometry resources also match | I\|R\|B | 4/6 numbers; CRS84/CRS84h baseline | No spatial restriction | Select intersections and retain resources without geometry | 400 arity/type/range | Geometry visibility before predicate | Spatial index; antimeridian/3D support | Boundary, antimeridian, no geometry, multi-geometry, Property | 013/015/026 | Includes Property/Procedure through Part 1's resource adaptation; `bbox-crs` needs a separate class |
| Every Part 1 resources endpoint | Glaux `bbox` geometry set | P-011-14; Features multi-geometry implementation choice | Evaluate every representation-independent geometry designated query-relevant in QReg | P\|X\|M | IDR-SRV-015/026-approved family geometry set | Registered set | Match if any registered geometry intersects; retain no-geometry resources | 400 only for input/complexity, not geometry absence | Register only releasable geometry facts | Family spatial indexes | Every registered geometry and multi-geometry disagreement | 015/026/039 | Project mapping; exact sets remain downstream-owned |
| Part 1 feature/Property/Procedure endpoints | `datetime` | [Features Reqs 25–26](https://docs.ogc.org/is/17-069r4/17-069r4.html#req_core_fc-time-definition); [P1 Req 3](https://docs.ogc.org/is/23-001/23-001.html#_req_api-common_datetime) | Intersect `validTime`; absent/null validity matches | I+N\|R\|M | RFC 3339 instant or slash interval with open bound | No temporal restriction | Selects descriptions valid during query | 400 grammar/range | Apply after auth; avoid validity inference | Temporal range index; null bucket | Instant/interval/open/boundary/null | 013/018/025 | Tagged date-only/`now`/`latest` are not baseline |
| Every CS resource family | `id` | [P1 Reqs 38–39](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_resource-by-id) | Match local IDs or UIDs; terminal `*` prefixes UIDs | N\|C\|B | One nonempty homogeneous comma list | All visible resources | OR across requested identifiers | 400 empty/mixed/malformed/wildcard misuse | Unknown/unseen identifier returns empty | Unique local/UID and UID-prefix index | Local/UID/list/prefix/unseen/mixed | 013/016/025 | No separate normative `uid` parameter |
| Every CS resource family | `q` semantics | [P1 Req 40](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_resource-by-keyword) | Match at least one term in human-readable fields, always including name/description | N\|C\|B | Comma list; each 1–50 chars | No text restriction | OR terms; AND other filters | 400 empty/too long/duplicate | Search only visible text; budget timing | Search index over registered fields | Term length, OR/AND, absent text | 013/015/025 | Formal requirement controls over overview “prefix” wording |
| Every CS resource family | Glaux `q` comparison algorithm | P-011-04; Unicode 17.0.0 Chapter 3/UAX #15 | `toNFKC_Casefold` field and term, then locale-independent code-point substring containment | P\|X\|M | Versioned Unicode algorithm | No linguistic expansion | Predictable case-insensitive literal containment | 400 follows normative term grammar | Avoid locale/timing divergence | Versioned normalized-text index/rebuild | Unicode edge cases, version, no stemming/fuzzy behavior | 013/015/025/028 | Project mapping; Unicode-data change gets compatibility review |
| Geometry-bearing Part 1 features | `geom` semantics | [P1 Req 41](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_feature-by-geom) | WKT intersection against server-designated query geometry; exclude no-geometry features | N\|C\|B | Valid WKT in baseline CRS | No geometry restriction | Select intersections; exclude no-geometry features | 400 WKT/CRS/complexity | Geometry authorization and cost caps | Spatial index plus bounded WKT parser | WKT types/bounds/complexity/no geometry | 013/023/026/054 | “Operator” artifact wording creates no grammar |
| Geometry-bearing Part 1 features | Glaux `geom` geometry set | P-011-14; same set as Glaux `bbox` mapping | Evaluate every representation-independent geometry designated query-relevant in QReg | P\|X\|M | IDR-SRV-015/026-approved family geometry set | Registered set | Match if any registered geometry intersects | 400 only for input/complexity | Same releasable set as spatial registry | Same family spatial indexes | All geometries and bbox/geom no-geometry distinction | 015/023/026/039 | Project mapping; exact sets remain downstream-owned |
| Systems | `recursive` | [P1 Reqs 10–12](https://docs.ogc.org/is/23-001/23-001.html#_req_subsystem_recursive-param) | Root: top-level vs all descendants; nested: direct vs all descendants; filter every processed level | N\|C\|B | Boolean | `false` | Changes candidate hierarchy, not item identity | 400 invalid Boolean; budget overflow policy later | Authorization at every node; no hidden-child leak | Cycle-safe closure/traversal | Multi-depth/cycle/dedup/filters/auth | 013/015/017/025 | Only canonical/hierarchy routes; explicit-root-`parent` policy is separate below |
| Deployments | `recursive` | [P1 Reqs 20–22](https://docs.ogc.org/is/23-001/23-001.html#_req_subdeployment_recursive-param) | Same hierarchy rules for Deployments | N\|C\|B | Boolean | `false` | Root/nested candidate set changes | 400 invalid Boolean | Per-node authorization | Deployment closure/traversal | Multi-depth/cycle/dedup/filters/auth | 013/015/017/025 | Only canonical/hierarchy routes; tagged root OAS omission tracked by #169/#196 |
| Systems | `parent`, `procedure` semantics | [P1 Reqs 42–43](https://docs.ogc.org/is/23-001/23-001.html#clause-systems-query-params) | Direct parent or implemented Procedure match | N\|C\|B | `ID_List` each | No relation restriction | Select matching Systems | 400 list grammar | Do not reveal hidden parent/Procedure | Parent and System–Procedure indexes | Local/UID/multi/nested/conflict | 016/017/025 | Normative selection; root candidate precedence is separate below |
| Systems | Root `parent` precedence | P-011-05; tagged hierarchy description | Explicit `parent` directly queries children despite the root top-level default | P\|X\|M | Valid `ID_List` already parsed | No explicit parent | Overrides only the root candidate seed; `recursive` does not alter direct-parent meaning | Standard list errors | No hidden-parent disclosure | Parent index independent of root seed | Root/nested/recursive combinations and unseen parent | 013/017/025/039 | Project interpretation needed to make the approved filter usable |
| Systems | `foi`, `observedProperty`, `controlledProperty` obligation | [P1 Reqs 44–46](https://docs.ogc.org/is/23-001/23-001.html#clause-systems-query-params) | Direct/indirect FOI or capability; parent matches recursive subsystem capability | N\|C\|M? | `ID_List` each | No relation restriction | Select Systems satisfying any listed value for each supplied filter | 400 list grammar | Prevent relationship/capability inference | FOI chain and capability closure | Parent-only descendant match; derived/hidden links | 017/024/025/039 | Obligation is normative; representation-independent source remains unsettled |
| Deployments | `parent`, `system`, `foi`, `observedProperty`, `controlledProperty` obligation | [P1 Reqs 47–51](https://docs.ogc.org/is/23-001/23-001.html#clause-deployments-query-params) | Parent, deployed System, or deployed-System FOI/capability | N\|C\|M? | `ID_List` each | No relation restriction | Select matching Deployments | 400 list grammar | Hidden deployment/System relations cannot leak | Deployment/Systems/FOI/property joins | Direct/transitive/temporal/auth cases | 017/018/024/025/039 | Obligation normative; representation and validity mapping need overlay |
| Procedures | `observedProperty`, `controlledProperty` obligation | [P1 Reqs 52–53](https://docs.ogc.org/is/23-001/23-001.html#clause-procedures-query-params) | Procedure capable of observing/controlling requested Property | N\|C\|M? | `ID_List` each | No relation restriction | Select capable Procedures | 400 list grammar | Capability facts caller-visible only | Normalized Procedure capability index | Description mappings and negative cases | 015/017/024/025 | Obligation normative; source representation remains under #179 |
| Sampling Features | `foi`, `observedProperty`, `controlledProperty` obligation | [P1 Reqs 54–56](https://docs.ogc.org/is/23-001/23-001.html#clause-sf-query-params) | FOI association; properties via associated streams per ATS | N\|C\|M? | `ID_List` each | No relation restriction | Select matching Samples | 400 list grammar | No dereference or hidden-chain leak | Cycle-safe sample chain; stream property joins | Local/UID/href/chain/cycle/external | 017/024/025/039 | Obligation normative; exact chain/source mapping remains under #165/#179 |
| Properties | `baseProperty`, `objectType` | [P1 Reqs 57–58](https://docs.ogc.org/is/23-001/23-001.html#clause-prop-query-params) | Direct/transitive base derivation; exact object type | N\|C\|M? | `ID_List`; compare normalized identifiers | No relation restriction | Select matching Properties | 400 list/identifier grammar | Avoid hidden taxonomy inference | Property transitive closure; type index | Derivation depth/cycle/CURIE/URI/local | 016/024/025 | Use `objectType`, not erroneous `object` example; local-ID meaning unresolved |
| Systems, Deployments, Procedures, Sampling Features, Observations, ControlStreams | FOI/capability mapping overlay | P-011-15; issues [#165](https://github.com/opengeospatial/ogcapi-connected-systems/issues/165) and [#179](https://github.com/opengeospatial/ogcapi-connected-systems/issues/179) | Use normalized local ID/UID/stored href, no live dereference, cycle-safe sampling chains, explicit capability facts, and Deployment-validity checks | P\|X\|M | Bounded normalized relationship facts | No relation restriction | Supplies a representation-independent execution mapping without widening normative membership | Standard filter grammar; unresolved external values yield no match | No hidden-chain traversal or external oracle | Normalized facts and bounded closures | Each mapping, cycle, external href, hidden relation, validity | 015/017/018/024/025/039 | Defined project overlay pending owner acceptance; track upstream changes via IDR-SRV-010A |
| DataStreams | `datetime` | [P2 Req 7](https://docs.ogc.org/is/23-002/23-002.html#_req_datastream_resources-endpoint) | Intersect DataStream `validTime` | N+I\|R\|M | Approved time string | No restriction | Select valid streams | 400 invalid time | Visible streams only | Valid-time range index | Null/open/boundaries/nested | 013/018/027 | Tagged OAS omits it |
| DataStreams | `phenomenonTime`, `resultTime`, `observedProperty`, `foi` | [P2 Reqs 45–48](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_datastream-by-phenomenontime) | Match stream time extents, properties, or FOI | N\|C\|B | Time string or `ID_List` | No restriction | Select streams | 400 grammar | Avoid revealing hidden activity | Temporal extents and relation indexes | Each filter, combination, every endpoint | 017/018/024/027 | Tagged `system` is artifact-only |
| Observations | `phenomenonTime`, `resultTime` | [P2 Reqs 49–50](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_obs-by-phenomenontime) | Intersect the named Observation property | N\|C\|B | Approved time string; `latest` only for resultTime | No restriction | Select time-matching Observations | 400 invalid/unsupported special | Activity timing can be sensitive | Time-series composite indexes | Bounds/ties/latest/top/nested | 013/018/027 | `now` example is not approved grammar |
| Observations | `resultTime=latest` obligation | [P2 Req 50D](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_obs-by-resulttime) | Retain observations at the latest result time | N\|C\|M? | Exact special token `latest` | Not applied unless supplied | Select latest result-time membership with ties | 400 outside `resultTime` | Maximum computed from visible set | Max/result-time index | Canonical/nested, ties, no matches | 018/027/039 | Standard does not state canonical endpoint grouping explicitly |
| Observations | Glaux `latest` scope | P-011-03; #64/closed-unmerged PR #88 history | Compute the literal maximum after auth, route scope and every other filter; retain all ties | P\|X\|M | Approved `latest` token already parsed | Not applied unless supplied | Endpoint-wide visible maximum; nested route naturally stream-scoped | 400 for unsupported combinations | No hidden maximum inference | Same maximum/result-time plan | Global/nested/ties/other filters/auth | 018/027/039 | Project interpretation; per-stream grouping is not baseline |
| Observations | `foi` obligation | [P2 Req 51](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_obs-by-foi) | Match sampling or ultimate/domain FOI | N\|C\|M? | `ID_List` | No restriction | Select related Observations | 400 list grammar | No hidden chain leak | Observation–sample–FOI joins | Direct/chain/external/unseen | 017/024/025/027/039 | Execution mapping supplied separately by P-011-15; tagged extras are artifact-only |
| ControlStreams | Base `datetime` | [P2 Req 20](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream_resources-endpoint) | Intersect ControlStream `validTime` | N+I\|R\|M | Approved time string | No restriction | Select valid ControlStreams | 400 grammar | Tasking capability is sensitive | Valid-time range index | Canonical/nested/deployment/null/open | 013/018/025/036/039 | Interpret copied DataStream/CommandStream target wording as ControlStream |
| ControlStreams | `issueTime`, `executionTime`, `controlledProperty`, `foi` obligation | [P2 Reqs 52–55](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_controlstream-by-issuetime) | Match Command time extents, controlled property, or FOI | N\|C\|M? | Approved time string or `ID_List` | No restriction | Select matching streams | 400 grammar | Tasking capability/activity is sensitive | Temporal and relation indexes | Each filter, copy-target repair, every route | 017/018/024/025/036/039 | Time obligations are direct; relationship mapping is supplied by P-011-15 |
| Commands | `issueTime`, `executionTime` | [P2 Reqs 56–57](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_cmd-by-issuetime) | Intersect named Command time | N\|C\|B | Approved time string | No restriction | Select Commands | 400 grammar | Task timing/releasability | Command time indexes | Instant/range/open/nested/auth | 018/025/036/039 | No created/modified query in CSAPI |
| Commands | `statusCode`, `sender`, `foi` | [P2 Reqs 58–60](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_cmd-by-status) | Current status, issuing sender, or FOI | N\|C\|B | Enum list; string list; `ID_List` | No restriction | Select Commands | 400 invalid enum/list | Sender/status/target may be restricted | Current-status, sender and FOI indexes | All enums/case/list/current/history/auth | 017/020/025/036/039 | Tagged controlledProperty/system/controlStream are artifact-only |
| CommandStatus | Base `limit`, `datetime` obligation | [P2 Req 31](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream_status-resources-endpoint) | Page and time-filter status reports | N+I\|R\|M? | Inherited limit; approved time string | No restriction; inherited limit default | Select status reports by the intended time property | 400 grammar | Conceal Command existence/status | `(command, reportTime, RID)` | Limit/time/null/bounds/auth | 013/018/020/025/036 | Normative wire name is `datetime`; property mapping is P-011-15 below |
| CommandStatus | `id`, `q`, `statusCode` obligation | [P2 §13.1](https://docs.ogc.org/is/23-002/23-002.html#clause-advanced-filtering); [Req 61](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_status-by-statuscode) | Common advanced filters plus exact status-code match | N\|C\|B | Standard common types; nine-token enum list | No restriction | Select status reports | 400 grammar/enum | Conceal Command existence/status | Identity/text/status indexes | Common filters, all enums, auth | 013/020/025/036/039 | No current-status interpretation: each report matches its own status |
| CommandStatus | `datetime` property mapping | P-011-15; tagged model/OAS evidence | Bind canonical `datetime` to `reportTime` | P\|X\|M | Approved time grammar | No restriction | Executes normative time filter against report time | Standard time errors | No hidden report timing | Report-time index | Canonical name/property mapping/auth | 013/014/018/020/025 | Explicit project repair of prose/model gap |
| CommandStatus | `reportTime` compatibility alias | P-011-16; pinned [artifact parameter](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/parameters/reportTime.yaml) | Candidate read-only alias; not baseline or canonical | A+P\|F\|D | If later accepted, same grammar and mutually exclusive with `datetime` | Absent | No current baseline effect | Baseline 400; future lane rejects both names together | Must not widen or leak | Same report-time index if adopted | Strict-lane rejection and future adapter suite | 013/014/014A-G/056 | Excluded from CSAPI conformance claims unless separately accepted as an adapter |
| CommandResult | `limit` | [P2 result endpoint](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream_result-resources-endpoint) | Page visible results | I\|R\|B | Inherited integer limit | Inherited default | Cap first-level results | Standard limit errors | Results may be highly sensitive | Limit pushdown | Paging/count/auth | 013/025/036/039 | No result-specific query parameter |
| CommandResult | `id`, `q` | [P2 §13.1 Advanced Filtering dependency](https://docs.ogc.org/is/23-002/23-002.html#clause-advanced-filtering) | Carry Part 1 common advanced filters to every Part 2 resource type | N\|C\|B | Standard common types | No restriction | Select visible results | 400 grammar | Results may be highly sensitive | Identity/text index if useful | Empty/multiple/auth | 013/025/036/039 | Direct Part 2 class dependency, not Features inheritance |
| SystemEvent | Base `limit`, `datetime` obligation | [P2 Req 41](https://docs.ogc.org/is/23-002/23-002.html#_req_system-event_resources-endpoint) | Page and time-filter System Events | N+I\|R\|M? | Inherited limit; approved time string | No restriction; inherited limit default | Select events by the intended time property | 400 grammar | Events can reveal operations | Event-time index | Time/null/bounds/nested/auth | 013/018/020/025/039 | Property name differs between prose and tagged JSON; mapping is separate |
| SystemEvent | `id`, `q`, `eventType` obligation | [P2 §13.1](https://docs.ogc.org/is/23-002/23-002.html#clause-advanced-filtering); [Req 62](https://docs.ogc.org/is/23-002/23-002.html#_req_advanced-filtering_event-by-type) | Common filters plus event-type match | N\|C\|M? | Common types; exact string list | No restriction | Select events | 400 grammar | Events can reveal operations | Event type/text indexes | Prose/schema mapping, casing, nested/auth | 013/018/020/025/039 | `eventType` property binding needs P-011-15; tagged `system` is artifact-only |
| SystemEvent | Time/type property mapping | P-011-15; tagged JSON schema/OAS and ATS conflict | Bind `datetime`→JSON `time` and `eventType`→JSON `definition` | P\|X\|M | Normative wire grammars | No restriction | Supplies encoding-specific execution mapping | Standard input errors | No hidden event inference | Normalized event time/type facts | JSON/prose mapping and alternate encodings | 012/014/015/020/025 | Track upstream; do not rename canonical wire parameters |
| Feasibility Commands | Nested/typed Command query contract | [P2 Feasibility](https://docs.ogc.org/is/23-002/23-002.html#clause-command-feasibility) | Nested and advertised typed Feasibility resources endpoints use the Command query contract | N\|C\|B | `limit`; conditional `id`, `q`, issue/execution time, status, sender, FOI | No restriction; inherited limit default | Select scoped feasibility requests | Standard query/path errors | Highest tasking sensitivity | Command-like indexes | Nested/typed/filter/status/result/auth | 013/014/036/037/039 | Requirement 36 and 39C make this normative |
| Feasibility Commands | Root `/feasibility` query binding | Accepted IDR-SRV-010 route plus P-011-17 | Apply the same Command query contract to the accepted project root list | P\|X\|M | Same as nested/typed contract | No restriction; inherited limit default | Select root-scoped visible feasibility requests | Standard query/path errors | Highest tasking sensitivity | Same Command-like indexes | Root/nested parity and auth | 013/014/036/037/039 | Defined adapter, not unresolved and not an OGC claim |
| Every filtered collection | Combined predicates | [P1 Req 59](https://docs.ogc.org/is/23-001/23-001.html#_req_advanced-filtering_combined-filters) | Different filters are AND; values inside one list are OR | N\|C\|B | Registered parameters only | No predicates | Intersection of caller-visible predicates | 400 duplicates/incompatible syntax | Authorization is prior logical relation | Composable typed predicates/planner | Pairwise/all-filter/property tests | 013/023/025 | Empty selection is successful 200 |
| Every paged collection | Inherited paging link | [Features paging](https://docs.ogc.org/is/17-069r4/17-069r4.html#fc-response); [P1 §7.7](https://docs.ogc.org/is/23-001/23-001.html) | A `next` link identifies additional not-yet-returned selected members | I\|O\|B | Server-authored URL | First page | Continue the same selected resource set | Expired page may become unavailable | Link cannot reveal hidden membership | Any coherent paging mechanism | Link presence/absence and not-yet-returned membership | 013/014/025 | Inherited recommendation/permission; token syntax is not standardized |
| Every paged collection | Glaux continuation contract | P-011-07; consistency/security analysis | Require known-more `next`, opaque keyset state, pseudonymous auth binding, confidentiality/integrity, and snapshot binding when repeatability is promised | P\|X\|M | Random server reference or authenticated encryption; restricted MAC-only lane | First page | Continue same visible filters/order/representation/snapshot | 400 forged; expiry policy to 013 | No raw sensitive identifiers; cross-context replay fails | Keyset + optional snapshot/watermark | Union/no gaps/duplicates/loops/tamper/expiry/key rotation | 013/025/027/029/039 | No public `offset`/`cursor`; Glaux elevates known-`next` to required |
| Every JSON collection envelope | Inherited count/time fields | [Features Reqs 30–32](https://docs.ogc.org/is/17-069r4/17-069r4.html#req_core_fc-numberMatched) | When present, generation time and counts are exact for the described selection/page | I\|C\|B | Nonnegative exact integers; timestamp | Fields optional/conditional | Metadata describes the same visible selection/page | Never label an approximation with an exact standard field | Counts/extents cannot include hidden rows | Exact plan or honest omission | Two principals, absent total, page count | 012/014/025/039 | Obligation binds only when the field is supplied |
| Every JSON collection envelope | Glaux count/time profile | P-011-08; approved schema overlay policy | Always include exact `numberReturned` where envelope permits; include `timeStamp` where permitted; exact visible `numberMatched` only when safe/affordable | P\|X\|M | Standard field types | Exact page count; other fields conditional | Stronger predictable metadata without forcing expensive total scans | Omit unsafe/unaffordable total; never approximate | Operates on caller-visible relation only | Count plan or omit; no forced full scan | Page count, absent total, timestamp, two principals | 012/014/025/039 | Project elevation separated from inherited conditional fields |
| All list families | Default total order | No CSAPI sort requirement; [issue #175](https://github.com/opengeospatial/ogcapi-connected-systems/issues/175) | Internal deterministic order needed for paging | P\|X\|B | Family key plus RID ascending | Provisional pre-release value in QReg | Stable sequence; no client control | N/A; later invalid sort=400 | Sort visible relation only | Composite order indexes | Ties/mutation/replay/all endpoints and external clients | 014/014A-G/025/027/029 | Validate before stable release; after release the observable default is a stable contract |
| All representations | Full-resource selection | Pinned [Part 1 OAS](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/openapi-connectedsystems-1.yaml) / [Part 2 OAS](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/openapi-connectedsystems-2.yaml) and approved schemas | Return complete selected resources | P\|X\|B | No `select`/`fields`/`expand` baseline | Full representation | Schema-conformant item bodies | 400 undeclared projection params | Property authorization remains mandatory | Avoid projection-specific plans | Reject parameters; validate complete schemas | 012/014/023/039 | “Selected” means selected resources, not fields |
| Registered scalar fields | Standards simple-equality recommendation | [Features Rec 16](https://docs.ogc.org/is/17-069r4/17-069r4.html#rec_core_fc-filters); P1 recommendation | Offer equality parameters for useful simple-valued properties | I\|O\|B | Same logical type as the response property | No restriction | Exact property equality | 400 invalid value when parameter offered | Do not make protected fields queryable | Per-field index only after evidence | Offered fields, exact equality, encoding parity | 014/015/024/025/039 | Recommendation does not authorize arbitrary JSON property names |
| Registered scalar fields | Glaux simple-equality surface | P-011-13; IDR-SRV-015-owned allowlist | Expose only QReg/OAD-declared representation-independent fields | P\|X\|M | Approved logical field/type registry | No restriction | Exact allowlisted equality | 400 undeclared/invalid; no wildcard/nested paths | Security review precedes registration | Allowlisted field indexes | Runtime/OAD/schema parity and unknown names | 014/015/024/025/039 | Project mapping; no arbitrary-property baseline |

### 6.1 Route-Applicability Ledger

This ledger joins the query rules above to the accepted [Part 1 route inventory](idr-srv-010-collections-resources-links-and-navigation-behavior-report.md#62-part-1-canonical-and-nested-inventory), [Part 2 route inventory](idr-srv-010-collections-resources-links-and-navigation-behavior-report.md#63-part-2-canonical-and-nested-inventory), and [primary Glaux path interpretations](idr-srv-010-collections-resources-links-and-navigation-behavior-report.md#65-primary-glaux-path-interpretation). A family contract applies to every canonical, nested, and typed **resources endpoint** identified below; path scope is an implicit AND predicate. An item or schema endpoint is not a second filterable collection.

| Key | Family query contract, in addition to route scope |
|---|---|
| `Q-SYS` | `limit`, `bbox`, `datetime`; conditional `id`, `q`, `geom`, `parent`, `procedure`, `foi`, `observedProperty`, `controlledProperty` |
| `Q-DEP` | `limit`, `bbox`, `datetime`; conditional `id`, `q`, `geom`, `parent`, `system`, `foi`, `observedProperty`, `controlledProperty` |
| `Q-PROC` | `limit`, `bbox`, `datetime`; conditional `id`, `q`, `observedProperty`, `controlledProperty` |
| `Q-SF` | `limit`, `bbox`, `datetime`; conditional `id`, `q`, `geom`, `foi`, `observedProperty`, `controlledProperty` |
| `Q-PROP` | `limit`, resource-adapted `bbox`/`datetime`; conditional `id`, `q`, `baseProperty`, `objectType` |
| `Q-DS` | `limit`, `datetime`; conditional `id`, `q`, `phenomenonTime`, `resultTime`, `observedProperty`, `foi` |
| `Q-OBS` | `limit`; conditional `id`, `q`, `phenomenonTime`, `resultTime` including `latest`, `foi` |
| `Q-CS` | `limit`, `datetime`; conditional `id`, `q`, `issueTime`, `executionTime`, `controlledProperty`, `foi` |
| `Q-CMD` | `limit`; conditional `id`, `q`, `issueTime`, `executionTime`, `statusCode`, `sender`, `foi` |
| `Q-STAT` | `limit`, `datetime`; conditional `id`, `q`, `statusCode` |
| `Q-RES` | `limit`; conditional `id`, `q`; no result-specific advanced filter |
| `Q-EVT` | `limit`, `datetime`; conditional `id`, `q`, `eventType` |
| `Q-FEAT` | Ordinary OGC API - Features behavior for the returned non-CSAPI feature type; CSAPI Advanced Filtering does not automatically apply |

| Family | Resources endpoints receiving the contract | Contract and scope caveats |
|---|---|---|
| System | `/systems`; `/systems/{parentId}/subsystems`; typed System `/collections/{cid}/items` | `Q-SYS`. `recursive` additionally applies only to canonical `/systems` and nested `/subsystems`, not typed collections. Item routes have no collection filter contract. |
| Deployment | `/deployments`; conditional `/systems/{sysId}/deployments`; `/deployments/{parentId}/subdeployments`; typed Deployment items | `Q-DEP`. `recursive` applies only to canonical `/deployments` and nested `/subdeployments`, not System-scoped or typed collections. |
| Procedure | `/procedures`; typed Procedure items | `Q-PROC`. The tagged OAS omission of `bbox` cannot narrow the approved contract. |
| Sampling Feature | `/samplingFeatures`; `/systems/{sysId}/samplingFeatures`; conditional `/datastreams/{dsId}/samplingFeatures`; conditional `/controlstreams/{csId}/samplingFeatures`; typed Sampling Feature items | `Q-SF` on every listed resources endpoint. Stream-nested routes return Sampling Features and therefore do not use stream-family filters. |
| Property | `/properties`; typed Property items | `Q-PROP`. The tagged OAS omissions of `bbox`/`datetime` cannot override Part 1 Requirement 35. |
| Ultimate/local FOI | Conditional `/datastreams/{dsId}/featuresOfInterest`; `/controlstreams/{csId}/featuresOfInterest` | `Q-FEAT`, not `Q-SF`; an ultimate FOI need not be a Sampling Feature or CSAPI resource. |
| DataStream | `/datastreams`; `/systems/{sysId}/datastreams`; `/deployments/{depId}/datastreams`; typed DataStream items | `Q-DS`. Deployment scope also constrains membership and applicable valid time. `/schema` and child-family routes use their own contracts. |
| Observation | `/observations`; `/datastreams/{dsId}/observations`; typed Observation items | `Q-OBS`. Item and transaction-qualified item routes are not collection-filter endpoints. |
| ControlStream | `/controlstreams`; `/systems/{sysId}/controlstreams`; conditional `/deployments/{depId}/controlstreams`; typed ControlStream items | `Q-CS`. Use the accepted plural route and ControlStream interpretation despite copied DataStream/CommandStream wording. |
| Command | `/commands`; accepted `/controlstreams/{csId}/commands`; typed Command items | `Q-CMD`. The published numbered nested route's singular spelling is repaired to plural. |
| CommandStatus | Accepted `/commands/{cmdId}/status`; `/feasibility/{feasId}/status` | `Q-STAT`. No canonical or typed Status family. `datetime→reportTime` is the P-011-15 semantic mapping. |
| CommandResult | Accepted `/commands/{cmdId}/result`; `/feasibility/{feasId}/result` | `Q-RES`. No canonical or typed Result family. Part 2 §13.1 carries Part 1 `id`/`q` to these Part 2 resources. |
| Feasibility | Normative `/controlstreams/{csId}/feasibility`; optional typed Feasibility items; accepted project root `/feasibility` | `Q-CMD` is normative for the nested and advertised typed resources endpoints. P-011-17 applies it to the Glaux root as a documented adapter; child status/result use `Q-STAT`/`Q-RES`. |
| SystemEvent | `/systemEvents`; `/systems/{sysId}/events`; typed SystemEvent items | `Q-EVT`. Canonical camel-case and nested `/events` are normative; isolated ATS spellings do not replace them. |
| Removed System History | Tagged `/systems/{id}/history` routes | `H\|Ø\|J`: do not register or advertise them as CSAPI 1.0. Retained historical dynamic resources do not revive the removed System History class. |

For `/collections/{cid}/items`, QReg selects the family contract from the accepted `featureType`/`itemType`; `/collections/{cid}/items/{itemId}` remains an item endpoint rather than a filterable list. That item route is inherited/normative for feature collections and Part 2 resource collections; applying it to Part 1 Property collections is the accepted Glaux interpretation from IDR-SRV-010 because Part 1 does not say so explicitly. Part 1 typed collections are required for supported families, while Part 2 typed collections are conditional when advertised.

---

## 7. Filtering Semantics Findings

### 7.1 Parsing and Canonicalization

**P-011-13** accepts one occurrence of each QReg-declared scalar or list parameter. List parameters use OpenAPI `form`, `explode=false`: a comma-separated value in one occurrence. Empty elements, mixed local-ID/UID lists, malformed URI-form UIDs, a wildcard anywhere except the final character of a UID, duplicate occurrences, undeclared names, and properties outside the explicit simple-equality allowlist are `400`. A syntactically valid but unknown or unauthorized identifier yields an empty filtered collection, not a distinguishable existence error.

Glaux should preserve the raw request for diagnostics but create a canonical typed predicate for execution and cursor binding. Canonicalization may normalize URI/CURIE comparison according to the later identifier policy; it must not silently lowercase case-sensitive local IDs, sender identifiers, status tokens, or event types.

### 7.2 Identifier and Text Search

`id` searches both canonical local IDs and the resource UID field. A terminal `*` is prefix matching only on a UID. It is not a general glob, and there is no normative `uid` parameter. Matching alternatives inside the list are OR.

The Part 1 overview calls `q` “prefix search only,” while Requirement 40 and the tagged parameter say human-readable fields contain one of the terms. The formal requirement controls. **P-011-04** makes the first stable behavior exact: apply Unicode 17.0.0 `toNFKC_Casefold` to both the stored field and each term, then use locale-independent Unicode code-point substring containment over at least `name` and `description`, OR across terms. The server may register additional human-readable fields, but QReg/OAD must name them. Unicode-data version is contract metadata; changing it requires compatibility review. Lemmatization, stemming, fuzzy similarity, ranking, locale-dependent collation, and language-dependent tokenization are disabled initially because they would make membership less predictable.

### 7.3 Spatial Filtering

Features permits the server to choose which geometry or geometries to test when a feature exposes more than one. **P-011-14** therefore makes the choice explicit: for both spatial parameters, Glaux evaluates every representation-independent geometry designated query-relevant in QReg; IDR-SRV-015/026 must approve each family set before the contract freezes. For `bbox`, a resource matches if any designated geometry intersects. Resources with no geometry still match, because the inherited Features requirement says so. The server must support CRS84/CRS84h coordinate order, valid bounds, degenerate boxes, and antimeridian-spanning boxes. Alternate `bbox-crs` is absent unless the corresponding Features Part 2 capability is separately implemented and declared.

For `geom`, Glaux should parse bounded WKT and perform intersection against that same designated geometry set. Geometry-less resources do not match. The tagged description's word “operator” does not create an operator grammar; intersection is the only approved behavior. Vertex, nesting, byte-size, coordinate, and topology limits protect the parser and spatial engine without changing the predicate.

### 7.4 Temporal Filtering

The baseline grammar is one RFC 3339 instant or slash-delimited interval. A closed interval uses `start/end`; a half-bounded interval uses `../end`, `/end`, `start/..`, or `start/`. Intersection includes boundaries. The tagged shared schema conflicts with this grammar by modeling intervals as arrays, omitting `..`, and adding date-only, `now`, and `latest` globally.

Glaux should implement the approved grammar and these property bindings:

- Part 1 descriptions, DataStreams, and ControlStreams: `datetime` → `validTime`;
- CommandStatus: `datetime` → `reportTime` as the narrow P-011-15 model-based repair;
- SystemEvent: `datetime` → tagged JSON `time` (the prose calls it `eventTime`) under the same provisional overlay;
- DataStream: `phenomenonTime`/`resultTime` → corresponding stream extent;
- Observation: corresponding instant property;
- ControlStream/Command: corresponding issue/execution time property.

Only Observation `resultTime` has an approved `latest` token. Glaux should calculate the maximum result time after authorization, route scope, and every other supplied predicate, retain all ties, and paginate that result. On a nested DataStream endpoint this is naturally stream-scoped; on the canonical endpoint it is the literal maximum of the complete visible selection. A per-DataStream grouping interpretation is not baseline because the standard does not say so.

### 7.5 Hierarchy and Nested Scope

Route scope is an implicit predicate. A nested route first limits candidates to resources related to the parent in the path; explicit query filters then combine with that scope using AND. A conflicting but valid filter returns an empty 200 collection.

On canonical `/systems` and `/deployments`, omitted/false `recursive` normally starts from top-level resources; true starts from the complete hierarchy. However, the approved `parent` filter would be unusable at the root if it were applied only after a top-level-only candidate restriction. The tagged OAS description confirms the practical intent. **P-011-05** therefore treats an explicit `parent` filter as a direct-child query over the hierarchy regardless of the root default; `recursive` does not change the meaning of direct parent. On nested child routes, `recursive=false` is direct children and `true` is all descendants before other predicates.

Every traversal must be cycle-safe, depth/fan-out bounded, and deduplicated by canonical identity. The System FOI and capability requirements independently cause a top-level System to match when a recursive subsystem matches, even if the returned candidate set itself is top-level only.

### 7.6 Relationship, Property, and FOI Matching

Until issues #165 and #179 receive an adopted correction, **P-011-15** supplies this bounded documented interpretation:

- Preserve normalized local ID, UID, and submitted `href` for accepted links. Match a known local resource through any of those representations; do not require arbitrary URL parsing or live external dereferencing.
- Traverse `sampleOf` through the proximate sampling chain and `sampledFeature` toward the ultimate/domain feature cycle-safely. Match both Sampling Features and ultimate Features of Interest as the requirements state.
- Systems and Procedures match normalized capability facts explicitly established by their descriptions; Systems also include descendant capabilities as required.
- Deployments match capability/FOI through deployed Systems during the Deployment's applicable validity.
- Sampling Features match observed/controlled properties through associated DataStreams/ControlStreams, following the ATS interpretation.
- Derived-property matching follows transitive `baseProperty` relationships where the standard recommends/defines it, with cycle protection.

Glaux should not broaden System/Procedure capability matching to every associated runtime stream until the SWG resolves #179 or the project exposes that behavior as a separately documented extension.

### 7.7 Status, Availability, and Tasking

Commands match `statusCode` against current status, not every historical status report. CommandStatus entries match their own `statusCode`. Both use the nine uppercase approved tokens exactly. `sender` is exact identifier matching; case folding would change identity. Command and Feasibility queries are authorization-sensitive and must never widen the caller's ability to discover a ControlStream, target, sender, or result.

CSAPI 1.0 defines no generic availability, created-time, modified-time, owner, link-target, or lifecycle-state parameter beyond the filters listed here. Availability and other dynamic properties normally belong in DataStreams/Observations and are handed to IDR-SRV-020. No direct availability filter should be invented in this topic.

### 7.8 Simple Properties and Unsupported Filters

Features and Part 1 recommend equality parameters for useful simple-valued properties. Glaux should implement this only through an explicit allowlist derived from the accepted resource model and surfaced identically in QReg, OAD, and tests. A property absent from that allowlist is an unknown parameter. Wildcards, nested JSON paths, arbitrary property names, and encoding-dependent field names are not implied.

The first stable baseline does not include `owner`, `collection`, `link`, `created`, `modified`, `offset`, `cursor`, `uid`, `filter`, `filter-lang`, `sort`, `order`, `sortby`, `select`, `fields`, `properties`, `expand`, or `embed`, except where a future separately advertised profile explicitly defines one.

---

## 8. Sorting, Pagination, and Consistency Findings

### 8.1 Standards Boundary

CSAPI and Features Core define no client-controlled ordering. Issue #175 confirms the gap. The tagged OAS has no active sort parameter. Records Version 1.0 defines `sortby` with `+`/`-` directions and a Sortables resource, and the newly approved Common Part 3 / Features Part 5 standardizes Sortables discovery; neither is inherited by CSAPI, and Sortables alone does not bind sorting behavior. Glaux must not claim a nonexistent CSAPI sort capability.

### 8.2 Default Total Order

Paging still requires an implementation order. **P-011-06** proposes these provisional pre-release ascending tuples, each ending with immutable canonical RID:

| Family | Default order |
|---|---|
| `/collections` descriptors and Part 1 descriptive resources | `(RID)` |
| DataStreams and ControlStreams | `(RID)` |
| Observations | `(resultTime, RID)` |
| Commands and Feasibility Commands | `(issueTime, RID)` |
| CommandStatus | `(reportTime, RID)` |
| CommandResult | `(RID)` |
| SystemEvent | `(time, RID)` |

Temporal keys compare chronological UTC instants, not source-string spelling: offsets and fractional-second spellings denoting the same instant tie and then compare by RID. New writes with malformed time keys are rejected. If a legacy or conditionally optional time key can be absent/null, it sorts after all non-null values, then by RID. The exact normalization, null placement, and immutable tie-breaker are part of QReg and cursor identity.

Ascending temporal order supports deterministic replay and limited-connectivity resume; clients seeking the newest Observation can use the approved `resultTime=latest`. The deliberate tradeoff is that newest-first Command/status/event clients may have to traverse or follow later pages. These directions are Glaux policy, not OGC obligations. Accepting this report adopts them as implementation-planning defaults, not as an irreversible stable-release freeze: IDR-SRV-014A–014G must test the tradeoff with external clients and may recommend a pre-release change. Once a stable release publishes them, changing the order is breaking under IDR-SRV-010A.

### 8.3 Continuation Contract

The client-facing paging interface is `limit` plus links. Glaux should not publish `offset` or `cursor` as baseline input parameters. A `next` target may contain a private query name, but clients treat the entire URL as opaque and reproduce no token themselves.

The continuation must be either a cryptographically random opaque reference to server-side state or an authenticated-encrypted token. A MAC/signature-only self-contained token is permitted only when every embedded value is non-sensitive or pseudonymous; integrity without confidentiality does not protect tenant, compartment, policy, or classified identifiers. It should protect or reference:

- canonical route and normalized filter hash;
- complete stable sort tuple and last returned key;
- effective `limit`;
- pseudonymous principal/security-context binding and authorization-policy version;
- representation/profile discriminator supplied by IDR-SRV-012;
- data snapshot, revision, or high-watermark when consistency is promised; and
- issued/expiry times and key version.

Tokens must contain no raw principal, tenant, compartment, policy, protected-resource, or classified identifier. A forged token is invalid input; an expired server-managed link may return 404 under inherited guidance. Exact Problem Details and concealment are assigned to IDR-SRV-013/039.

### 8.4 Mutation and Snapshot Behavior

Features explicitly warns that ordinary paging is not necessarily safe when data changes. Keyset continuation avoids the basic offset-shift failure but does not by itself preserve records whose filter/sort value changes. Glaux should provide two documented implementation levels:

1. **Stable-key continuation:** minimum general behavior; keyset order, no duplicates from simple inserts, and explicit no-snapshot caveat.
2. **Snapshot-bound continuation:** required for repeatable mission/export queries and targeted for the reference profile when persistence permits; all pages reflect one authorized selection snapshot or watermark.

The first page should read `limit + 1` visible keys when practical so the server can omit an unnecessary terminal `next` link without calculating `numberMatched`. No response should loop to an earlier token. `prev` is omitted initially unless the selected persistence design can implement it coherently.

### 8.5 Counts, Extents, and Authorization

Glaux should include exact `numberReturned` on every JSON collection page and a response-generation `timeStamp` wherever the selected JSON collection schema permits it. It should include `numberMatched` only when an exact visible count is safely available within the query budget; otherwise omit it. Approximate values must use a future separately named field, never `numberMatched`. Any extent associated with a filtered response describes the same complete visible selection, not the current page, or is omitted.

Authorization is part of the relation before count and pagination. Therefore two users issuing the same wire query may receive different counts, page boundaries, continuations, and extents. A continuation cannot be replayed across principals or a material policy change.

### 8.6 Future Client-Controlled Sorting

If operational evidence later justifies sorting, Glaux should align with `sortby` and a representation-independent Sortables resource rather than define separate `sort` and `order`. The extension must define sortable keys for canonical as well as typed collection endpoints, direction, multiple keys, null/missing values, collation, geometry prohibition or metric semantics, status ordering, immutable tie-breakers, authorization, OAD, conformance identifier, and cursor interaction. It should wait for issue #175 disposition and the applicable approved Features sorting binding.

---

## 9. Selection and Projection Findings

CSAPI's use of “selected resources” means resources satisfying predicates, not selected fields. Neither approved Part nor the tagged OAS defines a general field mask, projection, link expansion, embedded relationship, summary, or detail query. Commented `select` references are not active evidence.

The initial Glaux profile should return complete schema-conformant resources in the negotiated encoding. Links remain links unless that representation already embeds content by its schema. This preserves JSON/GeoJSON/SensorML/SWE validation, generated clients, authorization review, and caching. Content negotiation and schema-specific Observation/Command formats belong to IDR-SRV-012.

Projection is not an authorization mechanism. Omitting a forbidden property from one representation does not replace object-property authorization, schema review, or prevention of side-channel leakage through filters and counts. A later field-selection profile would require an explicit stable vocabulary independent of JSON versus SensorML spelling, required-field rules, link behavior, schema/profile identity, cache keys, and complete security tests.

EDR's `parameter-name` is domain-specific selection of observed parameters, not a generic CSAPI precedent. It may inform a future Observation result-component profile. Common Part 3 Returnables can advertise a logical schema but does not by itself create a CSAPI projection parameter.

---

## 10. Security, Authorization, and Resource-Consumption Implications

### 10.1 Authorization Invariants

1. Authorization and releasability produce the candidate relation before every user predicate.
2. Relationship traversal never crosses into a hidden node merely to reveal a visible aggregate fact unless the later policy explicitly permits that inference.
3. `numberMatched`, `numberReturned`, extents, sorting, lookahead, and `next` are computed only from visible rows.
4. Valid identifiers that are absent or unauthorized have indistinguishable empty-collection behavior on list queries; direct-item 404 concealment policy belongs to IDR-SRV-039/040.
5. Cursors bind pseudonymous principal/security context, tenant/compartment scope, policy version, route, filters, order, snapshot, and representation without exposing raw sensitive identifiers.
6. Cache keys and validators must separate authorization and negotiated variants; exact `Vary`/private-cache policy belongs to IDR-SRV-012/039.
7. Error messages never echo a protected canonical ID, relationship, count, or allowed-value set that the caller cannot discover.

### 10.2 Query Budgets

The query registry must provide configurable, published or operator-visible budgets for:

- `ID_List`, keyword, sender, status, and event-type cardinality;
- aggregate query-string and decoded-value bytes;
- WKT bytes, vertices, rings, nesting, dimensions, topology complexity, and coordinate count;
- temporal span and result-density estimates on high-volume families;
- recursive depth, visited nodes, fan-out, cycles, and relationship joins;
- scanned rows, execution time, memory, response bytes, decompressed bytes, and concurrent expensive queries;
- connection/request rate and burst; and
- CQL AST depth/operators/functions/joins only if a future CQL2 profile exists.

A valid page size above the declared maximum is clamped; other budget breaches must not be disguised as a different filter result. `429` with optional `Retry-After` is appropriate for rate limits, while invalid/over-complex request taxonomy is finalized in IDR-SRV-013. Tightening a documented stable budget can be client-breaking and follows IDR-SRV-010A.

### 10.3 Timing and Enumeration

Prefix UID searches, full-text search, relationship closures, exact counts, and deep recursion can create existence and cardinality side channels. Authorization-before-query is a logical and observable contract: no index probe, optimizer choice, count shortcut, timing distinction, or page boundary may become an unauthorized existence oracle. A physical optimizer may safely reorder predicates or index work only when the result, errors, metadata, timing controls, and other observable behavior preserve that contract. The server should use constant-shape errors, cap work, avoid partial unauthorized facts, and instrument cost without logging sensitive raw values. Security testing must compare overlapping principals rather than test only anonymous versus administrator.

---

## 11. Persistence, Indexing, and Performance Implications

This section records capabilities the later persistence design must support; it does not select a database.

| Query need | Required logical support | Later owner |
|---|---|---|
| Local ID, UID, UID prefix | Unique exact indexes and safe prefix strategy | 016/025 |
| `q` | Unicode 17.0.0 `toNFKC_Casefold` code-point-containment index over registered fields; versioned rebuild strategy | 025/028 |
| `bbox`, `geom` | 2D/3D spatial intersection, antimeridian handling, designated geometry set | 026 |
| Valid/phenomenon/result/issue/execution/report/event time | Interval/instant indexes and family order composites | 018/025/027 |
| System/Deployment recursion | Cycle-safe hierarchy closure or bounded recursive traversal | 015/017/025 |
| FOI and sampling chain | Local/UID/href identity facts plus `sampleOf`/`sampledFeature` closure | 017/024/025 |
| Capability/property filters | Representation-independent normalized facts and base-property closure | 015/024/025/028 |
| Command current status | Transactionally maintained current-state index plus immutable status history | 020/025/029/036 |
| Stable paging | Composite keyset indexes ending in RID; MVCC/revision/high-watermark option | 025/027/029 |
| Visible exact counts | Authorization-compatible count plan or honest omission | 025/039/054 |

The planner should reject or budget a query before high-cost traversal when possible, but a cost refusal must not alter the logical result silently. Filter selectivity, cardinality, latency, scanned rows, spill/memory, page-token failures, and count omission should be observable as metrics with sensitive values redacted.

### 11.1 Execution-Mode Boundary

Ordinary bounded collection queries remain synchronous, filtered, and paged. Internal precomputation is appropriate for hierarchy/relationship closure, normalized text and capability facts, current-status facts, authorized count strategies, and temporal extents, provided it preserves the same visible predicate result and carries a defined freshness/transaction boundary. A stale precompute must never silently substitute a different answer.

Broad exports, cross-family analytics or aggregation, and exceptionally large exact-total jobs may justify a separately advertised asynchronous job/process service. They must never be triggered implicitly by an ordinary collection GET. The server instead refuses over-budget WKT, graph traversal, scan, response, concurrency, snapshot, or authorization-sensitive work through the stable error policy. Storage/precompute design belongs to IDR-SRV-025–029; any public job surface requires final architecture synthesis and its own conformance, authorization, lifecycle, and tests; load/refusal behavior belongs to IDR-SRV-054.

### 11.2 Preliminary Implementation Complexity

| Work item | Relative complexity | Rough implementation range | Assumption |
|---|---|---|---|
| Typed QReg, parser, normalization, OAD projection | High | 3–5 engineer-weeks | Stable route/resource registries already exist |
| Part 1 common/spatial/text filters | High | 4–7 engineer-weeks | Spatial/full-text backend selected later |
| Hierarchy, FOI, and property traversal | Very high | 5–9 engineer-weeks | Accepted normalized relationship model |
| Part 2 temporal/status/tasking filters | High | 4–7 engineer-weeks | Time-series and Command models stable |
| Keyset/snapshot cursor and counts | Very high | 4–8 engineer-weeks | Persistence can expose revision/snapshot support |
| Authorization-safe query integration | Very high | 4–8 engineer-weeks | Security policy model supplied by later topics |
| OAS/ATS supplement and client suites | High | 4–7 engineer-weeks | Automated fixtures and external clients available |

Ranges overlap and are not a delivery plan. They identify risk concentration for later implementation planning.

---

## 12. Error, Failure, and Validation Implications

| Condition | Baseline outcome | Notes / later owner |
|---|---|---|
| Undeclared query parameter | 400 | Prevent silent ignored filters; 013/014 |
| Duplicate parameter occurrence | 400 | Canonical one-occurrence grammar; compatibility adapters must be explicit |
| Empty/mixed/malformed `ID_List` or illegal wildcard | 400 | Valid unseen identifiers instead return empty 200 |
| Invalid keyword/list length or enum token | 400 | Case-sensitive status/event/sender rules as defined |
| Invalid WKT, bbox arity/range, or time grammar | 400 | Exact machine codes and safe detail to 013 |
| `limit` below min/noninteger | 400 | Above max clamps to advertised max |
| Unsupported `now`/`earliest`/generalized `latest` | 400 | Observation `resultTime=latest` is the exception |
| Unsupported sort/projection/CQL2/offset/cursor input | 400 | Unless a separately advertised extension accepts it |
| Incompatible canonical and compatibility aliases together | 400 | Example: `datetime` plus a future `reportTime` alias |
| Valid filters with no visible match | 200 empty collection | No existence distinction |
| Unknown parent path resource | 404 | Direct route semantics; concealment to 039/040 |
| Forged continuation | 400 or concealed 404 decision | Final taxonomy to 013/039 |
| Expired server-managed `next` | 404 permitted by inherited guidance | Stable error body to 013 |
| Rate limit | 429, optional `Retry-After` | RFC 6585; rate policy to 039/048/055 |
| Unacceptable representation | 406 | Content negotiation to 012 |
| Safe exact count unavailable | Omit `numberMatched` | Never fabricate or approximate under the exact field |

OpenAPI must declare every accepted stable or compatibility parameter, its route/family scope, grammar, enum, defaults, limits, examples, and 400 behavior. It must not advertise a parameter that the runtime ignores. Final validation should distinguish parsing, semantic validity, resource budget, authorization, and backend failure without exposing protected information.

---

## 13. Interoperability and Existing-Implementation Implications

### 13.1 What External Clients Actually Need

Interoperable clients should inspect `/conformance`, use the exact documented wire names, and follow returned `next` links rather than construct offsets or assume counts. The pinned Explorer's “cursor” mode is more accurately server-link-following: the supplied link may itself contain an offset. Its separate `limit=1000` item-count fallback is capped, transfers full resources, can race the page request, and is not an exact-count contract. The pinned OS4CSAPI client models `offset`, `cursor`, and separate `uid`, but modeling/serializing those conveniences neither proves server support nor creates obligations. Glaux should smoke-test these exact pins while keeping its standards contract honest.

Two bounded live checks illustrated server differences: one OpenSensorHub deployment did not advertise Part 1 Advanced Filtering and used offset links without standard envelope counts, while one CS GO deployment advertised Advanced Filtering and supplied counts. Because exact URLs and response captures were not preserved, these observations are non-reproducible illustrations and do not support a requirement or recommendation. The immutable pins independently show that OpenSensorHub's collection envelopes omit `numberMatched`/`numberReturned` while it exposes a separate count subresource, and connected-systems-go emits exact counts with offset links. The standards—not either implementation—establish that clients should discover optional classes and tolerate omitted counts.

### 13.2 Tagged OpenAPI Delta and Disposition Ledger

| Artifact finding | Standards comparison | Glaux disposition |
|---|---|---|
| `/collections` advertises `bbox`, `datetime`, `geom`, `q`, `limit` | Features `/collections` lists collection descriptors; these filters belong on resource collections | Do not baseline them on `/collections`; return complete configured inventory per IDR-SRV-010 |
| Generic collection-items path omits every family-specific filter | Advanced Filtering applies family semantics to typed collections | QReg drives runtime applicability and a documented OAD union/conditional description; exact OAD design to 014 |
| Procedure path advertises `foi` | Procedure FOI requirement is commented out and association is unsettled | Reject in strict baseline; no conformance claim or silent implementation |
| Procedure omits `bbox`; Property omits `bbox`/`datetime` | Their endpoint requirements literally import Features §§7.15.2–7.15.8 | Implement/document inherited parameters; typically no-op inclusion where resources lack geometry/time |
| `/deployments` omits required `recursive` | Part 1 Requirement 21 requires it; #169/#196 confirms omission | Implement and advertise; pin #196 as unmerged maintenance evidence only |
| Part 1 datetime schema adds date-only, `now`, `latest`, array intervals and omits `..` | Approved grammar is RFC 3339 slash form; only Part 2 Observation resultTime adds `latest` | Correct Glaux schema/OAD overlay; reject unsupported specials |
| Property example uses `object=`; parameter file uses URI-only `objectType` | Requirement/ATS uses `objectType: ID_List` | Use `objectType` only; compare normalized semantic identifiers; record local-ID ambiguity |
| Part 2 DataStream/ControlStream paths omit required `datetime` | Resource endpoint requirements explicitly require it | Add runtime and OAD binding |
| Part 2 Observations/Commands/Status/Results omit inherited `q`; Events omit inherited `id` | Part 2 Advanced Filtering inherits Part 1 common filters | Add to every applicable runtime/OAD endpoint |
| CommandStatus OAS uses `reportTime` instead of normative `datetime` | Model makes `reportTime` the intended property but approved wire name is `datetime` | Baseline uses only `datetime`; `reportTime` is a future adapter candidate under P-011-16, not adopted by this report |
| DataStream OAS adds `system`; Observation adds `dataStream`, `system`, `observedProperty`; ControlStream adds `system`; Command adds `controlStream`, `system`, `controlledProperty`; Event adds `system` | No corresponding approved requirements; some relationship semantics remain under #179 | Do not claim as CSAPI; defer to focused interoperability studies or a separately registered extension |
| ControlStream paths duplicate `q` | No semantic justification | Remove duplicate in generated OAD |
| System history paths and `validTime` remain | Published Part 2 removed System History | Exclude from CSAPI core; only a separate future extension may expose history |
| Feasibility and deployment-scoped Part 2 paths are absent | Approved text requires them | Runtime/OAD must follow approved requirements and accepted route overlay, not omission |
| Part 2 collection schemas make links optional and omit count/time fields | Imported response semantics remain applicable where requirements say so | Supplement schemas/tests; do not let schema validation replace semantic checks |
| No active sort/projection/CQL2/offset/cursor parameter in 424 tagged API files | Confirms absence, but absence alone is not normative proof | Keep first baseline free of these extensions |

### 13.3 Tagged Part 2 Parameter-to-Route Inventory

This is the reproducible active-query ledger for every collection or schema-discovery path in the pinned Part 2 OAS. “Repair” means generate the runtime/OAD binding from approved text; “artifact-only” means the tag does not create a standard obligation.

| Tagged path source | Active tagged query names | Standards comparison and disposition |
|---|---|---|
| [`/datastreams`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/paths/dataStreams.yaml) | `id`, `q`, `phenomenonTime`, `resultTime`, `system`, `foi`, `observedProperty`, `limit` | Repair missing `datetime`; reject/defer artifact-only `system`. |
| [`/systems/{systemId}/datastreams`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/paths/systemDataStreams.yaml) | `q`, `phenomenonTime`, `resultTime`, `limit` | Repair missing `datetime`, `id`, `observedProperty`, `foi`; System scope is in the path. |
| [`/datastreams/{dataStreamId}/observations`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/paths/dataStreamObservations.yaml) | `id`, `phenomenonTime`, `resultTime`, `foi`, `observedProperty`, `limit` | Repair missing `q`; reject/defer artifact-only Observation `observedProperty`. |
| [`/observations`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/paths/observations.yaml) | `id`, `phenomenonTime`, `resultTime`, `dataStream`, `system`, `foi`, `observedProperty`, `limit` | Repair missing `q`; reject/defer artifact-only `dataStream`, `system`, `observedProperty`. |
| [`/controlstreams`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/paths/controlStreams.yaml) | `id`, `q`, `issueTime`, `executionTime`, duplicate `q`, `system`, `foi`, `controlledProperty`, `limit` | Repair missing `datetime`; remove duplicate; reject/defer artifact-only `system`. |
| [`/systems/{systemId}/controlstreams`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/paths/systemControlStreams.yaml) | `id`, `q`, `issueTime`, `executionTime`, duplicate `q`, `limit` | Repair missing `datetime`, `controlledProperty`, `foi`; remove duplicate. |
| [`/commands`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/paths/commands.yaml) | `id`, `issueTime`, `executionTime`, `statusCode`, `sender`, `controlStream`, `system`, `foi`, `controlledProperty`, `limit` | Repair missing `q`; reject/defer the three artifact-only relationship names. |
| [`/controlstreams/{controlStreamId}/commands`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/paths/controlStreamCommands.yaml) | `id`, `issueTime`, `executionTime`, `statusCode`, `sender`, `foi`, `controlledProperty`, `limit` | Repair missing `q`; reject/defer artifact-only `controlledProperty`. |
| [`/commands/{cmdId}/status`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/paths/commandStatuses.yaml) | `id`, `reportTime`, `statusCode`, `limit` | Repair canonical `datetime` and inherited `q`; `reportTime` is future adapter-lane only under P-011-16. |
| [`/commands/{cmdId}/result`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/paths/commandResults.yaml) | `id`, `limit` | Repair inherited `q`; invent no result-specific filter. |
| [`/systemEvents`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/paths/systemEventsAll.yaml) / [`/systems/{systemId}/events`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/paths/systemEvents.yaml) | `datetime`, `eventType`, `q`, `system`, `limit` | Repair missing `id`; reject/defer `system`, which is redundant on the nested route. |
| [Removed System History](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/paths/systemHistory.yaml) | `validTime`, `q`, `limit` | `H\|Ø\|J`: exclude; the published standard removed this class. |
| [`/datastreams/{dataStreamId}/schema`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/paths/dataStreamSchemas.yaml) | required `obsFormat` | [`/req/datastream/schema-op`](https://docs.ogc.org/is/23-002/23-002.html#_req_datastream_schema-op), `N\|R\|B`: selects the Observation encoding whose schema is requested; no enum/default. |
| [`/controlstreams/{controlStreamId}/schema`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/paths/controlStreamSchemas.yaml) | optional `cmdFormat` | [`/req/controlstream/schema-op`](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream_schema-op), `N\|R\|M?`: selects the Command encoding whose schema is requested; omitted behavior to IDR-SRV-012; correct the description's erroneous `commandFormat`. |

The exact parameter files are pinned [`obsFormat.yaml`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/parameters/obsFormat.yaml) (raw SHA-256 `cefc3b97afb79e7364d1591fc5e116352199b0ba5cb6a7ded71bd6c63bd8289d`) and [`cmdFormat.yaml`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/parameters/cmdFormat.yaml) (`acc7958e5a21cb74bd641bd223573318d125786521e3d8ef6e92ec099c79192f`). `obsFormat`/`cmdFormat` select the **subject encoding** for which a schema is requested; `Accept` selects the schema response representation. The unreferenced [`format.yaml`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/parameters/format.yaml) defines `f` but is inactive (`A\|Ø\|J`) and creates no baseline parameter. Exact media/profile behavior and tagged item/collection inconsistencies belong to IDR-SRV-012.

The concrete tagged response inconsistencies handed to IDR-SRV-012 are artifact observations only:

| Resource/response | Tagged item media | Tagged collection/schema media | Disposition |
|---|---|---|---|
| Observation | [`application/json`, `application/swe+json`, `application/swe+csv`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/responses/observation.yaml) | [Collection: JSON and SWE JSON, no SWE CSV](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/responses/observationCollection.yaml) | `A\|X\|S`: reconcile against selected encoding classes; do not infer final negotiation here. |
| Command | [`application/json`, `application/swe+csv`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/responses/command.yaml) | [Collection: JSON and SWE JSON](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/responses/commandCollection.yaml) | `A\|X\|S`: item/collection SWE bindings conflict. |
| SystemEvent | [`application/sml+json`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/responses/systemEvent.yaml) | [Collection: `application/json`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/responses/systemEventCollection.yaml) | `A\|X\|S`: item/collection media differ. |
| Observation/Command schema | N/A | [Observation schema](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/responses/observationSchema.yaml) and [Command schema](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/responses/commandSchema.yaml): `application/json`, `application/schema+json`, `text/plain` | `A\|X\|S`: `Accept` chooses among these response forms; exact contract to IDR-SRV-012. |

The tagged OAS entirely omits Feasibility paths and deployment-scoped DataStream/ControlStream paths; §6.1 records their approved route contracts. All `select` references are comments and inactive.

### 13.4 Current History Disposition

- **#64 / PR #88:** closure did not adopt `earliest`; the unmerged PR cannot amend the standard or tag.
- **#165:** implement the bounded local-ID/UID/href and cycle-safe FOI policy in §7.6, mark it as Glaux policy, and track SWG disposition.
- **#175:** no current sort obligation; future work should align with OGC Sortables/`sortby` after a binding is approved.
- **#179:** use the ATS-supported resource mapping provisionally, but do not invent representation-dependent capability rules or broaden OAS-only filters.
- **#169 / PR #196:** implement required Deployment recursion even though the approved repair is not merged.
- **#182:** distinguish query interval grammar from resource `validTime` encoding; hand the latter to IDR-SRV-018/023.

### 13.5 Compatibility Lanes

Tests and documentation should distinguish:

1. **Strict CSAPI lane:** approved requirements plus accepted Glaux project policies; undeclared OAS extras fail.
2. **Published-artifact adapter lane:** only separately approved safe aliases; P-011-16 leaves CommandStatus `reportTime` as a future candidate, never baseline or canonical output and never part of CSAPI conformance claims.
3. **Experimental extension lane:** future CQL2/sorting/projection features under separate identifiers and OAD/profile material.
4. **External-client lane:** pinned Explorer, OS4 client, generated clients, and later IDR-SRV-014A–014G targets.

---

## 14. Test-Strategy Implications

### 14.1 Required Test Layers

| Layer | Minimum evidence |
|---|---|
| Parser/unit | Every grammar boundary, list serialization, Unicode rule, enum, time interval, WKT, duplicate, unknown and limit clamp |
| Predicate/property | OR within lists, AND across filters, inclusive boundaries, null/no-geometry differences, exact property binding |
| Resource-family contract | Every matrix row on canonical, nested, deployment-scoped, feasibility-related, and typed collection routes |
| Hierarchy/graph | Three-plus levels, cycles, duplicates, parent precedence, descendant capability, sampling chains, external links |
| Paging invariant | Stable total order, ties, union equals selection, no duplicates/gaps/loops, effective limit, terminal link, counts |
| Mutation/consistency | Inserts, updates, deletes and authorization-policy changes between pages for stable-key and snapshot modes |
| Security | Two or more overlapping principals; hidden IDs/relations/counts/extents; cursor replay/tampering; cache separation; timing probes |
| Schema/OAD | Runtime↔QReg↔OAD parameter parity; schema closure; complete representations; no stale/duplicate/artifact-only bindings |
| Conformance | Every selected official ATS test plus documented supplemental tests for ATS gaps/defects |
| Interoperability | Pinned Explorer, OS4 client, generated clients and later implementation study fixtures |
| Load/fuzz | Oversized lists/WKT/time/recursion, high limit, expensive counts, concurrency, cancellation, rate limits and memory |

### 14.2 High-Value Invariants

For a fixed authorized snapshot and normalized query:

- concatenating all pages in `next` order yields exactly the filtered selection once;
- every page preserves the same predicate, order, limit, representation, principal, policy, and snapshot identity;
- `numberReturned` equals serialized first-level members;
- `numberMatched`, when present, equals the complete caller-visible selection;
- removing a predicate cannot reduce the result set, except when a cost/security refusal replaces ordinary execution;
- adding a list alternative cannot remove matches; adding a different filter cannot add matches;
- `bbox` and `geom` differ only as intentionally defined, including no-geometry behavior;
- `recursive=true` is a superset of the ordinary hierarchy candidate set, before a direct `parent` predicate;
- changes in hidden data do not alter a caller's visible count/link metadata; and
- a forged/cross-principal cursor never returns data.

### 14.3 ATS Gaps Requiring Supplemental Tests

The official ATS does not adequately test combined-filter AND behavior, invalid values, unknown parameters, open temporal bounds, `limit` clamping, count/link semantics, `latest`, sorting absence, projection rejection, nested/custom endpoint coverage, mutation, or authorization. The exact tagged defects below require a versioned Glaux overlay; literal execution of a copied defect is not conformance.

| Tagged ATS identifier / source | Exact defect | Glaux supplemental correction |
|---|---|---|
| [`/conf/advanced-filtering/system-by-parent`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/standard/sections/annex-abstract-test-suite.adoc#L940-L952) → `/req/advanced-filtering/system-by-parent` | Follows `parentSystem` but checks the child ID against requested parent IDs | Check the followed parent System identifier. |
| [`system-by-obsprop` / `system-by-controlprop`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/standard/sections/annex-abstract-test-suite.adoc#L996-L1028) → corresponding `/req/advanced-filtering/...` IDs | Retrieves descendants through nonexistent `/components` | Use accepted `/subsystems` and test recursive descendant capability. |
| [`/conf/advanced-filtering/deployment-by-parent`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/standard/sections/annex-abstract-test-suite.adoc#L1036-L1048) → `/req/advanced-filtering/deployment-by-parent` | Follows `parentSystem` for a Deployment and checks the child itself | Follow the parent Deployment relationship and validate its identifier. |
| [`/conf/advanced-filtering/deployment-by-system`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/standard/sections/annex-abstract-test-suite.adoc#L1054-L1066) → `/req/advanced-filtering/deployment-by-system` | UID repeat step mistakenly sets `foi`, not `system` | Repeat with `system={UID-list}`. |
| [`/conf/advanced-filtering/indirect-prop`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/standard/sections/annex-abstract-test-suite.adoc#L1274-L1306) → `/rec/advanced-filtering/indirect-prop` | Sampling Feature portion repeats `/systems` requests | Exercise `/samplingFeatures` for the transitive-property check. |
| [`/conf/advanced-filtering/status-by-statuscode`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/annex-abstract-test-suite.adoc#L1158-L1168) | Calls returned entries Commands and checks `currentStatus` | Iterate CommandStatus resources and check each `statusCode`. |
| [`/conf/feasibility/canonical-url`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/annex-abstract-test-suite.adoc#L637-L647) | Iterates `itemType=Command`, not `Feasibility` | Use `itemType=Feasibility`. |
| [`/conf/feasibility/ref-from-controlstream`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/annex-abstract-test-suite.adoc#L652-L660) | Tests a Command route instead of Feasibility | Exercise the accepted Feasibility route and assert only its scoped resources. |
| [`/conf/system-event/canonical-url` and endpoint tests](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/annex-abstract-test-suite.adoc#L735-L772) | Uses ControlStream resources/test target for SystemEvent | Use SystemEvent, `itemType=SystemEvent`, and its resources-endpoint test. |
| [`/conf/system-event/ref-from-system`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/annex-abstract-test-suite.adoc#L778-L786) | Uses `/systems/{sysId}/systemEvents` | Use normative `/systems/{sysId}/events`. |
| [`/conf/advanced-filtering/event-by-type`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/annex-abstract-test-suite.adoc#L1176-L1188) | Lowercases `/systemevents`; checks prose `.type` despite tagged JSON `.definition` | Use `/systemEvents` and test the P-011-15 encoding mapping explicitly. |
| [`/conf/advanced-filtering/obs-by-resulttime`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/annex-abstract-test-suite.adoc#L931-L943) | Omits normative `latest` cases | Add canonical/nested maximum, ties, no-match, other-filter, and authorization-scope cases. |
| [Status/result resource endpoint tests](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/annex-abstract-test-suite.adoc#L556-L600) | Check only GET/200/media/schema | Add `limit`, query selection, invalid input, link, count, and authorization semantics. |

The Part 2 Advanced Filtering class's [positive-test list](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/annex-abstract-test-suite.adoc#L811-L833) contains no dedicated negative, combination, limit-clamp, count/link, mutation, or authorization suite. Those remain explicit Glaux supplemental obligations.

For exact requirement-to-test traceability, the affected Part 2 rows target these published identifiers:

| Tagged test | Target requirement identifier |
|---|---|
| `/conf/advanced-filtering/status-by-statuscode` | `/req/advanced-filtering/status-by-statuscode` |
| `/conf/feasibility/canonical-url` | `/req/feasibility/canonical-url` |
| `/conf/feasibility/ref-from-controlstream` | `/req/feasibility/ref-from-controlstream` |
| `/conf/system-event/canonical-url` | `/req/system-event/canonical-url` |
| `/conf/system-event/canonical-endpoint` | `/req/system-event/canonical-endpoint` |
| `/conf/system-event/ref-from-system` | `/req/system-event/ref-from-system` |
| `/conf/advanced-filtering/event-by-type` | `/req/advanced-filtering/event-by-type` |
| `/conf/advanced-filtering/obs-by-resulttime` | `/req/advanced-filtering/obs-by-resulttime` |
| `/conf/controlstream/status-resources-endpoint` | `/req/controlstream/status-resources-endpoint` |
| `/conf/controlstream/result-resources-endpoint` | `/req/controlstream/result-resources-endpoint` |

### 14.4 Suggested Fixture Families

- Two authorized principals with overlapping but nonidentical Systems, streams, Commands, and events.
- Three-level System and Deployment hierarchies with a deliberate cycle fixture rejected or safely contained.
- Sampling Features with local targets, UID-only targets, stored external `href`, proximate chains, and ultimate FOI.
- Spatial features crossing boundaries/antimeridian plus geometry-less descriptions.
- Temporal instants, closed/open intervals, equal timestamps, null validity, and `latest` ties.
- Every status token, sender/FOI variations, current-status versus historical-status differences.
- Dataset mutations between pages; expiring/rotated random-reference and authenticated-encryption keys; and a restricted MAC-only fixture containing only non-sensitive pseudonymous state.
- Artifact-defect fixtures for missing/extra OAS parameters, casing, history, and CommandStatus time alias.

---

## 15. Downstream Topic Handoff Matrix

| Topic(s) | Required handoff from IDR-SRV-011 |
|---|---|
| IDR-SRV-012 | Representation/profile discriminator in links/cursors, complete-resource negotiation, cache `Vary`, Observation/Command format parameters |
| IDR-SRV-013 | Stable Problem Details taxonomy for invalid/unknown/duplicate/incompatible filters, forged/expired cursors, budgets, 429, 404 concealment |
| IDR-SRV-014 | Generate OAD from QReg; exact parameter schemas/defaults/routes; conditional typed-collection applicability; artifact overlay and aliases |
| IDR-SRV-014A–014G / 056 | Verify OAS-only relationship filters, client assumptions, implementation differences, counts and paging links |
| IDR-SRV-015 | Representation-independent capability facts, query-relevant geometry fields, simple-property allowlist, family order keys |
| IDR-SRV-016 | Local ID/UID/CURIE normalization, UID prefix behavior, canonical RID, unresolved link identifiers |
| IDR-SRV-017 | Parent/child, deployed-System, stream, FOI, sampledFeature/sampleOf relationship semantics and closures |
| IDR-SRV-018 | `validTime`, open resource bounds, time-property definitions, `latest`, history and snapshot windows |
| IDR-SRV-020 | Command current status, status history, availability and SystemEvent vocabulary/property mapping |
| IDR-SRV-023 | Parsers, list/time/WKT validation, complexity limits, schema closure and semantic validation beyond schemas |
| IDR-SRV-024 | Representation-independent Property, FOI, sampling-chain and capability facts used by relationship filters; no live dereference |
| IDR-SRV-025 | QReg execution abstraction, indexes, planner budgets, count strategy, keyset/snapshot cursor feasibility |
| IDR-SRV-026 | CRS84/CRS84h, antimeridian, multiple geometry, bbox/geom indexing and WKT limits |
| IDR-SRV-027 | Observation/stream time indexes, resultTime order/latest, high-volume pages and watermarks |
| IDR-SRV-028 | Full-text and document-property facts without encoding-dependent query names |
| IDR-SRV-029 | Snapshot/MVCC guarantees, concurrent mutation, cursor revision, transactional current status |
| IDR-SRV-036/037 | Command/control lifecycle query facts, Feasibility root/typed/nested applicability, and any separately advertised async job boundary |
| IDR-SRV-039/039A/040 | Authorization-first relation, releasability, counts/extents/links, cursor binding, timing/cache/log side channels |
| IDR-SRV-048 | Redacted query-cost, rate, count-omission, continuation-failure, and resource-consumption metrics without raw sensitive values |
| IDR-SRV-050/051 | Official ATS plus defect overlay, requirement↔QReg↔test traceability and truthful class gates |
| IDR-SRV-053 | Fixtures listed in §14.4, golden pages/tokens/OAD and mutation/auth scenario corpus |
| IDR-SRV-054/055 | Cost/load/fuzz budgets, rate limits, concurrent expensive queries, Command/security query cases |
| Final implementation-plan synthesis | Resolve any still-pending root Feasibility, snapshot, async/export, optional sorting/CQL2/projection, and persistence tradeoffs before build sequencing |

---

## 16. Recommendations and Decision Analysis

### 16.1 Options Considered

| Option | Benefits | Costs / risks | Standards / compatibility impact | Recommendation |
|---|---|---|---|---|
| Copy tagged OAS literally | Quick scaffolding | Misses requirements, adds undefined semantics, stale routes/schemas | False/incomplete conformance | Reject |
| Minimum classes only | Smaller implementation | Undermines accepted best-reference goal and client usefulness | Can conform narrowly but conflicts with Glaux target | Reject as end state |
| Complete CSAPI filters plus strict QReg | Standards-grounded, testable, reusable, honest | Significant graph/time/index work | Fits accepted all-class profile | **Adopt** |
| Add bespoke sort/offset/fields immediately | Familiar convenience | New contract, unstable paging, schema/security complexity | Not CSAPI; future collision risk | Reject for first stable profile |
| Add CQL2 immediately | Powerful standard filter language | Parser/planner/DoS and conformance-class breadth | Optional; not inherited | Defer pending demonstrated need |
| Opaque keyset links with optional snapshots | Deterministic and scalable; hides implementation | Token/key/snapshot complexity | Fully compatible with Features link model | **Adopt** |
| Offset as public contract | Easy and common | Duplicates/gaps under mutation; clients synthesize links | Not required; weaker reference behavior | Reject as baseline |
| Full resources only | Strong validation/interoperability | Larger payloads | Exactly matches current CSAPI surface | **Adopt** |

### 16.2 Key Recommendations

1. Accept proposed decisions **P-011-01 through P-011-17** in Appendix C as the downstream planning baseline.
2. Implement Advanced Filtering as one complete capability, never as a route-by-route marketing label.
3. Make QReg the authoritative query contract and generate runtime parsers, OAD, docs, authorization hooks, cost controls, index requirements, and tests from it.
4. Freeze `limit` at `1/10/10000` for the initial reference profile, with above-max clamping and exact OAD parity.
5. Implement only approved slash-form temporal grammar and the single normative `resultTime=latest` special case.
6. Preserve exact spatial, list, AND/OR, hierarchy, and no-match semantics from §§6–7.
7. Use the provisional deterministic ascending family orders ending in RID and link-only opaque keyset continuation; validate order direction before stable release and add snapshot-bound behavior when storage proves it.
8. Always include exact `numberReturned`; include exact caller-visible `numberMatched` only when safe and affordable.
9. Return complete resources and reject undeclared sort/projection/CQL2/offset/cursor parameters.
10. Repair missing normative OAS bindings and isolate, defer, or reject artifact-only behavior through §13.2.
11. Treat authorization, budgets, and cursor integrity as part of query correctness, not later middleware decoration.
12. Preserve the documented open-issue interpretations as Glaux policy, track upstream, and process any later resolution through IDR-SRV-010A compatibility review.

---

## 17. Risks, Constraints, and Open Questions

### 17.1 Risks and Constraints

| Risk | Consequence | Mitigation / owner |
|---|---|---|
| OAS copied as truth | Missing obligations and invented filters | QReg + artifact ledger; 014/050 |
| Undefined order | Duplicate/missing pages, nondeterministic tests | P-011-06 and composite indexes; 025/027 |
| Continuation not confidentially bound to auth/policy | Cross-user, stale-policy, or token-content disclosure | Random server-side reference or authenticated encryption; restricted non-sensitive MAC-only lane; 039/040 |
| Exact totals forced on expensive queries | Denial of service and latency | Honest omission and budget; 025/054 |
| Relationship semantics encoded from one JSON shape | SensorML/GeoJSON drift and wrong matches | Normalized facts; 015/017/024 |
| Broad `q`/UID prefix/WKT/recursion | Resource consumption and enumeration | Limits, authorization first, fuzz/load tests |
| Special temporal tokens generalized from OAS | Ambiguous/breaking result membership | Approved grammar only; track #64/#182 |
| Projection added casually | Invalid schemas, protected-property leaks, cache explosion | Full-resource baseline; separate future profile |
| Client conveniences mistaken for standard | Contract proliferation and future collision | Strict/adapter/extension lanes |
| Snapshot target exceeds storage capability | False consistency promise | Persistence proof before claim; document stable-key minimum |

### 17.2 Resolved Plan Open Questions

- **CQL2:** preserve architectural room but defer from the first CSAPI profile. Adoption requires explicit Queryables/Filter/binding/language classes, cost controls, and a demonstrated client use case.
- **CSAPI Explorer:** it needs exact wire names, conformance discovery, and supplied links; its local `offset`/count conveniences are not server requirements.
- **Authorization-filtered counts:** exact caller-visible `numberMatched` or omission; never count unauthorized data or label an estimate exact.
- **Resource limits:** QReg budgets every high-cost dimension; initial page min/default/max is 1/10/10000; higher valid values clamp.

### 17.3 Remaining Unresolved Issues

1. The SWG has not settled representation-independent System/Procedure property capability sources (#179).
2. Exact external-link and sampling-chain matching remains open (#165); §7.6 is the proposed bounded Glaux policy.
3. CSAPI has no approved client sorting; #175 and the future applicable Features sorting binding must be tracked.
4. `objectType` formally says `ID_List` while artifacts/model favor URI/CURIE identifiers.
5. Part 2 does not explicitly bind CommandStatus `datetime` to `reportTime` or resolve SystemEvent prose/schema names.
6. Canonical `resultTime=latest` grouping is not explicit; this report chooses the literal visible endpoint maximum with ties.
7. Snapshot-bound paging feasibility and retention duration depend on later persistence/consistency research.
8. Final numeric budgets beyond the stable page-size contract depend on storage, deployment profile, threat model, and performance evidence.
9. Applying the normative Command query contract to the accepted project root `/feasibility` remains a Glaux adapter decision; nested and advertised typed Feasibility endpoints are not ambiguous.
10. Exact query-relevant geometry sets, simple-property allowlists, and omitted-`cmdFormat` behavior remain assigned to IDR-SRV-015/026 and IDR-SRV-012 respectively.

None prevents a defensible implementation plan if the proposed Glaux interpretations are accepted and preserved as explicit policy rather than mislabeled standards text.

---

## 18. Validation Against the Research Plan

### 18.1 Success Criteria

| Topic-plan success criterion | Status | Evidence |
|---|---|---|
| Query/filter/sort/page/selection requirements identified with anchors | Met | §§3, 5–9; §6 matrix |
| CSAPI-specific versus inherited OGC behavior distinguished | Met | §§3.4, 5.1–5.4; classification in §6 |
| Behavior mapped to resource families | Met | §§5.2–5.3 and complete §6 matrix |
| Spatial, temporal, identifier, relationship, status, dynamic and tasking implications assessed | Met | §§7.2–7.7; matrix |
| Sorting/pagination consistency documented | Met | §8 |
| Selection/projection classified | Met | §§5.4, 9, 16.1 |
| Security/authorization/consumption/leakage identified | Met | §10 and security column in §6 |
| Persistence/OAD/error/validation/performance/test handoffs documented | Met | §§11–15 |
| Recommendations decision-usable and bounded | Met | §§1, 16–17; Appendix C |
| References explicit and reproducible | Met | §§3, 19; commit/hash pins |

### 18.2 Deliverable Requirements

All nineteen required report areas are Sections 1–19. The §6 matrix includes all fourteen required fields. Appendix A accounts for all 55 detailed questions, Appendix B supplies a compact contract view, Appendix C records proposed decisions with acceptance pending, and Appendix D records research-execution completeness and the controlled next actions.

### 18.3 Methodology Phases

| Phase | Status | Output |
|---|---|---|
| 1. Source collection/framework | Complete | §3 pins, prerequisites and classifications |
| 2. Standards extraction | Complete | §5 standards and artifact cross-checks |
| 3. Resource-family mapping | Complete | §6 matrix |
| 4. Sorting/paging/selection | Complete | §§8–9 |
| 5. Security/performance/persistence/error | Complete | §§10–12 |
| 6. Synthesis | Complete | §§1, 13–18 and appendices |

---

## 19. References

### 19.1 Controlling Standards

- [OGC API - Connected Systems - Part 1: Feature Resources, OGC 23-001](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2: Dynamic Data, OGC 23-002](https://docs.ogc.org/is/23-002/23-002.html)
- [OGC API - Features - Part 1: Core corrigendum, OGC 17-069r4](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [OGC API - Features - Part 3: Filtering, OGC 19-079r2](https://docs.ogc.org/is/19-079r2/19-079r2.html)
- [Common Query Language (CQL2), OGC 21-065r2](https://docs.ogc.org/is/21-065r2/21-065r2.html)
- [OGC API - Records - Part 1: Core, OGC 20-004r1](https://docs.ogc.org/is/20-004r1/20-004r1.html)
- [OGC API - Features Part 5 / OGC API - Common Part 3: Schemas, OGC 23-058r2](https://docs.ogc.org/is/23-058r2/23-058r2.html)
- [OGC API - Environmental Data Retrieval, OGC 19-086r6](https://docs.ogc.org/is/19-086r6/19-086r6.html)
- [OGC API - Processes - Part 1: Core, OGC 18-062r2](https://docs.ogc.org/is/18-062r2/18-062r2.html)
- [RFC 9110, HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)
- [RFC 9111, HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111.html)
- [RFC 6585, Additional HTTP Status Codes](https://www.rfc-editor.org/rfc/rfc6585.html)
- [OpenAPI Specification 3.1.0](https://spec.openapis.org/oas/v3.1.0.html)
- [Unicode Standard 17.0.0, Chapter 3](https://www.unicode.org/versions/Unicode17.0.0/core-spec/chapter-3/) and [UAX #15 Revision 57](https://www.unicode.org/reports/tr15/tr15-57.html)
- [OWASP API Security Top 10 2023: API1, API3, API4](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)

### 19.2 Official Artifacts and History

- [Official CSAPI `v1.0.0` source at commit `8e03b236...`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)
- [Versioned Part 1 published schema/OAS tree](https://schemas.opengis.net/ogcapi/connected-systems/part1/1.0/openapi/)
- [Versioned Part 2 published schema/OAS tree](https://schemas.opengis.net/ogcapi/connected-systems/part2/1.0/)
- [Current pinned official branch commit `3fd86c73...`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f)
- [Issue #64](https://github.com/opengeospatial/ogcapi-connected-systems/issues/64) and [unmerged PR #88](https://github.com/opengeospatial/ogcapi-connected-systems/pull/88)
- [Issue #165](https://github.com/opengeospatial/ogcapi-connected-systems/issues/165)
- [Issue #169](https://github.com/opengeospatial/ogcapi-connected-systems/issues/169) and [open PR #196](https://github.com/opengeospatial/ogcapi-connected-systems/pull/196)
- [Issue #175](https://github.com/opengeospatial/ogcapi-connected-systems/issues/175)
- [Issue #179](https://github.com/opengeospatial/ogcapi-connected-systems/issues/179)
- [Issue #182](https://github.com/opengeospatial/ogcapi-connected-systems/issues/182)
- [Glaux upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md)

### 19.3 Project and Supporting Evidence

- [Glaux Server goal and definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Controlling overall IDR plan](../IDR%20Plans/overall-idr-research-plan.md)
- [IDR-SRV-011 topic plan](../IDR%20Plans/idr-srv-011-query-filtering-sorting-pagination-and-selection-semantics.md)
- [Research-planning governance](../../../../../Governance/research-planning-approach.md)
- [Research-plan creation prompt](../../../../../Governance/research-plan-creation-prompt.md)
- [Research-plan template](../../../../../Governance/research-plan-template.md)
- [Research-report template](../../../../../Governance/research-report-template.md)
- [Overall research-report template](../../../../../Governance/overall-research-report-template.md)
- [OS4CSAPI research-plan exemplar corpus pinned at `75441189...`](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans), including [blueprint-first](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans/01-pr114-blueprint-analysis.md), [inventory/sourcing](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans/15-fixture-sourcing-organization.md), and [synthesis](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans/38-testing-playbook-synthesis.md) exemplars
- [Accepted IDR-SRV-006 Part 1 baseline](idr-srv-006-csapi-part-1-requirement-baseline-report.md)
- [Accepted IDR-SRV-007 Part 2 baseline](idr-srv-007-csapi-part-2-requirement-baseline-report.md)
- [Accepted IDR-SRV-008 conformance mapping](idr-srv-008-conformance-class-and-requirement-mapping-report.md)
- [Accepted IDR-SRV-009 entry-point behavior](idr-srv-009-landing-page-api-definition-and-conformance-declaration-behavior-report.md)
- [Accepted IDR-SRV-010 resource/navigation behavior](idr-srv-010-collections-resources-links-and-navigation-behavior-report.md)
- [Accepted IDR-SRV-010A compatibility policy](idr-srv-010a-api-versioning-backward-compatibility-and-deprecation-strategy-report.md)
- [connected-systems-go pinned source](https://github.com/OS4CSAPI/connected-systems-go/tree/e900da88738cca92872038b703c4ad537fc0c8fd)
- [OpenSensorHub pinned source](https://github.com/OS4CSAPI/osh-core/tree/b2badae59aaa78455c5638ad73b452ccdee40207)
- [OS4CSAPI client pinned source](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/6fdf39669e8b11fe1e9c0c8914278c5f11a3b57f)
- [CSAPI Explorer pinned source](https://github.com/OS4CSAPI/ogc-csapi-explorer/tree/00f1c188e05738ee03390fd95f09d351e073a9c3)

---

## Appendix A. Detailed Research-Question Coverage

This ledger accounts for every top-level detailed question in the topic plan. “Complete” means the report reached a standards-grounded answer, a bounded Glaux recommendation, or an explicit downstream handoff; it does not mean that the plan owner has accepted the proposed project decisions.

### A.1 Standards and Query Sources

| ID | Plan question, shortened | Answer | Report location | Status |
|---|---|---|---|---|
| S1 | Part 1 parameters and semantics | Part 1 supplies inherited `limit`/spatial/temporal behavior and Advanced Filtering for IDs, text, geometry, hierarchy, relationships, capabilities and resource families. | §§5.1–5.2, 6–7 | Complete |
| S2 | Part 2 parameters and semantics | Part 2 adds stream, Observation, Command, status, feasibility and event filters, while reusing Part 1 common filters and pagination. | §§5.3, 6–7 | Complete |
| S3 | Features Part 1 inheritance | Only behavior incorporated by the applicable CSAPI requirements is inherited: chiefly `limit`, paging links, counts, `bbox`, `datetime`, unknown-parameter handling and no-match behavior. | §§3.4, 5.1, 8 | Complete |
| S4 | Relevant optional OGC building blocks | Features Filtering/CQL2, Records sorting, Sortables, Returnables, EDR and Processes are useful patterns but do not create CSAPI obligations. | §§5.4, 8.6, 9, 16 | Complete |
| S5 | OAS/schema-only behavior | Every tagged Part 1/2 parameter and binding was inventoried; omissions, extras, temporal conflicts, stale paths and schema gaps are dispositioned rather than copied. | §§3.2, 4, 13.2–13.3 | Complete |
| S6 | Implementations/client expectations | Pinned implementations and clients show useful link/count/offset precedents, but client conveniences and observed server behavior remain informative only. | §§3.3, 13.1, 13.5 | Complete |

### A.2 Resource Families and Query Scope

| ID | Plan question, shortened | Answer | Report location | Status |
|---|---|---|---|---|
| R1 | Systems, Procedures, Deployments, Sampling Features, Properties | Family-specific rows define IDs, search, geometry/time, hierarchy, FOI, capabilities and object type as applicable. | §§5.2, 6, 7.2–7.6 | Complete |
| R2 | DataStreams and Observations | DataStreams use valid/phenomenon/result extents plus FOI/observed-property relationships; Observations use instant phenomenon/result time, FOI and `latest`. | §§5.3, 6, 7.4 | Complete |
| R3 | ControlStreams, Commands, status and feasibility | Rows define controlled-property, FOI, issue/execution time, sender, current/status-entry state and feasibility scope. | §§5.3, 6, 7.4, 7.7 | Complete |
| R4 | Events, status, availability, dynamic properties | System Events use ID, `q`, time and type; status is handled on Commands/status entries; generic availability/dynamic-property filters are not defined. | §§6, 7.7 | Complete |
| R5 | Collection, item, nested, related and history scope | Route scope is an implicit predicate; canonical/nested/recursive scopes are defined; item lookup stays distinct; removed System History is excluded. | §§5.2–5.3, 6.1, 7.5, 13.2 | Complete |
| R6 | Uniform versus family-specific behavior | Parsing, AND/OR, authorization, ordering and paging are uniform; applicable filter names, property bindings and default order keys vary by family. | §§1.2, 5.1–5.3, 6, 8.2 | Complete |

### A.3 Filtering Semantics

| ID | Plan question, shortened | Answer | Report location | Status |
|---|---|---|---|---|
| F1 | Identifier, type, text, relationship, ownership, collection and link filters | Approved ID/text/type/relationship filters are mapped; generic owner, collection and link-target parameters are absent from the baseline. | §§6, 7.2, 7.6, 7.8 | Complete |
| F2 | Geospatial filters | `bbox` uses inherited intersection and includes geometry-less features; `geom` is bounded WKT intersection and excludes them. | §§6, 7.3 | Complete |
| F3 | Temporal filters | Approved instant/slash-interval grammar and per-family property bindings are defined; unsupported OAS-only tokens are rejected; Observation `resultTime=latest` is bounded. | §§6, 7.4 | Complete |
| F4 | Status and availability | Command/current-status and status-entry filters use exact approved tokens; no generic CSAPI availability filter exists. | §§6, 7.7 | Complete |
| F5 | Command/tasking filters | ControlStream, controlled property, issue/execution time, sender, current status, status-entry and feasibility behavior is mapped with authorization constraints. | §§6, 7.7, 10 | Complete |
| F6 | Normative/optional/project/future classification | Every matrix row separates authority, conditionality/optionality/future status, and implement/map/defer/reject disposition. | §§3.4, 6 | Complete |

### A.4 Sorting Semantics

| ID | Plan question, shortened | Answer | Report location | Status |
|---|---|---|---|---|
| SO1 | Required or observed sorting | CSAPI, Features Core and tagged CSAPI OAS define no client sort; existing OGC `sortby` patterns are comparative only. | §§5.4, 8.1, 13.2 | Complete |
| SO2 | Safe/useful sort fields | The first baseline exposes none; a future profile must advertise logical sortable keys and prohibit or define problematic geometry/status/collation semantics. | §8.6 | Complete |
| SO3 | Temporal, spatial, text, ID, status and relationship ordering | Default ordering uses time or RID only; richer client ordering is deferred until each type, null, collation and authorization rule is specified. | §§8.2, 8.6 | Complete |
| SO4 | Default family orders | Provisional pre-release ascending tuples are specified for descriptive resources, streams, Observations, Commands, status, results and events, all ending in RID and subject to required external-client validation. | §8.2 | Complete |
| SO5 | Paging stability | Every order is total and ends with immutable RID; cursors bind the order and last key. | §§8.2–8.4 | Complete |
| SO6 | Persistence/index handoff | Composite keyset indexes, family time indexes, collation and snapshot support are routed to storage topics. | §§11, 15 | Complete |

### A.5 Pagination Semantics

| ID | Plan question, shortened | Answer | Report location | Status |
|---|---|---|---|---|
| P1 | Required/inherited paging | Collection responses support `limit` and link-driven continuation; counts are optional but exact when supplied. | §§5.1, 6, 8.1–8.5 | Complete |
| P2 | Limit, offset, cursor, links and counts | Publish `limit` 1/10/10000, clamp higher valid values, require known `next`, omit public offset/cursor, always return `numberReturned`, conditionally exact `numberMatched`. | §§1.1, 8.3, 8.5 | Complete |
| P3 | Large/high-volume histories | Use keyset continuation, stable family orders, query budgets and snapshot/watermark support for repeatable mission/export cases. | §§8.2–8.4, 10.2, 11 | Complete |
| P4 | Sorting/filtering/auth/mutation interaction | The logical authorization-visible relation precedes predicates, ordering and page boundaries; continuations bind normalized query, pseudonymous security context, policy, representation and consistency state. | §§1.2, 8.3–8.5, 10.1 | Complete |
| P5 | Duplicate/missing/ambiguous continuation prevention | Stable unique tie-breakers, confidential/integrity-protected state, keyset traversal, no loops and snapshot mode address the risks; minimum mode declares its mutation caveat. | §§8.3–8.4, 14.2 | Complete |
| P6 | Storage handoff | Keyset indexes, MVCC/revision/high-watermark, count strategy and retention are routed to IDR-SRV-025/027/029. | §§11, 15 | Complete |

### A.6 Selection and Projection Semantics

| ID | Plan question, shortened | Answer | Report location | Status |
|---|---|---|---|---|
| SEL1 | Required/expected projection | No approved CSAPI field-selection or projection parameter exists. | §§5.4, 9 | Complete |
| SEL2 | Subsets, expansion, embedding, summaries | The initial profile rejects `select`/`fields`/`properties`/`expand`/`embed` and returns full resources. | §§7.8, 9 | Complete |
| SEL3 | Schema/representation interaction | Full schema-conformant selected representations preserve JSON, GeoJSON, SensorML and SWE validation; negotiation is handed to IDR-SRV-012. | §§9, 15 | Complete |
| SEL4 | Interoperability hazards | Ad hoc wire-field masks would vary by encoding, break required fields/generated clients and complicate authorization/cache identity. | §§9, 17.1 | Complete |
| SEL5 | Deferred behavior | Any logical field-selection profile needs a representation-independent vocabulary, schema/profile identity and separate security/conformance work. | §§9, 16 | Complete |

### A.7 Authorization, Security, and Policy Interaction

| ID | Plan question, shortened | Answer | Report location | Status |
|---|---|---|---|---|
| A1 | Authorization/releasability interaction | Authorization creates the candidate relation before route scope, predicates, order, counts and paging. | §§1.2, 10.1 | Complete |
| A2 | Queries spanning visible/hidden resources | Return only visible matches; valid absent and unauthorized identifiers are indistinguishable on list queries. | §§7.1, 10.1 | Complete |
| A3 | Metadata leakage | Counts, extents, page boundaries, links and timing operate on visible data; cursor and cache identity bind security context. | §§8.5, 10.1, 10.3 | Complete |
| A4 | Limits and resource consumption | Page, list, text, WKT, time, recursion, scan, time, memory, bytes, concurrency and rate dimensions receive explicit budgets. | §10.2 | Complete |
| A5 | Security-topic handoff | Threat, policy, releasability, cursor, cache, timing, logging and load/security tests are routed to 039/039A/040/055. | §§10, 15 | Complete |

### A.8 Error and Failure Semantics

| ID | Plan question, shortened | Answer | Report location | Status |
|---|---|---|---|---|
| E1 | Query error cases | Invalid, unknown, duplicate, incompatible and over-complex inputs are enumerated, including temporal, bbox/WKT, sorting/projection and token failures. | §§7.1, 10.2, 12 | Complete |
| E2 | Standard versus project errors | Inherited unknown/invalid parameter and limit rules are separated from Glaux token, budget and concealment policy. | §§3.4, 5.1, 12 | Complete |
| E3 | Error-taxonomy handoff | Status categories and required distinctions are supplied to IDR-SRV-013 without defining its final Problem Details schema. | §§12, 15 | Complete |
| E4 | Negative tests | Parser, predicate, route, paging, mutation, security, schema/OAD and fuzz negative cases are enumerated. | §14 | Complete |
| E5 | OpenAPI error documentation | OAD must match runtime parameters and describe grammars, limits and 400 behavior; final error schemas belong to 013/014. | §§12, 15 | Complete |

### A.9 Performance, Persistence, and Indexing Implications

| ID | Plan question, shortened | Answer | Report location | Status |
|---|---|---|---|---|
| PF1 | Common/critical/expensive queries | Discovery IDs/text, spatial/time windows, graph relationships, current status, exact counts, recursion and high-volume paging are identified. | §§10.2, 11 | Complete |
| PF2 | Index needs | Exact/prefix identity, normalized text, spatial, temporal, graph closure, capability, status and composite keyset needs are mapped. | §11 | Complete |
| PF3 | Specialized storage needs | Spatial, time-series, recursive relationship, full-text, normalized-property and snapshot capabilities are handed off without selecting a database. | §§11, 15 | Complete |
| PF4 | Limits/async/precompute/refusal | Ordinary queries stay synchronous/paged; semantics-preserving precompute is bounded; heavy jobs require a separate advertised service; over-budget work is refused honestly. | §§8.5, 10.2, 11.1 | Complete |
| PF5 | Category E/performance handoff | Concrete requirements are routed to 025–029 and 054, including indexes, time series, snapshots, counts and load/fuzz evidence. | §§11, 15 | Complete |

### A.10 Interoperability and Existing-Implementation Lessons

| ID | Plan question, shortened | Answer | Report location | Status |
|---|---|---|---|---|
| I1 | Existing implementation behavior | Pinned Go and Java implementations show divergent optional filtering, offset-link and count behavior; none overrides the standards. | §§3.3, 13.1 | Complete |
| I2 | Client smoke-test compatibility | Interoperable clients should inspect conformance and follow links; offset/cursor/UID conveniences need later adapter tests rather than automatic server adoption. | §§13.1, 13.5 | Complete |
| I3 | Explorer/external-client patterns | Explorer needs documented names, full resources and supplied paging links; its `limit=1000` fallback is not an exact-count contract. | §13.1 | Complete |
| I4 | Differences Glaux should account for | Strict, artifact-adapter, experimental-extension and external-client lanes isolate differences without corrupting conformance claims. | §§13.2–13.5 | Complete |
| I5 | Interoperability handoff | OAS-only filters, aliases, link/count behavior and client assumptions are routed to IDR-SRV-014A–014G and 056. | §§13.3–13.5, 15 | Complete |

---

## Appendix B. Compact Query Contract

| Contract area | Initial Glaux rule |
|---|---|
| Accepted inputs | Only QReg-declared parameters for the route; one occurrence; comma-separated list values; strict grammar and type validation |
| Predicate meaning | Authorization-visible route scope AND every named filter; OR among alternatives inside one list; valid no-match returns an empty collection |
| Text search | Unicode 17.0.0 `toNFKC_Casefold` on field and term, followed by locale-independent code-point substring containment; versioned Unicode data |
| Ordering | No client sort; fixed ascending family total order ending in immutable RID |
| Pagination | `limit` 1/10/10000; opaque `next`; keyset continuation; snapshot/watermark when repeatability is promised; no public `offset` or `cursor` |
| Counts | Exact `numberReturned` on JSON pages; exact visible `numberMatched` when safe and affordable, otherwise omitted |
| Selection | Full schema-conformant resources only; no field mask, expansion or summary query in the initial profile |
| Extensions | Sorting, CQL2 and projection require separately advertised, standards-aligned profiles and compatibility review |
| Safety | Authorization-first observable contract; confidential/integrity-protected pseudonymous principal/policy-bound continuations; bounded query cost; no hidden-resource leakage |

---

## Appendix C. Proposed Decision and Evidence Register

These are project recommendations, not OGC requirements. The Glaux Project Lead accepted them with the report on August 2, 2026.

| Decision | Proposed Glaux policy | Main evidence | Acceptance |
|---|---|---|---|
| P-011-01 | Implement both CSAPI Advanced Filtering classes completely before claiming them. | Accepted IDR-SRV-008 target; CSAPI Parts 1/2 | Accepted August 2, 2026 |
| P-011-02 | Publish `limit` minimum 1, default 10, maximum 10,000; clamp a larger valid integer and reject malformed/below-minimum values. | Tagged parameter plus inherited Features rule | Accepted August 2, 2026 |
| P-011-03 | Use approved RFC 3339/slash interval grammar; permit `latest` only for Observation `resultTime`, selecting the visible endpoint-wide maximum after other filters and retaining ties. | CSAPI requirements; #64/PR #88 disposition | Accepted August 2, 2026 |
| P-011-04 | Apply Unicode 17.0.0 `toNFKC_Casefold` to field and term, then locale-independent code-point substring containment over at least `name`/`description`; OR terms; version the Unicode data; no fuzzy or linguistic expansion. | Part 1 Requirement 40; Unicode Chapter 3/UAX #15; artifact wording conflict | Accepted August 2, 2026 |
| P-011-05 | Treat explicit root `parent` as a direct-child query independent of the root `recursive` default; keep all graph traversal cycle-safe and bounded. | Part 1 hierarchy requirements; tagged parameter intent | Accepted August 2, 2026 |
| P-011-06 | Adopt §8.2's ascending total orders as provisional pre-release planning defaults—UTC-instant comparison, null after non-null, immutable RID tie-break—and expose no baseline client sort; IDR-SRV-014A–014G may recommend a change before stable release. | Paging determinism; DDIL/replay analysis; absence of CSAPI sorting; #175 | Accepted August 2, 2026 |
| P-011-07 | Use random server-side references or authenticated-encrypted keyset continuations; permit MAC-only tokens only for non-sensitive/pseudonymous state; bind snapshot/watermark wherever repeatability is promised. | Features mutation caveat; confidentiality, integrity and consistency analysis | Accepted August 2, 2026 |
| P-011-08 | Treat authorization as the prior logical relation for predicates, order, counts, extents and pages; always include exact `numberReturned` in supported JSON envelopes and include exact visible `numberMatched` only when safe/affordable, otherwise omit it. | Security analysis; Features optional-count contract | Accepted August 2, 2026 |
| P-011-09 | Return complete resources and expose no baseline projection, expansion or summary parameter. | CSAPI/OAS inventory; schema/interoperability analysis | Accepted August 2, 2026 |
| P-011-10 | Defer client sorting and CQL2; if adopted later, align with OGC `sortby`/Sortables and explicit Features Filtering/CQL2 profiles. | Records, Common Part 3/Features Part 5, Features Part 3, CQL2; #175 | Accepted August 2, 2026 |
| P-011-11 | Generate runtime/OAD/tests from QReg and maintain an explicit overlay for tagged OAS omissions, extras and aliases. | Mechanical OAS/ATS/schema review; IDR-SRV-010A policy | Accepted August 2, 2026 |
| P-011-12 | Enforce typed query budgets for lists, text, WKT, time, recursion, work, memory, bytes, concurrency and rate without silently changing predicate meaning. | Security/performance analysis; RFC 6585 | Accepted August 2, 2026 |
| P-011-13 | Accept one occurrence of each QReg-declared parameter; return 400 for duplicates, malformed lists, undeclared names, and properties outside an IDR-SRV-015-approved simple-equality allowlist. | Features unknown-parameter behavior; tagged `form`/`explode=false`; parser/security analysis | Accepted August 2, 2026 |
| P-011-14 | Evaluate `bbox` and `geom` against every representation-independent geometry designated query-relevant in QReg, preserving their opposite no-geometry behavior; IDR-SRV-015/026 approve the set. | Features multi-geometry implementation choice; CSAPI `geom` | Accepted August 2, 2026 |
| P-011-15 | Provisionally bind CommandStatus `datetime`→`reportTime`, SystemEvent `datetime`→JSON `time`, and `eventType`→JSON `definition`; use bounded local-ID/UID/stored-href, cycle-safe FOI, normalized capability, and Deployment-validity relationship facts; track #165/#179. | Approved prose/schema conflicts; ATS; open issues #165/#179 | Accepted August 2, 2026 |
| P-011-16 | Keep `datetime` the sole canonical CommandStatus time query. `reportTime` is not baseline; if separately accepted later, make it a read-only, mutually exclusive, OAD-declared adapter excluded from CSAPI conformance claims. | Tagged OAS versus Part 2 Requirement 31; compatibility policy | Accepted August 2, 2026 |
| P-011-17 | Apply the normative Command query contract to the accepted Glaux root `/feasibility` list so it behaves consistently with nested and advertised typed Feasibility resources endpoints; document this as a project adapter, not an OGC obligation. | Accepted IDR-SRV-010 root route; Part 2 Requirements 36/39C; route-consistency analysis | Accepted August 2, 2026 |

---

## Appendix D. Completion and Workflow Handoff

### D.1 Completion Check

- [x] Exactly one research topic, IDR-SRV-011, was executed.
- [x] Prior report IDR-SRV-010A was accepted by the project lead before IDR-SRV-011 began.
- [x] All five core and 55 detailed research questions were answered.
- [x] All six methodology phases were completed.
- [x] All ten success criteria were met.
- [x] All nineteen required content areas and fourteen required query-matrix fields were supplied.
- [x] Approved standards, tagged OAS/schema/ATS artifacts, current official issues and pull requests, and bounded implementation/client evidence were reviewed.
- [x] Standards obligations, artifact observations, history, analysis and Glaux recommendations are visibly distinguished.
- [x] Evidence limitations and unresolved issues are recorded.
- [x] The deliverable was reviewed for coverage, traceability, consistency and Markdown integrity.
- [x] Proposed decisions P-011-01 through P-011-17 and the report were accepted by the Glaux Project Lead on August 2, 2026.

### D.2 Recorded Transition

The Glaux Project Lead's August 2, 2026 combined instruction accepted IDR-SRV-011 and authorized execution of exactly IDR-SRV-012. This acceptance record does not authorize IDR-SRV-013 or any later topic.

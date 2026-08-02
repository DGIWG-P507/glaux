# Section 010: Collections, Resources, Links, and Navigation Behavior - Research Report

**Topic ID:** IDR-SRV-010<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-010 Collections, Resources, Links, and Navigation Behavior](../IDR%20Plans/idr-srv-010-collections-resources-links-and-navigation-behavior.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core and 41 detailed questions; all six methodology phases, ten success criteria, fifteen required content areas, and thirteen minimum behavior-matrix fields are validated<br>
**Methodology Used:** Reconciliation of accepted IDR-SRV-006 through IDR-SRV-009 with the approved CSAPI Parts 1 and 2 and OGC API - Features standards; normative abstract tests and schemas; tagged OpenAPI and example artifacts; bounded official issue, pull-request, and release history; Web-linking authorities; and pinned implementation, client, and test evidence<br>
**Research Time:** Approximately 5 hours of AI-assisted elapsed execution time, including three parallel independent read-only standards, artifact/history, and interoperability audits, on August 1, 2026<br>
**Primary Sources:**

- OGC 23-001, *OGC API - Connected Systems - Part 1: Feature Resources*, Version 1.0
- OGC 23-002, *OGC API - Connected Systems - Part 2: Dynamic Data*, Version 1.0
- OGC 17-069r4, *OGC API - Features - Part 1: Core*, Version 1.0 with Corrigendum
- RFC 8288, *Web Linking*, and the IANA Link Relation Types registry

**Supporting Resources:** Accepted IDR-SRV-001 through IDR-SRV-009 reports; official CSAPI tag `v1.0.0`; versioned OGC schemas; tagged modular OAS and examples; bounded official maintenance history; official Features executable tests; CSAPI Explorer; and pinned implementation evidence<br>
**Document Purpose:** Establish the server-visible collection, endpoint, link, canonical-identity, and traversal contract that the Rust Glaux reference server and its downstream resource, URI, relationship, validation, persistence, and test designs must implement<br>
**Author:** OpenAI Codex, with independent read-only Part 1, Part 2, and artifact/history/interoperability audits<br>
**Accepted By:** TBD pending Glaux Project Lead review<br>
**Acceptance Date:** TBD pending acceptance<br>
**Date:** August 1, 2026<br>
**Last Updated:** August 1, 2026

---

## 1. Executive Summary

### 1.1 What the Server Must Make Navigable

Glaux cannot be merely a database with CSAPI-shaped JSON. A client must be able to start at the API root, follow the landing-page `data` link to `/collections`, discover typed collections, retrieve a collection's items, open an item, recover that resource's canonical URL, and then traverse the relationships that make the connected-system model useful.

The approved standards establish four overlapping views:

1. **Canonical type endpoints** such as `/systems`, `/datastreams`, and their `/{id}` item URLs expose the type-scoped canonical view and the canonical identity of each resource. When hierarchy classes apply, `/systems` and `/deployments` return top-level members by default; `recursive=true` exposes the complete hierarchy.
2. **Typed OGC collections** under `/collections/{collectionId}` group resources for discovery or a particular purpose. One resource may belong to multiple collections.
3. **Nested relationship endpoints** such as `/systems/{id}/subsystems`, `/datastreams/{id}/observations`, and `/commands/{id}/status` expose relationship-filtered views.
4. **Links in representations and HTTP headers** connect the current representation to its canonical resource, collection, alternate representations, schemas, parents, children, streams, observations, commands, results, events, and other related resources.

These are views of one resource graph, not separate copies. A System retrieved from a typed collection or nested endpoint remains the System at `/systems/{id}`. CSAPI therefore requires canonical links on noncanonical retrievals and, when the applicable transaction class is implemented, requires canonical changes to be visible through every collection membership.

### 1.2 Principal Standards Findings

1. **The inherited collection spine is mandatory, with one bounded adaptation question.** OGC API - Features requires `GET /collections`, `GET /collections/{collectionId}`, `GET /collections/{collectionId}/items`, and an individual item route for feature collections; Part 2 Requirement 2 imports the resource analogue. Part 1 clearly adapts collection-items behavior for Property but is less explicit about its individual non-feature collection route, so Glaux adopts that route as a documented project interpretation. Named metadata fields must match between aggregate and detail when present, and every aggregate-entry link must appear in detail; the detail response may add links.
2. **Part 1 requires typed collections.** For every supported Part 1 family, at least one advertised collection is required: System, Deployment, Procedure, Sampling Feature, and Property. The first four are feature collections with exact `featureType` markers; Property is a non-feature resource collection.
3. **Part 2 typed collections are conditional.** DataStream, Observation, ControlStream, Command, Feasibility, and SystemEvent collections are optional to expose, but once exposed their exact `itemType` and item-endpoint behavior are mandatory.
4. **Canonical endpoints are the identity backbone.** Part 1 defines five canonical families. Part 2 defines six in its common endpoint inventory, although Feasibility lacks the same numbered canonical-list requirement as the other five. Command Status, Command Result, and stream schema resources remain subordinate rather than independent top-level families.
5. **Nested endpoints are relationship views.** They must contain only members related to the parent named in the path. Hierarchical System and Deployment endpoints also have defined direct-versus-recursive behavior.
6. **The basic Web links are precise.** `self`, `alternate`, `items`, `collection`, `canonical`, `next`, `describedby`, and `license` have distinct meanings. Applicable inherited Features links require `rel` and `type`; payload links should usually be repeated in HTTP `Link` headers.
7. **CSAPI defines a specialized association vocabulary.** Part 1 Table 3 lists `ogc-rel:` values for parent/child, sampling, deployment, procedure, stream, and tasking relationships. They are unregistered application-specific extension relation URIs using the `ogc-rel` URI scheme, not IANA-registered relation tokens. Glaux must preserve the published lexical values and map them explicitly rather than guessing from internal field names.
8. **The released relation evidence conflicts.** Encoding requirements say to use the association name; incorporated mapping-table footnotes direct `ogc-rel:` prefixes; tagged examples contain unprefixed values; and post-release PR #176 fixes only part of those examples. The relation table also contains lexical-case and model-coverage inconsistencies. These are controlled defects, not permission to discard evidence or treat bare relations as normative; RFC case-insensitive comparison still applies.
9. **Part 2 contains route contradictions.** Several numbered requirements use singular `/controlstream` or `/command`, while the endpoint overview, other requirements, abstract tests, and tagged OAS use plural `/controlstreams` and `/commands`. The System Event requirement uses `/systems/{id}/events`, while one abstract test says `systemEvents`. The tagged OAS also omits Feasibility and Deployment-scoped routes and contains obsolete System History routes.
10. **The released OAS and examples are evidence, not the contract.** They are useful for finding implementation traps but are incomplete and sometimes contradict the approved text. Glaux must generate its API surface from the reconciled standards baseline.
11. **Stream schemas are first-class navigation targets.** `/datastreams/{id}/schema` and `/controlstreams/{id}/schema` tell a client how to interpret observation results and command parameters; they cannot be treated as incidental documentation.
12. **The standards do not settle every operational policy.** Authorization-sensitive links, moved/deleted resource handling, tombstones, external-link health, alias lifetime, transaction boundaries, and reverse-link materialization remain project decisions owned by later topics.

### 1.3 Recommended Baseline

Accept a single, typed **navigation registry** as the downstream planning baseline. It should describe every resource family, canonical and nested route, collection marker, link relation, related family, capability prerequisite, representation, and conformance/test anchor. Rust handlers, payload links, HTTP `Link` headers, OpenAPI paths, collection metadata, authorization filtering, and tests should all project from that registry and the same underlying resource graph.

For published Glaux URLs:

- use the plural canonical and nested paths consistently;
- advertise only canonical plural paths in links and OpenAPI;
- treat singular forms and the `events`/`systemEvents` conflict as explicit compatibility-test adapters pending IDR-SRV-010A, not alternate identities;
- expose a root `/feasibility` list as a Glaux interoperability baseline because the standard's common endpoint inventory and tests assume canonical discovery, while recording the missing numbered list requirement;
- provide at least one collection for every Part 1 family and, for a best-of-breed reference implementation, one clearly named default collection for every top-level Part 2 family;
- emit the Table 3 `ogc-rel:` spellings deterministically, compare extension relation URIs case-insensitively as RFC 8288 requires, keep unprefixed variants out of ordinary output, and preserve controlled bare-value fixtures/adapters until authoritative maintenance resolves the remaining conflicts; and
- reject deployment when routes, collection descriptors, links, OAS, and conformance evidence disagree.

Acceptance of this report establishes a planning baseline. It does not finalize internal Rust domain types, identifier generation, relationship storage, query algorithms, content negotiation, error bodies, database design, authorization policy, or alias/deprecation lifetime.

---

## 2. Scope and Plan Alignment

### 2.1 In Scope and Completed

- Collection aggregate, collection-detail, collection-items, and item behavior inherited from OGC API - Features.
- Part 1 and Part 2 canonical, nested, subordinate, relationship, schema, and typed-collection endpoint inventory.
- Required, recommended, conventional, and project-selected link relations and navigation paths.
- Root-to-collection, collection-to-item, item-to-canonical, and cross-resource traversal.
- Part 1-to-Part 2 relationships across systems, deployments, sampling features, procedures, properties, streams, observations, commands, status, results, feasibility, and events.
- Identity, URI, lifecycle, validation, consistency, persistence, and transaction implications without finalizing their designs.
- Tagged OAS/schema/example review and bounded relevant official issue/PR history.
- Supporting implementation, CSAPI Explorer, OGC tooling, and executable-test implications.
- Positive, negative, invariant, contradiction-adapter, and interoperability test needs.

### 2.2 Explicitly Out of Scope

- Final Rust resource/domain model: IDR-SRV-015.
- Identifier allocation, URI construction policy, aliases, tombstones, and lifecycle: IDR-SRV-016.
- Stored relationship representation and complete forward/reverse-link rules: IDR-SRV-017.
- Query, filtering, sorting, paging implementation, and selection semantics: IDR-SRV-011.
- Media types and content negotiation: IDR-SRV-012.
- Complete error representations and status mapping: IDR-SRV-013.
- OAS generation and documentation architecture: IDR-SRV-014.
- Schema resolver/validation architecture: IDR-SRV-023.
- Database schema, indexes, and transaction boundaries: IDR-SRV-025 through 029.
- Full implementation-specific comparisons: IDR-SRV-014A through 014G.
- Authorization, policy filtering, and security controls: IDR-SRV-039 through 041.
- Writing Rust server code or asserting that a Glaux implementation already conforms.

### 2.3 Core Research-Question Coverage

| ID | Core question | Status | Primary evidence |
|---|---|---|---|
| CQ1 | Required Part 1/Part 2 collection and resource behavior | Complete | §§5–7; Appendix B |
| CQ2 | Links and traversal needed for client discovery | Complete | §§7–9 |
| CQ3 | Part 1 feature resources and Part 2 dynamic-data relationships | Complete | §§7–9; Appendix B |
| CQ4 | Required, inherited, conventional, and project-choice distinctions | Complete | §§3–5, 7, 11, 13 |
| CQ5 | Downstream design, validation, interoperability, and test implications | Complete | §§11–17 |

Appendix A maps all 41 detailed questions individually. Appendix E validates every methodology phase, success criterion, content requirement, and required matrix field.

### 2.4 Reconciliation with Accepted Prior Research

IDR-SRV-006 supplies the accepted Part 1 route, collection, identity, hierarchy, relation, mutation, and encoding requirements. IDR-SRV-007 supplies the corresponding Part 2 dynamic-data baseline and records the singular/plural, Feasibility, System Event, OAS, and schema defects. IDR-SRV-008 controls conformance-class selection and evidence gating. IDR-SRV-009 establishes one combined API root whose `data` link leads to `/collections`, plus one capability registry for routes, OAS, conformance, and discovery.

This report does not reopen those conclusions. It extends the IDR-SRV-009 capability registry one level deeper into an explicit resource/navigation graph and assigns unresolved model or policy questions to their owning later topics.

---

## 3. Evidence Base and Authority Classification

### 3.1 Primary Sources Reviewed

| Source | Version / pin | Authority | Stable anchors used | Access | Availability / limitations |
|---|---|---|---|---|---|
| [OGC 23-001, CSAPI Part 1](https://docs.ogc.org/is/23-001/23-001.html) | Version 1.0; approved; HTML SHA-256 `555c4b3bea06ab91b980bdaa3c99d265e6718dbad943ca1cbec39fbbf283c92a` | Controlling normative CSAPI source | §§7.5–7.9, 8.4–8.5, 9–15, 17–19; Requirements 1–37, 63–91; Annex A | 2026-08-01 | Contains material link-relation and ATS/artifact conflicts |
| [OGC 23-002, CSAPI Part 2](https://docs.ogc.org/is/23-002/23-002.html) | Version 1.0; approved; HTML SHA-256 `e840613693c282a41b1dda709eb266905683697fb430168ff348833e8f50df5e` | Controlling normative CSAPI source | §§7.4, 8.3, 9–12, 16; Requirements 3–44, 95–106; Annex A | 2026-08-01 | Contains singular/plural, Feasibility, event-route, and ATS inconsistencies |
| [OGC 17-069r4, OGC API - Features Part 1](https://docs.ogc.org/is/17-069r4/17-069r4.html) | Version 1.0.1 corrigendum; source tag `1.0.1` at `4ff19a5734578cf1f815d03ab192e8e0dc407e9f` | Inherited normative source | §§7.12–7.16; Requirements 12–35; Annex A.2 | 2026-08-01 | CSAPI adapts feature wording for non-feature resources |
| [RFC 8288, Web Linking](https://www.rfc-editor.org/rfc/rfc8288) | Standards Track, October 2017 | Normative Web-linking context | §§2.1, 2.1.1, 2.1.2, 3 | 2026-08-01 | Does not define the CSAPI extension-relation URI vocabulary |
| [IANA Link Relation Types](https://www.iana.org/assignments/link-relations/link-relations.xhtml) | Registry last updated 2026-06-12 | Authoritative registered-token registry | Selected anchors: `alternate`, `canonical`, `collection`, `describedby`, `item`, `license`, `next`, `prev`, `self`, `service-desc`, `service-doc`, `status` | 2026-08-01 | OGC's `items`, `data`, and `conformance` are not IANA-registered tokens; `ogc-rel:` values are unregistered extension relation URIs |
| [Versioned OGC API - Features schemas](https://schemas.opengis.net/ogcapi/features/part1/1.0/openapi/schemas/) | Part 1 Version 1.0 schema tree | Normative where incorporated | `collections.yaml`, `collection.yaml`, `featureCollectionGeoJSON.yaml`, `featureGeoJSON.yaml`, `link.yaml` | 2026-08-01 | Structure alone cannot prove semantic link or membership correctness |

### 3.2 Official Artifact and Maintenance Evidence

| Source | Pin / state | Evidence class and use | Limitation |
|---|---|---|---|
| [Official CSAPI repository at release](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2) | `v1.0.0`, commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2` | Exact published-support source, OAS, schemas, examples, ATS source | Supporting artifacts cannot override approved normative text |
| [Part 1 modular OAS](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/openapi-connectedsystems-1.yaml) | SHA-256 `745fc5357a173e127f174f7413f36bc22b55f1b7cfa560902c63bb67dd82ea78` | Mechanical route and example comparison | Incomplete/defective example API, not a conformance contract |
| [Part 2 modular OAS](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/openapi-connectedsystems-2.yaml) | SHA-256 `2df6c9b48bc19a21f0b44219b947a0d3be29e76f842cb9bb106c3cf7a5c9dd82` | Mechanical route, schema, and omission comparison | Omits normative surface and includes excluded System History paths |
| [Part 1 released OAS bundle](https://github.com/opengeospatial/ogcapi-connected-systems/releases/download/v1.0.0/ogcapi-connectedsystems-1.bundled.oas31.yaml) | SHA-256 `69da631d5d05f01716381cca7b7ee6311402f2752a8fd79a9b72b663539555aa` | Release-asset dependency-closure comparison | Retains 32 relative example references; not standalone |
| [Part 2 released OAS bundle](https://github.com/opengeospatial/ogcapi-connected-systems/releases/download/v1.0.0/ogcapi-connectedsystems-2.bundled.oas31.yaml) | SHA-256 `86ed005f9e7cf176264d6deb72581a0b521a227cd7a198b6cb1bd32b39d83667` | Release-asset dependency-closure comparison | Retains 51 relative references; not standalone |
| [Tagged shared Link schema](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/common/link.json) | Release commit `8e03b236...`; SHA-256 `61660fcb768672af3221167d80f1f853a88f66ac7665affefc5db21bd7f50889` | Exact structural Link contract comparison | Schema permissiveness cannot prove endpoint-specific relation semantics |
| [Upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md) | Version 1.3, refreshed 2026-08-01 | Bounded dispositions including issues #2, #4, #22, #30, #47, #62, #91, #141, #149, #164, #165, #169, #173, #177 and linked changes | Maintenance evidence; never silently amends Version 1.0 |
| [Current official branch](https://github.com/opengeospatial/ogcapi-connected-systems/commit/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f) | `master` at `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f`; accessed 2026-08-01 | Confirms post-release example maintenance and residual defects | Mutable; retrieval-date pin required |

### 3.3 Supporting Implementation, Client, and Test Evidence

| Source | Pin | Relevance | Limitation |
|---|---|---|---|
| [Official OGC API - Features ETS](https://github.com/opengeospatial/ets-ogcapi-features10/tree/a314c1e6a9278b14ab9a2ed865cfe36d202f0125) | `a314c1e6a9278b14ab9a2ed865cfe36d202f0125` | Executable inherited collection/detail/items/item discovery checks | Not a CSAPI ETS |
| [CSAPI Explorer](https://github.com/OS4CSAPI/ogc-csapi-explorer/tree/00f1c188e05738ee03390fd95f09d351e073a9c3) | `00f1c188e05738ee03390fd95f09d351e073a9c3`; inspected [`helpers.ts`](https://github.com/OS4CSAPI/ogc-csapi-explorer/blob/00f1c188e05738ee03390fd95f09d351e073a9c3/src/ogc-api/csapi/helpers.ts), [`url_builder.ts`](https://github.com/OS4CSAPI/ogc-csapi-explorer/blob/00f1c188e05738ee03390fd95f09d351e073a9c3/src/ogc-api/csapi/url_builder.ts), and [`command-routing.ts`](https://github.com/OS4CSAPI/ogc-csapi-explorer/blob/00f1c188e05738ee03390fd95f09d351e073a9c3/src/ogc-api/csapi/command-routing.ts) | Real client link discovery, collection navigation, and tolerance evidence | Client behavior is informative, not normative |
| [connected-systems-go](https://github.com/OS4CSAPI/connected-systems-go/tree/e900da88738cca92872038b703c4ad537fc0c8fd) | `e900da88738cca92872038b703c4ad537fc0c8fd`; inspected [`router.go`](https://github.com/OS4CSAPI/connected-systems-go/blob/e900da88738cca92872038b703c4ad537fc0c8fd/internal/api/router.go), [`feature_handler.go`](https://github.com/OS4CSAPI/connected-systems-go/blob/e900da88738cca92872038b703c4ad537fc0c8fd/internal/api/feature_handler.go), and [`association_links.go`](https://github.com/OS4CSAPI/connected-systems-go/blob/e900da88738cca92872038b703c4ad537fc0c8fd/internal/model/formaters/association_links.go) | Existing route/link construction patterns and compatibility risks | Implementation precedent only; Glaux remains Rust |
| [OpenSensorHub core](https://github.com/OS4CSAPI/osh-core/tree/b2badae59aaa78455c5638ad73b452ccdee40207) | `b2badae59aaa78455c5638ad73b452ccdee40207`; inspected [`ConSysApiService.java`](https://github.com/OS4CSAPI/osh-core/blob/b2badae59aaa78455c5638ad73b452ccdee40207/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/ConSysApiService.java), [`SystemAssocs.java`](https://github.com/OS4CSAPI/osh-core/blob/b2badae59aaa78455c5638ad73b452ccdee40207/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/system/SystemAssocs.java), and [`CollectionHandler.java`](https://github.com/OS4CSAPI/osh-core/blob/b2badae59aaa78455c5638ad73b452ccdee40207/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/home/CollectionHandler.java) | Mature CSAPI server organization and traversal precedent | Implementation precedent only; live deployments may differ |
| [Restricted SECD interoperability evidence](https://github.com/Sam-Bolling/csapi-server-interop-secd/tree/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0) | `f018fd129bf0d0d1ce75e68198e3ab4d99d937a0`; authenticated workspace access | Preserved client/server HTTP evidence and navigation failure modes | Private/restricted and not publicly reproducible; deployment-specific, informative evidence only |
| [OS4CSAPI testing corpus](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing) | `754411897173c2ec4debaa9bcf4ed9e0f8a9e230` | Inventory, fixture, traceability, and scenario method | Research-method exemplar, not technical authority |

### 3.4 Authority and Classification Rules

1. Approved normative requirements and their incorporated schemas control server obligations.
2. An inherited Features requirement is normative when the applicable CSAPI requirement/class invokes it.
3. Normative prose and endpoint inventories help interpret numbered requirements but are not silently promoted into missing `SHALL` statements.
4. The normative ATS is conformance evidence, but a test defect does not rewrite its requirement.
5. Tagged OAS, schemas not incorporated by a requirement, examples, and release packaging are supporting evidence.
6. Closed issues, merged PRs, and current-branch commits explain intent or maintenance state; only a new controlling publication changes the approved release.
7. Implementation and client behavior can justify robustness or compatibility recommendations, never a standards obligation.
8. This report labels Glaux choices explicitly and hands designs outside Topic 010 to their owning plans.

### 3.5 Evidence Limitations and Conflicts

- No official public executable CSAPI ETS was found. The standards' abstract tests and future Glaux supplemental harness remain necessary.
- The released OAS descriptions are incomplete and not reference-closed; no conclusion assumes that OAS path presence equals a requirement or that absence cancels one.
- Current-branch changes after `v1.0.0` remain maintenance evidence only.
- Some association semantics name a relationship without prescribing one exact target URL. The later relationship model must choose the target while preserving the published relation meaning.
- Live external links and deployments are mutable. Pinned source and preserved evidence were preferred.
- The SECD repository was available only through the authenticated project workspace and returned public 404; its observations are disclosed as restricted supporting evidence and are never the sole basis for a standards or project recommendation.
- Detailed alias lifetime, authorization-sensitive discovery, tombstone/move behavior, link health, and transaction policy remain unresolved because their controlling topic has not yet run.

---

## 4. Research Method and Reconciliation

### 4.1 Six-Phase Execution

| Plan phase | Work completed | Output | Status |
|---|---|---|---|
| 1. Source collection/framework | Pinned controlling and mutable sources; accepted prerequisites reconciled; 13-field extraction model defined | Evidence inventory and authority rules in §3 | Complete |
| 2. Collections/endpoints | Extracted inherited collection spine and all Part 1/Part 2 canonical, nested, subordinate, schema, and collection patterns | §§5–7; Appendix B | Complete |
| 3. Links/navigation | Classified registered, OGC-defined, CSAPI-specific, recommended, and project links; mapped traversal and conflicts | §§7–10; Appendices B–C | Complete |
| 4. Identity/URI/lifecycle | Isolated canonical-identity and multi-view invariants; bounded aliases, deletion, moved/stale, and external-resource questions | §10 | Complete |
| 5. Validation/interoperability/tests | Compared normative text, ATS, OAS, examples, clients, implementations, and failure modes | §§11–12, 15; Appendix D | Complete |
| 6. Synthesis | Produced recommendations, decision analysis, risks, handoffs, and exhaustive plan validation | §§13–18; Appendices A and E | Complete |

### 4.2 Extraction Framework

Each behavior was recorded with the plan's thirteen required fields: family; endpoint/pattern; source/anchor; rule summary; classification; links; related resources; Glaux implication; conformance/requirement; schema/OAS artifact; test implication; downstream handoff; and unresolved notes. Appendix B is the consolidated matrix.

Classifications used below are:

- **N** — normative approved requirement or directly inherited obligation;
- **R** — approved recommendation;
- **C** — published convention or informative standard text;
- **A** — analyst interpretation needed to reconcile evidence;
- **P** — Glaux project recommendation or choice;
- **D** — documented defect/contradiction requiring an adapter, negative fixture, or upstream watch.

### 4.3 Conflict-Reconciliation Rule

Where a numbered requirement, endpoint overview, ATS, OAS, schema, example, or issue disagrees, this report retains every source and its authority. It recommends one primary Glaux behavior only where a coherent server needs one, records the competing form as a test case, and assigns compatibility/lifecycle policy to IDR-SRV-010A or the other owning topic. This prevents an AI implementer from “fixing” a contradiction by losing evidence.

---

## 5. Collection and Collection-Metadata Findings

### 5.1 Inherited Collection Spine

The Part 1 API Common class inherits OGC API - Features Core through Part 1 §7.5; its family collection requirements (Requirements 8, 18, 28, 33, and 37) apply the collection behavior to each Part 1 family. Part 2 Requirement 2 explicitly incorporates Features §§7.14–7.16 for arbitrary resource collections, and Part 2 §8.3 states the terminology adaptation. The resulting minimum discoverable spine is:

| Operation | Server obligation | Key response behavior | Source |
|---|---|---|---|
| `GET /collections` | Return the advertised collection inventory | HTTP 200; top-level `links` and `collections`; `self` and every `alternate`; `rel` and `type` on those links | [Features Requirements 12–17](https://docs.ogc.org/is/17-069r4/17-069r4.html#_feature_collections) |
| `GET /collections/{collectionId}` | Return every advertised collection individually | HTTP 200; each named metadata field is identical when present, and every aggregate-entry link is included; detail may add links | [Features Requirements 18–19](https://docs.ogc.org/is/17-069r4/17-069r4.html#_feature_collection) |
| `GET /collections/{collectionId}/items` | Return the items in every advertised collection | HTTP 200; applicable collection query and response rules; `self`, alternates, correct link types, and paging/count invariants | [Features Requirements 20–32](https://docs.ogc.org/is/17-069r4/17-069r4.html#_features) |
| `GET /collections/{collectionId}/items/{itemId}` | Return an advertised feature item; Part 2 imports the resource analogue, while Glaux applies it to Part 1 Property as a project interpretation | HTTP 200; `self`, alternates, and `collection`; CSAPI adds `canonical` outside the canonical item URL | [Features Requirements 33–35](https://docs.ogc.org/is/17-069r4/17-069r4.html#_feature), CSAPI Part 1 §7.6.3, and Part 2 Requirement 2 |

Unknown collection or item identifiers produce 404. Invalid inherited `limit`, `bbox`, or `datetime` values produce 400 where those parameters apply. Detailed parameter selection belongs to IDR-SRV-011 and final error representations to IDR-SRV-013.

Features permits `/collections` to advertise only a selection when the server has many collections. Glaux's complete configured collection inventory is therefore a project policy for reference-server discoverability, not an inherited `SHALL` that every possible collection must be listed.

Part 1's Property resource is not a feature. The common adaptation clearly requires its collection-items behavior, but the publication is less explicit about the individual non-feature route at `/collections/{id}/items/{itemId}`. Glaux should implement that route as the coherent resource analogue, return the Property's canonical link, and record it as a project interpretation in the conformance adapter. Part 2's individual resource route is not merely analogous: Requirement 2 normatively incorporates Features §§7.14–7.16 for every exposed Part 2 collection.

### 5.2 Collection Object Contract

The inherited schema requires each collection object to contain `id` and `links`. The standard also defines optional `title`, `description`, and `extent`; CSAPI adds exact type markers. The operational contract is stronger than schema validation:

- `id` is the path identifier and must be stable enough for its published URI.
- every supported items representation has an `items` link with correct `href`, `rel`, and `type`;
- a collection `self` link is formally recommended because of a Features corrigendum compatibility constraint, but should be ordinary Glaux output;
- `alternate`, `describedby`, and `license` links are included as applicable;
- supplied spatial/temporal extents cover the collection rather than merely the current page;
- aggregate and detail views are generated from the same descriptor so named fields match when present and every aggregate link appears in detail, while permitting detail-only links; and
- `featureType` must also remain consistent in Glaux even though the inherited detail-consistency sentence names `itemType` but not the CSAPI extension field.

### 5.3 Part 1 Required Typed Collections

For every Part 1 resource class Glaux implements, the class requires at least one matching collection. The accepted all-class target therefore requires all five:

| Family | Minimum availability | Exact `itemType` | Exact `featureType` | Collection items behave as |
|---|---|---|---|---|
| System | At least one | `feature` | `sosa:System` | System resources endpoint |
| Deployment | At least one | `feature` | `sosa:Deployment` | Deployment resources endpoint |
| Procedure | At least one | `feature` | `sosa:Procedure` | Procedure resources endpoint |
| Sampling Feature | At least one | `feature` | `sosa:Sample` | Sampling Feature resources endpoint |
| Property | At least one | `sosa:Property` | Not applicable | Property resources endpoint |

These are case-sensitive wire values. Tagged examples that use `ssn:System` or `ssn:Deployment` are wrong and belong in the negative-fixture corpus.

### 5.4 Part 2 Conditional Typed Collections

Part 2 permits any number—including zero—of arbitrary collections for each dynamic-data family. If one is exposed, the exact marker and endpoint behavior become normative:

| Family | Exact `itemType` | Items endpoint contract | Availability |
|---|---|---|---|
| DataStream | `DataStream` | DataStream resources endpoint | Optional to advertise; strict if present |
| Observation | `Observation` | Observation resources endpoint | Optional to advertise; strict if present |
| ControlStream | `ControlStream` | ControlStream resources endpoint | Optional to advertise; strict if present |
| Command | `Command` | Command resources endpoint | Optional to advertise; strict if present |
| Feasibility | `Feasibility` | Command resources endpoint adapted for Feasibility | Optional to advertise; strict if present |
| SystemEvent | `SystemEvent` | System Event resources endpoint | Optional to advertise; strict if present |

CommandStatus, CommandResult, observation/command schema resources, and operational “status” are not generic top-level collection families.

**Project recommendation:** advertise one plainly named default collection for each of the six top-level Part 2 families. This is not needed merely to satisfy each conditional collection requirement, but it gives generic clients a complete `/collections` inventory and avoids requiring prior route knowledge.

### 5.5 Membership and Overlap

CSAPI expressly allows a resource to appear in multiple, overlapping collections. Canonical type endpoints expose the canonical type-scoped view, subject to the top-level-default hierarchy rules for Systems and Deployments; arbitrary collections expose selected memberships; nested endpoints expose relationship-derived memberships. Therefore:

- membership is not identity;
- the same local resource ID remains stable across all same-type collections;
- each noncanonical item representation points to its canonical item;
- a canonical update is visible in every view when the applicable transaction class is implemented;
- deletion from a custom collection removes membership, while deletion at the canonical resource removes the resource and all memberships when the applicable transaction class is implemented; and
- paging links apply to a particular selection/view, not to the canonical identity.

Collection criteria may be arbitrary. Glaux should describe them clearly enough that a client can understand the view; this is a project recommendation because Features does not prescribe a criteria-description field. The final membership write policy and transaction semantics remain IDR-SRV-016, 017, 029, and 031 work.

---

## 6. Resource and Endpoint Findings

### 6.1 Endpoint Taxonomy

CSAPI Part 1 §7.6 distinguishes:

- a **resources endpoint**, which returns a set;
- a **resource endpoint**, which returns one item;
- a **canonical resources endpoint**, which returns the canonical type-scoped set, subject to a family's hierarchy/query rules;
- a **canonical resource endpoint**, which gives one item its preferred API URL;
- a **nested resources endpoint**, which returns the members related to a parent; and
- a **collection items endpoint**, which returns an advertised grouping.

This vocabulary matters. `/datastreams` and `/collections/default_datastreams/items` may select the same current objects, but they are distinct Web resources and links must identify the correct context.

### 6.2 Part 1 Canonical and Nested Inventory

| Family / category | Canonical set and item | Nested or relationship endpoint | Obligation / condition | Primary requirement anchors |
|---|---|---|---|---|
| System · descriptive feature | `/systems`; `/systems/{id}` | — | Required when System class selected; default list is top-level only if Subsystem class applies | [Reqs 5–8](https://docs.ogc.org/is/23-001/23-001.html#_req_system_canonical-url) |
| Subsystem · descriptive feature | Same System identity | `/systems/{parentId}/subsystems` | Required when Subsystem class selected; direct children by default, all descendants with `recursive=true` | Reqs 9–13 |
| Deployment · descriptive feature | `/deployments`; `/deployments/{id}` | `/systems/{sysId}/deployments` | Canonical routes required for class; when Subdeployment applies, default list is top-level only and `recursive=true` returns the complete hierarchy; System-scoped route when the association is provided | Reqs 14–18 |
| Subdeployment · descriptive feature | Same Deployment identity | `/deployments/{parentId}/subdeployments` | Required when Subdeployment class selected; direct/all-descendant semantics parallel Systems | Reqs 19–23 |
| Procedure · descriptive feature without geometry | `/procedures`; `/procedures/{id}` | No fixed association path | Required for Procedure class | Reqs 25–28 |
| Sampling Feature · descriptive feature | `/samplingFeatures`; `/samplingFeatures/{id}` | `/systems/{sysId}/samplingFeatures` | Canonical routes plus parent-System endpoint; every System has the scoped route when SF class applies | Reqs 29–33 |
| Property · supporting non-feature metadata | `/properties`; `/properties/{id}` | No fixed association path | Required for Property class | Reqs 34–37 |

When the Subsystem class applies, every System's subsystem collection includes all permanently attached subsystems. Requirement 9 also uses the undefined uppercase keyword `CAN` for currently deployed subsystems; Recommendation `/rec/subsystem/collection-datetime` supplies the usable policy: omitted, instant, and interval `datetime` values select current, point-in-time, and interval-associated membership respectively. Glaux should implement that recommendation and retain the publication ambiguity in its traceability evidence.

Part 1 Requirements 13 and 23 also define recursive association content, not just recursive child lists. A System with subsystems must have `samplingFeatures`, `datastreams`, and `controlstreams` association target sets covering that System and every descendant recursively. A Deployment with subdeployments must aggregate `deployedSystems`, `samplingFeatures`, `featuresOfInterest`, `datastreams`, and `controlstreams` across that Deployment and every descendant recursively.

The tagged Part 1 OAS contains 20 paths in total: 14 Connected Systems family/nested paths plus six root, conformance, and collection paths. It contains no nested item route such as `/systems/{id}/subsystems/{childId}`; a client retrieves a child from the nested list, then follows its `canonical` link.

### 6.3 Part 2 Canonical, Nested, and Subordinate Inventory

| Family / category | Canonical set and item | Required/conditional related endpoints | Obligation and defect note | Primary anchors |
|---|---|---|---|---|
| DataStream · dynamic-data description | `/datastreams`; `/datastreams/{id}` | `/systems/{sysId}/datastreams`; `/deployments/{depId}/datastreams`; `/datastreams/{id}/samplingFeatures`; `/datastreams/{id}/featuresOfInterest`; `/datastreams/{id}/schema`; `/datastreams/{id}/observations` | Canonical routes and schema required; scoped routes have stated Part 1/association/local-target conditions | [Reqs 3–11](https://docs.ogc.org/is/23-002/23-002.html#_req_datastream) |
| Observation · time-varying data record | `/observations`; `/observations/{id}` | `/datastreams/{dsId}/observations`; transaction-qualified `.../{obsId}` | Canonical and parent-stream lists required; nested item appears in transaction text but not tagged OAS | Reqs 12–16 |
| ControlStream · control-schema description | `/controlstreams`; intended `/controlstreams/{id}` | `/systems/{sysId}/controlstreams`; `/deployments/{depId}/controlstreams`; `/controlstreams/{id}/samplingFeatures`; `.../featuresOfInterest`; `.../schema`; `.../commands` | Numbered canonical-item requirement incorrectly says `/controls/{id}`; several nested requirements have copied/singular defects | [Reqs 17–25](https://docs.ogc.org/is/23-002/23-002.html#_req_controlstream) |
| Command · task/control record | `/commands`; `/commands/{id}` | intended `/controlstreams/{csId}/commands`; `/commands/{id}/status`; `/commands/{id}/result` | Canonical routes required; numbered nested/status/result clauses use singular parent segments while ATS/OAS/transactions use plurals | Reqs 26–34 |
| CommandStatus · task-lifecycle record | No top-level canonical family | `/commands/{cmdId}/status`; qualified item `.../{statusId}` | Parent-scoped history; collection required for every Command, item route arises through representation/transaction behavior | Reqs 31–32 and transaction classes |
| CommandResult · task-result record | No top-level canonical family | `/commands/{cmdId}/result`; qualified item `.../{resultId}` | Parent-scoped, when a result can exist | Reqs 33–34 and transaction classes |
| Feasibility · pre-task Command variant | Common inventory `/feasibility`; item `/feasibility/{id}` | intended `/controlstreams/{csId}/feasibility`; `/feasibility/{id}/status`; `.../result` | Canonical item required; no parallel numbered canonical-list rule; OAS omits all paths; ATS has copied errors | [Reqs 35–39](https://docs.ogc.org/is/23-002/23-002.html#_req_feasibility) |
| SystemEvent · event/lifecycle record | `/systemEvents`; `/systemEvents/{id}` | `/systems/{sysId}/events`; qualified item `.../{eventId}` | Requirement/OAS/transactions use `events`; one ATS test says `systemEvents` | [Reqs 40–44](https://docs.ogc.org/is/23-002/23-002.html#_req_system-event) |

Query parameters on the schema endpoints identify the observation or command format: `obsFormat` and `cmdFormat`, respectively. Detailed media selection belongs to IDR-SRV-012.

### 6.4 Resource-Category Conclusions

| Category | Resources | Navigation consequence |
|---|---|---|
| Descriptive feature resources | System, Deployment, Procedure, Sampling Feature | Feature collection/item behavior and GeoJSON/SensorML relationship links |
| Descriptive non-feature resource | Property | Resource-adapted collection behavior; links to definitions and users |
| Dynamic stream descriptions | DataStream, ControlStream | Canonical identity plus schema, system/deployment, SF/FOI, and child-record navigation |
| Dynamic/history records | Observation, SystemEvent | Global canonical discovery plus stream/system scoped histories |
| Tasking/control records | Command, Feasibility | Canonical command-like identity plus parent ControlStream, status, and result traversal |
| Subordinate task records | CommandStatus, CommandResult | Parent-scoped addressability, not independent top-level families |
| Supporting schema resources | Observation/Command schemas | Format-specific resources needed before decoding/writing dynamic payloads |

Operational system status is not CommandStatus. CSAPI represents it as a DataStream with `type="status"` whose Observations carry the changing values. CommandStatus describes task progress; SystemEvent describes discrete events. Downstream models must keep all three separate.

### 6.5 Primary Glaux Path Interpretation

Glaux should emit and document the plural route spine because the endpoint overviews, most ATS/transaction clauses, and tagged OAS converge on it:

- `/controlstreams/{id}`, not the isolated `/controls/{id}`;
- `/controlstreams/{id}/commands` or `/controlstreams/{id}/feasibility`, not `/controlstream/...`;
- `/commands/{id}/status` or `/commands/{id}/result`, not `/command/...`; and
- `/systems/{id}/events`, not the isolated ATS `systemEvents` form.

IDR-SRV-010A must decide the lifetime and status of read-compatible defect aliases. If offered, they should resolve or respond using the plural canonical identity, stay out of ordinary links and OAS except a clearly marked compatibility surface, and never create independent resource state.

Glaux should expose `GET /feasibility` as a project interoperability baseline because the common canonical inventory and ATS assume discoverability, while recording that the class lacks an equivalent numbered set-endpoint requirement.

`/systems/{id}/history` and `.../history/{revId}` are not CSAPI 1.0 obligations. They survived in the tagged OAS after the feature was deliberately removed ([issue #149](https://github.com/opengeospatial/ogcapi-connected-systems/issues/149)). Glaux must not advertise them as conformance behavior.

---

## 7. Link Relation and Association Findings

### 7.1 Generic Link Contract

The shared CSAPI Link schema structurally requires only `href` and permits `rel`, `type`, `hreflang`, `title`, `uid`, `rt`, and `if`. That permissiveness does not weaken endpoint-specific semantics. Inherited collection and item rules often require `rel` and `type`, and CSAPI association rules constrain `rel`.

`role` and `profile` are not defined members of the tagged Link schema. RFC 8288's `anchor` is an HTTP Link target attribute; issue #22 discusses its utility for representation-specific `describedby` links, but the tagged JSON schema does not define a JSON `anchor` member. Glaux must not infer JSON semantics merely because open additional properties allow the field to validate.

### 7.2 Standard and OGC Navigation Relations

| Relation | Class | Meaning in this API | Use / strength |
|---|---|---|---|
| `self` | IANA registered | This exact response/representation URL | Required on collection aggregates and item lists; required/recommended elsewhere as specified |
| `alternate` | IANA registered | Another representation of the same Web resource | Required for every other supported representation on the affected Features responses |
| `canonical` | IANA registered | Preferred API item URL for the conceptual CSAPI resource | CSAPI-required whenever retrieved through a noncanonical URL |
| `collection` | IANA registered | Collection containing the item | Required by inherited feature-item response behavior |
| `items` | OGC-defined token | Items resource belonging to a collection descriptor | Required for each supported item encoding; do not “correct” it to IANA's singular `item` |
| `next` | IANA registered | Next page of this selection | Recommended when further results exist; target may be opaque and may expire |
| `prev` | IANA registered | Previous page | Conventional/permitted, not generally required |
| `related` | IANA registered | Generic related resource | Optional fallback only; do not replace a more precise CSAPI association relation |
| `describedby` | IANA registered | Schema or semantic description of the context | Recommended when an applicable description exists; useful for collection and representation schema discovery |
| `license` | IANA registered | Applicable license | Recommended for collections/dataset |
| `service-desc`, `service-doc` | IANA registered | Machine API definition and human documentation | Root behavior accepted in IDR-SRV-009 |
| `data`, `conformance` | OGC-defined tokens | Collection root and conformance declaration | Root behavior accepted in IDR-SRV-009 |
| `status` | IANA registered | Status resource for the context | Available for a Glaux navigation design, but Part 2 does not prescribe it for CommandStatus |

RFC 8288 distinguishes registered tokens from extension relation URIs. The CSAPI `ogc-rel:` strings are syntactically absolute, application-specific extension relation URIs using an unregistered `ogc-rel` scheme; they are not IANA-registered tokens. RFC 8288 §2.1.2 requires extension relation types to be compared character by character in a case-insensitive fashion and recommends all-lowercase URIs. Therefore `ogc-rel:controlStreams` and `ogc-rel:controlstreams` identify the same relation under RFC comparison even though the CSAPI publication is lexically inconsistent. In an HTTP `Link` header, the `ogc-rel:` value must be quoted because its colon is not permitted in a token. Glaux must map these values explicitly rather than expand or dereference them, and its relation comparison key must be case-insensitive.

### 7.3 Published CSAPI Association Vocabulary

Part 1 §7.9 publishes thirteen values:

| Exact published value | Published source → target applicability |
|---|---|
| `ogc-rel:parentSystem` | Subsystem System or Sampling Feature → parent System |
| `ogc-rel:subsystems` | System → child/descendant Systems view |
| `ogc-rel:samplingFeatures` | System or Deployment → associated Sampling Features |
| `ogc-rel:deployments` | System → associated Deployments |
| `ogc-rel:procedures` | System → associated Procedures |
| `ogc-rel:parentDeployment` | Subdeployment → parent Deployment |
| `ogc-rel:subdeployments` | Deployment → child/descendant Deployments |
| `ogc-rel:featuresOfInterest` | System or Deployment → associated features of interest |
| `ogc-rel:implementingSystems` | Procedure → implementing Systems |
| `ogc-rel:sampledFeature` | Sampling Feature → ultimate feature of interest; representation placement conflicts with the generic-links rule |
| `ogc-rel:sampleOf` | Sampling Feature → other Sampling Features of which it is a sample |
| `ogc-rel:datastreams` | System, Deployment, or Sampling Feature → associated DataStreams |
| `ogc-rel:controlStreams` | System, Deployment, or Sampling Feature → associated ControlStreams |

The exact capital `S` in `controlStreams` is lexically inconsistent with lower-case `controlstreams` throughout model fields and endpoint terminology; the association-name mapping footnote also implies the lower-case form. RFC 8288 makes the two spellings equivalent relation identifiers, so this is an output-spelling defect rather than an alias or a second relation. The table also lists a System `featuresOfInterest` applicability that the System model/mappings do not define. Glaux should:

- emit the exact published Table 3 value `ogc-rel:controlStreams` as its deterministic table-first output spelling;
- compare all extension relation URIs case-insensitively, including the lower-case spelling, as a normative RFC rule rather than a compatibility accommodation;
- map relation tokens explicitly to internal relationship identifiers;
- treat only bare, unprefixed variants as compatibility inputs/fixtures rather than ordinary output;
- not create a System `featuresOfInterest` edge solely from the stray table applicability; and
- preserve these conflicts in the traceability/test registry pending authoritative correction.

### 7.4 Association-Name Conflict

Part 1 GeoJSON Requirement 79 and SensorML Requirement 91 say the relation type is the “association name.” Incorporated association-mapping footnotes require that name to be prefixed with `ogc-rel:`. Parts of the ATS and tagged examples use bare names. [Issue #173](https://github.com/opengeospatial/ogcapi-connected-systems/issues/173) and [PR #176](https://github.com/opengeospatial/ogcapi-connected-systems/pull/176) confirm prefixed intent, but they do not resolve the separate output-spelling inconsistency; current `master` fixes only three relations in one example and does not amend Version 1.0 or all remaining examples.

**Glaux interpretation:** ordinary output uses the prefixed Table 3 spellings, including `ogc-rel:controlStreams`, while comparison and lookup use the RFC-required case-insensitive key. Compatibility parsing may recognize known bare values with telemetry, but must not treat arbitrary unknown strings as capabilities. If an uncorrected external ATS requires bare output, run that test through a named published-1.0 adapter rather than duplicating conflicting links in normal responses.

### 7.5 Direct Properties, Local IDs, and Links Are Different

The representation models deliberately use several relationship forms:

- `system@link`, `procedure@link`, `deployment@link`, `featureOfInterest@link`, `samplingFeature@link`, `result@link`, `observation@link`, `observationSet@link`, `datastream@link`, and `external@link` carry Link objects in their respective models and result variants;
- direct named association properties also include `platform@link`, `deployedSystems@link`, and `sampledFeature@link`; Requirement 79/91's generic `links` mapping rule does not automatically govern these named properties;
- `datastream@id`, `controlstream@id`, `command@id`, and `samplingFeature@id` carry API-local identifiers;
- `systemKind` is a Procedure association mapped to GeoJSON `properties/systemKind@link` and SensorML `typeOf`, rather than a generic semantic URI/CURIE field;
- `baseProperty`, `objectType`, and `statistic` are examples of URI/CURIE-valued semantic references; and
- collection/nested association links point to resource sets rather than one item.

Table 3's inclusion of `sampledFeature` alongside generic association links conflicts with its direct-property placement in the representation mapping. Glaux must keep that inconsistency visible and let IDR-SRV-017 define the exact serialization/validation rule.

The Rust model must preserve these distinctions. A universal “relationship URL” field would lose required local-ID, external-link, set-link, and semantic-identifier behavior.

---

## 8. Navigation and Traversal Findings

### 8.1 Generic Client Walk

A standards-aware client can perform this basic walk:

`/` → landing `data` link → `/collections` → collection `items` link → `/collections/{id}/items` → item `self`/view → item `canonical` → canonical item → association links or documented nested endpoints.

The landing page need not link directly to every CSAPI canonical family. Direct family links may be added as helpful Glaux extensions, but the required generic discovery route remains `data` to `/collections`.

### 8.2 System and Deployment Walks

A System can navigate to:

- parent and child Systems;
- its Sampling Features;
- its Deployments when the association is implemented;
- Procedures through association links even though no fixed nested Procedure path is prescribed;
- DataStreams and ControlStreams through Part 1 association links and Part 2 system-scoped endpoints; and
- System Events through Part 2 `/systems/{id}/events`.

A Deployment can navigate to:

- parent/child Deployments;
- deployed Systems and related features/Sampling Features through its representation;
- DataStreams and ControlStreams generated/controlled by deployed Systems, when the applicable Part 2 association and temporal intersection conditions hold.

Recursive System/Deployment lists return direct/top-level members by default and all descendants with `recursive=true`. Other filters apply at every visited level. The subsystem view includes all permanently attached subsystems and follows the recommended `datetime` membership behavior for current, instant, or interval deployment association.

Recursive association aggregation applies even when the client is not traversing the child-list route: a parent System's Sampling Feature, DataStream, and ControlStream target sets include the parent and all descendant Systems; a parent Deployment's deployed-System, Sampling Feature, feature-of-interest, DataStream, and ControlStream target sets include the Deployment and all descendant Deployments. Cycle protection, deterministic deduplication, and exact query interaction belong to IDR-SRV-011/015/017/025, but the navigation implementation must leave room for them.

### 8.3 Observation Walk

The coherent dynamic-data traversal is:

System → DataStreams → one DataStream → its format-specific schema → its Observations → one Observation → canonical Observation, System, observed Properties, Sampling Feature/feature of interest, Procedure, result or result link, and temporal context.

The numbered standard requires the scoped Observation endpoint but does not require a named `observations` link in every DataStream representation. Tagged examples use bare `observations`, which is informative only. Glaux should make the scoped endpoint explicitly navigable, but IDR-SRV-017 must finalize a standards-safe Glaux extension relation rather than mislabel the example token as normative.

An Observation's temporal context is carried primarily by its inline `phenomenonTime` and `resultTime` fields (and the parent DataStream's temporal aggregates), not by an assumed temporal Link. A Link is used only where the selected representation actually defines one.

### 8.4 Control and Feasibility Walk

The coherent tasking traversal is:

System → ControlStreams → one ControlStream → command/feasibility schema → Commands or Feasibility requests → one Command-like resource → status history and result resources → any inline result, Observation(s), DataStream, or external dataset.

CommandResult supplies the primary bridge from tasking back to observations. Its variants are inline `data`, `observation@link`, `observationSet@link`, `datastream@link`, or `external@link`.

Each ControlStream also carries or exposes its required System relationship and may identify target Sampling Features and features of interest through the applicable `@id`, `@link`, and scoped endpoints. Those target relationships must agree with the System-scoped ControlStream view before a client follows the stream into Commands or Feasibility.

As with observations, Part 2 does not publish a complete link-relation vocabulary for Commands, status, results, Feasibility, events, or schema endpoints. Routes alone let an OAS-aware client navigate but not a purely hypermedia client. Glaux should include explicit links for every available scoped edge and classify their relation values as project extensions until IDR-SRV-017 finalizes the vocabulary.

### 8.5 Reverse, Parent/Child, and External Traversal

Required scoped endpoints establish many set-valued reverse views; representation fields establish other forward or item links. They do not imply that every relationship must be materialized in both directions:

- a System's stream set and each stream's `system@link` should agree;
- a nested item must resolve to its canonical item;
- Deployment stream membership is derived from relationship plus temporal intersection, not URL ownership;
- external features, schemas, results, and related APIs may remain external;
- a Procedure may link to an implementing-System set without a mandated fixed path; and
- a globally exhaustive reverse Deployment link may be impossible when related systems or deployments are external.

Glaux should enforce bidirectional invariants only where both sides are locally authoritative or required. IDR-SRV-017 must classify stored, derived, optional, and external relationships so a later implementation does not manufacture false completeness.

---

## 9. Part 1 / Part 2 Resource-Relationship Analysis

| Source resource | Target / view | Standard navigation surface | Direction and condition | Glaux implication |
|---|---|---|---|---|
| System | Subsystems | `ogc-rel:subsystems` → `/systems/{id}/subsystems` | Required by Subsystem class | Same System identities; permanent membership plus recommended time-dependent membership; recursive selector |
| Subsystem | Parent System | `ogc-rel:parentSystem` → parent item | Required association/mapping | Stable canonical parent target |
| System | Sampling Features | `ogc-rel:samplingFeatures` → `/systems/{id}/samplingFeatures` | Required when SF class applies | Empty collection remains navigable |
| Sampling Feature | Parent / sampled feature / samples | `ogc-rel:parentSystem`; direct GeoJSON `properties/sampledFeature@link` / corresponding SensorML property; `ogc-rel:sampleOf` | Model/mapping-specific | Preserve generic-link versus direct-property placement; local or external targets; no invented route |
| Sampling Feature | DataStreams / ControlStreams | `ogc-rel:datastreams`, `ogc-rel:controlStreams` | Published association vocabulary; no fixed target path | Registry selects the correct local set target; output uses the table spelling and comparison is case-insensitive |
| System | Deployments | `ogc-rel:deployments` → `/systems/{id}/deployments` | Conditional on association | Membership matches deployed-System relationship |
| Deployment | Platform/deployed Systems; subdeployments / parent | direct `platform@link` and `deployedSystems@link`; `/deployments/{id}/subdeployments`; `ogc-rel:subdeployments`, `ogc-rel:parentDeployment` | Representation mappings and Subdeployment class | Preserve direct-property direction; same Deployment identity; recursive selector and descendant association aggregation |
| System / Procedure | Procedure kind, Procedures, and implementing Systems | direct `systemKind@link`; System→Procedure set via `ogc-rel:procedures`; Procedure→System set via `ogc-rel:implementingSystems` | Association; no fixed set-target path | Preserve direct-property direction separately from generic association links |
| System | DataStreams | `ogc-rel:datastreams` + `/systems/{id}/datastreams` | Part 1 hook; Part 2 route conditioned on System class | Forward link and reverse `system@link` agree |
| System | ControlStreams | `ogc-rel:controlStreams` + `/systems/{id}/controlstreams` | Same | Deterministic table spelling; lower-case input is RFC-equivalent, not an alias |
| Deployment | DataStreams/ControlStreams | association links + deployment-scoped routes | Part 2 conditions; valid-time intersection for DataStream | Derived view, not ownership |
| DataStream | Sampling Features / FOI | scoped endpoints + Link/local-ID fields | Conditional on classes, represented association, and local hosting | Do not emit local link to absent/external target |
| DataStream | Schema / Observations | fixed schema and nested Observation routes | Required for class | Add Glaux navigability relations pending IDR-SRV-017 |
| Observation | DataStream/System/Property/SF/FOI/Procedure/result | required local ID plus optional links/result form | Representation rules | Preserve ID vs link; validate against parent schema |
| ControlStream | Sampling Features / FOI | scoped endpoints + Link/local-ID fields | Conditional | Same integrity rules as DataStream |
| ControlStream | Schema / Commands / Feasibility | fixed intended routes | Required/conditional with publication defects | Primary plural route plus named adapters |
| Command/Feasibility | Status / Result | parent-scoped endpoints | Required as applicable | Parent-scoped identity; no top-level status/result family |
| CommandResult | Observation(s), DataStream, external or inline result | mutually exclusive representation forms | JSON/model constraint | Task-to-data traceability |
| System | System Events | `/systems/{id}/events` | Part 2 SystemEvent class | Nested binding and global canonical event agree |
| Status DataStream | Status Observations | ordinary DataStream/Observation navigation | DataStream `type=status` | Separate from CommandStatus |

### 9.1 Hypermedia Completeness Finding

CSAPI is navigable but not uniformly hypermedia-complete. Part 1 defines a specialized relation vocabulary; Part 2 mostly defines paths, local `@id` references, and Link-valued fields without a complete set of relation names for all new edges. A client using a complete, corrected Glaux OAS can construct every route; the tagged official OAS cannot. A client relying only on response links may not discover every Part 2 subordinate endpoint.

**Project recommendation:** Glaux should close this usability gap with explicit, typed links from each resource to every available related set/schema/status/result/event endpoint, while keeping the exact extension-relation vocabulary in the IDR-SRV-017 decision register. That makes Glaux a better reference implementation without falsely labeling a project extension as a CSAPI `SHALL`.

---

## 10. Canonical Identity, URI, and Lifecycle Implications

### 10.1 Three Part 1 Identity Layers

Part 1 distinguishes:

1. a local ID, unique within a resource type across every collection containing that type;
2. a UID, which is a valid URI and unique across all collections served by the API; and
3. a canonical API URL such as `/systems/{id}`.

They are not interchangeable. Part 1 Recommendation 1 strengthens the UID rule by recommending globally unique identifiers drawn from registered namespaces, with UUID URNs as one suitable form; global uniqueness is not the Requirement 2 `SHALL`. Part 2 Common inherits Part 1 Common, so any UID exposed on a Part 2 representation remains subject to the inherited UID rules even though Part 2 does not define a second resource-level UID field. CommandStatus and CommandResult are parent-scoped and have no global canonical family.

### 10.2 Required Identity Invariants

- one logical resource has one canonical API item, even when represented through several collection, nested, or compatibility paths;
- local ID lookup for each top-level family reaches the same resource from every view;
- a noncanonical response includes `canonical`;
- `self` identifies the current Web representation and must not be substituted for `canonical`;
- canonical mutation is reflected across memberships when the applicable transaction class is implemented;
- a nested item's parent relationship is true at the response's effective state/time;
- an external Link remains a Link and is not coerced into a local ID; and
- opaque paging URLs identify selections/pages, not resources.

### 10.3 Lifecycle Questions Handed to IDR-SRV-016

IDR-SRV-016 must decide:

- local-ID generation, case sensitivity, allowed characters, escaping, and reuse prohibition;
- UID allocation and external UID collision handling;
- canonical URL stability behind reverse proxies and public-base changes;
- alias, redirect, and deprecation behavior for the published route defects;
- whether moves use 301/308, a tombstone, or another policy;
- deleted-resource and stale-link behavior;
- external-reference retention and health metadata;
- parent-scoped Status/Result ID rules; and
- whether a canonical URL is ever reassigned after deletion.

The standards do not settle these policies. This report only requires that later choices preserve the canonical/multi-view invariants.

### 10.4 Write and Collection-Membership Boundary

Issue #164 confirms that Version 1.0 does not cleanly distinguish links generated by the server from association links a client may submit. Until IDR-SRV-017 and mutation topics decide otherwise, Glaux should treat navigation links (`self`, `canonical`, collection, paging, route-derived relationship sets) as response-generated and non-writable. Client-supplied relationship fields must be accepted only through explicitly modeled inputs, never by replaying an arbitrary response `links` array into storage.

---

## 11. Validation, Error, and Consistency Implications

### 11.1 Structural and Semantic Validation Are Separate

JSON/OpenAPI schema validation can establish that `links` is an array and that a Link contains `href`. It cannot establish that:

- `rel` is the correct value for the source/target relationship;
- `type` matches the representation returned by `href`;
- a canonical link resolves to the same underlying identity;
- a member truly belongs to the collection or parent resource;
- aggregate/detail named metadata fields agree when present and every aggregate-entry link is included in detail;
- the target exists or is visible to the current principal;
- a reverse relationship agrees;
- a recursive result is complete, deduplicated, and cycle-safe; or
- the OAS, implemented route, emitted link, and conformance claim describe the same capability.

Glaux needs both representation validation and a graph/navigation invariant layer.

### 11.2 Minimum Invariant Set

| Invariant | Required source or project reason | Failure significance |
|---|---|---|
| Collection aggregate/detail named-field equality and aggregate-link inclusion | Features Req 19 | Discovery clients receive contradictory metadata |
| Exact collection type markers and homogeneous family behavior | CSAPI family collection requirements | Generic client chooses wrong decoder/handler |
| Every advertised `items` target exists and returns the declared family | Features Req 15 + CSAPI adaptation | Broken discovery |
| Item-list `self`/`alternate` links and link parameters have correct meaning | Features Reqs 28–29 | Selection and negotiation corruption |
| Individual-item `self`, `alternate`, `collection`, and CSAPI `canonical` targets have correct meaning | Features Req 35 + applicable CSAPI canonical requirements | Identity and negotiation corruption |
| Nested endpoint contains only resources related to the path parent | CSAPI endpoint requirements | False graph edges/data leakage |
| Hierarchical System/Deployment association target sets include the parent and all descendants required by Requirements 13/23 | Part 1 Reqs 13 and 23 | Incomplete recursive graph traversal |
| Canonical and noncanonical views resolve one logical resource | CSAPI canonical contract | Duplicate state and inconsistent updates |
| Local IDs and UIDs meet their uniqueness scopes | Part 1 Reqs 1–2 | Ambiguous resolution |
| Stream/schema/child records agree | Part 2 schema and parent constraints | Observation/command cannot be safely decoded |
| System/deployment stream forward and reverse views agree where locally authoritative | Cross-Part relationship contract | Broken traversal |
| Paging/count/time values and links describe the same selection | Features response rules | Lost/duplicated results |
| Route registry, OAS, collection descriptors, link emitters, and conformance evidence agree | Glaux project invariant | Published API is not truthful |

### 11.3 Missing, Inaccessible, Stale, Deleted, and Moved Targets

The inherited Features baseline establishes 404 for unknown collection/item paths and explicitly warns that an opaque `next` link may later return 404. It does not define a complete related-link lifecycle policy. CSAPI also does not prescribe:

- whether an unauthorized target is omitted or linked and then returns 401/403;
- whether a deleted target yields 404, 410, a tombstone, or retained historical metadata;
- whether a moved canonical item redirects;
- how external-link health is reported; or
- whether stale relationship data is an error, warning, or reconciliation state.

Until IDR-SRV-013, 016, 039, and 040 decide these:

- never emit a locally generated relationship link that is already known to be dangling;
- apply the same authorization decision to link visibility and target retrieval so links do not leak hidden resources;
- retain external links as external evidence without promising availability;
- report invariant violations operationally rather than silently dropping state; and
- keep moved/deleted/unauthorized cases as explicit test dimensions instead of hard-coding one premature response policy.

These are bounded project safeguards, not claims that CSAPI mandates a particular status code.

### 11.4 Consistency and Transaction Boundary

Operations that modify canonical resources, collection memberships, parent-child relationships, stream schemas, or derived reverse views can affect several navigation surfaces at once. IDR-SRV-029 must decide atomicity, idempotency, concurrency, and repair policy. The required outcome is clear even though the mechanism is not: after a successful externally visible change, all generated routes and links must describe one coherent committed graph.

---

## 12. Interoperability and Existing-Implementation Findings

### 12.1 Official OAS and Schema Artifacts

The two tagged modular OAS documents are explicitly example specifications with OpenAPI 3.1.0 and `info.version: 0.0.1`. Their route inventories are useful but not authoritative:

- Part 1 has 20 top-level paths and broadly matches its fixed navigation surface.
- Part 2 has 23 paths, but omits Deployment-scoped stream endpoints, stream-to-Sampling-Feature/FOI endpoints, every Feasibility route, and `/systemEvents/{id}`.
- Part 2 includes removed `/systems/{id}/history` routes.
- The release assets [`ogcapi-connectedsystems-1.bundled.oas31.yaml`](https://github.com/opengeospatial/ogcapi-connected-systems/releases/download/v1.0.0/ogcapi-connectedsystems-1.bundled.oas31.yaml) and [`ogcapi-connectedsystems-2.bundled.oas31.yaml`](https://github.com/opengeospatial/ogcapi-connected-systems/releases/download/v1.0.0/ogcapi-connectedsystems-2.bundled.oas31.yaml) retain 32 and 51 relative references respectively. Neither is standalone.
- The shared Link schema requires only `href`, leaves `rel` unconstrained, cites superseded RFC 5988, and cannot validate endpoint semantics.
- The shared collection schema lacks `featureType`; open additional properties let examples carry it but do not enforce it.
- two Part 2 by-reference examples use mutable raw-`master` references.

The Part 2 ATS also contains copied-target defects that a literal harness must not mistake for the requirement: Feasibility A.35 checks Command `itemType`, A.36 checks the Commands path, SystemEvent A.40 names ControlStream collections, A.42 invokes a ControlStream resources test, and A.43 uses the wrong nested `systemEvents` path. Requirement 31(B)'s normative text names both `limit` and `datetime` but cites only Features §7.15.2; `datetime` is defined in §7.15.4. The requirement baseline and a documented overlay control those cases.

Glaux OAS must therefore be generated from the normative-overlay registry, not copied from either official example. IDR-SRV-014 owns the construction strategy.

### 12.2 Tagged Example Defects

Examples reviewed at `v1.0.0` include:

- bare association relations where the published table uses `ogc-rel:`;
- a collection-list-level `canonical` link even though Part 1's rule places `canonical` on each noncanonical resource representation;
- wrong `ssn:` collection type markers;
- an `items` target missing `/items`;
- singular `item`;
- missing link `type`;
- missing inherited item `collection`;
- a nonstandard `/components` hierarchy route;
- generic/malformed alternate URLs;
- `/controls/{id}/commands`;
- stale Observation reference fields;
- a SystemEvent link without `rel`; and
- server-response examples missing IDs required by their response models.

They should be retained as named negative or compatibility fixtures. “Make our output look like the example” is not an acceptable implementation rule.

### 12.3 Official Maintenance History

The bounded history review found no later release and no post-tag Part 2 repair. Relevant context includes:

- [#22](https://github.com/opengeospatial/ogcapi-connected-systems/issues/22): explains canonical-vs-collection identity and `describedby`/HTTP-anchor rationale;
- [#47](https://github.com/opengeospatial/ogcapi-connected-systems/issues/47) and commit `da6ce68`: introduced the published relation table;
- [#149](https://github.com/opengeospatial/ogcapi-connected-systems/issues/149): records removal of System History;
- [#164](https://github.com/opengeospatial/ogcapi-connected-systems/issues/164): leaves client-written versus generated links unresolved;
- [#165](https://github.com/opengeospatial/ogcapi-connected-systems/issues/165): leaves local/external Sampling Feature matching questions unresolved;
- [#173](https://github.com/opengeospatial/ogcapi-connected-systems/issues/173) / PR #176: confirms prefixed example intent but fixes only part of the current branch; and
- [#177](https://github.com/opengeospatial/ogcapi-connected-systems/issues/177): tracks missing Deployment-scoped stream OAS paths.

These items explain design state; they do not amend the approved release.

### 12.4 CSAPI Explorer

At commit `00f1c188...`, CSAPI Explorer can use explicit URLs and recognizes several bare or implementation-specific patterns, but it does not recognize the standards-published `ogc-rel:` association vocabulary. It can synthesize stale/nonstandard paths such as `history` or `cancel`, does not automatically exhaust `next` pages, and contains an OpenSensorHub-specific fallback from top-level Commands to ControlStream-scoped Commands.

Explorer is valuable as:

- a smoke client;
- an adversarial compatibility corpus; and
- evidence that explicit links and canonical routes matter.

It is not an implementation oracle. Glaux should not emit wrong relation names or invent routes merely to satisfy its heuristics. IDR-SRV-056 should test standards-strict traversal first and record Explorer accommodations separately.

### 12.5 OpenSensorHub and connected-systems-go

OpenSensorHub exposes a mature hierarchy of canonical and nested routes, but uses many bare/custom association relations, treats some collection `items` links as direct typed routes, lacks a fully registered generic collection-items handler, and retains history behavior. Its collection wrappers and counts/links diverge from the inherited Features baseline in places.

`connected-systems-go` covers broad canonical/nested routes, redirects generic collection items to canonical typed resources, emits counts, and normalizes several association relations to `ogc-rel:`. It also retains history, omits Deployment-scoped streams, and emits relation names outside or differently cased from the published table. Its lenient equality between prefixed and bare relations is useful input-policy precedent, not output authority.

Both are valuable differential-test targets and sources of implementation lessons. Neither changes what the Rust Glaux server should publish.

### 12.6 Official Features ETS and Preserved SECD Evidence

The official Features ETS checks inherited landing, `/collections`, collection metadata, `items`, item-list links/counts, and feature-item `self`/`alternate`/`collection`. It does not test CSAPI type markers beyond feature discovery, canonical back-links, specialized associations, Part 2 resource envelopes, or the publication contradictions.

Restricted SECD evidence available through the authenticated project workspace includes a deployment where direct typed routes worked while `/collections` returned 404, nested relations used `alternate`, top-level Observations were absent, and a DataStream wrapper used `datastreams` instead of `items`. The pinned repository returns public 404, so this evidence is not independently public. It is useful evidence of how an apparently functional server becomes unusable to generic clients; it is not standards proof and no conclusion depends on it alone.

---

## 13. Decision Analysis

| Decision area | Option | Benefits | Costs / compatibility effect | Disposition |
|---|---|---|---|---|
| Primary conflicting routes | Implement every literal singular/typo form as primary | Superficial adherence to isolated requirement text | Contradicts overview, ATS, transactions, OAS, and client practice; fragments identity | Reject |
| Primary conflicting routes | Publish plural route spine; track named compatibility aliases | Coherent and best-supported interpretation; one canonical identity | Requires adapter and explicit version policy | **Adopt; alias lifetime to IDR-SRV-010A** |
| Relation output | Emit bare association names from examples/ATS | Works with some current clients/tests | Contradicts relation table/mapping intent and issue #173 | Reject as normal output |
| Relation output | Emit prefixed Table 3 `ogc-rel:` spellings; compare extension relation URIs case-insensitively; accept known bare legacy inputs separately | Deterministic output, RFC-correct identity, controlled interoperability | Requires comparison and compatibility tests; publication spelling remains inconsistent | **Adopt; retain upstream watch** |
| Relation output | Duplicate prefixed and bare links in every response | May satisfy more heuristic consumers | Ambiguous/noisy graph, possible double traversal, hides conflict | Reject by default; adapter-only if a test requires |
| Part 2 collections | Advertise none | Meets conditional wording | Weak generic discovery; poor reference-server usability | Reject for Glaux target |
| Part 2 collections | One default typed collection per top-level family, plus configured collections | Complete generic discovery and clear baseline | More descriptors/tests | **Adopt as project behavior** |
| Feasibility list | Omit `/feasibility` because numbered set requirement is missing | Minimal literal class surface | Conflicts with common inventory and ATS assumptions; weak discovery | Reject |
| Feasibility list | Expose `/feasibility`, flag interpretation | Coherent canonical discovery | Must not miscite as numbered `SHALL` | **Adopt as project behavior** |
| Part 2 related links | Rely only on OAS/path construction | Avoids project relation vocabulary | Hypermedia clients cannot discover every edge | Reject for reference implementation |
| Part 2 related links | Emit explicit links with project-extension classification | Best navigability; testable graph | Exact relation vocabulary must be designed | **Adopt; vocabulary to IDR-SRV-017** |
| Navigation implementation | Hand-code routes, collection metadata, OAS, and links separately | Locally simple | Predictable drift and false conformance | Reject |
| Navigation implementation | One typed registry projected to all surfaces | Consistency, traceability, test generation | Upfront design work | **Adopt** |

---

## 14. Key Recommendations

1. **Use one typed navigation registry.** Record canonical, collection, nested, subordinate, schema, and compatibility routes; exact relations; related types; prerequisites; representations; source anchors; and tests.
2. **Project every public surface from it.** Rust routing, collection descriptors, payload links, HTTP `Link` headers, OAS paths, conformance evidence, and tests must share the registry.
3. **Maintain one resource identity across views.** Canonical, nested, arbitrary-collection, and compatibility URLs never create duplicate state.
4. **Implement the complete inherited collection spine.** Aggregate, detail, items, and item routes must be mutually consistent and actually traversable.
5. **Advertise all required Part 1 typed collections.** Enforce exact `itemType`/`featureType`, non-vacuous availability, and member-family correctness.
6. **Advertise one default Part 2 collection per top-level family.** Treat this as Glaux reference-server behavior, not a conditional standard mandate.
7. **Use plural paths as primary.** Keep the published singular/typo forms in a named contradiction register and finalize read-alias policy in IDR-SRV-010A.
8. **Expose the Feasibility canonical list.** Clearly classify the missing numbered set requirement as a publication gap.
9. **Use prefixed `ogc-rel:` values with RFC-correct comparison.** Emit the published Table 3 spellings deterministically, compare extension relation URIs case-insensitively, and isolate only bare legacy alternatives under controlled compatibility policy.
10. **Do not invent normative Part 2 relations.** Add project-classified navigability links for schema, observations, commands, feasibility, status, result, and events; finalize their extension vocabulary in IDR-SRV-017.
11. **Keep Link, local ID, and semantic URI fields distinct in Rust.** Do not flatten `@link`, `@id`, UID, CURIE, canonical URL, and set-link forms.
12. **Treat navigation links as response-generated by default.** Accept writable relationships only through explicit request models defined by later topics.
13. **Run structural and graph validation.** Validate schemas, then identity, membership, targets, link meanings, reverse/derived views, paging, and cross-surface parity.
14. **Use official artifact defects as fixtures.** Never copy tagged OAS/examples as the server contract.
15. **Test in separate evidence lanes.** Preserve literal requirements, normative ATS/Features ETS, Glaux interpretations, defect adapters, client smoke tests, and implementation differentials as distinct results.
16. **Fail release readiness on drift.** A route, link, collection marker, OAS declaration, or conformance claim without matching implementation evidence blocks publication.

---

## 15. Test-Strategy Implications

### 15.1 Required Test Lanes

| Lane | Purpose | Examples |
|---|---|---|
| Approved normative behavior | Prove controlling obligations | Every canonical/nested route; type markers; required links; scoped membership |
| Official ATS/ETS | Preserve recognized external procedures | Features Core ETS; CSAPI Annex A tests with exact provenance |
| Normative overlay | Test obligations omitted or misstated in OAS/ATS | Deployment streams, SF/FOI routes, Feasibility, event route, relation prefix |
| Artifact conflict | Prevent accidental copying of defects | Wrong examples, missing routes, history extras, singular paths |
| Glaux project behavior | Prove adopted reference-server enhancements | Default Part 2 collections, explicit Part 2 navigability, registry parity |
| Compatibility adapter | Isolate tolerated legacy behavior | Bare relations, singular aliases, `controls`, event ATS spelling |
| External client | Measure real interoperability | Explorer standards-first smoke, then documented accommodation cases |
| Differential implementation | Learn without adopting divergences | OpenSensorHub and connected-systems-go response comparisons |

Every class test must include at least one correctly typed, reachable resource and collection before evaluating `for-each` assertions. This non-vacuity guard prevents several Part 1 ATS loops from passing merely because the implementation exposes no qualifying collection.

### 15.2 Minimum Scenario Corpus

The fixture graph should contain:

- all Part 1 and Part 2 top-level families;
- at least one correctly typed collection for each family and one resource in overlapping collections;
- a three-level System hierarchy and three-level Deployment hierarchy;
- direct, recursive, empty, external, missing, unauthorized, stale, and deleted relationship targets;
- a System and Deployment with several DataStreams/ControlStreams and temporal edge cases;
- Observation data with inline and linked results;
- synchronous and asynchronous Command/Feasibility status/result histories;
- a status DataStream distinct from CommandStatus and SystemEvent;
- multiple representations and paging with opaque `next`;
- published, alternate-case, and bare legacy relation spellings;
- plural canonical routes and every named defect alias; and
- examples/OAS fragments preserved as expected-negative inputs.

### 15.3 Positive and Negative Assertions

For every endpoint family, tests should assert:

- route/method/status and response representation;
- family purity and parent/collection membership;
- correct path ID, local ID, UID scope, and canonical equivalence;
- required/recommended links and target media types;
- required association endpoints and links remain present even when their result set is empty; only capability-conditional links may be absent when that capability or target is genuinely unavailable;
- metadata consistency, accurate extents, and accurate counts when counts are present;
- paging without duplicates or loss;
- cycle-safe recursive traversal;
- schema resolution for stream payloads;
- OAS/runtime/registry parity; and
- authorization and lifecycle outcomes once owning policies exist.

Negative tests must include wrong collection markers, cross-family IDs, dangling targets, incorrect case-sensitive relation comparison, missing/incorrect relation prefixes, stale aliases, malformed hrefs, wrong media `type`, mismatched parent membership, duplicate UIDs, path/OAS drift, and every contradiction in Appendix C.

### 15.4 Conformance Result Discipline

Passing Features ETS proves only the inherited lane. Passing an implementation-specific Explorer smoke test proves only that client interaction. Passing a compatibility alias test does not make that alias canonical. Glaux's final conformance evidence must preserve those distinctions so a green dashboard cannot conceal a standards conflict.

---

## 16. Implementation Implications and Planning Estimates

### 16.1 Rust Design Shape

Later implementation planning should expect explicit Rust types along these lines:

- `ResourceFamily` for every top-level and subordinate family;
- `RouteKind` for canonical set/item, collection, nested, schema, subordinate, and compatibility routes;
- `RelationId` that preserves exact wire spelling independently of internal field names;
- separate `LocalId`, `Uid`, `CanonicalUrl`, `LinkTarget`, and external semantic-reference types;
- `CollectionDescriptor` with exact type markers and generated links;
- `NavigationEdge` with source/target family, cardinality, direction, derivation, capability guard, authorization rule, and provenance;
- a public-base-aware URL builder; and
- semantic validators operating on the resulting graph.

These are design implications, not a final domain model. IDR-SRV-015 through 017 own the actual type design.

### 16.2 Relative Work Estimate

| Work area | Relative complexity | Main risk | Owning later work |
|---|---|---|---|
| Typed registry and URL builder | Medium | Bad abstraction freezes publication defects | 014–017, 044–045 |
| Collection aggregate/detail/items/item service | High | Feature/resource adaptation and metadata drift | 011–015 |
| Canonical/nested route projection | High | Scope conditions and shared identity | 015–017, 025 |
| Link serialization and HTTP-header projection | Medium | Media/context/legacy relation mismatch | 012, 017 |
| Graph and membership validators | High | Derived/reverse/external/authorized state | 017, 023, 029 |
| Compatibility adapters | Medium | Aliases become permanent or falsely canonical | 010A, 013 |
| Generated OAS and parity validation | High | Example OAS defects leak into contract | 014, 051 |
| Multi-lane automated tests/fixtures | High | Test greenwashing and vacuous class checks | 050–056 |

A detailed schedule now would be false precision. Framework choice, persistence model, relationship cardinalities, and test architecture have not yet been researched. The implementation plan should estimate work only after those reports are accepted.

---

## 17. Risks, Constraints, and Open Questions

| Risk / unresolved item | Current conclusion | Owner / gate |
|---|---|---|
| Singular/plural and `controls` route defects | Plural primary; aliases not yet final | IDR-SRV-010A |
| Exact alias status/lifetime and deprecation headers | Unresolved | IDR-SRV-010A |
| Part 2 extension relation vocabulary | Links recommended; exact URIs/tokens unresolved | IDR-SRV-017 |
| `ogc-rel:controlStreams` versus lower-case model/mapping/ecosystem use | Emit table spelling; compare case-insensitively as the same relation | Upstream watch; 017 |
| Bare relation ATS/example behavior | Named adapter, never silent normal output | IDR-SRV-050/051 |
| Property collection individual-resource route | Implement coherent adapted item route as Glaux interpretation | IDR-SRV-014/015/051 |
| Feasibility root list lacks numbered requirement | Expose as Glaux interoperability baseline | Plan-owner acceptance; 010A/014 |
| Writable versus generated links | Generated-only default | IDR-SRV-017/031 |
| Reverse-link completeness | Only where locally authoritative/required | IDR-SRV-017/029 |
| Missing, moved, deleted, stale, external, or forbidden targets | Invariants defined; response policy unresolved | IDR-SRV-013/016/039/040 |
| Authorization-sensitive discovery | Must avoid leakage; mechanism unresolved | IDR-SRV-039/040 |
| Stream schema link relation and representation anchoring | `describedby` useful; exact media/anchor behavior unresolved | IDR-SRV-012/017/023 |
| Recursive cycles/dedup/order | Required robustness; algorithm unresolved | IDR-SRV-011/015/025 |
| OAS/ATS/artifact defects | Retain in overlay/adapter registry | IDR-SRV-014/050/051 |
| External client expectations conflict with standard | Standards-strict first; accommodations isolated | IDR-SRV-056 |

The largest implementation risk is not route coding. It is allowing route handlers, stored relationships, collection descriptors, links, OAS, and tests to become six different and inconsistent descriptions of the same graph.

### 17.1 Decisions Made by Accepting This Report

Plan-owner acceptance approves these planning baselines:

- plural routes are Glaux's primary public contract;
- prefixed Table 3 `ogc-rel:` spellings are the deterministic Glaux output policy, with RFC-required case-insensitive comparison and the publication's lexical inconsistency recorded;
- all Part 1 and default Part 2 typed collections are advertised;
- `/feasibility` is exposed and explicitly classified as a project interpretation;
- Part 2 scoped edges are made link-navigable with project classification; and
- one typed navigation registry controls public surfaces.

Acceptance does not choose alias lifetime, final project extension-relation names, error codes/bodies, ID algorithms, authorization behavior, persistence design, or Rust framework.

---

## 18. Downstream Handoff and Current Status

### 18.1 Handoff Matrix

| Downstream topic | Required input from this report |
|---|---|
| IDR-SRV-010A | Route/relation defect aliases, versioning, deprecation, Features Part 4 delta policy |
| IDR-SRV-011 | Resources-endpoint classes, nested/recursive scopes, paging links, selection invariants |
| IDR-SRV-012 | Link `type`, alternate representations, schema format parameters, HTTP Link/anchor behavior |
| IDR-SRV-013 | Unknown, stale, expired, unauthorized, moved, deleted, and external target failures |
| IDR-SRV-014 | Normative-overlay route inventory, collection/link components, OAS omissions/extras, generated parity |
| IDR-SRV-015 | Family taxonomy, canonical-versus-view identity, subordinate resources |
| IDR-SRV-016 | ID/UID/canonical URL, path escaping, aliases, redirects, tombstones, lifecycle |
| IDR-SRV-017 | Exact relationship/cardinality model, extension relations, reverse/derived edges, link writes |
| IDR-SRV-018/020 | Deployment temporal membership; status DataStreams; System Events/history boundary |
| IDR-SRV-021/023/024 | GeoJSON/SensorML link mappings, schema resolution, stream-schema validation |
| IDR-SRV-025/026/029/031 | Graph/membership storage, indexes, derived views, atomic writes, ingestion |
| IDR-SRV-034/036/037 | Observation/status and Command/Feasibility lifecycle traversal |
| IDR-SRV-039/040 | Authorization-sensitive discovery and link visibility |
| IDR-SRV-044/045 | Rust registry, route composition, link builder, module boundaries |
| IDR-SRV-050/051/053/056 | ATS adapters, traceability, graph fixtures, external-client matrix |
| IDR-SRV-057 | Final resource/navigation baseline and unresolved-decision ledger |

### 18.2 Current Status

- Research and independent evidence audits: complete.
- All five core and 41 detailed questions: answered.
- All six methodology phases: complete.
- All ten success criteria and fifteen deliverable areas: satisfied.
- Deliverable: complete and internally reviewed.
- Plan-owner acceptance: **pending**.

No downstream topic was executed.

### 18.3 Next Controlled Transition

The next two actions are:

1. the Glaux Project Lead accepts or returns IDR-SRV-010; then
2. after acceptance, execute exactly one next plan: IDR-SRV-010A.

A combined instruction may approve this report and authorize exactly IDR-SRV-010A in one action. Until that instruction is received, this report remains `In Review` and IDR-SRV-010A remains unstarted.

---

## 19. References

### Controlling Standards

- [OGC API - Connected Systems - Part 1: Feature Resources, OGC 23-001](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2: Dynamic Data, OGC 23-002](https://docs.ogc.org/is/23-002/23-002.html)
- [OGC API - Features - Part 1: Core corrigendum, OGC 17-069r4](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [RFC 8288, Web Linking](https://www.rfc-editor.org/rfc/rfc8288.html)
- [IANA Link Relation Types](https://www.iana.org/assignments/link-relations/link-relations.xhtml)
- [Versioned OGC API - Features Part 1 schemas](https://schemas.opengis.net/ogcapi/features/part1/1.0/openapi/schemas/)

### Official CSAPI Artifacts and History

- [Official CSAPI `v1.0.0` source at commit `8e03b236`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)
- [Part 1 tagged OAS](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/openapi-connectedsystems-1.yaml)
- [Part 2 tagged OAS](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/openapi-connectedsystems-2.yaml)
- [Part 1 released OAS bundle](https://github.com/opengeospatial/ogcapi-connected-systems/releases/download/v1.0.0/ogcapi-connectedsystems-1.bundled.oas31.yaml)
- [Part 2 released OAS bundle](https://github.com/opengeospatial/ogcapi-connected-systems/releases/download/v1.0.0/ogcapi-connectedsystems-2.bundled.oas31.yaml)
- [Tagged shared Link schema](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/common/link.json)
- [Current audited official commit `3fd86c73`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f)
- [Glaux bounded upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md)

### Supporting Implementation, Client, and Test Evidence

- [Official OGC API - Features ETS](https://github.com/opengeospatial/ets-ogcapi-features10/tree/a314c1e6a9278b14ab9a2ed865cfe36d202f0125)
- [CSAPI Explorer](https://github.com/OS4CSAPI/ogc-csapi-explorer/tree/00f1c188e05738ee03390fd95f09d351e073a9c3)
- [OpenSensorHub core](https://github.com/OS4CSAPI/osh-core/tree/b2badae59aaa78455c5638ad73b452ccdee40207)
- [connected-systems-go](https://github.com/OS4CSAPI/connected-systems-go/tree/e900da88738cca92872038b703c4ad537fc0c8fd)
- [Restricted SECD preserved interoperability evidence](https://github.com/Sam-Bolling/csapi-server-interop-secd/tree/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0) (authenticated workspace; public 404)
- [OS4CSAPI testing/research exemplar corpus](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing)

### Accepted Project Inputs

- [IDR-SRV-006 report](idr-srv-006-csapi-part-1-requirement-baseline-report.md)
- [IDR-SRV-007 report](idr-srv-007-csapi-part-2-requirement-baseline-report.md)
- [IDR-SRV-008 report](idr-srv-008-conformance-class-and-requirement-mapping-report.md)
- [IDR-SRV-009 report](idr-srv-009-landing-page-api-definition-and-conformance-declaration-behavior-report.md)
- [Glaux Server goal and definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Research-report template](../../../../../Governance/research-report-template.md)

---

## Appendix A: Detailed Research-Question Coverage

### A.1 Collections and Collection Metadata

| ID | Plan question (short form) | Coverage | Evidence |
|---|---|---|---|
| C1 | Part 1 required/implied collections | Complete | §§5.1, 5.3 |
| C2 | Part 2 required/implied collections | Complete | §§5.1, 5.4 |
| C3 | Required/recommended collection metadata | Complete | §5.2 |
| C4 | Feature, dynamic, control, status, event categories | Complete | §§6.3–6.4 |
| C5 | Inherited Features behavior | Complete | §5.1 |
| C6 | Membership, IDs, collection and item links | Complete | §§5.2, 5.5 |
| C7 | Metadata effects on OAS, schema, discovery | Complete | §§5.2, 12.1 |

### A.2 Resource Categories and Endpoints

| ID | Plan question (short form) | Coverage | Evidence |
|---|---|---|---|
| R1 | Required resources across all named families | Complete | §§6.2–6.4 |
| R2 | Collection, item, subordinate, relationship endpoint types | Complete | §§6.1–6.3 |
| R3 | Static/descriptive versus dynamic/time-varying | Complete | §6.4 |
| R4 | Independently addressable resources | Complete | §§6.2–6.3, 10 |
| R5 | Nested, linked, referenced, derived resources | Complete | §§6.3, 8–9 |
| R6 | Canonical-model handoff to IDR-SRV-015 | Complete | §§10, 18.1 |

### A.3 Links and Link Relations

| ID | Plan question (short form) | Coverage | Evidence |
|---|---|---|---|
| L1 | Link objects, relations, href/type/title/role/profile | Complete | §§7.1–7.2 |
| L2 | Root, collection, item, related, API, conformance, alternate, schema links | Complete | §§7.2, 8 |
| L3 | CSAPI-specific relationship links | Complete | §§7.3–7.5 |
| L4 | Part 1-to-Part 2 links | Complete | §9 |
| L5 | IANA, Features, CSAPI, and other conventions | Complete | §§7.2–7.4 |
| L6 | External-client interoperability patterns | Complete | §§8, 12 |

### A.4 Navigation and Traversal

| ID | Plan question (short form) | Coverage | Evidence |
|---|---|---|---|
| N1 | Root → collections → items | Complete | §8.1 |
| N2 | System → deployment/procedure/streams/status/commands/events | Complete | §§8.2, 9 |
| N3 | Observation → stream/system/property/SF/result/time | Complete | §§8.3, 9 |
| N4 | ControlStream → command/status/feasibility/target | Complete | §§8.4, 9 |
| N5 | Reverse, parent-child, cross-resource links | Complete | §§8.5, 9 |
| N6 | Required versus implementation-choice traversal | Complete | §§7–9, 13 |

### A.5 Resource Identity and URI Implications

| ID | Plan question (short form) | Coverage | Evidence |
|---|---|---|---|
| I1 | Required/implied URI patterns | Complete | §§6.2–6.5 |
| I2 | Resources requiring stable URIs | Complete | §§10.1–10.2 |
| I3 | Resources requiring canonical item URLs | Complete | §§6, 10 |
| I4 | Alternate and external identifiers | Complete | §§7.5, 10 |
| I5 | Questions handed to IDR-SRV-016 | Complete | §10.3 |
| I6 | Lifecycle questions from navigation | Complete | §§10.3–10.4 |

### A.6 Validation, Error, and Consistency

| ID | Plan question (short form) | Coverage | Evidence |
|---|---|---|---|
| V1 | Membership/link/navigation validation | Complete | §§11.1–11.2 |
| V2 | Missing/unavailable/unauthorized/stale/deleted/moved | Complete; final policy deferred | §11.3 |
| V3 | Link consistency errors/warnings | Complete; response policy deferred | §§11.1–11.3 |
| V4 | Failure handoff to IDR-SRV-013 | Complete | §§11.3, 18.1 |
| V5 | Consistency/transaction handoff to IDR-SRV-029 | Complete | §§11.4, 18.1 |

### A.7 Interoperability and Existing Implementations

| ID | Plan question (short form) | Coverage | Evidence |
|---|---|---|---|
| X1 | Existing implementation navigation | Complete | §12.5 |
| X2 | Smoke-test/interoperability compatibility issues | Complete | §§12.4–12.6 |
| X3 | Explorer, external-client, OGC-tooling patterns | Complete | §§12.4, 12.6 |
| X4 | Implementation differences Glaux must account for | Complete | §§12.4–12.6, 13 |
| X5 | Handoffs to 014A–014G and 056 | Complete | §18.1 |

All 41 detailed questions are answered. “Final policy deferred” means the question's Topic 010 implication and downstream owner are established; it is not missing research.

---

## Appendix B: Consolidated 13-Field Behavior Matrix

**Abbreviations:** P1 = OGC 23-001; P2 = OGC 23-002; F = OGC 17-069r4; N/R/C/A/P/D use §4.2 classifications. Requirement numbers are from the named source.

| Family | Endpoint / navigation pattern | Source / anchor | Rule summary | Class | Required/recommended links | Related resources | Glaux implication | Conformance / requirement | Schema / OAS | Test implication | Handoff | Notes / unresolved |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Collections aggregate | `GET /collections` | F §7.13; P1/P2 API Common | Return advertised descriptors; top-level links/collections | N/P | `self`, alternates; `rel`+`type` | All advertised families | Generate complete configured inventory from registry | F Reqs 12–17 | `collections.yaml`; P1 OAS | Schema + descriptor/link walk | 012, 014, 023 | Features permits a limited selection when many collections exist; complete Glaux inventory is P |
| Collection detail | `GET /collections/{cid}` | F §7.14 | Named fields equal when present; every aggregate link included; detail may add links | N | inherited links; self/alternates recommended | One collection | Same descriptor object with detail additions | F Reqs 18–19 | `collection.yaml` | Named-field equality + aggregate-link inclusion | 014, 023 | Include CSAPI `featureType` consistency as P |
| Collection items | `GET /collections/{cid}/items` | F §7.15; P1 §7.5 + family collection reqs; P2 Req 2 and §8.3 | Return family-correct selection and applicable inherited paging behavior | N | self, alternates, next if more | Collection members | One query/view service | F Reqs 20–32 + CSAPI family reqs | Collection response schemas/OAS | Family purity, filters, counts, paging | 011–013, 015 | Feature wording adapted for resources |
| Collection item | `GET /collections/{cid}/items/{id}` | F §7.16; P1 §7.5/family reqs; P2 Req 2 | Retrieve member; link current view to collection and canonical resource | N/A/P | self, alternates, collection, canonical | One member/canonical item | Implement for all Glaux families | F Reqs 33–35; P2 Req 2; CSAPI canonical reqs | P1 generic item path | Identity equivalence and 404 | 013–016 | P2 route is N; P1 Property route is P because wording is not fully explicit |
| System | `/systems[/{id}]` | P1 Reqs 5–13 + association mappings | Canonical System view; top-level only by default when Subsystem applies | N/P | canonical; outgoing set relations including `ogc-rel:subsystems`, `ogc-rel:samplingFeatures`, `ogc-rel:deployments`, `ogc-rel:procedures`, `ogc-rel:datastreams`, and `ogc-rel:controlStreams`; direct `systemKind@link` | System graph, Procedures, P2 | Primary family route; preserve direct-property direction and case-insensitive relation identity | P1 /conf/system and related classes | P1 system schemas/OAS | Top-level/recursive sets, direct/generic placement, case comparison, IDs | 015–017 | Do not invent System FOI solely from Table 3; `recursive=true` reaches complete hierarchy |
| System hierarchy | `/systems/{id}/subsystems` | P1 Reqs 9–13 | Permanent membership; recommended datetime membership; direct/all descendants; association targets aggregate parent + descendants | N/R | `ogc-rel:subsystems`, `ogc-rel:parentSystem`, canonical | Parent/child Systems; SF/DS/CS aggregate sets | Cycle-safe graph projection | P1 /conf/subsystem | P1 OAS | 3 levels, time cases, cycles, dedup, aggregate relations | 011, 015, 017, 025 | Requirement 9 `CAN` is undefined; no nested item route |
| Deployment | `/deployments[/{id}]`; `/systems/{id}/deployments` | P1 Reqs 14–23 + mappings | Canonical Deployment view; top-level by default and complete with `recursive=true` when Subdeployment applies; System-scoped only related deployments when association exists | N | canonical; outgoing `ogc-rel:parentDeployment`, `ogc-rel:subdeployments`, `ogc-rel:samplingFeatures`, `ogc-rel:featuresOfInterest`, `ogc-rel:datastreams`, `ogc-rel:controlStreams`; direct `platform@link`/`deployedSystems@link`; incoming System→Deployment `ogc-rel:deployments` | Systems, subdeployments, SF/FOI, P2 streams | Preserve direction and relationship, not ownership | P1 /conf/deployment and /conf/subdeployment | P1 OAS | Default/recursive sets, direct/generic placement, condition-off, canonical | 015–018 | System-scoped association is capability-conditional; extension relations compare case-insensitively |
| Deployment hierarchy | `/deployments/{id}/subdeployments` | P1 Reqs 19–23 | Direct/all descendants; association targets aggregate parent + descendants | N | `ogc-rel:subdeployments`, `ogc-rel:parentDeployment`, canonical | Deployments; aggregated Systems/SF/FOI/DS/CS | Parallel hierarchy engine | P1 /conf/subdeployment | P1 OAS | Depth, cycle, dedup, aggregate relations | 011, 015, 017, 025 | Same identity as Deployment |
| Procedure | `/procedures[/{id}]` | P1 Reqs 25–28 | Canonical all/item; feature without geometry | N | canonical; outgoing `ogc-rel:implementingSystems`; incoming System→Procedure `ogc-rel:procedures` | Systems, streams | Preserve direction; no invented fixed reverse path | P1 /conf/procedure | P1 OAS/schema | Discovery, direction, canonical, external target | 015–017 | Association target path not fixed |
| Sampling Feature | `/samplingFeatures[/{id}]`; `/systems/{id}/samplingFeatures` | P1 Reqs 29–33 + mappings | Canonical all/item plus System-scoped related set | N/P | canonical; outgoing `ogc-rel:parentSystem`, `ogc-rel:sampleOf`, `ogc-rel:datastreams`, and `ogc-rel:controlStreams`; direct `sampledFeature@link`; incoming `ogc-rel:samplingFeatures` | System/domain features/DS/CS | Preserve direction/direct-property placement; empty view remains navigable | P1 /conf/sf | P1 OAS/schema | Every System, direction, membership, external sample, stream targets, case comparison | 015–017 | Table 3 `sampledFeature` placement conflicts; alternate case is the same relation identity |
| Property | `/properties[/{id}]` | P1 Reqs 34–37 | Canonical non-feature resource family | N | canonical; semantic URI fields are not ordinary association links | Systems/streams/observations | Resource-adapted collection/item | P1 /conf/property | P1 property schema/OAS | Non-feature envelope and identity | 012, 015–017 | Individual collection-item interpretation P |
| DataStream | `/datastreams[/{id}]`; System/Deployment scoped | P2 Reqs 3–11 | Canonical all/item; related stream views where stated conditions hold; schema | N/P | canonical; incoming Part 1 `ogc-rel:datastreams`; project-classified SF/FOI/schema/observation links | Systems, Deployment, SF/FOI, Obs | Registry guards every scoped edge | P2 /conf/datastream | P2 schemas/OAS; OAS gaps | All scopes; condition-off; valid-time intersection | 011–018 | Part 2 assigns no SF/FOI relation values; OAS omits deployment and SF/FOI routes |
| Observation | `/observations[/{id}]`; `/datastreams/{id}/observations` | P2 Reqs 12–16 | Canonical all/item and parent stream history | N | canonical; representation @id/@link; project traversal links | DS, System, Property, SF/FOI, Procedure | Keep result/id/link forms distinct | P2 /conf/datastream | observation schemas/OAS | Parent membership, result/schema, canonical | 015–018, 023, 034 | Nested item transaction-qualified |
| ControlStream | `/controlstreams[/{id}]`; System/Deployment scoped | P2 Reqs 17–25 | Canonical, related sets, schema, command parent | N/D/P | canonical; incoming Part 1 `ogc-rel:controlStreams`; project-classified SF/FOI/schema/command links | Systems, Deployment, SF/FOI, Command | Plural primary; deterministic table spelling with case-insensitive identity; `/controls` adapter record | P2 /conf/controlstream | P2 schemas/OAS | Literal/primary/adapter/case-equivalence tests | 010A, 014–018 | Part 2 assigns no SF/FOI relation values; Req URL says `/controls/{id}` |
| Command | `/commands[/{id}]`; `/controlstreams/{id}/commands` | P2 Reqs 26–30 | Canonical all/item and parent ControlStream set | N/D | canonical; project status/result links | ControlStream, SF/FOI, status/result | Plural primary, singular adapter record | P2 /conf/controlstream | command schemas/OAS | Parent membership, schema, aliases | 010A, 015–017, 036 | Requirement uses singular parent |
| Command Status/Result | `/commands/{id}/status[/{childId}]`; `/commands/{id}/result[/{childId}]` | P2 Reqs 31–34 + transactions | Parent-scoped histories/results; no global family | N/D | project `status`/result relation; external/result links | Command, Observations, DS, external | Parent-scoped IDs and route | P2 controlstream class | status/result schemas/OAS | Parent ID, variants, terminal history | 013, 015–017, 036 | Singular requirement vs plural evidence |
| Feasibility | `/feasibility[/{id}]`; ControlStream/status/result scoped | P2 §7.4, Reqs 35–39 | Canonical item, scoped tasking, optional typed collection; root list gap | N/C/P/D | canonical; project parent/status/result links | ControlStream, status/result | Expose root; plural primary | P2 /conf/feasibility | Command schemas; absent OAS | Sync/async, root, aliases, ATS defect | 010A, 014–017, 037 | No numbered canonical-list rule |
| SystemEvent | `/systemEvents[/{id}]`; `/systems/{id}/events` | P2 Reqs 40–44 | Canonical global events and System-scoped history | N/D | canonical; project System/events link | System | `events` nested primary | P2 /conf/system-event | event schema; partial OAS | Global/nested identity; ATS spelling | 010A, 013–017, 020 | One ATS says `systemEvents` |
| Stream schemas | `/datastreams/{id}/schema`; `/controlstreams/{id}/schema` | P2 Reqs 11, 25 | Return format-selected child payload schema | N | project schema/`describedby` link | DS/Observation; CS/Command | First-class discoverable target | P2 data/control classes | schema endpoints/OAS | Each supported format; target type | 012, 014, 017, 023 | Anchor semantics not finalized |
| Paging | Collection items and other resources endpoints whose family requirement imports the response behavior | F Reqs 21, 28–32 + applicable CSAPI incorporation | Limit selection; correct counts/time where required; opaque next | N/R/P | self, alternates; next if more | Current result set | Use uniform paging as P outside direct incorporation; token is not resource identity | Features Core + family class | envelope schemas | No loss/dup; expired 404; scope-specific applicability | 011, 013, 025 | Algorithm/duration unspecified; uniformity beyond direct incorporation is P |
| Compatibility aliases | `/controls`, singular parents, event ATS form, bare rels | P2 defects; P1 #173 | Never primary/canonical; optional named adapter | P/D | canonical points to primary; no normal advertising | Same underlying resources | Central adapter + telemetry | Not independent conformance class | OAS compatibility view only if chosen | Strict vs adapter lanes | 010A, 014, 050/051 | Lifetime and write methods unresolved |

---

## Appendix C: Publication and Artifact Contradiction Register

| ID | Conflict | Controlling interpretation for Glaux planning | Required treatment |
|---|---|---|---|
| C-01 | P2 ControlStream canonical requirement says `/controls/{id}`; overview/list/ATS/transactions/OAS use `/controlstreams/{id}` | Plural primary | Literal negative/adapter fixture; 010A alias decision |
| C-02 | Command nested requirement says `/controlstream/{id}/commands`; other evidence uses plural | Plural primary | Same |
| C-03 | Command status/result requirements say `/command/{id}/...`; other evidence uses plural | Plural primary | Same |
| C-04 | Feasibility nested requirement says singular ControlStream; transactions use plural | Plural primary | Same |
| C-05 | Feasibility overview/ATS assumes root list but class lacks numbered canonical-list requirement | Expose root as P | Preserve missing `SHALL` distinction |
| C-06 | Feasibility ATS A.35 checks Command `itemType`, A.36 checks the Commands path, and OAS omits Feasibility | Requirements + project overlay | Adapt ATS; retain each copied-target defect; generate OAS from registry |
| C-07 | SystemEvent requirement/OAS use `events`; ATS A.40 names ControlStream collections, A.42 invokes the ControlStream resources test, and A.43 says `systemEvents` | `events` primary; SystemEvent class controls | Separate copied-target and path adapter fixtures |
| C-08 | ControlStream SF requirement contains copied DataStream/observation terms and `dsId` | Intended ControlStream scoped edge | Supplemental semantic test |
| C-09 | Other ControlStream/Command endpoint prose contains copied DataStream/Observation nouns | Resource-class context controls | Negative semantic fixture |
| C-10 | P2 OAS omits Deployment streams, SF/FOI, Feasibility, and canonical SystemEvent item | Normative overlay controls | Generated complete OAS; path parity test |
| C-11 | P2 OAS retains removed System History | Not CSAPI 1.0 | Expected-absent conformance fixture |
| C-12 | Encoding sentence/examples/ATS disagree with `ogc-rel:` mapping intent | Prefixed table-first output | Bare-input compatibility; named ATS adapter |
| C-13 | Table spells `controlStreams` while model/mapping/endpoints use `controlstreams`; table applies FOI to System without a model edge and lists `sampledFeature` despite its direct-property placement | Spell output deterministically but treat both cases as one RFC-equivalent relation; do not invent System FOI or flatten direct properties | Case-insensitive identity test + separate lexical/coverage/placement conflict fixtures |
| C-14 | P1 OAS/examples contain wrong type markers, `item`, bad items URLs, missing type/collection links, `components` | Normative Features/CSAPI requirements control | Retain artifacts as negative fixtures |
| C-15 | Link schema requires only href while endpoint rules require rel/type and relation semantics | Semantic requirements control | Structural + semantic validation |
| C-16 | Release “bundles” retain relative references | Not standalone inputs | Vendor/pin complete graph; closure test |
| C-17 | Requirement 31(B) normative text names `limit` and `datetime` but cites only Features §7.15.2; `datetime` is §7.15.4 | Apply both inherited parameters from their actual clauses | Citation-defect fixture and requirement-to-test overlay |

No contradiction is silently discarded. This register is input to IDR-SRV-010A, 014, 017, 023, 050, and 051.

---

## Appendix D: Evidence and Test Artifacts to Preserve

| Artifact | Pin / identifier | Planned use |
|---|---|---|
| CSAPI official release source | `v1.0.0` / `8e03b236...` | Reproducible standards-support baseline |
| Current official source | `3fd86c73...` | Post-release delta watch |
| P1 modular OAS | SHA-256 `745fc5357a173e127f174f7413f36bc22b55f1b7cfa560902c63bb67dd82ea78` | Route/component differential |
| P2 modular OAS | SHA-256 `2df6c9b48bc19a21f0b44219b947a0d3be29e76f842cb9bb106c3cf7a5c9dd82` | Route/component differential |
| [P1 released bundle](https://github.com/opengeospatial/ogcapi-connected-systems/releases/download/v1.0.0/ogcapi-connectedsystems-1.bundled.oas31.yaml) | SHA-256 `69da631d5d05f01716381cca7b7ee6311402f2752a8fd79a9b72b663539555aa` | Residual-reference negative fixture |
| [P2 released bundle](https://github.com/opengeospatial/ogcapi-connected-systems/releases/download/v1.0.0/ogcapi-connectedsystems-2.bundled.oas31.yaml) | SHA-256 `86ed005f9e7cf176264d6deb72581a0b521a227cd7a198b6cb1bd32b39d83667` | Residual-reference negative fixture |
| Shared Link schema | SHA-256 `61660fcb768672af3221167d80f1f853a88f66ac7665affefc5db21bd7f50889` | Demonstrate structural/semantic gap |
| Official Features ETS | `a314c1e6...` | Inherited executable lane |
| CSAPI Explorer | `00f1c188...` | Standards-first/adversarial client lane |
| OpenSensorHub | `b2badae5...` | Differential server lane |
| connected-systems-go | `e900da88...` | Differential server/code-pattern lane |
| Restricted SECD evidence | `f018fd12...`; authenticated workspace only; public 404 | Preserved non-normative interoperability cases; never sole decision basis |

The future fixture repository should retain source license/provenance, exact checksums, expected authority class, and expected pass/fail interpretation. A mutable download without that metadata is not reproducible evidence.

---

## Appendix E: Plan Completion Validation

### E.1 Success Criteria

| Plan success criterion | Result | Evidence |
|---|---|---|
| Collection/item behavior identified with source anchors | Satisfied | §§3, 5–6; Appendix B |
| CSAPI-specific and inherited Features behavior distinguished | Satisfied | §§3.4, 5, 7 |
| Required/recommended/conventional/project links identified | Satisfied | §§4.2, 7; Appendix B |
| Root→collections→items→related navigation documented | Satisfied | §§8–9 |
| Part 1/Part 2 relationships identified | Satisfied | §9 |
| URI/ID/lifecycle/relationship implications handed off | Satisfied | §10; §18.1 |
| Validation/interoperability/test implications documented | Satisfied | §§11–12, 15 |
| Broken/stale/inconsistent/incomplete discovery risks identified | Satisfied | §§11, 17 |
| Recommendations decision-usable and bounded | Satisfied | §§13–14, 17.1 |
| References explicit and reproducible | Satisfied | §§3, 19; Appendix D |

### E.2 Required Deliverable Content

| Required content area | Location | Result |
|---|---|---|
| 1. Executive summary | §1 | Present |
| 2. Scope and plan alignment | §2; Appendix A | Present |
| 3. Evidence base / authority | §3 | Present |
| 4. Collection/resource endpoints | §§5–6; Appendix B | Present |
| 5. Links/navigation | §§7–8 | Present |
| 6. Part 1 / Part 2 relationships | §9 | Present |
| 7. Identity/URI/lifecycle | §10 | Present |
| 8. Validation/consistency | §11 | Present |
| 9. Interoperability/implementations | §12 | Present |
| 10. Test strategy | §15; Appendix C–D | Present |
| 11. Downstream handoff matrix | §18.1 | Present |
| 12. Recommendations | §§13–14 | Present |
| 13. Risks/constraints/open questions | §17 | Present |
| 14. Success-criteria validation | Appendix E.1 | Present |
| 15. References | §19 | Present |

### E.3 Minimum Behavior-Matrix Fields

Appendix B includes all thirteen required fields as explicit columns:

1. resource/collection family;
2. endpoint/navigation pattern;
3. source/anchor;
4. requirement/convention summary;
5. classification;
6. links;
7. related resources;
8. Glaux implication;
9. conformance class/requirement;
10. schema/OAS;
11. test implication;
12. downstream handoff; and
13. notes/unresolved issues.

### E.4 Methodology and Review Gate

All six phases are complete as recorded in §4.1. The report was reconciled against the accepted prerequisite reports and three independent read-only evidence audits. Automated document checks and the final independent completeness/accuracy review were completed before publication. Plan-owner acceptance remains intentionally pending.

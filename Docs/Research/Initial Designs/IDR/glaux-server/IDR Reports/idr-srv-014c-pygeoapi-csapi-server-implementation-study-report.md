# Section 014C: pygeoapi CSAPI Server Implementation Study - Research Report

**Topic ID:** IDR-SRV-014C<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-014C pygeoapi CSAPI Server Implementation Study](../IDR%20Plans/idr-srv-014c-pygeoapi-csapi-server-implementation-study.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core and 41 detailed questions; all six methodology phases, ten success criteria, nineteen required content areas, and twelve minimum implementation-findings fields are validated<br>
**Methodology Used:** Commit- and tag-pinned review of geopython core, the 52°North dependency fork, and the 52°North CSAPI proof-of-concept; route, provider, model, configuration, schema, OpenAPI, issue, pull-request, workflow, and documentation inventories; bounded live-deployment probes; and comparison with accepted IDR-SRV-006 through IDR-SRV-014B<br>
**Research Time:** Approximately 4.5 hours of AI-assisted elapsed execution on August 31, 2026<br>
**Primary Sources:** 52°North `connected-systems-pygeoapi` main commit `6ad75aa7fbfc76517c8a24c21446471845007855`; geopython pygeoapi release `0.24.0` at `f765a64fa65350dc93d9df0a1b38755bef6c31b0`; PoC dependency pin `ec1eb38d9a64d93ec9a2e1b9db6fea6dc05f194a`; approved OGC API - Connected Systems Parts 1 and 2; and accepted Glaux IDR-SRV-006 through IDR-SRV-014B reports<br>
**Supporting Resources:** PoC tag `v0.6` at `835cacb36145da0f46eab4b689157e00d820191c`; OGC schema submodule commit `636277919d96d2274844ccb981b22119d01e2f9e`; repository issues, pull requests, workflows, and two dated deployment probes current through August 31, 2026<br>
**Document Purpose:** Establish a reproducible pygeoapi/52°North CSAPI implementation findings baseline for Glaux architecture, configuration, persistence, representation, validation, conformance, interoperability, and test design without treating the framework or proof-of-concept as standards authority<br>
**Author:** OpenAI Codex<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** August 31, 2026<br>
**Date:** August 31, 2026<br>
**Last Updated:** August 31, 2026

---

## Reading Guide

Evidence labels used throughout are:

- **N — normative:** approved CSAPI or incorporated-standard evidence;
- **P — project baseline:** an accepted prior Glaux IDR conclusion;
- **C — code-supported:** behavior established in pinned source;
- **T — test-supported:** behavior exercised by pinned automated tests;
- **G — configuration-supported:** behavior or topology selected through configuration;
- **D — documented:** repository documentation, generated artifacts, or examples;
- **A — API-observed:** response observed from a dated public deployment;
- **H — history:** issue, pull request, tag, workflow, or maintainer evidence; and
- **I — inference:** bounded analysis rather than a directly observed runtime fact.

“pygeoapi core” means geopython's general OGC API framework. “PoC” means `52North/connected-systems-pygeoapi`. The PoC is not a normal pygeoapi configuration or a merged core plugin: it is a standalone Quart service that imports selected pygeoapi internals and registers custom providers and routes. A returned conformance URI is a claim, not proof. The approved standards and accepted Glaux reports control whenever implementation evidence differs.

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. pygeoapi Source Inventory and Evidence Classification
4. pygeoapi Architecture, Provider, and Configuration Findings
5. pygeoapi CSAPI Part 1 Behavior Findings
6. pygeoapi CSAPI Part 2 Behavior Findings
7. pygeoapi Conformance Posture and Standards-Alignment Matrix
8. pygeoapi API Behavior Findings
9. pygeoapi Dynamic-Data, Status, and Tasking Findings
10. pygeoapi Persistence, Configuration, and Fixture Implications
11. pygeoapi Documentation, OpenAPI, and Examples Findings
12. Interoperability and Client-Compatibility Findings
13. Test-Strategy Implications
14. Lessons for Glaux Server
15. Downstream Topic Handoff Matrix
16. Recommendations
17. Risks, Constraints, and Open Questions
18. Validation Against the Research Plan
19. References
Appendix A. Detailed Question Ledger
Appendix B. Reproducible Study Record
Appendix C. Completion and Handoff

---

## 1. Executive Summary

Current geopython pygeoapi does not implement OGC API - Connected Systems. The relevant implementation is 52°North's separate `connected-systems-pygeoapi` proof-of-concept. Its current main commit is active and labels the package `0.6.0`, but the last tag, `v0.6`, dates to October 2025 and the project publishes no GitHub Releases. It depends on an exact upstream pygeoapi commit reporting `0.22.dev0`, reached through a 52°North fork URL but also present unchanged in geopython history. The PoC's own open issue #10 accurately says development diverged from seamless pygeoapi integration into a standalone Quart service that uses pygeoapi as a library.

The PoC supplies useful implementation evidence. It extends pygeoapi request, template, collection, OpenAPI, plugin-loader, and provider concepts; introduces CSAPI resource routes; uses Elasticsearch for Part 1 metadata and datastream descriptions; uses TimescaleDB/PostGIS for observations and derived time extents; bundles approved-tag OGC JSON Schemas for local request validation; supports SensorML JSON, GeoJSON, JSON, OM JSON, HTML, and configuration-driven providers; and places Basic authentication in front of write routes by default. These are practical patterns for configuration, provider isolation, schema bundling, mixed static/dynamic collections, and horizontal storage scaling.

The implementation is nevertheless a prototype, not a conformance or production baseline. Main contains 33 Python files, zero automated test files, 40 explicit TODO/not-supported markers, six open pull requests, and numerous open behavior issues. The tag-triggered workflow builds/publishes a container but does not run a test suite. Local execution was unavailable because the research environment had neither Python nor Docker, and the current dependency declaration contains both `shapely==2.0.6` and nonexistent `shapely==2.6.0`; open PR #23 documents that normal local dependency resolution fails while the Docker/system-package workaround masks it.

The largest design warning is representation-dependent storage. Elasticsearch documents contain separate nested `smljson` and `geojson` fields, and incomplete transcoders can leave one view empty or semantically different. On the dated Oracle-hosted deployment, systems and procedures were populated through `application/sml+json` but empty through `application/json`; datastreams and observations were populated separately. The deployment also still returned empty results for both the sampling-feature `system` filter and equivalent nested route despite top-level sampling features being populated. Current main includes newer JSON-fallback code, so the deployment cannot be treated as a current-main runtime; its OpenAPI metadata still reports `0.16.dev0`. The observation remains decisive as an interoperability scenario: every representation must be generated from one canonical resource and tested for equivalent identity, fields, relationships, and population.

Other material findings include:

- only systems, deployments, procedures, sampling features, properties, datastreams, datastream schemas, and observations appear in the PoC entity enum; generic features, system events/history, control streams, commands, status/results, and feasibility are absent;
- `/conformance` returns only Common Part 1 core and intentionally makes no CSAPI claim; this is safer than overclaiming but supplies no evidence of CSAPI class satisfaction;
- code declares 49 unique abstract CSAPI method/path pairs, while the checked-in OpenAPI has only 11 paths and 18 operations, four operation IDs, no security schemes, stale localhost/`0.16.dev0` metadata, and ten malformed external references containing backslashes;
- source configuration for Part 2 strict validation mistakenly updates the Part 1 flag, so configured validation behavior is not reliable;
- Part 1 replace always constructs a `System`, regardless of requested resource type, and PATCH is commented out;
- cross-store mutations have no distributed transaction, startup performs ad hoc table/index creation, an unlogged derived datastream-time table has incomplete crash recovery, and its update trigger uses incorrect comparison columns;
- offset paging is explicitly nonstandard, Elasticsearch itself notes the need for `search_after` beyond 10,000 records, and “next” links can be emitted merely because a page is full;
- errors use legacy `{code,type,description}` bodies and broadly map provider/internal failures to 400, not the accepted Glaux problem/status model; and
- Basic authentication is shared-route protection rather than fine-grained policy, is absent from OpenAPI, and deployment examples contain fixed development credentials.

Glaux should reuse the provider boundary, configuration layering, schema-bundle reproducibility, separate metadata/time-series optimization, and capability-oriented route composition. It should reject representation-specific canonical storage, dual-write without an outbox/transaction strategy, untyped environment overrides, ad hoc startup migration, unchecked OpenAPI drift, and tests externalized into live demos and issues. One typed canonical model and capability registry must drive storage, every representation, routes, navigation, conformance, OpenAPI, authorization, validation, and tests.

Acceptance of this report would make IDR-SRV-014D the next authorized topic. It would not assert that geopython core supports CSAPI, certify the 52°North PoC, authorize source copying, or choose Glaux's Rust framework and persistence engines.

## 2. Scope and Plan Alignment

### 2.1 Included and Excluded Scope

The study directly reviewed geopython pygeoapi's current release and provider/configuration model, the exact pygeoapi dependency commit, the PoC's main and last tag, custom routes/providers/models/configuration, bundled schemas and their generation tool, OpenAPI, container/workflow, architecture documentation, open issues and pull requests, and public deployments identified in issue evidence. It compared these sources with accepted IDR-SRV-006 through IDR-SRV-014B.

The study does not certify conformance, benchmark storage, test authenticated mutations, mutate public deployments, audit every pygeoapi provider, resolve the PoC roadmap, or decide Glaux technology choices. No Python/Docker execution was possible, and the PoC contains no automated test suite to execute. Live observations are GET-only and deployment-specific.

### 2.2 Plan Coverage Matrix

| Plan question group | Coverage | Evidence location |
|---|---|---|
| Repositories, branches, pins, deployments, issues, artifacts, licensing | Complete | Sections 3 and 17; Appendix B |
| Core architecture, PoC boundary, providers, plugins, and configuration | Complete | Section 4 |
| Part 1 and Part 2 behavior | Complete | Sections 5–7 |
| Landing, navigation, query, negotiation, errors, and OpenAPI | Complete | Sections 8 and 11 |
| Dynamic data, events, streaming, status, and tasking | Complete | Sections 6 and 9 |
| Data sources, deployment, schemas, and fixtures | Complete | Section 10 |
| Validation and test evidence | Complete within explicit limitations | Section 13 |
| Client and interoperability observations | Complete within available deployments | Section 12 |
| Glaux lessons, recommendations, risks, and handoffs | Complete | Sections 14–17 |

## 3. pygeoapi Source Inventory and Evidence Classification

### 3.1 Reproducible Source Baseline

| Source | Role | Pin / state | Evidence | Availability / limitation |
|---|---|---|---|---|
| [52°North CSAPI PoC main](https://github.com/52North/connected-systems-pygeoapi/tree/6ad75aa7fbfc76517c8a24c21446471845007855) | Current CSAPI source baseline | Commit `6ad75aa7fbfc76517c8a24c21446471845007855`; tree `a9cd2b7163709251895c055ba9e1a9314375f8f4`; 2026-08-17 | C/G/D/H | Active main, not an immutable release; no automated tests. |
| [PoC tag `v0.6`](https://github.com/52North/connected-systems-pygeoapi/tree/835cacb36145da0f46eab4b689157e00d820191c) | Last tagged baseline | Commit `835cacb36145da0f46eab4b689157e00d820191c`; tree `3cca1588ef1a5cd25b32d50ee004006512f8b1b8`; 2025-10-21 | H/C | Main has 477 additions and 268 deletions across 21 files since the tag. No GitHub Release exists. |
| [geopython pygeoapi `0.24.0`](https://github.com/geopython/pygeoapi/tree/f765a64fa65350dc93d9df0a1b38755bef6c31b0) | Current core-framework comparison | Tag commit `f765a64fa65350dc93d9df0a1b38755bef6c31b0`; tree `f81efc93526b560b41a79e3ab63b604cf0c0ac6`; 2026-07-28 | C/T/D | Contains no CSAPI-specific implementation; informs framework lessons only. |
| [PoC pygeoapi dependency](https://github.com/geopython/pygeoapi/tree/ec1eb38d9a64d93ec9a2e1b9db6fea6dc05f194a) | Actual framework revision used by source | Commit `ec1eb38d9a64d93ec9a2e1b9db6fea6dc05f194a`; `0.22.dev0`; 2025-10-20 | C | URL points to 52°North fork, but the exact commit is unchanged in geopython history. |
| [OGC schema submodule](https://github.com/opengeospatial/ogcapi-connected-systems/tree/636277919d96d2274844ccb981b22119d01e2f9e) | Source for local schema bundles | Commit `636277919d96d2274844ccb981b22119d01e2f9e`; tagged `v1.0.0`; 2025-03-24 | N/tool input | Bundle generation rewrites identifiers/references and request-required sets; generated output is not byte-identical normative source. |
| [PoC issues](https://github.com/52North/connected-systems-pygeoapi/issues) and [pull requests](https://github.com/52North/connected-systems-pygeoapi/pulls) | Defects, roadmap, live observations, pending fixes | State retrieved 2026-08-31 | H | Mutable; source and dated probes distinguish current behavior. |
| `https://csa.demo.52north.org/` | Historically documented live deployment | Probed 2026-08-31 | A | TLS validation failed in the study environment; no current positive responses established. |
| `https://129-80-248-53.sslip.io/csapi-pygeoapi/` | Oracle-hosted live deployment named in issue #27 | Probed 2026-08-31 | A | Available, but OpenAPI reports `0.16.dev0`; deployed code/config pin is unknown and not equated to current main. |
| Accepted IDR-SRV-006–014B and approved CSAPI Parts 1–2 | Controlling comparison | Accepted/published pins in prior reports | N/P | Control over all implementation precedent. |

### 3.2 Artifact Classification

The PoC has 134 tracked files: 33 Python source/tool files, ten generated JSON Schema bundles, one 4,000-plus-line OpenAPI YAML, HTML/templates/static viewer material, an arc42-derived generated documentation site, one Dockerfile, one development Compose file, request snippets, a simulator, a schema bundler, and one image-publish workflow. It has zero test files.

The geopython core `0.24.0` release has 483 tracked files, 164 Python files, 71 Python test files, six workflows, CITE configuration, provider/formatter/process/manager/pubsub plugins, and Flask/Starlette/Django entry points. Those tests and CITE materials validate core features, not this separate CSAPI service.

The PoC's OGC submodule and bundler are provenance evidence. The ten bundled schemas are modified request-validation artifacts: the tool rewrites `$id`/`$ref` values to a `connected-systems.n52` base and removes read-only properties from required lists. That is a legitimate request-schema transformation idea, but it must not be confused with unmodified normative response schemas.

### 3.3 Licensing and Evidence Strength

The PoC includes Apache-2.0 license text. pygeoapi core is MIT-licensed. The OGC source schemas and generated bundles require separate standards-artifact provenance/licensing review before copying. Glaux may independently apply architecture lessons; direct reuse must retain component-level provenance and compatible notices.

Code plus a passing PoC test would be strong evidence, but no PoC tests exist. Code is moderate evidence. Successful image build/publish runs establish build completion at `v0.6`, not functional or conformance correctness. Issue reports and live probes are dated observations. Documentation contains both real architecture content and unreplaced arc42 template guidance, so every claim is corroborated against source where possible.

## 4. pygeoapi Architecture, Provider, and Configuration Findings

### 4.1 Three-Layer Architecture Boundary

The relevant stack has three distinct layers:

1. **geopython core:** an OGC API framework with YAML configuration, provider/formatter/process/manager/pubsub plugins, landing/collections/OpenAPI/conformance helpers, templates, and multiple web adapters;
2. **the dependency pin:** an upstream `0.22.dev0` commit fetched through a 52°North fork URL but not carrying a PoC-specific fork delta at that pin; and
3. **the PoC service:** a custom Quart application importing pygeoapi internals, overriding collection/meta/request behavior, registering custom blueprints, and injecting provider classes into pygeoapi's plugin dictionary.

The PoC is therefore evidence for selective framework reuse, not evidence that CSAPI can be expressed as ordinary pygeoapi YAML or a standard provider plugin. Open issue #10 and PR #26 explicitly seek documentation of this divergence.

### 4.2 Composition and Route Model

`app.py` constructs `CustomQuart`, applies CORS and Basic-auth configuration, registers read and read/write blueprints, registers selected core OGC API blueprints, exposes landing/OpenAPI/conformance/health routes, and opens/closes provider connections during service lifecycle. `api.py` subclasses the meta behavior, multiplexes Part 1 versus Part 2 providers, parses requests, validates mutations, formats responses, and registers two provider names in `PLUGINS`.

This creates a recognizable composition root, but much behavior depends on wildcard `<path:path>` routes and request-path parsing. Typed resource/method metadata is not the source of truth for routes, OAD, conformance, and authorization. Glaux should retain composable resource modules while generating all contract surfaces from a typed registry.

### 4.3 Provider and Storage Model

The Part 1 provider uses Elasticsearch document classes for collections, systems, deployments, procedures, sampling features, properties, and datastream descriptions. The Part 2 provider uses TimescaleDB/PostgreSQL for observations and a small unlogged table for derived datastream time bounds, while referencing Elasticsearch for datastream metadata.

The split matches workload types and offers horizontal-scaling potential. It also creates cross-store consistency hazards. Datastream creation saves Elasticsearch first and then inserts the Timescale row without a shared transaction or compensating action. Deletes and derived-time updates span stores. Glaux should use one transaction where feasible or an outbox/saga with explicit consistency state, idempotency, repair, and observability.

### 4.4 Representation Model

Part 1 Elasticsearch documents store separate nested `smljson` and `geojson` representations. Queries select `_source` using the requested format. Transcoders are partial: GeoJSON-to-SensorML returns an empty object, and several mappings use implementation-specific or questionable source fields. This is the mechanism behind format-dependent resource visibility and semantic drift.

Glaux requires one encoding-neutral canonical domain model. SensorML, GeoJSON, JSON, HTML, and future formats must be projections over the same identity and relationships. A representation may omit format-inapplicable fields, but it must not select a different population or canonical resource.

### 4.5 Configuration Model

Core pygeoapi uses typed-by-convention YAML sections for server, metadata, resources, providers, and formatters. The PoC adds `dynamic-resources` entries for Part 1 and Part 2 providers and accepts `CSA_` environment overrides whose underscore encoding traverses configuration paths.

This is flexible for demos and multi-source deployments. It is not type-safe: environment values remain strings, missing paths are created dynamically, and later `bool(value)` conversion makes strings such as `"False"` truthy. A separate bug assigns Part 2's `strict_validation` setting to `strict_validation1`, leaving Part 2 unchanged. Glaux should deserialize configuration into versioned typed structures, reject unknown keys, distinguish secrets, validate cross-field invariants, and expose an effective redacted configuration/capability view.

## 5. pygeoapi CSAPI Part 1 Behavior Findings

### 5.1 Resource Coverage

The entity enum and routes cover systems/subsystems, deployments, procedures, sampling features, properties, and mandatory CSAPI collections. Systems expose nested subsystems, deployments, sampling features, and datastreams. The PoC merges core-configured normal collections with dynamic CSAPI collections.

Generic CSAPI feature resources, subdeployment traversal, history, and system events are not represented by entity types. Root create routes exist for systems, deployments, and procedures; sampling features and datastreams are created in nested system context. Property routes do not register POST. PATCH logic is commented out.

### 5.2 Mutation Correctness

Part 1 create dispatches resource-specific document types and performs some duplicate, parent, procedure, and deployment-system link checks. Replace does not: it always builds a `System` regardless of the requested Part 1 entity. Deletion checks some system children, but cascade returns a `ProviderGenericError` object rather than completing or raising it, and datastream/control-stream dependencies are TODOs.

The generic upsert wrapper catches every exception and reports HTTP 400 `InvalidParameterValue`, including internal provider faults. This obscures conflicts and server failures. Glaux should use typed commands and errors per resource, then test create/replace/update/delete independently for every class.

### 5.3 Encodings and Relationships

SensorML JSON and GeoJSON are intended for systems, deployments, and procedures; GeoJSON is used for sampling features; SensorML JSON for properties. Mandatory collection documents include navigation links to the corresponding roots.

Relationship handling is incomplete. Current open issue #27 and the dated Oracle probe show the parent-system sampling-feature filter and nested route returning empty while top-level/item reads work. Elasticsearch maps the association as analyzed text with a keyword subfield, while current filtering targets the analyzed field. Link construction also mixes local IDs, UIDs, `urn`, and `@link` members across models.

### 5.4 Query Behavior

Part 1 parameter classes include `id`, `q`, `datetime`, `bbox`, `geom`, parent/system/procedure/property associations, `limit`, and explicitly nonstandard `offset`. Unknown parameters are ignored because parsing iterates only the declared list. Invalid numbers and times become 400 errors.

Elasticsearch paging uses slices and source selection. Source comments acknowledge `search_after` is required above 10,000 records. The “next” condition uses `count >= limit + offset`, which emits a next link when the current page ends exactly at the total. There is no stable sort/cursor contract. Glaux should use deterministic keyset paging and supplied navigation links, not expose offset as its scalable baseline.

## 6. pygeoapi CSAPI Part 2 Behavior Findings

### 6.1 Implemented Scope

Part 2 scope is limited to datastreams, datastream schemas, and observations. The service exposes top-level and system-nested datastream reads, system-nested datastream creation, schema GET/PUT, nested observation GET/POST, top-level observation GET, item GET/PUT/DELETE routes, and provider methods for query/create/replace/delete.

Observation replacement/update explicitly reports unsupported. No system-event, status, control-stream, command, result, or feasibility entity/route exists. This is a narrow observation PoC, not broad Part 2 support.

### 6.2 Observation Persistence and Querying

Observations are stored in a Timescale hypertable with UUID, result time, phenomenon time, datastream ID, JSONB result, sampling feature, procedure link, and parameters. Current source sorts ascending by phenomenon time and supports ID/datastream/time/limit/offset filters. Limit is capped below 100,000, but nonpositive values are not robustly constrained.

A full page is assumed to imply a next page, producing possible empty-next navigation. Offset paging under concurrent inserts is not snapshot-stable. “latest” remains a TODO. No selection, result-format negotiation beyond JSON/OM JSON, or SWE text/binary implementation was found.

### 6.3 Derived Datastream Time Extents

An unlogged `datastreams` table caches time bounds. Startup attempts recovery, but the branch for observations present and cache empty logs `TODO: Restore datastreams` without rebuilding. The insert trigger calculates `resulttime_end` from `resulttime_start` and `phenomenontime_end` from `phenomenontime_start`, producing incorrect extrema after multiple observations and handling initial NULLs poorly.

This is useful negative evidence. Derived metadata needs a declared source of truth, transactional update or rebuildable materialized view, backfill, crash recovery, and verification tests. Glaux should not make a lossy unlogged cache part of its externally visible canonical state without deterministic repair.

### 6.4 Dynamic Behavior

The PoC supports persisted historical HTTP retrieval only. Its future-work document lists MQTT/AMQP/WebSockets as future push-ingestion work. Core pygeoapi has general pubsub plugins, but the PoC does not connect them to CSAPI live observation, replay, event, status, or command semantics. Framework capability is not PoC CSAPI behavior.

## 7. pygeoapi Conformance Posture and Standards-Alignment Matrix

### 7.1 Conformance Declaration

The Part 1 provider returns only `http://www.opengis.net/spec/ogcapi-common-1/1.0/conf/core` with a source TODO to verify actual support. The Part 2 provider returns an empty list, and the meta handler consults only Part 1. Both inspected live `/conformance` behavior and pinned source therefore make no Connected Systems claim.

This underclaim is safer than the static overclaims found in IDR-SRV-014A/B, but it is not proof of Common core satisfaction and gives capable clients no standards-based way to discover the implemented CSAPI subset. No official CSAPI ATS result or PoC conformance test exists.

### 7.2 Implementation Findings Matrix

| Area | pygeoapi / PoC anchor | Evidence | Observed behavior | CSAPI / Glaux baseline | Alignment | Strength / useful pattern | Gap / risk | Glaux applicability | Test implication | Handoff | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Core framework | pygeoapi `0.24.0` | C/T/D | YAML resources, plugins, helpers, multiple web adapters | Modular service baseline | Informative | Mature provider/config ecosystem | No core CSAPI support | Reuse boundary ideas only | Framework adapter contract tests | 015, 018 | Core tests do not cover PoC |
| Integration boundary | PoC `app.py`, `api.py`, issue #10 | C/H | Standalone Quart service importing pygeoapi internals | Explicit component ownership | Partial | Selective reuse enabled fast PoC | Private/internal API coupling and identity confusion | Keep stable internal traits | dependency-upgrade tests | 018, 044 | PR #26 documents reality, still open |
| Configuration | `default-config.yml`, env override | G/C | Static resources plus dynamic Part 1/2 providers | Typed, traceable configuration | Useful/divergent | Flexible demos and multiple sources | String coercion, unknown-key creation, secret/default drift | Typed versioned config | config matrix/redaction tests | 047, 052 | Part 2 validation flag bug |
| Canonical model | Elasticsearch document models | C/A/H | Separate SML/GeoJSON nested sources | One resource, multiple representations | Divergent | Fast source-format retrieval | Empty/different populations and lossy transcoding | Reject representation-specific truth | cross-format identity/round-trip | 015, 023, 056 | Highest design lesson |
| Part 1 resources | enum/routes/provider | C | Systems, deployments, procedures, SFs, properties | Approved Part 1 families/classes | Partial | Broad metadata prototype | Missing generic features/history/events and class completeness | Use behavior inventory | per-resource CRUD/relationship suite | 015, 019 | Property POST absent |
| Part 2 resources | Part 2 provider/routes | C/A | Datastreams, schemas, observations | Approved dynamic/tasking families | Narrow partial | Real historical time-series persistence | No events/status/control/command/feasibility/SWE encodings | Separate capability modules | positive and absent-class tests | 034, 036 | No live/replay transport |
| Conformance | provider `get_conformance` | C/A | Only Common core returned | IDR-SRV-008 accepted class set | Underdeclared/unclear | Avoids CSAPI overclaim | Not capability/test-derived; CSAPI undiscoverable | Generate from verified registry | exact capability declaration | 050, 051 | Common claim has TODO |
| Routing | `routes/csa.py` | C | 49 unique abstract method/path pairs | Canonical routes/methods | Partial | Read/readwrite blueprint split | Wildcard parsing; invalid collection-level verbs; no PATCH | Typed route metadata | method/path/error inventory | 019, 050 | 69 declarations include duplicates |
| Queries | params/util/providers | C/A/H | ID/q/time/spatial/relations/limit/offset | Canonical vocabulary and semantics | Partial/divergent | Central parser | Unknown ignored, latest missing, association/filter bugs | Typed per-resource grammar | negative/boundary/cross-route | 011, 051 | Offset explicitly nonstandard |
| Pagination | ES/Timescale query code | C | Offset slices and unconditional-full-page next | Supplied links and scalable paging | Divergent | Simple demo behavior | false next, 10k ES limit, write drift | Opaque keyset cursors | terminal/concurrent/tamper tests | 011, 025 | No stable order Part 1 |
| Negotiation | `AsyncAPIRequest`, live probes | C/A/H | `f` and Accept order; format-specific sources | Accepted negotiation/status behavior | Partial/divergent | Multiple encodings and q-token parsing | Ignores q weights; unsupported Accept falls back; `f` error is 400 | Central standards negotiator | q/wildcard/406/415/Vary | 012, 056 | Current main adds JSON fallback |
| Validation | bundle tool, ten schemas | C/G | Draft 2020-12 local request validation | Approved schema + semantic validation | Useful partial | Offline reproducible bundles | Rewritten schemas; config bug; no tests; broad 400 | Separate request/response schemas | provenance and mutation corpus | 023, 053 | Pin is tagged `v1.0.0` |
| Errors | `get_exception`, wrappers | C/A | `{code,type,description}`, broad 400, framework HTML 404 | IDR-SRV-013 problem contract | Divergent | Consistent OGC-style body on handled paths | Conflict/internal/auth/media distinctions lost | Typed central mapper | exact status/body/header matrix | 013, 052 | Provider status attribute ignored |
| Persistence | ES + Timescale/PostGIS | C/D | Metadata/time-series split | Durable canonical store | Useful partial | Workload-specialized stores | Dual-write, startup DDL, derived-cache corruption/recovery | Adapt only with consistency design | failure/recovery/migration tests | 025, 028 | No versioned migrations |
| OpenAPI | `default-openapi.yml`, `/openapi` | C/D/A | 11 paths, 18 ops, 126 source schemas | Complete deployment OAD | Divergent | OAS 3.1 and rich schema components | Most routes absent; four IDs; stale info; malformed refs; no security | Generate/verify from registry | route/OAD/ref/client CI | 014, 050 | Live has 129 schemas, same path set |
| Security | Basic-auth blueprints/config | C/G/A | Read open default; writes Basic-auth default; live wildcard CORS | Later Glaux zero-trust baseline | Prototype only | Random defaults avoid known write password when unset | Shared Basic auth, no scopes/tenants/audit, absent OAD | Security policy first | principal/tenant/command tests | 038, 039A, 052 | Fixed DB demo credentials |
| Testing/release | repo/workflow/issues | T/H | No PoC tests; tag-only image build | TDD and traceable releases | Divergent | Build workflow and live feedback exist | Bugs discovered in demos; no CI behavior gate/release objects | Tests before feature claims | unit/integration/ATS/interop/release | 050, 051, 053 | Six open PRs |

## 8. pygeoapi API Behavior Findings

### 8.1 Entry Points and Navigation

The PoC exposes `/`, `/openapi`, `/conformance`, `/collections`, collection items, CSAPI roots, a `/connected-systems/` overview, `/status`, `/metrics`, and selected core OGC API routes. The landing page and overview build links programmatically. Mandatory CSAPI collections are stored in Elasticsearch and recreated at provider setup, while configured core collections are merged into the response.

This demonstrates mixed framework and domain collections but makes discovery dependent on successful live provider calls. Core pygeoapi issue #2141 separately illustrates how provider instantiation/metadata retrieval can make `/collections` slow. Glaux should cache capability/collection metadata with explicit freshness and health semantics, not require every backend to answer synchronously for discovery.

### 8.2 URI and Method Behavior

Mechanical parsing found 69 read/readwrite route declarations representing 49 unique abstract method/path pairs. Wildcard item paths reduce explicit route clarity. Several collection routes advertise PUT or DELETE without an item path even though handlers dereference `path[1]`; the resulting behavior is not a valid collection mutation contract. CORS/live headers advertise PATCH while PATCH is commented out.

Glaux should enumerate every method/path/resource pair statically and generate method-not-allowed behavior, OpenAPI, auth scopes, and tests from that set.

### 8.3 Query and Pagination

Query parameters are represented by dataclasses, which is a useful typed direction. Parsing still accepts only listed fields while silently ignoring others, performs direct `int`/`fromisoformat` conversion, and allows configuration/provider semantics to leak into public behavior. Date parsing uses local `now` and permits implementation-dependent naive timestamps. `latest` is TODO.

Both ES and Timescale use offset. Part 1 may hit Elasticsearch's 10,000 window, and next-link conditions can point to empty pages. Part 2 orders by phenomenon time ascending in current main, while issue/PR history debates result-time versus phenomenon-time order. Glaux must make default ordering and `latest` normative-baseline decisions, with tuple tie-breakers and deterministic paging.

### 8.4 Content Negotiation

The custom request class recognizes HTML, JSON, GeoJSON, SensorML JSON, OM JSON, and SWE JSON aliases. It gives `f` precedence and walks Accept entries in header order after stripping parameters. It does not evaluate q weights. Unknown `f` returns HTTP 400 `InvalidMimetype`; on the live deployment, unsupported `Accept: application/xml` and `*/*` fell back to the default SensorML response rather than 406.

Current main adds an application/json fallback to a concrete Part 1 representation, but a response must not merely change format: it must return the same resource population. The live Oracle snapshot's empty JSON versus populated SensorML results remain a required interoperability regression even if main's latest fallback fixes it.

### 8.5 Error Behavior

Handled errors use `code`, `type`, and `description`. Live invalid format and limit probes returned useful 400 bodies; an unknown path returned Quart's HTML 404. Provider conflict status attributes are ignored by wrappers that always choose 400, and broad exception catches can expose raw exception arguments as client errors. Other provider-generic failures may escape to framework 500 behavior.

Glaux should use the accepted centralized problem model and explicitly separate syntax, schema, semantic, not found, conflict, authn, authz, unsupported media, unacceptable response, throttling, dependency, and internal errors.

## 9. pygeoapi Dynamic-Data, Status, and Tasking Findings

### 9.1 Historical Observations

The PoC's only CSAPI dynamic-data behavior is persistent historical observation ingestion and retrieval, datastream description/schema handling, and derived time ranges. JSONB results support heterogeneous scalar/structured values, and TimescaleDB is a credible time-series optimization. There is no evidence for a complete result-type/encoding matrix or large-object/coverage strategy.

### 9.2 Streaming, Replay, and Events

No CSAPI WebSocket, SSE, MQTT, AMQP, replay, or event-publication implementation exists. The architecture site's future-work section lists push ingestion through MQTT/AMQP/WebSockets. Core pygeoapi pubsub plugins are used for general framework workflows and cannot be counted as Connected Systems transport behavior.

### 9.3 Status and Tasking

The `/status` endpoint is application health, not CSAPI system/command status. No system-event resource, control stream, command, command status/result, or feasibility route/model/provider appears. Glaux should keep operational health, resource status, system events, tasking state, and transport notifications as separate concepts.

### 9.4 Handoff Lesson

The split store shows why dynamic-data design must cover more than table shape: canonical ownership, association integrity, schema version, derived metadata, ingest idempotency, cross-store consistency, time ordering, live/history boundary, and recovery all affect API correctness.

## 10. pygeoapi Persistence, Configuration, and Fixture Implications

### 10.1 Elasticsearch and TimescaleDB

Elasticsearch provides flexible nested documents, full-text filters, geo queries, and format-specific source selection. TimescaleDB/PostGIS provides time-series storage and SQL time filtering. Architecture documentation explicitly chooses the split for metadata versus observations and delegates scale to providers.

Glaux may consider separate read models or specialized stores, but canonical writes and identities must remain coherent. Denormalized format documents should be derived/rebuildable projections, not the only truth. Cross-store operations require transactional or eventual-consistency contracts with repair.

### 10.2 Schema Evolution and Recovery

Provider startup creates indices, tables, extensions, hypertables, triggers, and mandatory collection documents. There is no numbered migration history. The unlogged cache recovery is unfinished. Recreating mandatory collections by delete/save on every startup can overwrite operator changes and makes startup stateful.

Glaux should use explicit versioned migrations and idempotent seed-data ownership. Derived views need rebuild commands, checksums/version markers, health degradation, and repair tests.

### 10.3 Configuration and Secrets

The default config contains localhost endpoints and development database passwords; Compose exposes Timescale, Elasticsearch, Kibana, and pgAdmin with fixed credentials. The app service in `docker-compose-dev.yml` is commented out, and README references untracked `docker/examples` plus removed requirements files. Environment override paths include hyphenated dynamic-resource keys and untyped values, increasing operational error risk.

Glaux should provide schema-validated configuration, secret references rather than values, safe environment profiles, redacted diagnostics, and deployable examples continuously exercised in CI.

### 10.4 Fixtures and Simulator

The repository includes request snippets and a Python observation simulator, while bundled schemas can generate positive/negative fixtures. No committed response-golden corpus or automated use of these assets exists. Live deployments have effectively served as integration fixtures, which is useful for discovery but unstable for regression.

Glaux should preserve local deterministic scenarios for systems, relationships, datastreams, observations, errors, and representation equivalence. External demos then become a separate interoperability layer.

## 11. pygeoapi Documentation, OpenAPI, and Examples Findings

### 11.1 OpenAPI Inventory

The checked-in OpenAPI 3.1 document has 11 paths, 18 operations, only four operation IDs, approximately 126 source component schemas, no security scheme, a localhost server, `pygeoapi default instance` title, and `0.16.dev0` version. Ten external OGC schema references contain a Windows-style backslash after `schemas.opengis.net`, making them invalid portable URLs.

The documented paths cover landing, collections, conformance, OpenAPI, and a systems/deployments subset. They omit most actual roots and all Part 2 routes. Live `/openapi` showed the same 11-path/18-operation set, 129 schemas, no security, localhost server, and `0.16.dev0`. This is a rich schema bundle but an incomplete deployment contract.

### 11.2 Schema Bundle

The bundler pins an OGC commit that is contained in approved tag `v1.0.0`, normalizes references to a local namespace, bundles dependencies, and removes read-only fields from request-required lists. Pinning and offline resolution are excellent patterns. The transformation must be deterministic, versioned, tested, and produce separately named request versus response schemas; modifying `required` in a general schema without that distinction weakens traceability.

### 11.3 Documentation State

The README supplies minimal installation and simulator instructions but references absent directories and removed dependency files. Architecture pages contain a few substantive choices—Quart, Elasticsearch, TimescaleDB, container scaling, future transports/access control—but much of the generated site is unreplaced arc42 guidance. It labels itself `1.0.0-BETA` while code is `0.6.0` and OpenAPI is `0.16.dev0`.

Open PR #26 would document the true pygeoapi relationship; its continued open state confirms current ambiguity. Documentation/version/OAD identity should be one release-derived value in Glaux.

### 11.4 Build and Release Evidence

The only project workflow builds and publishes images on tags/manual dispatch. GitHub records successful runs for `v0.6` and the commit shared by `v0.2`/`v0.4`. The workflow has no lint, unit, integration, schema, OAD, security, or conformance gates. It uses several major-version action references and does not establish SBOM/signature/provenance.

No GitHub Releases exist. Current main's `pyproject.toml` has duplicate incompatible Shapely pins and no lockfile; Docker uses system packages and override rules that hide the invalid dependency. A successful old tag image does not prove current main installability.

## 12. Interoperability and Client-Compatibility Findings

### 12.1 Dated Live Observations

On August 31, 2026, the Oracle-hosted deployment returned:

- HTTP 200 JSON landing, conformance, OpenAPI, collections, datastreams, observations, and sampling features;
- only Common Part 1 core from `/conformance`;
- populated SensorML systems/procedures but empty `application/json` FeatureCollections for the same roots;
- populated JSON datastreams and observations;
- populated top-level sampling features; and
- empty sampling-feature results from both `?system=rr-seed-desert-weather-alpha` and `/systems/rr-seed-desert-weather-alpha/samplingFeatures`.

Its OpenAPI reports `0.16.dev0`, so these results are deployment-snapshot evidence, not confirmation of main `6ad75aa`. The 52°North demo endpoint failed TLS validation in this study environment; no insecure bypass was used.

### 12.2 Client Compatibility Patterns

Generic OGC API clients commonly request application/json/GeoJSON, follow links, inspect conformance, and consume OpenAPI. A populated server that appears empty in one representation is worse than an explicit unsupported-format error because clients silently conclude no resources exist. Incomplete operation IDs/paths and malformed refs undermine generated clients. A Common-only conformance response prevents CSAPI feature discovery.

Issue #15 documents the representation/data issue from CSAPI client work. Issue #16 initially reported procedure-type ambiguity and then corrected its standards reading, illustrating why client findings must preserve later normative adjudication. Issue #27 provides a strong reproducible association-traversal case.

### 12.3 Smoke-Test Derivations

External client smoke tests should compare:

1. collection/item identity and count across every supported representation;
2. direct filter versus equivalent nested traversal;
3. landing/conformance/OAD links and operation completeness;
4. `f` versus Accept, q weights, wildcards, 406/415, and response `Vary`;
5. terminal-page navigation and stable order;
6. exact errors/statuses for malformed, conflict, missing, and unauthorized cases; and
7. schema-generated clients against real responses and auth declarations.

## 13. Test-Strategy Implications

### 13.1 Existing Evidence

Core pygeoapi `0.24.0` has substantial tests, six workflows, configuration/OpenAPI tests, and CITE setup. These support confidence in core behavior at its release. They do not execute the PoC's custom async routes, providers, models, schemas, storage split, authentication, or OpenAPI.

The PoC has no test files. Its schema `tester.js` is an exploratory script with one inline system, not a suite. Request snippets, simulator, live deployments, issues, and successful image builds are useful raw assets but not automated acceptance evidence.

### 13.2 Priority Regression Corpus

Glaux should explicitly retain tests inspired by current PoC defects:

- same resources/count/identity across SensorML and GeoJSON/JSON;
- lossless bidirectional encoding for all writable fields and links;
- direct association filter equals nested route;
- Part 1 replace constructs the correct resource type;
- Part 1/Part 2 validation flags are independently configured and typed;
- conflict/internal exceptions retain correct status and safe detail;
- datastream + derived-time write rolls back or repairs across failures;
- derived extents handle first value, earlier/later values, null phenomenon time, crash, and rebuild;
- last full page does not advertise a nonexistent next page;
- OAD routes/security/operation IDs/refs match runtime; and
- clean local/container dependency resolution uses the same lock/provenance.

### 13.3 Required Glaux Test Layers

1. Unit/property tests for parsers, typed config, encoders, canonical models, cursors, and error mapping.
2. Schema tests against pinned unmodified response schemas and explicitly transformed request schemas.
3. Repository contract tests reusable across in-memory/test and production stores.
4. Transaction/recovery/migration tests including injected cross-store failures.
5. Generated route/conformance/OAD/authorization parity tests.
6. Official/project ATS with requirement IDs and unsupported-class assertions.
7. Client/generated-client/CSAPI Explorer interoperability tests.
8. Security, load, backpressure, multi-tenant, dependency-failure, and release-provenance tests.

## 14. Lessons for Glaux Server

### 14.1 Adopt

- Provider traits separating domain behavior from storage engines.
- Declarative deployment metadata and resource/provider selection, but parsed into typed Rust configuration.
- Pinned, offline-capable schema bundles with explicit transformation provenance.
- Workload-aware metadata and time-series projections, provided canonical consistency is designed first.
- Separate read and write authorization boundaries as an initial route-composition concept.
- Mixed deterministic static fixtures and live/provider integrations as separate test layers.
- Dated issue/live evidence that becomes local regression tests.

### 14.2 Avoid

- Treating a library's general provider framework as proof that a domain standard is implemented.
- Canonical state stored separately by representation.
- Wildcard path routing as the primary resource contract.
- Untyped environment override traversal and truthy-string booleans.
- Cross-store dual writes without transaction/outbox/repair semantics.
- Ad hoc startup DDL and unrecoverable unlogged public-state caches.
- OAD maintained independently from routes/security and released with stale identity.
- Build-only CI and live demos as substitutes for an automated suite.

### 14.3 Investigate

- Whether the PoC intends to merge, rebase, or remain a separate product from pygeoapi.
- Which open fixes will land after current main and whether a new immutable tag/release will follow.
- The exact Oracle and 52°North deployment commits/configuration and TLS maintenance state.
- Production-scale ES/Timescale consistency, query plans, recovery, and auth patterns.
- Whether any official CSAPI ATS execution exists outside the repository.

## 15. Downstream Topic Handoff Matrix

| Topic | Handoff | Required use |
|---|---|---|
| IDR-SRV-014D | Compare SECD's actual product boundary, routes, persistence, OAD, and test evidence | Use same core/extension/deployment separation |
| IDR-SRV-014E | Include representation-population, nested-equivalence, negotiation, and OAD cases | Preserve deployment pin/unknown-version classification |
| IDR-SRV-014F | Compare SECD interop failures with PoC issues #15/#27 and error behavior | Separate normative defect from client tolerance |
| IDR-SRV-014G | Preserve corrected/invalid issue dispositions and framework/product naming lessons | Establish durable adjudication workflow |
| IDR-SRV-015 | Require an encoding-neutral canonical model and typed relationships | Never use SML/GeoJSON documents as separate truth |
| IDR-SRV-023 | Adopt pinned schema bundles with separate request/response transformations | Test transformation diff and provenance |
| IDR-SRV-025 | Evaluate specialized projections with transaction/outbox/repair | Include derived-view rebuild and migrations |
| IDR-SRV-028 | Distinguish canonical metadata, format projections, and documents | Define indexing/rebuild ownership |
| IDR-SRV-034 | Define ordering, latest, derived extents, ingest consistency, and live/history | Use PoC trigger/paging failures as negative tests |
| IDR-SRV-047 | Require typed versioned configuration and secret references | Reject unknown/untyped environment paths |
| IDR-SRV-050 | Generate claims from tested capabilities | Common-only underclaim and no PoC ATS are comparison evidence |
| IDR-SRV-051 | Link issues, requirements, fixes, deployments, and regressions | Retain later corrections such as issue #16 |
| IDR-SRV-053 | Convert request snippets/simulator cases into deterministic fixtures | Keep external demo as separate layer |
| IDR-SRV-056 | Test cross-representation equality, negotiation, traversal, OAD, and errors | Include generic OGC and generated clients |

## 16. Recommendations

1. **Make the canonical model encoding-neutral.** Priority: High. All supported representations must derive from one resource identity, relationships, and version; cross-format equivalence becomes a release gate.
2. **Use one typed capability/contract registry.** Priority: High. It should drive explicit routes, navigation, conformance, OAD, auth scopes, configuration validation, and required tests.
3. **Design projection consistency before selecting multiple stores.** Priority: High. Use transactions or outbox/saga patterns, idempotent consumers, repair jobs, health state, and rebuildable projections.
4. **Adopt versioned typed configuration.** Priority: High. Parse environment values by schema, reject unknowns, validate dependencies, reference secrets, and publish a redacted effective capability view.
5. **Generate and parity-test OpenAPI.** Priority: High. Require complete paths/methods, operation IDs, schemas, security, error responses, valid refs, deployed server URL, and release-derived version.
6. **Separate normative response schemas from request transforms.** Priority: High. Pin both, record transformations, test diffs, and never present a modified bundle as the original standard artifact.
7. **Build deterministic regressions before implementation breadth.** Priority: High. Prioritize representation equality, associations, mutation type safety, validation toggles, cross-store failure, paging terminals, and error/status mapping.
8. **Keep implementation identity explicit.** Priority: Medium. A framework, extension, fork, standalone service, demo configuration, and deployment snapshot must each have distinct names and pins.
9. **Treat Basic authentication as a development boundary only.** Priority: High. Glaux needs principals, scopes, tenant/resource policy, audit, secret lifecycle, and OAD declarations before writes/tasking.
10. **Publish reproducible releases, not only images.** Priority: Medium. Lock dependencies, test clean install/container paths, sign provenance/SBOM/checksums, and record deployment commit/config compatibility.

## 17. Risks, Constraints, and Open Questions

### 17.1 Risks and Constraints

- **Evidence boundary:** core pygeoapi maturity must not be transferred to untested custom PoC behavior.
- **Mutable source:** main is newer than the last tag and has no immutable Release object.
- **Execution:** Python and Docker were unavailable locally; PoC source contains no tests.
- **Installability:** current duplicate/nonexistent Shapely pins and missing lockfile prevent confident clean setup.
- **Representation:** per-format storage can silently hide or alter resources.
- **Consistency:** ES/Timescale dual writes and derived cache have no complete recovery contract.
- **Contract drift:** code, OAD, README, architecture site, tags, package version, and deployment version disagree.
- **Security:** Basic auth and fixed demo credentials are inadequate for production/resource/tasking policy.
- **Deployment:** one endpoint failed TLS validation and the other exposes an unknown/stale code version.
- **Standards:** local schema transforms and PoC interpretation are informative, not normative.

### 17.2 Open Questions

1. Will the PoC adopt PR #26's standalone-product identity and publish an updated architecture/roadmap?
2. Which of PRs #19, #23, #25, #26 and issues #3–5, #15, #27 will be resolved in the next tag?
3. What exact commits/configuration back the two public deployments, and will TLS/version metadata be repaired?
4. Can the PoC produce a complete CSAPI capability declaration and official/project ATS record?
5. What is the intended canonical source when SensorML and GeoJSON representations disagree?
6. What consistency/recovery mechanism is intended across Elasticsearch, TimescaleDB, and the unlogged cache?
7. Is Basic auth temporary, and what production identity/authorization/audit model is intended?

These questions limit claims about PoC conformance, production readiness, and current deployment behavior. They do not block the report's design and test lessons.

## 18. Validation Against the Research Plan

| Topic plan success criterion | Status | Evidence |
|---|---|---|
| Exact pygeoapi/CSAPI sources, pins, deployments, and dates identified | Met | Section 3; Appendix B |
| Core, PoC, providers, plugins, config, tests, demos, docs, and inference distinguished | Met | Reading Guide; Sections 3–4 |
| Architecture, provider, configuration, and documentation behavior summarized | Met | Sections 4, 10, and 11 |
| Part 1 and Part 2 compared to Glaux baseline | Met | Sections 5–7 |
| Conformance assessed without assumption | Met | Section 7 |
| Entry point, navigation, query, representations, errors, OAD, dynamic data, status, and tasking assessed | Met | Sections 8–11 |
| Strengths, gaps, risks, and assumptions identified | Met | Sections 7, 14, and 17 |
| Design, validation, test, and interoperability lessons documented | Met | Sections 12–16 |
| Downstream handoffs explicit | Met | Section 15; Appendix C |
| References explicit and reproducible | Met | Sections 3 and 19; Appendix B |

All six phases are complete. All five core and 41 detailed questions are answered or explicitly bounded in Appendix A. Section 7.2 contains all twelve required implementation-matrix fields. The report is ready for project-lead review; acceptance remains pending.

## 19. References

### Controlling and Glaux Sources

- [OGC API - Connected Systems landing page](https://ogcapi.ogc.org/connectedsystems/)
- [OGC API - Connected Systems Part 1](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems Part 2](https://docs.ogc.org/is/23-002/23-002.html)
- [OGC Connected Systems approved-tag source](https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0)
- [OGC API - Features Part 1](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html)
- [SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html)
- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)
- [IDR-SRV-006 through IDR-SRV-014B reports](./)

### pygeoapi and PoC Sources

- [52°North CSAPI PoC current main pin](https://github.com/52North/connected-systems-pygeoapi/tree/6ad75aa7fbfc76517c8a24c21446471845007855)
- [PoC routes](https://github.com/52North/connected-systems-pygeoapi/blob/6ad75aa7fbfc76517c8a24c21446471845007855/connected-systems-api/routes/csa.py)
- [PoC API dispatcher/validation](https://github.com/52North/connected-systems-pygeoapi/blob/6ad75aa7fbfc76517c8a24c21446471845007855/connected-systems-api/api.py)
- [PoC Part 1 provider](https://github.com/52North/connected-systems-pygeoapi/blob/6ad75aa7fbfc76517c8a24c21446471845007855/connected-systems-api/provider/part1/part1.py)
- [PoC Part 2 provider](https://github.com/52North/connected-systems-pygeoapi/blob/6ad75aa7fbfc76517c8a24c21446471845007855/connected-systems-api/provider/part2/part2.py)
- [PoC configuration](https://github.com/52North/connected-systems-pygeoapi/blob/6ad75aa7fbfc76517c8a24c21446471845007855/connected-systems-api/default-config.yml)
- [PoC OpenAPI](https://github.com/52North/connected-systems-pygeoapi/blob/6ad75aa7fbfc76517c8a24c21446471845007855/connected-systems-api/default-openapi.yml)
- [PoC schema bundler](https://github.com/52North/connected-systems-pygeoapi/blob/6ad75aa7fbfc76517c8a24c21446471845007855/tools/bundler/bundler.js)
- [PoC issues](https://github.com/52North/connected-systems-pygeoapi/issues)
- [PoC pull requests](https://github.com/52North/connected-systems-pygeoapi/pulls)
- [Issue #10: pygeoapi relationship](https://github.com/52North/connected-systems-pygeoapi/issues/10)
- [Issue #15: representation/provider behavior](https://github.com/52North/connected-systems-pygeoapi/issues/15)
- [Issue #27: sampling-feature association](https://github.com/52North/connected-systems-pygeoapi/issues/27)
- [PR #25: provider/query defect set](https://github.com/52North/connected-systems-pygeoapi/pull/25)
- [geopython pygeoapi `0.24.0`](https://github.com/geopython/pygeoapi/tree/f765a64fa65350dc93d9df0a1b38755bef6c31b0)
- [pygeoapi configuration documentation](https://docs.pygeoapi.io/en/latest/configuration.html)
- [pygeoapi plugin documentation](https://docs.pygeoapi.io/en/latest/plugins.html)
- [pygeoapi OGC compliance documentation](https://docs.pygeoapi.io/en/latest/ogc-compliance.html)
- [pygeoapi security documentation](https://docs.pygeoapi.io/en/latest/security.html)

## Appendix A. Detailed Question Ledger

| ID | Detailed question short form | Disposition | Report evidence |
|---|---|---|---|
| S1 | Relevant repos/branches/forks/plugins/issues/configs/demos | Answered | Section 3 |
| S2 | Exact branch/commit/release/deployment snapshot | Answered; deployments explicitly unpinned | Section 3; Appendix B |
| S3 | Core/CSAPI/provider/config/test/fixture/doc/demo classification | Answered | Sections 3–4 |
| S4 | Licensing/reuse | Answered with component provenance condition | Section 3.3 |
| S5 | Active issues/PRs/discussions | Answered | Sections 3, 11, 12, 17 |
| A1 | pygeoapi architecture | Answered | Section 4.1 |
| A2 | Provider-to-resource mapping | Answered | Sections 4.3–4.4 |
| A3 | Landing/conformance/collections/links/OAD/negotiation generation | Answered | Sections 7–8 and 11 |
| A4 | CSAPI fit in provider/plugin/config model | Answered: standalone custom service | Sections 4.1–4.2 |
| A5 | Reusable versus Python-specific framework patterns | Answered | Sections 4 and 14 |
| C1 | Part 1 resources/behavior | Answered | Section 5 |
| C2 | Part 2 resources/behavior | Answered | Section 6 |
| C3 | Claimed/implied/unsupported/unclear classes | Answered | Section 7 |
| C4 | Conformance/OAD/media/links/schemas/errors | Answered | Sections 7–8 and 11 |
| C5 | Gaps versus Glaux baseline | Answered | Section 7.2 |
| C6 | Evidence per alignment/gap | Answered | Section 7.2 and references |
| B1 | Landing/OAD/conformance/collections/items/links | Answered | Section 8.1 |
| B2 | URIs/links/nesting/navigation | Answered | Sections 5 and 8.2 |
| B3 | Query/filter/sort/pagination/selection | Answered | Sections 5.4, 6.2, and 8.3 |
| B4 | Negotiation/media behavior | Answered | Section 8.4 |
| B5 | Error behavior | Answered | Section 8.5 |
| B6 | Generated OpenAPI versus behavior | Answered | Section 11.1 |
| D1 | Datastream/observation/status/event/dynamic representation | Answered; absent areas explicit | Sections 6 and 9 |
| D2 | Real-time/history/streaming/replay/publication | Answered | Sections 6.4 and 9.2 |
| D3 | Control/commands/feasibility/status | Answered: absent | Section 9.3 |
| D4 | Implemented/partial/absent/unclear dynamic features | Answered | Sections 6 and 9 |
| D5 | Dynamic/streaming/command handoffs | Answered | Section 15 |
| G1 | Configuration of collections/providers/metadata/links/schemas/API | Answered | Sections 4.5 and 10.3 |
| G2 | Useful deployment/demo/reference-data patterns | Answered | Sections 10 and 14 |
| G3 | Multiple sources/metadata/provider behavior/operations | Answered | Sections 4.3 and 10.1 |
| G4 | Publisher/Simulator/static/test integration lessons | Answered | Section 10.4 |
| G5 | Config patterns weakening traceability/type safety | Answered | Sections 4.5 and 10.3 |
| V1 | Tests/examples/fixtures/config/demo/OAD assets | Answered | Sections 3.2 and 13.1 |
| V2 | Candidate positive/negative/schema/conformance/golden/client/interop tests | Answered | Sections 12.3 and 13 |
| V3 | Comparison evidence versus normative truth | Answered | Reading Guide; Section 7 |
| V4 | Handoffs to 050/051/053/056 | Answered | Section 15 |
| I1 | Endpoints/examples tested with clients | Answered within issue/live evidence | Section 12.1 |
| I2 | Important client compatibility patterns | Answered | Section 12.2 |
| I3 | Implementation-specific interoperability risks | Answered | Sections 7.2 and 12 |
| I4 | Derived smoke tests | Answered | Section 12.3 |
| I5 | Handoffs to 014E/014F/056 | Answered | Section 15 |

## Appendix B. Reproducible Study Record

### B.1 Mechanical Inventory

| Item | Result |
|---|---|
| PoC current source | `6ad75aa7fbfc76517c8a24c21446471845007855`; tree `a9cd2b7163709251895c055ba9e1a9314375f8f4`; 2026-08-17 |
| PoC last tag | `v0.6`; `835cacb36145da0f46eab4b689157e00d820191c`; 2025-10-21 |
| PoC repository inventory | 134 tracked files; 33 Python files; 10 bundled schemas; 0 test files; 1 build/publish workflow |
| Explicit debt markers | 40 `TODO` / not-implemented / not-supported source matches |
| CSAPI route inventory | 69 blueprint declarations; 49 unique abstract method/path pairs |
| Source OpenAPI | 3.1.0; 11 paths; 18 operations; 4 operation IDs; about 126 schemas; 0 security schemes; 10 malformed backslash OGC refs |
| Core release | pygeoapi `0.24.0`; `f765a64fa65350dc93d9df0a1b38755bef6c31b0`; 483 files; 164 Python files; 71 Python test files |
| Actual core dependency | `ec1eb38d9a64d93ec9a2e1b9db6fea6dc05f194a`; reports `0.22.dev0`; exact geopython history commit |
| OGC schema input | `636277919d96d2274844ccb981b22119d01e2f9e`; contained in tag `v1.0.0` |
| Project releases | No GitHub Releases; seven tags; successful image workflow for `v0.6` and older shared `v0.2`/`v0.4` commit |
| Current open code/docs PRs | #9, #19, #21, #23, #25, #26 as retrieved 2026-08-31 |

### B.2 Live Probe Record

Oracle deployment, retrieved 2026-08-31:

- landing 200 JSON;
- conformance 200 with Common Part 1 core only;
- OpenAPI 3.1.0, 11 paths, 18 operations, 4 operation IDs, 129 schemas, no security, version `0.16.dev0`, localhost server;
- systems/procedures JSON empty, SensorML populated;
- datastreams/observations JSON populated;
- sampling features top-level populated but direct/nested system association queries empty;
- invalid `f=application/xml` and `limit=abc` returned structured 400;
- unknown path returned framework HTML 404; and
- unsupported Accept and wildcard fell back to default representation.

52°North demo, retrieved 2026-08-31: TLS connection validation failed for the bounded root/conformance/OpenAPI/resource probes. No insecure connection was used.

### B.3 Execution Limitations

- Local Python executable: unavailable.
- Local Docker executable: unavailable.
- PoC automated tests: none found.
- No mutations were sent to public deployments.
- Live deployment commit/configuration could not be established.

## Appendix C. Completion and Handoff

- Research phases 1–6: complete.
- Deliverable draft and internal review: complete.
- Report status: **In Review**.
- Acceptance: pending Glaux Project Lead.
- Next topic after acceptance: **IDR-SRV-014D — SECD CSAPI Server Implementation Study**.
- Workflow boundary: the next plain `proceed` accepts/finalizes/merges IDR-SRV-014C and executes exactly IDR-SRV-014D; it does not authorize IDR-SRV-014E.

---

## Report Completion Checklist

- [x] Topic ID matches overall research plan index
- [x] Topic research plan is linked and aligned
- [x] Core and detailed research questions are covered or explicitly bounded
- [x] Findings are evidence-backed with reproducible references
- [x] Normative and informative evidence are classified and not conflated
- [x] Mutable sources identify release, tag, commit, deployment, and retrieval date
- [x] Inaccessible, missing, and ambiguous evidence limitations are explicit
- [x] Source-backed findings, inference, and recommendations are distinguishable
- [x] Findings are reconciled with accepted IDR-SRV-006 through IDR-SRV-014B
- [x] Executive summary is independently readable
- [x] Recommendations are explicit and actionable
- [x] Risks and open questions are documented
- [x] Success criteria validation is complete
- [x] Plan-owner acceptance and acceptance date are recorded
- [x] Next handoff is explicit

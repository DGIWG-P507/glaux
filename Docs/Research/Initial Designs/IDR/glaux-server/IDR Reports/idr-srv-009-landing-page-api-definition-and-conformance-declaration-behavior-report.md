# Section 009: Landing Page, API Definition, and Conformance Declaration Behavior - Research Report

**Topic ID:** IDR-SRV-009<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-009 Landing Page, API Definition, and Conformance Declaration Behavior](../IDR%20Plans/idr-srv-009-landing-page-api-definition-and-conformance-declaration-behavior.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core and 31 detailed questions; all six methodology phases, ten success criteria, fifteen required content areas, and ten minimum behavior-matrix fields are validated<br>
**Methodology Used:** Reconciliation of accepted IDR-SRV-006 through IDR-SRV-008 with the approved CSAPI, OGC API - Common, and OGC API - Features standards; their normative abstract tests and schemas; the tagged modular and released bundled CSAPI OpenAPI artifacts; bounded official issue and pull-request history; the official Features executable test suite; and pinned implementation and client evidence<br>
**Research Time:** Approximately 4 hours of AI-assisted elapsed execution time, including three parallel independent read-only standards, OpenAPI/history, and implementation/test audits, on August 1, 2026<br>
**Primary Sources:**

- OGC 23-001, *OGC API - Connected Systems - Part 1: Feature Resources*, Version 1.0
- OGC 23-002, *OGC API - Connected Systems - Part 2: Dynamic Data*, Version 1.0
- OGC 19-072, *OGC API - Common - Part 1: Core*, Version 1.0.0
- OGC 17-069r4, *OGC API - Features - Part 1: Core*, Version 1.0 with Corrigendum

**Supporting Resources:** Accepted IDR-SRV-001 through IDR-SRV-008 reports; official CSAPI tag `v1.0.0`; released OAS 3.1 bundles; the bounded upstream-history register; official Features ETS; CSAPI Explorer; pinned OpenSensorHub, connected-systems-go, pygeoapi, and SECD evidence; and the OS4CSAPI exemplar corpus<br>
**Document Purpose:** Establish the decision-usable external contract for the first resources clients encounter on a complete Rust Glaux Server, while reserving detailed navigation, negotiation, error-model, and OpenAPI construction decisions for their owning topics<br>
**Author:** OpenAI Codex, with independent read-only normative, OpenAPI/history, and implementation/test audits<br>
**Accepted By:** TBD - Glaux Project Lead review pending<br>
**Acceptance Date:** TBD until accepted<br>
**Date:** August 1, 2026<br>
**Last Updated:** August 1, 2026

---

## How to Read This Report

For the project decision, read §§1, 5.2, 6.4, 7.3, 14, 16.2, and 19. They state what Glaux should expose, how it should avoid false conformance claims, and what remains for later research. Sections 3–13 and the appendices are the audit trail for implementers and later AI agents.

Four labels keep different kinds of statements separate:

- A **standards obligation** comes from approved normative text or a normative abstract test.
- A **source-backed finding** reports what an artifact, implementation, client, test suite, issue, or pull request contains.
- **Analysis** explains what the evidence means.
- A **project recommendation** states the bounded Glaux choice proposed for plan-owner acceptance.

In this report, **OAS** means OpenAPI Specification, **API definition** means the machine- or human-readable description linked from the landing page, **ATS** means normative abstract test suite, and **ETS** means executable test suite.

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Research Method and Reconciliation
5. Landing Page Behavior Findings
6. API Definition Behavior Findings
7. Conformance Declaration Behavior Findings
8. Representation and Media-Type Implications
9. Error, Failure, and Stale-Description Implications
10. OpenAPI Artifact and Upstream-History Findings
11. Interoperability and Existing-Implementation Findings
12. Unified Entry-Point Behavior Matrix
13. Test-Strategy Implications
14. Decision Analysis and Recommendations
15. Implementation Implications and Relative Estimates
16. Risks, Constraints, and Open Questions
17. Downstream Topic Handoff Matrix
18. Validation Against the Research Plan
19. Next Steps and Handoff
20. References
21. Appendices

---

## 1. Executive Summary

### 1.1 Plain-English Decision Brief

The Glaux entry-point design should be simple for clients and difficult for the server to misrepresent. A client starts at `GET {root}/`, follows clearly typed links to the API definition, conformance declaration, and collections, and then uses those resources to understand the live deployment. `GET {root}/conformance` reports only the conformance classes that the particular released build has actually implemented and proved. The API definition describes that same build, not the standard in the abstract and not a different demonstration server.

The standards create one easy-to-miss compatibility requirement. CSAPI Part 1 inherits both OGC API - Features Core and OGC API - Common Landing Page. Features requires a landing-page link whose relation is the short value `conformance`; Common requires the full relation URI `http://www.opengis.net/def/rel/ogc/1.0/conformance`. The safest exact implementation is two link objects, both pointing to the same `{root}/conformance` resource. The landing page must also use `data`, not the official CSAPI example's nonstandard `collections`, for its link to `{root}/collections`.

Glaux should publish both a machine-readable `service-desc` and a human-readable `service-doc`, with accurate `type` values and working GET operations. The detailed OAS version, generation mechanism, YAML option, and documentation renderer remain for IDR-SRV-014. Until that work is accepted, Glaux should not promise an OAS 3.0 conformance claim. Serving an OAS 3.1 document does not by itself satisfy either OGC OAS 3.0 conformance class.

The central implementation recommendation is to generate the landing page, runtime API definition, conformance declaration, and route/media advertisement from one versioned capability registry. The release process should fail if those views disagree. This prevents the exact failures found in the official examples and existing implementations: nonexistent class identifiers, obsolete versions, generic demo URLs, wrong link relations, incomplete API definitions, and hardcoded capability lists that drift from the server.

### 1.2 Principal Conclusions

1. **A single combined API root governs Parts 1 and 2.** Part 2 inherits Part 1 API Common and does not define a separate landing page, conformance resource, or standalone API definition.
2. **JSON is mandatory for the landing page and conformance declaration.** CSAPI Part 1 directly inherits the Common JSON class. HTML representations are optional and must not be claimed until implemented and tested.
3. **The landing page needs a small mandatory discovery spine:** an API-definition link, both conformance relation spellings, and a `data` link to `/collections`. `self` and representation `alternate` links should also be present.
4. **Direct root links to every CSAPI resource are not required.** CSAPI Part 1 says inherited Common links are sufficient. Any richer navigation graph belongs to IDR-SRV-010.
5. **Every advertised API-definition link must work.** GET must return `200`, and the representation must agree with the link's `type` and content negotiation.
6. **OpenAPI is recommended but not mandatory at the base landing-page level.** OGC's OAS 3.0 classes are conditional, separate conformance claims with additional JSON, HTML, completeness, implementation-parity, error, and security obligations.
7. **The live API definition must be Glaux-specific.** The official CSAPI OAS files are explicitly examples, advertise demo/local servers, use `info.version: 0.0.1`, and cannot serve as the contract for a Glaux deployment.
8. **`/conformance` is a truth statement, not a roadmap.** It lists exact `/conf/` identifiers for evidence-complete classes and their prerequisites—never planned, partial, experimental, deferred, `/req/`, test, friendly-name, or guessed identifiers.
9. **Schema validation alone is inadequate.** The conformance schema accepts any strings and does not prove exact identifiers, uniqueness, completeness, dependency closure, or implementation evidence. Glaux needs semantic validation.
10. **The all-class target and a release declaration are different.** The accepted IDR-SRV-008 target is all 25 direct CSAPI classes; a particular release declares only the subset and full prerequisite closure that it passes.
11. **Identifier resolution and identifier identity are different.** Part 2 class PURLs currently return 404, but the literal normative `http://www.opengis.net/spec/.../conf/...` strings remain the declaration identifiers.
12. **The official tagged landing and conformance examples must become negative fixtures, not templates.** They contain relation, version, completeness, and identifier defects.
13. **The official released OAS “bundles” are not fully reference-closed.** The Part 1 asset retains 32 relative references and Part 2 retains 51. Glaux must publish a validated, versioned, Glaux-owned definition.
14. **The official Features ETS is necessary but insufficient.** It can test inherited entry-point behavior; it cannot establish CSAPI conformance or validate the complete Glaux declaration.
15. **Interoperability depends on precision, not client-specific workarounds.** CSAPI Explorer expects recognizable API-definition/conformance links, uses `data` for collections, resolves relative links, and does not validate whether the OAS or conformance claims are actually true.

### 1.3 Recommended Plan-Owner Decision

Accept the following baseline for downstream planning:

- one combined Glaux API root;
- mandatory JSON landing and conformance resources;
- both conformance relation spellings, `data`, `self`, accurately typed machine and human API-definition links, and stable public-base URLs;
- an implementation-specific API definition whose exact OAS strategy is finalized in IDR-SRV-014;
- a deterministic, evidence-generated conformance declaration;
- a single capability registry from which routes, representations, documentation, and declarations are derived; and
- deployment/readiness failure when those external contracts disagree.

Acceptance does not select the final OAS version, finalize every landing-page extension, define detailed content negotiation or error bodies, or claim that a Glaux implementation already conforms.

---

## 2. Scope and Plan Alignment

### 2.1 In Scope and Completed

- Required and recommended landing-page operations, structure, links, metadata, and representations.
- API-definition discovery, retrieval, media agreement, live-behavior parity, and optional OAS-class consequences.
- Conformance endpoint location, structure, identifier rules, completeness, evidence gating, and staged-release behavior.
- The interaction among CSAPI Parts 1 and 2, Common Part 1, and Features Part 1.
- Tagged modular OAS, released OAS bundles, schemas, examples, and bounded relevant official issues and pull requests.
- Supporting implementation, CSAPI Explorer, official Features ETS, and interoperability evidence.
- Positive, negative, semantic, parity, reverse-proxy, and external-client test implications.
- Explicit handoffs to IDR-SRV-010, 010A, 012, 013, 014, 023, 050, 051, 056, and 057.

### 2.2 Explicitly Out of Scope

- The complete collection and resource navigation design; IDR-SRV-010 owns it.
- API versioning, compatibility, and deprecation policy; IDR-SRV-010A owns it.
- Full `Accept`, `f`, q-value, media-parameter, and alternate-URI policy; IDR-SRV-012 owns it.
- The final error model and status-code/body mapping; IDR-SRV-013 owns it.
- OAS version selection, source/generation strategy, modularization, renderer, and published documentation architecture; IDR-SRV-014 owns it.
- Complete schema-resolution and validation architecture; IDR-SRV-023 owns it.
- Executable CSAPI harness architecture and full requirement-to-test system; IDR-SRV-050 and IDR-SRV-051 own them.
- Detailed implementation comparisons reserved for IDR-SRV-014A through IDR-SRV-014G.
- Writing Rust server code or asserting current Glaux conformance.

### 2.3 Core Research-Question Coverage

| ID | Core question | Status | Primary evidence |
|---|---|---|---|
| CQ1 | Required landing-page behavior | Complete | §§5, 8, 12 |
| CQ2 | Required/supported API-definition behavior | Complete | §§6, 8–10, 12 |
| CQ3 | Required conformance-declaration behavior | Complete | §§7, 10, 12 |
| CQ4 | Honest exposure of CSAPI, SensorML, SWE, and Glaux capabilities | Complete | §§7.3–7.5, 14 |
| CQ5 | Test and validation implications | Complete | §§9, 11, 13, 17 |

Appendix B maps all 31 detailed questions individually.

### 2.4 Reconciliation with Accepted Prior Research

IDR-SRV-006 established that Part 1 API Common inherits Features Core and Common Core, Landing Page, and JSON. IDR-SRV-007 established Part 2 API Common's prerequisite on Part 1 API Common. IDR-SRV-008 established the exact direct CSAPI class identifiers, the all-25 direct-class end-state target, the directly invoked external dependencies, and an evidence-gated runtime declaration policy. Its external dependency list was not a recursive closure audit.

This report adds one planned refinement, not a correction requiring prior-report reopening. IDR-SRV-008 mentioned the Features `conformance` relation and explicitly handed exact landing behavior to IDR-SRV-009. The complete inherited analysis here shows that Glaux must also satisfy Common's full conformance relation URI. Both lexical values are therefore included.

---

## 3. Evidence Base and Authority Classification

### 3.1 Primary Sources Reviewed

| Source | Version / pin | Authority | Stable anchors | Access | Limitations |
|---|---|---|---|---|---|
| [OGC 23-001, CSAPI Part 1](https://docs.ogc.org/is/23-001/23-001.html) | Version 1.0; approved; HTML SHA-256 `555c4b3bea06ab91b980bdaa3c99d265e6718dbad943ca1cbec39fbbf283c92a` | Controlling normative CSAPI source | §§8.1–8.3; `/req/api-common`; Annex A.2 | 2026-08-01 | Direct entry behavior is mainly inherited rather than repeated |
| [OGC 23-002, CSAPI Part 2](https://docs.ogc.org/is/23-002/23-002.html) | Version 1.0; approved; HTML SHA-256 `e840613693c282a41b1dda709eb266905683697fb430168ff348833e8f50df5e` | Controlling normative CSAPI source | §8; `/req/api-common`; Annex A.1 | 2026-08-01 | Part 2 identifier PURLs are not currently mapped |
| [OGC 19-072, OGC API - Common Part 1](https://docs.ogc.org/is/19-072/19-072.html) | Version 1.0.0; approved; source tag `Common2023` at `3dc3c6f8322fb4bb35f6a51d3bed153232535a7d` | Inherited normative source | §§8–11; Requirements 12–27; Annex A.3–A.5 | 2026-08-01 | ATS JSON/HTML loops are broader than their normative requirement scope; adapter required |
| [OGC 17-069r4, OGC API - Features Part 1](https://docs.ogc.org/is/17-069r4/17-069r4.html) | Version 1.0.1 corrigendum; source tag `1.0.1` at `4ff19a5734578cf1f815d03ab192e8e0dc407e9f` | Inherited normative source | §§7.2–7.5, 9; Requirements 1–7 and 46–51; Annex A.2/A.7 | 2026-08-01 | Some PURLs redirect to the pre-corrigendum r3 rendering |
| [OGC Common landing schema](https://schemas.opengis.net/ogcapi/common/part1/1.0/openapi/schemas/landingPage.yaml) and [conformance schema](https://schemas.opengis.net/ogcapi/common/part1/1.0/openapi/schemas/confClasses.yaml) | Versioned OGC schema repository | Normative where cited by requirements | `links`; `conformsTo` | 2026-08-01 | Structural validation does not prove link semantics or declaration truth |
| [OpenAPI Specification](https://spec.openapis.org/oas/latest.html) | Latest published version 3.2.0, dated 2025-09-19; versioned 3.0.3 and 3.1.0 also reviewed | Normative only for a selected OAS version, not a CSAPI obligation by itself | OAS document, paths, responses, servers, security | 2026-08-01 | IDR-SRV-014 must select the Glaux strategy |

### 3.2 Official Artifact and History Evidence

| Source | Pin / state | Evidence class and use | Limitation |
|---|---|---|---|
| [Official CSAPI repository](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2) | `v1.0.0`, commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2` | Published-support artifacts and exact source provenance | OAS and examples are informative, not normative profiles |
| [Part 1 modular OAS](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/openapi-connectedsystems-1.yaml) | SHA-256 `745fc5357a173e127f174f7413f36bc22b55f1b7cfa560902c63bb67dd82ea78` | Entry-point artifact inspection | Example OAS 3.1 with generic servers and incomplete entry responses |
| [Part 2 modular OAS](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/openapi-connectedsystems-2.yaml) | SHA-256 `2df6c9b48bc19a21f0b44219b947a0d3be29e76f842cb9bb106c3cf7a5c9dd82` | Confirms additive Part 2 path document | Not a standalone API description |
| [Part 1 released bundle](https://github.com/opengeospatial/ogcapi-connected-systems/releases/download/v1.0.0/ogcapi-connectedsystems-1.bundled.oas31.yaml) | SHA-256 `69da631d5d05f01716381cca7b7ee6311402f2752a8fd79a9b72b663539555aa` | Released OAS 3.1 evidence | Retains 32 relative references |
| [Part 2 released bundle](https://github.com/opengeospatial/ogcapi-connected-systems/releases/download/v1.0.0/ogcapi-connectedsystems-2.bundled.oas31.yaml) | SHA-256 `86ed005f9e7cf176264d6deb72581a0b521a227cd7a198b6cb1bd32b39d83667` | Released OAS 3.1 evidence | Retains 51 relative references |
| [Upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md) | Version 1.2, refreshed 2026-08-01 | Bounded disposition of official issues, PRs, commits, and release inclusion | Supporting evidence; never changes approved obligations |
| Current official branch | `master` commit `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f` | Confirms no post-tag entry-point correction | Mutable; date-pinned |

### 3.3 Supporting Implementation, Client, and Test Evidence

| Source | Pin | Relevance | Limitation |
|---|---|---|---|
| [Official Features ETS](https://github.com/opengeospatial/ets-ogcapi-features10/tree/a314c1e6a9278b14ab9a2ed865cfe36d202f0125) | `a314c1e6a9278b14ab9a2ed865cfe36d202f0125` | Executable behavior for inherited Features root, API definition, and conformance | Not a CSAPI ETS; OAS test is OAS 3.0-specific |
| [CSAPI Explorer](https://github.com/OS4CSAPI/ogc-csapi-explorer/tree/00f1c188e05738ee03390fd95f09d351e073a9c3) | `00f1c188e05738ee03390fd95f09d351e073a9c3` | Real client discovery assumptions and failure tolerance | Client behavior is not standards authority |
| [connected-systems-go](https://github.com/OS4CSAPI/connected-systems-go/tree/e900da88738cca92872038b703c4ad537fc0c8fd) | `e900da88738cca92872038b703c4ad537fc0c8fd` | Positive landing construction and negative hardcoded declaration/OAS evidence | Implementation precedent only |
| [OpenSensorHub](https://github.com/OS4CSAPI/osh-core/tree/b2badae59aaa78455c5638ad73b452ccdee40207) | `b2badae59aaa78455c5638ad73b452ccdee40207` | Existing CSAPI server entry-point behavior | Implementation precedent only; live state mutable |
| [pygeoapi](https://github.com/geopython/pygeoapi/tree/bdf3b9ff70b15b4bd72e19df624d038d72c2f466) | `bdf3b9ff70b15b4bd72e19df624d038d72c2f466` | Capability-derived, deterministic conformance and instance OAS precedent | OGC API implementation, not a CSAPI implementation |
| [SECD interoperability evidence](https://github.com/Sam-Bolling/csapi-server-interop-secd/tree/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0) | `f018fd129bf0d0d1ce75e68198e3ab4d99d937a0` | Preserved HTTP evidence and negative/positive implementation cases | Informative and deployment-specific |
| [OS4CSAPI exemplar corpus](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing) | `754411897173c2ec4debaa9bcf4ed9e0f8a9e230` | Report structure, sourcing rigor, and test-scenario depth | Form and method only; technical assumptions may be superseded |

### 3.4 Authority Rules Applied

1. Approved standards and their normative ATS control obligations.
2. A schema controls the structure only to the extent a normative requirement cites it.
3. The accepted IDR-SRV-008 registry controls the Glaux identifier and evidence baseline unless new approved normative evidence contradicts it; none did.
4. Tagged OAS, examples, release assets, and official issues explain artifacts and history but cannot override normative text.
5. Merged post-publication work is maintenance evidence until it enters a controlling release or approved standard.
6. Implementations and client behavior reveal interoperability risks and useful patterns, not standards requirements.
7. When sources conflict, Glaux preserves each source's authority and tests the contradiction instead of choosing the easiest artifact to copy.

### 3.5 Material Evidence Limitations

- No official public CSAPI ETS was found. The standards' ATS and Glaux supplemental tests remain essential.
- The Features ETS proves only inherited Features behavior and does not reject unknown conformance strings.
- Live implementation checks are point-in-time observations; pinned code and preserved HTTP captures are preferred where available. A pinned source commit is not proof that a public deployment runs that binary.
- Features Part 4 remains an unpublished dependency for four target transaction classes. This affects the full target declaration, not the entry-point shape.
- Several PURLs have resolver defects or redirect to an older rendering. Literal identifiers and human evidence anchors are recorded separately.
- Full OAS linting, bundling, schema dialect, and generator evaluation belongs to IDR-SRV-014 and IDR-SRV-023.

---

## 4. Research Method and Reconciliation

### 4.1 Six-Phase Execution

| Plan phase | Work performed | Result |
|---|---|---|
| 1. Source collection and setup | Pinned approved standards, tags, current branch, OAS assets, schemas, prior reports, issues/PRs, ETS, clients, implementations, and exemplars | Reproducible evidence register and authority rules |
| 2. Landing extraction | Reconciled CSAPI inheritance with Common and Features root requirements, ATS, link schemas, and examples | Exact root and link baseline, including dual conformance relations |
| 3. API-definition extraction | Compared base requirements, optional OAS classes, CSAPI media-support requirements and ATS, official OAS variants, and implementation behavior | Retrieval/parity contract and IDR-SRV-014 handoff |
| 4. Conformance extraction | Reconciled endpoint schema, IDR-SRV-008 registry/dependencies, PURL behavior, examples, issue #23, and staged releases | Evidence-gated declaration policy and unresolved recursive-closure handoff |
| 5. Representation/error/interoperability/test analysis | Audited JSON/HTML/OAS media behavior, status guidance, Features ETS, Explorer, implementations, and negative cases | Test families and later-topic handoffs |
| 6. Synthesis | Performed three independent read-only audits, reconciled findings, and validated every plan question/criterion/deliverable | This report |

### 4.2 Direct Obligation and Test Count

The bounded entry-point and CSAPI media-support/test baseline contains 28 directly relevant requirements and 28 corresponding abstract tests.

| Source lane | Requirements | Abstract tests |
|---|---:|---:|
| Common Core HTTP | 1 | 1 |
| Common Landing Page | 6 | 6 |
| Common JSON | 2 | 2 |
| Features Core HTTP and entry points | 7 | 7 |
| CSAPI Part 1 runtime media support / ATS advertisement | 4 | 4 |
| CSAPI Part 2 runtime media support / ATS advertisement | 8 | 8 |
| **Total** | **28** | **28** |

Common Core tests A.2–A.11 and Features tests A.24–A.25 add twelve cross-cutting checks that use the API definition as an operational source for parameter names, values, capitalization, list encoding, and negative requests. They are not included in the direct 28 count, but they demonstrate that an API definition is part of the executable contract, not merely a documentation page.

### 4.3 Conflict Reconciliation

| Conflict | Authority-based resolution | Glaux consequence |
|---|---|---|
| Features uses `conformance`; Common uses the full OGC relation URI | CSAPI inherits both normative classes | Emit two links to the same conformance resource |
| Features requires `data`; tagged CSAPI example uses `collections` | Features requirement controls; example is informative | Emit `data`; preserve `collections` only as a negative fixture |
| OGC optional classes specify OAS 3.0; CSAPI artifacts use OAS 3.1 | Both facts stand; an example OAS cannot create a conformance claim | Select strategy in IDR-SRV-014 and withhold OAS 3.0 claim until proved |
| Tagged conformance example differs from published class inventory | Approved standards and accepted IDR-SRV-008 control | Never initialize Glaux from the example list |
| Part 2 PURLs return 404 | Approved document identifiers remain normative strings | Emit literal identifiers; do not depend on dereferencing them at runtime |
| Part 2 OAS lacks root/conformance | Part 2 depends on Part 1 API Common | Treat it as an additive path source, not a standalone deployment definition |
| Common JSON ATS loops over every landing operation | Requirements 20–21 limit JSON to landing and conformance | Use a documented ATS adapter; do not invent mandatory JSON API-definition behavior from the loop |

---

## 5. Landing Page Behavior Findings

### 5.1 Standards Obligations

**CSAPI inheritance.** [CSAPI Part 1 §8.1](https://docs.ogc.org/is/23-001/23-001.html#_overview_2) makes Features Core, Common Core, Common Landing Page, and Common JSON prerequisites of `/req/api-common`. [Part 1 §8.2](https://docs.ogc.org/is/23-001/23-001.html#_api_landing_page) says the inherited common links are sufficient. [Part 2 §8](https://docs.ogc.org/is/23-002/23-002.html#_requirements_class_common) depends on Part 1 API Common.

**Root operation and response.** Common `/req/landing-page/root-op` and Features `/req/core/root-op` require GET at `{root}/`. A successful response is `200`, is based on the applicable landing-page schema, and contains `links`. `{root}` is the API root and may include a deployment path prefix; it is not necessarily the domain root.

**Required discovery links.** The combined inherited contract requires:

- at least one `service-desc` or `service-doc` API-definition link;
- a short `conformance` link;
- a full `http://www.opengis.net/def/rel/ogc/1.0/conformance` link; and
- a `data` link to `{root}/collections`.

**Recommended links.** Features' root-links recommendation says the landing page should include `self` and an `alternate` link for every other supported representation. Although its identifier is mistakenly written under `/req/`, its wording is explicitly a recommendation using SHOULD.

Common and Features also recommend exposing applicable payload links through HTTP `Link` headers unless doing so is impractical. This is not a substitute for the required body links. Glaux should keep body and header link views consistent and let IDR-SRV-012 finalize header/negotiation behavior.

**JSON.** Common `/req/json/definition` and `/req/json/content`, inherited by CSAPI, require `application/json` support for landing and conformance `200` responses, valid RFC 8259 JSON, and conformance to the cited resource schemas.

### 5.2 Recommended Glaux Landing Contract

The following is the recommended minimum JSON representation. Values are illustrative; exact public URLs and the selected OAS media type are deployment/build outputs.

```json
{
  "title": "Glaux OGC API - Connected Systems",
  "description": "A Rust reference implementation of OGC API - Connected Systems Parts 1 and 2.",
  "attribution": "DGIWG P5.07 Glaux",
  "links": [
    {"rel": "self", "href": "https://example.org/csapi/", "type": "application/json", "title": "This landing page"},
    {"rel": "service-desc", "href": "https://example.org/csapi/api", "type": "application/vnd.oai.openapi+json;version=3.1", "title": "Machine-readable API definition"},
    {"rel": "service-doc", "href": "https://example.org/csapi/api.html", "type": "text/html", "title": "Human-readable API documentation"},
    {"rel": "conformance", "href": "https://example.org/csapi/conformance", "type": "application/json", "title": "Conformance declaration"},
    {"rel": "http://www.opengis.net/def/rel/ogc/1.0/conformance", "href": "https://example.org/csapi/conformance", "type": "application/json", "title": "Conformance declaration"},
    {"rel": "data", "href": "https://example.org/csapi/collections", "type": "application/json", "title": "Collections"}
  ]
}
```

The two conformance links are intentionally separate. A consumer may select either lexical relation form. Duplicate targets are preferable to silently failing one inherited contract. The OAS 3.1 media type in this illustration is not the IDR-SRV-014 version decision and must be replaced if that topic selects another machine representation.

### 5.3 Metadata and Link Fields

The formal Common schema requires only `links`. `title`, `description`, and `attribution` are optional; Common strongly advises a title. The nested link schema requires `href` and `rel`, while `type`, `title`, `hreflang`, and `length` are optional.

**Project recommendation:** Glaux should treat `title`, `description`, `type`, and a concise link `title` as release-quality requirements even where the standards do not. Accurate `type` values let a client make the exact negotiated request that Features requires and prevent ambiguous responses such as `Content-Type: auto` observed in implementation evidence. Attribution should be configurable and included when appropriate.

License, terms-of-service, support, and source-code information are useful but not mandated landing fields. Expose them through the API definition, human documentation, and/or properly related extra links after IDR-SRV-010 and IDR-SRV-014 select the exact vocabulary. Do not invent a relation and describe it as CSAPI-required.

### 5.4 CSAPI Resource Navigation

The landing page is not required to link separately to systems, deployments, procedures, sampling features, properties, datastreams, observations, control streams, commands, feasibility resources, results, statuses, or events. Part 1 explicitly says inherited Common links are sufficient. The required `data` link reaches the collections discovery resource; CSAPI resources are then discoverable through standard paths and links.

**Project recommendation:** keep the normative root spine small and reliable. IDR-SRV-010 should decide whether additional registered links to canonical CSAPI resources improve usability without confusing generic OGC clients. Any additional links must use accurate `href`, `rel`, `type`, and public-base handling.

### 5.5 Deployment Root and Link Construction

All entry links and the API definition's server URL must describe the externally visible deployment, including path prefixes and reverse proxies. Hardcoded localhost, internal host, or demonstration URLs are unacceptable for a live definition.

**Project recommendation:** make the public API base an explicit validated deployment setting. Generate absolute external discovery links from it, or use a deliberately tested relative-link policy. Test direct deployment, a non-root mount such as `/csapi/v1`, and reverse-proxy forwarding. Do not trust unvalidated forwarded-host headers as the sole source of public URLs; the security design belongs to IDR-SRV-039.

---

## 6. API Definition Behavior Findings

### 6.1 Base Standards Obligations

Common `/req/landing-page/api-definition-op` requires GET on every landing-page link with `service-desc` and every link with `service-doc`. Features `/req/core/api-definition-op` likewise requires GET on every referenced definition. A successful Common response must be `200`, must be an API Definition document, and must agree with the media type selected through content negotiation. Features specifically requires that an `Accept` value equal to the link's `type` return a document consistent with that type.

Common recommends `{root}/api`, but that path is not mandatory. Features permits the definition to be hosted as a resource under the API root and says it need not list itself in its own `paths`. Multiple definition formats and locations are allowed.

### 6.2 OpenAPI Is Conditional, Not Automatic

The base landing requirements mandate an API definition, not OpenAPI specifically. Both Common and Features recommend that an OAS 3.0 definition conform to their optional OAS 3.0 class.

If Glaux claims a Common or Features OAS 3.0 class, the conditional obligations expand materially. They include:

- a JSON OAS 3.0 representation using `application/vnd.oai.openapi+json;version=3.0`;
- a human-readable HTML definition using `text/html`;
- a valid OAS 3.0 document;
- implementation of every advertised capability;
- all server-generated success and error response codes and response objects;
- applicable security schemes and requirements; and
- passing the corresponding OAS abstract tests.

Serving OAS 3.1 or 3.2 does not satisfy a class whose identifier and tests are explicitly OAS 3.0. Glaux may serve a newer OAS without claiming OAS 3.0, or may additionally provide a tested OAS 3.0 representation. IDR-SRV-014 must compare those options.

### 6.3 CSAPI Uses the Definition as Operational Evidence

CSAPI Part 1 requirements 77, 78, 89, and 90 and Part 2 requirements 93, 94, 107, 108, 115, 116, 123, and 124 govern read/write media behavior for GeoJSON, SensorML, JSON, SWE JSON, SWE Text, and SWE Binary. Eleven of their twelve direct abstract tests inspect the implementation API definition to verify media advertisement; Part 2 A.93 also exercises JSON behavior directly.

Common and Features tests additionally use the definition to determine legal parameters, values, capitalization, list encoding, and negative requests. An API definition that returns `200` but contains only `openapi` and `info`, or describes generic standard paths without the deployed server, is not decision-usable evidence.

### 6.4 Recommended Glaux API-Definition Contract

Glaux should publish:

1. a machine-readable, versioned, implementation-specific `service-desc`;
2. a rendered, human-readable `service-doc` for the same contract;
3. stable links with accurate `type` values;
4. externally correct server/base-path information;
5. all implemented paths and operations, including `/` and `/conformance`;
6. exact request and response media types by operation;
7. all server-originated response statuses and response shapes selected by later error/security topics; and
8. a build/version identity that allows the document to be correlated with the running release.

The route table, request/response media matrix, security configuration, and API definition should be derived from the same capability registry. A parity test must compare the generated definition with the live router and release evidence.

The exact OAS version, JSON/YAML variants, modular versus bundled source, generation tool, rendered UI, caching/versioning, and whether to support an additional OAS 3.0 claim remain explicit IDR-SRV-014 decisions. A YAML representation may be useful, but it does not replace the JSON representation required by an OAS 3.0 conformance claim.

### 6.5 Human and AI Use

The human `service-doc` and machine `service-desc` are two views of the same contract. This supports developers, generic clients, generated clients, conformance tools, and AI-enabled development without creating a separate “AI API.” Useful descriptions, examples, stable operation identifiers, schemas, security requirements, and deployed server URLs matter because they reduce guesswork. They are quality requirements or later design choices unless a controlling standard makes them normative.

---

## 7. Conformance Declaration Behavior Findings

### 7.1 Standards Obligations

Common `/req/landing-page/conformance-op` requires GET at fixed `{root}/conformance` and at every landing-page link with the full OGC conformance relation. Features `/req/core/conformance-op` requires GET at `/conformance` and its ATS follows every short `conformance` link. All must return a successful `200` response satisfying the conformance schema.

The response contains required property `conformsTo`, an array of strings listing all OGC API conformance classes to which the server conforms. These are conformance-class identifiers, not requirement-class identifiers, abstract-test identifiers, document anchors, feature flags, or friendly labels.

### 7.2 What the Schema Does Not Prove

The normative schema proves only that `conformsTo` is present and is an array of strings. It does not require or prove:

- URI syntax;
- exact registry membership;
- uniqueness;
- deterministic ordering;
- non-emptiness;
- implementation evidence;
- prerequisite closure;
- completeness; or
- agreement with the API definition and running server.

Glaux must therefore add semantic validation to the structural schema test.

### 7.3 Evidence-Gated Declaration Policy

The accepted IDR-SRV-008 policy remains controlling:

- The **target profile** is all 25 direct CSAPI classes.
- A **released-build declaration** contains only the direct and inherited classes that the build has fully implemented and proved.
- A class becomes eligible only when all its prerequisites, unconditional requirements, triggered conditional requirements, normative abstract tests, and necessary supplemental tests pass for that build.
- Planned, partial, experimental, deferred, or coded-but-unproved behavior never enters `conformsTo`.
- A failed identifier resolver does not authorize changing the literal identifier.

The conformance declaration should be generated as a unique, deterministic list from the release evidence registry. Stable ordering is a Glaux quality rule, not a schema obligation. It improves review, caching, diffs, and reproducible tests.

### 7.4 Direct External Dependencies and Unresolved Recursive Closure

IDR-SRV-008 identified 15 external classes invoked directly by the all-25 direct CSAPI target before optional HTML or OAS classes:

- Common: `core`, `landing-page`, `json`;
- Features Part 1: `core`, `geojson`;
- Features Part 4 draft: `create-replace-delete`, `update`;
- SensorML 3: `json-simple-process`, `json-physical-system`, `json-deployment`, `json-derived-property`; and
- SWE Common 3: `json-record-components`, `json-encoding-rules`, `text-encoding-rules`, `binary-encoding-rules`.

Those 15 are direct dependencies, not the full transitive declaration closure. The official SensorML 3 dependencies add at least `json-core`, `json-aggregate-process`, and `json-physical-component`; SWE Common 3 adds at least `json-block-components`, `json-simple-components`, `json-simple-encodings`, and `general-encoding-rules`. Further indirect model and GeoPose prerequisites may apply. Therefore this report does not assert an exact total for the full evidence-complete declaration. IDR-SRV-051 must recursively audit and trace the complete dependency graph before a release can claim the full profile. The Features Part 4 dependencies remain draft-qualified, and every direct or prerequisite class remains evidence-gated.

### 7.5 Planned, Experimental, and Glaux-Specific Capabilities

The standard conformance document has no status vocabulary for planned, experimental, partial, or deferred capabilities. Adding such strings to `conformsTo` would mislead generic clients.

**Project recommendation:** publish roadmap and experimental information in human documentation and, if later needed, a separate namespaced Glaux capabilities resource. Link it using a relation selected in IDR-SRV-010/014. Never use the standard conformance array as a roadmap or internal feature-flag dump.

### 7.6 Identifier Rules

Use the exact literal `http://www.opengis.net/spec/.../conf/...` identifiers established by IDR-SRV-008. Do not emit:

- `/req/` identifiers;
- `/conf/` identifiers for individual abstract tests in place of a class;
- `https` substitutions made for stylistic reasons;
- old Common Part 2 `0.0` strings;
- nonexistent suffixes such as Part 1 `sampling` or Part 2 `geojson`;
- shortened names such as `common` or `system-features`; or
- prefix matches that merely look like a CSAPI URI.

Part 2 PURL resolver failures are an upstream publication problem tracked under issue #152 and IDR-SRV-057. Glaux must not require successful dereferencing to start or answer `/conformance`.

---

## 8. Representation and Media-Type Implications

### 8.1 Current Normative Baseline

| Resource | Required baseline | Optional/conditional additions |
|---|---|---|
| Landing page | `application/json`, valid JSON, landing schema | HTML or other representations only if implemented; `alternate` links for each |
| Conformance declaration | `application/json`, valid JSON, conformance schema | HTML or other representations only if implemented and accurately linked |
| Machine API definition | Representation identified by `service-desc` link `type`; GET/Accept must agree | OAS JSON/YAML and version selected in IDR-SRV-014 |
| Human API documentation | `text/html` when advertised as `service-doc` | Renderer and versioning selected in IDR-SRV-014 |

Common/Features do not make HTML landing and conformance representations a CSAPI prerequisite. Serving a `text/html` `service-doc` does not by itself claim the Common or Features HTML class for API resources.

### 8.2 Negotiation Boundary

HTTP content negotiation applies. Common notes that an `f` or `-f` style override may be used, but it is not mandatory. CSAPI Explorer currently appends `f=json`, so supporting a conventional format parameter may improve interoperability; its exact syntax, precedence against `Accept`, q-values, parameters, defaults, and failure behavior belong to IDR-SRV-012.

**Project recommendation:** always return a concrete registered `Content-Type`, never placeholders such as `auto` or a missing header. Link `type` and response `Content-Type` must be tested together.

### 8.3 Alternate Representations

When Glaux supports multiple landing or conformance representations, each representation should have a `self` link and `alternate` links for every other supported type. The link graph, negotiated response, query-format override, and API definition must agree. Do not advertise an alternate representation until it returns successfully and passes its schema/content tests.

---

## 9. Error, Failure, and Stale-Description Implications

### 9.1 Standards Guidance

Common and Features identify typical HTTP statuses including `405` for an unsupported method, `406` when content negotiation fails, and `500` for an internal server error. Features recommends HEAD for resources supporting GET. Common recommends Problem Details and `application/problem+json` for JSON error reports. These are general HTTP/error rules; IDR-SRV-013 must define the final Glaux status/body policy.

### 9.2 Broken or Unavailable API Definition

A landing page may not advertise a broken definition and remain conformant to the entry requirements: every advertised definition link must support GET, and a successful request must return `200` with the promised representation. The standards do not define a special “documentation stale” status.

**Project recommendation:** build and validate the machine and human definitions as release artifacts and fail deployment/readiness if they cannot be served. A transient post-startup failure may warrant `503`, but IDR-SRV-013 must decide exact runtime semantics. Do not silently omit the required API-definition link or continue serving a knowingly false definition.

### 9.3 Stale or Incomplete Definition

If an OAS 3.0 class is claimed, live-definition parity, response completeness, errors, and security are direct class obligations. Even without that optional claim, CSAPI media abstract tests and normal client behavior depend on accurate advertisement.

**Project recommendation:** treat route/OAS/media/declaration disagreement as a release-blocking defect. At minimum test:

- every live route is represented when intended by the selected OAS strategy;
- every advertised route exists;
- the public server/base path is correct;
- request and response media types match handlers;
- entry points appear in the combined definition;
- security and server-originated response codes selected by later topics are represented; and
- build/version identifiers agree.

### 9.4 Unsupported Representation and Method

Use `406` when no acceptable representation can be produced, subject to the final IDR-SRV-012/013 policy. Use normal HTTP method handling, with `405` and an accurate `Allow` header where applicable. HEAD support should be evaluated across all GET resources. Do not return a successful body under an unrelated or missing content type.

### 9.5 Declaration Failure Modes

Overclaiming and underclaiming are separate failures. Overclaiming tells clients a capability is safe when it is not. Underclaiming hides implemented standard capability and prevents generic clients from using it. Structural schema success does not catch either.

The release validator must reject duplicates, unknown identifiers, obsolete versions, aliases, missing prerequisites, roadmap leakage, missing evidence, implemented-but-undeclared target classes, and disagreement with the runtime/OAS media matrix.

---

## 10. OpenAPI Artifact and Upstream-History Findings

### 10.1 Tagged Modular OAS

Both official documents are explicitly example OAS 3.1.0 definitions with `info.version: 0.0.1`. Both advertise fixed demonstration/local servers. Neither defines an API-definition route or `operationId`. The absence of the definition route from its own `paths` is permitted, but the linked runtime definition must still exist.

Part 1 defines 20 paths, including `/` and `/conformance`. Those entry operations describe only a `200` JSON response and omit alternate representations, explicit errors, operation-specific security, and richer negotiation behavior. Part 2 defines 23 additive paths and contains neither `/` nor `/conformance`. A complete Glaux description must combine the implemented Parts 1 and 2 surface rather than serve Part 2's file alone.

### 10.2 Tagged Landing Example Defects

The [tagged landing example](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/examples/landingPage.json):

- omits `type` and `title` on its API-definition link;
- uses only short `conformance`, omitting the Common full relation URI;
- uses `collections` instead of required Features `data`;
- omits `self`, representation alternates, and `service-doc`; and
- points to a generic example URL.

The absence of direct links to every CSAPI resource is not a defect; Part 1 says inherited links are sufficient.

### 10.3 Tagged Conformance Example Defects

The [tagged conformance response component](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/responses/conformance.yaml) and its linked [example payload](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/examples/confClasses.json):

- use the description “requirements classes” even though the example payload lists conformance-class identifiers;
- claims Common/Features HTML without corresponding entry representations in the OAS;
- claims Common OAS 3.0 although the artifact is OAS 3.1;
- contains obsolete Common Part 2 `0.0` identifiers;
- contains nonexistent Part 1 `/conf/sampling` and Part 2 `/conf/geojson` classes;
- omits `/conf/api-common` and most direct CSAPI classes; and
- does not represent the accepted all-class target or any defensible release profile.

It must never seed a Glaux constant or fixture asserted as valid.

### 10.4 Released Bundle Limitations

The release assets inline the Part 1 entry schemas/examples, but are not fully reference-closed overall. Part 1 retains 32 `../examples/...` references. Part 2 retains 51 relative references—45 examples and six schemas. Those sibling resources are not delivered beside a separately downloaded GitHub release asset.

This does not change a standards obligation. It demonstrates why Glaux must validate and publish its own versioned implementation definition rather than expose the official bundle unchanged.

### 10.5 Relevant Official Issues and Pull Requests

| Item | Current disposition | IDR-SRV-009 meaning |
|---|---|---|
| [#12](https://github.com/opengeospatial/ogcapi-connected-systems/issues/12), [#13](https://github.com/opengeospatial/ogcapi-connected-systems/issues/13) | Closed | Confirm Part 1/2 OAS provenance as examples, not normative deployment contracts |
| [#23](https://github.com/opengeospatial/ogcapi-connected-systems/issues/23) | Open | Class-level encoding claims do not express every operation/media/schema combination; use OAS for detail, without treating the proposal as adopted |
| [#28](https://github.com/opengeospatial/ogcapi-connected-systems/issues/28), [PR #29](https://github.com/opengeospatial/ogcapi-connected-systems/pull/29) | Closed/merged, in release | Explains adopted identifier structure; does not validate stale example values |
| [#48](https://github.com/opengeospatial/ogcapi-connected-systems/issues/48) | Open | Original OAS 3.0 downgrade request is superseded by 3.1 direction, but formal resolution remains incomplete |
| [#77](https://github.com/opengeospatial/ogcapi-connected-systems/issues/77) | Open | OAS 3.1 was selected and both 3.0/3.1 bundles were discussed; release supplies only 3.1 |
| [#78](https://github.com/opengeospatial/ogcapi-connected-systems/issues/78), [#79](https://github.com/opengeospatial/ogcapi-connected-systems/issues/79), [#137](https://github.com/opengeospatial/ogcapi-connected-systems/issues/137) | Closed | Bundling automation produced release assets but not fully reference-closed documents |
| [#152](https://github.com/opengeospatial/ogcapi-connected-systems/issues/152) | Open | Part 2 PURL mappings remain incomplete; literal identifiers stay normative |
| [#186](https://github.com/opengeospatial/ogcapi-connected-systems/issues/186) | Open proposal | Versioned bundle use could improve discovery, but its standalone-consumption acceptance criterion is not currently met |

Current official `master` has no post-tag landing, API-definition, or conformance correction. The relevant example and OAS findings therefore apply to both the published tag and current branch as checked.

---

## 11. Interoperability and Existing-Implementation Findings

### 11.1 CSAPI Explorer

Pinned CSAPI Explorer behavior shows that it:

- requests JSON using `f=json`;
- identifies a landing page through API-definition and conformance links;
- can walk upward from a nested starting URL;
- resolves relative `href` values;
- follows the conformance link;
- uses `data`, not `collections`, for collection discovery; and
- does not retrieve or validate whether the advertised OAS is complete or whether conformance identifiers are canonical.

Its broad prefix matching can accept nonexistent CSAPI suffixes. Glaux must interoperate through precise standards behavior, not imitate that leniency. A passing Explorer smoke test proves discoverability, not conformance truth.

### 11.2 OpenSensorHub

Pinned source and preserved HTTP evidence show a static conformance list containing draft, obsolete, or noncanonical identifiers; a landing page using `collections` and omitting `self`; and links to generic official OAS files rather than an instance-specific deployment definition. A live read-only recheck also observed nonconcrete `Content-Type: auto` entry responses. These are useful negative cases, not a profile oracle.

### 11.3 connected-systems-go

Its landing handler provides a useful positive pattern: configured base URL and typed `self`, `service-desc`, `conformance`, and `data` links. Its hardcoded conformance list contains invalid/nonexistent identifiers, and a live `/api` check returned an extremely small OAS object without paths or server information. This combination demonstrates why correct links alone are insufficient and why hardcoded declarations drift.

### 11.4 pygeoapi

Current upstream pygeoapi provides a useful architecture precedent outside CSAPI: it generates an instance-specific OAS, derives conformance from enabled providers, deduplicates and deterministically sorts declarations, and tests that shared global state is not mutated. Glaux should reuse the principle, not its product-specific class list or architecture.

### 11.5 SECD and Other Public Deployments

SECD evidence provides a positive example of a typed landing page and instance-specific OAS with correct public server information, while its noncanonical CSAPI suffixes remain negative conformance fixtures. A check of the 52North deployment exposed an expired TLS certificate, an invalid `Content-Type: None`, no CSAPI declaration, and a localhost OAS server. Normal TLS access was therefore blocked and these HTTP observations required diagnostic certificate bypass. They support offline fixture preservation, public-base tests, TLS-independent contract tests, and exact identifiers. They are dated implementation evidence, not normative findings.

The following read-only checks were performed on 2026-08-01. Deployed binary versions were not identifiable and must not be equated automatically with the repository pins in §3.3.

| Deployment | Exact endpoints checked | Bounded observation |
|---|---|---|
| OpenSensorHub S1 | [`/sensorhub/api`](https://129-80-248-53.sslip.io/sensorhub/api); [`/sensorhub/api/conformance`](https://129-80-248-53.sslip.io/sensorhub/api/conformance); linked [Part 1](https://opengeospatial.github.io/ogcapi-connected-systems/api/part1/openapi/openapi-connectedsystems-1.yaml) and [Part 2](https://opengeospatial.github.io/ogcapi-connected-systems/api/part2/openapi/openapi-connectedsystems-2.yaml) OAS | JSON required `?f=json`; default responses reported `Content-Type: auto`; root lacked `self`; declaration contained obsolete/noncanonical strings; linked OAS documents were generic upstream examples rather than an instance definition |
| connected-systems-go S2 | [`/csapi-go-v2/`](https://129-80-248-53.sslip.io/csapi-go-v2/); [`/conformance`](https://129-80-248-53.sslip.io/csapi-go-v2/conformance); [`/api`](https://129-80-248-53.sslip.io/csapi-go-v2/api) | Root used useful standard relations; declaration contained known invalid/nonexistent identifiers; the advertised 88-byte OAS 3.0 JSON response had no `paths`, `servers`, or components |
| 52North S6 | [`/`](https://csa.demo.52north.org/); [`/conformance`](https://csa.demo.52north.org/conformance); [`/openapi`](https://csa.demo.52north.org/openapi) | Expired certificate blocked normal TLS; diagnostic bypass exposed `Content-Type: None`, no CSAPI declaration, and an OAS 3.1 document that advertised `http://localhost:5000` |
| SECD S4 | [`/api/1.0`](https://cs.ogc.secd.eu/api/1.0); [`/conformance`](https://cs.ogc.secd.eu/api/1.0/conformance); [`/api-docs/openapi.json`](https://cs.ogc.secd.eu/api/1.0/api-docs/openapi.json) | Typed discovery links and a deployment-specific OAS 3.1 document used the correct public base; the declaration's CSAPI suffixes were noncanonical |

### 11.6 Interoperability Conclusion

The recurring failure is independent documents maintained by hand: one root response, one hardcoded class list, one generic OAS, and one runtime router. Glaux should instead expose consistent projections of one release capability model. External clients should not need product-specific fallback logic to compensate for wrong relations, missing content types, invalid class names, or localhost URLs.

---

## 12. Unified Entry-Point Behavior Matrix

The matrix contains every minimum field required by the topic plan. `Normative` means an approved obligation; `Recommended` means a standard recommendation; `Project` means the proposed Glaux quality rule; and `Conditional` means it becomes normative when the named class/feature is claimed.

| Behavior area | Source / anchor | Requirement or convention summary | Classification | Glaux implication | Related class / requirement | Schema / OAS artifact | Test implication | Handoff | Notes / unresolved |
|---|---|---|---|---|---|---|---|---|---|
| API root | Common Req 12; Features Req 1 | GET `{root}/` | Normative | Route at configured public API root, including path prefixes | Common landing; Features core | Landing schema | GET, 200, direct/proxy paths | 010A, 050 | Domain root not required |
| Root representation | Common Req 13; Features Req 2; Common Reqs 20–21 | 200, schema-based body, JSON support | Normative | Always serve valid `application/json` with `links` | Common landing/json; Features core | `landingPage.yaml` | Schema, JSON, Content-Type | 012, 023, 050 | HTML optional |
| API-definition discovery | Common Req 13; Features Req 2 | At least `service-desc` or `service-doc` | Normative | Publish both machine and human links | Common landing; Features core | Link schema | Presence, type, dereference | 014, 050 | Exact OAS type pending 014 |
| Conformance discovery | Common Reqs 13/16; Features Reqs 2/5 | Full URI relation and short relation | Normative reconciliation | Emit both links to same resource | Common landing; Features core | Link schema | Select/follow both independently | 010, 050, 056 | Lexical values differ |
| Collections discovery | Features Req 2 | `data` link to `/collections` | Normative | Use `data`, not example `collections` | Features core | Landing example is negative fixture | Explorer + ETS discovery | 010, 056 | Exact collections behavior belongs 010 |
| Self/alternates | Features root-links recommendation | `self` and all representation alternates | Recommended | Treat as release-quality baseline | Features core | Link schema | Cross-link/media closure | 012, 050 | Erroneous `/req/` identifier still uses SHOULD |
| HTTP Link headers | Common Recommendation 1 and Features Recommendation 10, both `/rec/core/link-header` | Reproduce applicable payload links as HTTP `Link` headers unless impractical | Recommended | Generate headers from the same link model as the body | Common/Features core | Link objects / HTTP headers | Body/header parity and quoting | 012, 050 | Headers supplement rather than replace body links |
| Landing metadata | Common schema/prose | Optional title, description, attribution; title strongly advised | Recommended/project | Include configurable title/description; attribution where appropriate | Common landing | Common landing schema | Presence/configuration/escaping | 014, 039 | Attribution may contain markup |
| Direct CS links | CSAPI P1 §8.2 | Inherited common links are sufficient | Optional | Do not invent required root graph here | CSAPI P1 API Common | Tagged example | Test only adopted extras | 010 | Systems/etc. navigation deferred |
| Definition retrieval | Common Req 14; Features Req 3 | GET every `service-desc` and `service-doc` | Normative | Every advertised link must return successfully | Common landing; Features core | Runtime definition | Follow every link, 200 | 014, 050 | `{root}/api` recommended, not required |
| Definition media agreement | Common Req 15; Features Req 4 | Response agrees with negotiation/link `type` | Normative | Always include accurate link `type` and response Content-Type | Common landing; Features core | OAS/HTML output | Accept equal to type; parse | 012, 014, 023, 050 | `f` optional |
| Optional OAS 3.0 class | Common Reqs 22–27; Features Reqs 46–51 | JSON OAS3 + HTML; validity; implementation/error/security parity | Conditional normative | Do not declare until independently proved | Common/Features `oas30` | OAS 3.0 document | ATS plus parity/security/error tests | 013, 014, 023, 050 | OAS 3.1 does not automatically qualify |
| CSAPI media support / ATS advertisement | P1 77/78/89/90; P2 93/94/107/108/115/116/123/124 | Runtime `Accept`/`Content-Type` media support is normative; 11 of 12 ATS verify it through API-definition advertisement, while P2 A.93 exercises runtime JSON retrieval | Conditional normative | Derive both handlers and OAS media matrix from the same class registry | Claimed CSAPI encoding/transaction classes | Runtime handlers and implementation OAS | 12 direct tests: 11 definition-based; A.93 runtime; A.115 adapter | 012, 014, 050, 051 | Issue #23 remains open |
| Conformance operation | Common Req 16; Features Req 5 | GET fixed path and advertised links | Normative | Serve same representation at all advertised targets | Common landing; Features core | Conformance response | Fixed path plus both links | 050, 056 | Do not rely only on link discovery |
| Conformance content | Common Req 17; Features Req 6 | `conformsTo` lists all classes actually satisfied | Normative | Generate evidence-complete exact identifiers | Direct/inherited classes | `confClasses.yaml` | Schema plus semantic registry/dependency checks | 050, 051 | Schema is permissive |
| Planned/experimental state | No standard status model | Do not mix roadmap state into conformance | Project | Separate docs/namespaced capability resource | None | Human docs | Assert no roadmap leakage | 010, 014, 051 | Relation/resource design deferred |
| Unsupported media/method | HTTP status guidance | 406 negotiation; 405 unsupported method; HEAD recommended | Recommended/HTTP | Apply consistent later error policy | Common/Features core | API definition error responses | Negative Accept/method/Allow/HEAD | 012, 013, 050 | Exact bodies/status edge cases deferred |
| Stale/broken definition | Entry requirements; optional OAS class; CSAPI media ATS | Broken link or false definition fails the relevant contract | Normative consequence/project | Fail build/deployment readiness on drift | Entry, media, optional OAS classes | Router/OAS/build registry | Route/OAS/media/declaration parity | 014, 023, 048, 050 | Runtime 503 policy deferred |
| Public deployment URLs | Link/OAS semantics and observed failures | Links and servers must describe reachable deployment | Project interoperability rule | Validated public base; proxy-aware output | Cross-cutting | Landing/OAS `servers` | Direct, base-path, reverse-proxy tests | 010A, 014, 039, 046 | Avoid localhost/demo/internal hosts |

---

## 13. Test-Strategy Implications

### 13.1 Normative Test Lanes

1. Execute the 28 directly relevant Common, Features, and CSAPI abstract-test obligations identified in §4.2.
2. Execute Common A.2–A.11 and Features A.24–A.25 where the API definition supplies parameters and negative-test input.
3. Run the official Features ETS for inherited Features behavior.
4. Adapt Common JSON A.18 to its normative landing/conformance scope rather than forcing JSON on the API definition.
5. Adapt Part 2 A.115's copy/paste error: the Text class must advertise `application/swe+text`, not `application/swe+binary`.
6. Record every adapter against the normative requirement, defective test text, implementation behavior, and supplemental test.

The normative Features ATS requires landing-page conformance-link testing. The current executable Features ETS instead derives conformance test points through the API definition, and its API-definition test recognizes only the exact OAS 3.0 JSON media type. The ETS therefore does not cover Glaux's dual conformance relations or an OAS 3.1-only description; those remain explicit Glaux supplemental tests.

### 13.2 Positive Tests

- `GET {root}/` and `GET {root}/conformance` return `200` with `application/json` and validate structurally.
- Root includes `self`, at least one correctly typed machine definition, a human definition, both conformance relation forms, and `data`.
- Every relative advertised link resolves correctly against the configured public base; every absolute external link is explicitly allowed, reachable, and accurately typed.
- Machine definition parses, contains meaningful `paths`, includes `/` and `/conformance`, advertises the deployed server/base path, and represents implemented routes/media.
- Conformance is nonempty for a claimed CSAPI build, unique, deterministic, allowlisted, complete, and prerequisite-closed.
- Every declared class has release evidence and every evidence-complete implemented target class is declared.
- Repeated requests are stable and do not mutate shared capability state.
- Direct and reverse-proxied deployments produce equivalent externally correct contracts.

### 13.3 Negative Tests

- Missing `links`, API-definition, either conformance relation, or `data`.
- `collections` substituted for `data` without the required relation.
- Malformed link object; broken href; wrong host/path prefix; inaccurate or absent `type`; wrong Content-Type.
- Unsupported `Accept`, invalid format parameter, unsupported method, and incorrect HEAD/Allow handling after IDR-SRV-012/013 decisions.
- API definition returns `200` but lacks paths, points to localhost/demo servers, omits live operations, advertises nonexistent operations, or disagrees on media/security/errors.
- Missing `conformsTo`; wrong property type; non-string member; duplicate; empty build declaration; unknown or obsolete class; `/req/` string; test URI; alias; prefix-only false positive.
- Overclaim, underclaim, missing prerequisite, draft qualification loss, planned/experimental leakage, and stale evidence.
- Literal `/conformance`, short relation link, and full relation link return different content.
- Official tagged landing/conformance examples and sampled implementation failures are retained as named negative fixtures.

### 13.4 Interoperability Tests

- Bootstrap an unmodified CSAPI Explorer from the root and from a nested resource URL.
- Run the official Features ETS as a separate inherited-conformance lane.
- Validate the machine definition with at least two independent parsers/validators selected in IDR-SRV-014/023.
- Exercise browser/human documentation and generated-client flows.
- Exercise absolute and relative link resolution if both are supported.
- Preserve dated offline fixtures from OpenSensorHub, connected-systems-go, SECD, and other public deployments so external availability does not make regression tests nondeterministic.

### 13.5 What Passing Does and Does Not Prove

Passing CSAPI Explorer proves basic discoverability for that client, not exact declarations or OAS truth. Passing the Features ETS proves inherited Features behavior, not a direct CSAPI class. Parsing an OAS proves syntax, not route parity. Schema-valid `conformsTo` proves shape, not correctness. The conformance claim gate must combine all applicable evidence lanes.

---

## 14. Decision Analysis and Recommendations

### 14.1 Decision Options

| Decision | Option | Benefits | Costs/risks | Standards/compatibility impact | Recommendation |
|---|---|---|---|---|---|
| Entry-point source | Copy official examples/OAS | Fast initial appearance | Known invalid identifiers, demo URLs, incomplete responses, unresolved refs | Can fail inherited and semantic behavior | Reject |
| Entry-point source | Maintain root, OAS, routes, and class list independently | Simple local code per endpoint | High drift/overclaim risk observed in implementations | Difficult to prove parity | Reject |
| Entry-point source | Generate all external views from one capability registry | Consistent, testable, release-specific | Requires disciplined model and build validation | Strongest compatibility and evidence posture | Adopt |
| Conformance relation | Emit only short or only full value | Smaller response | Fails one inherited lexical contract/client path | Incomplete dual inheritance | Reject |
| Conformance relation | Emit both to one target | Satisfies both standards and clients | Two link objects | Conservative exact compatibility | Adopt |
| API definition | Serve generic official CSAPI OAS | Standards-shaped source | Not deployment-specific; known defects | Misleads clients and tools | Reject |
| API definition | Publish Glaux-specific definition and human view | Accurate release contract | Generation/validation work | Meets base entry requirements; optional class separate | Adopt |
| OAS class | Preclaim OAS 3.0 because an OAS exists | Attractive class list | False if serving 3.1 only or lacking HTML/parity | Direct overclaim | Reject |
| OAS class | Withhold pending IDR-SRV-014 and proof | Honest staged behavior | Fewer initial claims | Fully compatible with modular conformance | Adopt |
| Declaration state | Publish target/roadmap classes | Shows ambition | Misleads generic clients | Violates “classes the API conforms to” meaning | Reject |
| Declaration state | Publish evidence-complete released classes only | Honest and automatable | Requires evidence registry | Aligns with standards and IDR-SRV-008 | Adopt |

### 14.2 Key Recommendations

1. **Create one versioned capability registry as the entry-point source of truth.**
   - It should represent routes, methods, media types, schemas, enabled classes, prerequisites, release evidence, public base, and build version.
   - Landing links, conformance output, runtime API definition, and release validation should be projections of it.
   - Priority: High.

2. **Adopt the exact landing discovery spine in §5.2.**
   - Include `self`, typed `service-desc`, typed `service-doc`, both conformance relations, and `data`.
   - Add alternates only when they are implemented and tested.
   - Priority: High.

3. **Publish a Glaux-owned implementation API definition, not the official example.**
   - It must describe the running release and external deployment.
   - The OAS version and generation strategy remain for IDR-SRV-014.
   - Priority: High.

4. **Generate `/conformance` from evidence-complete classes and exact identifiers.**
   - Enforce allowlisting, uniqueness, stable ordering, dependency closure, completeness, and parity.
   - Keep planned/experimental state outside `conformsTo`.
   - Priority: High.

5. **Make contract parity a release and readiness gate.**
   - Reject builds where route, media, schema, OAS, link, declaration, public-base, or evidence views disagree.
   - Priority: High.

6. **Use official defects and implementation failures as regression fixtures.**
   - Preserve provenance and expected failure reason.
   - Do not normalize them into valid examples.
   - Priority: Medium.

7. **Run several independent evidence lanes.**
   - Common/Features/CSAPI ATS, official Features ETS, schema validation, semantic declaration validation, OAS parsing/parity, reverse-proxy tests, and external-client smoke tests each prove different things.
   - Priority: High.

### 14.3 What Acceptance Decides

Acceptance makes the recommendations above the baseline for later planning. It does not approve a particular Rust web framework, OAS generator, documentation UI, deployment architecture, authentication model, API version path, content-negotiation algorithm, or error body. Those remain with their indexed topics.

---

## 15. Implementation Implications and Relative Estimates

### 15.1 Architecture Implications

- The Rust service architecture needs a capability model available to routing, representation, documentation, conformance, startup validation, and tests without mutable global drift.
- Public-base construction must be configuration-aware and security-reviewed.
- The conformance registry needs exact identifiers, prerequisites, conditions, evidence state, and release gating—not just strings.
- OAS generation or assembly must support the complete combined Part 1/Part 2 deployment and be checked against live routes.
- Entry resources should be deterministic and inexpensive to serve; immutable per-build material can be generated at build/startup and cached where appropriate.

### 15.2 Relative Planning Estimate

These are complexity bands, not calendar commitments. Framework, security, deployment, content-negotiation, and OAS decisions are still pending.

| Work item | Relative complexity | Planning estimate | Assumptions |
|---|---|---|---|
| Capability/evidence registry model | High | One major implementation unit | Must support all later classes and release evidence, not only three endpoints |
| JSON landing and conformance handlers | Low–Medium | One small implementation unit | Registry and public-base utilities already exist |
| Link/public-base/proxy handling | Medium | One medium unit | Direct and reverse-proxy deployment both supported |
| Glaux-specific machine/human API definition | High | One major unit | Exact OAS/rendering strategy selected in IDR-SRV-014 |
| Route/OAS/media/declaration parity validator | High | One major unit | Router and schemas expose machine-readable metadata |
| Entry-point normative and negative tests | Medium–High | One major test unit | Harness architecture from IDR-SRV-050/051 available |
| External-client/ETS interoperability lane | Medium | One medium test unit | Reproducible containers/fixtures selected later |

The entry handlers themselves are not the hard part. The demanding work is maintaining a truthful relationship among routes, representations, documentation, class evidence, and deployments across the full server.

---

## 16. Risks, Constraints, and Open Questions

### 16.1 Risk Register

| Risk / constraint | Consequence | Control / owner |
|---|---|---|
| Only one conformance relation emitted | Fails one inherited standard/client discovery path | Dual-link baseline; IDR-SRV-010/050 |
| Official examples copied | Invalid relations/classes and misleading profile | Negative fixtures; generated Glaux contract |
| Independent hardcoded documents | OAS/declaration/runtime drift and overclaim | One registry; release parity gate |
| OAS version selected casually | Incorrect optional class claim or tool incompatibility | IDR-SRV-014 decision and separate validation lanes |
| Permissive schemas treated as semantic proof | Unknown, duplicate, incomplete, or false classes pass | Supplemental registry/dependency/evidence validation |
| PURL availability treated as identity | Correct Part 2 identifiers rejected or rewritten | Literal identifier registry; separate resolver monitoring |
| Public base inferred incorrectly | Clients receive localhost/internal/wrong-path URLs | Validated config and proxy/security tests |
| Features ETS treated as CSAPI certification | False confidence | Explicit evidence-lane labels; CSAPI ATS harness |
| Common A.18 / Part 2 A.115 defects followed literally | False JSON/API-definition or binary/text expectations | Documented adapters in IDR-SRV-050/051 |
| Released OAS bundles assumed standalone | Parser/generator failures on relative refs | Glaux-owned reference-closed publication |
| Fifteen direct external dependencies mistaken for full closure | Required transitive SensorML/SWE/model classes are omitted from the declaration | Recursive dependency audit and traceability gate in IDR-SRV-051 |
| Full target declared before evidence | Credibility and interoperability harm | Evidence-gated per-release declaration |
| Features Part 4 remains draft | Qualification and future-delta risk for four direct target classes | Pin, qualify, monitor, and review at publication |

### 16.2 Unresolved Questions and Disposition

1. **Which OAS version(s) will Glaux publish and claim?**
   - Unresolved by design; IDR-SRV-014 owns the decision. Current baseline requires an accurate machine definition and human documentation but no OAS 3.0 class preclaim.
2. **Will landing and conformance support HTML representations?**
   - IDR-SRV-012/014 must assess value and cost. JSON remains mandatory; HTML class claims remain withheld.
3. **What exact `f` parameter and negotiation precedence will be supported?**
   - IDR-SRV-012 owns it. Explorer behavior is useful evidence, not authority.
4. **Which additional direct CSAPI-resource links belong on the landing page?**
   - IDR-SRV-010 owns the graph. None is required by the current entry baseline.
5. **What error bodies/statuses apply to transient definition or capability-registry failure?**
   - IDR-SRV-013 owns the final policy. Build/readiness failure is recommended for known inconsistency.
6. **How will evidence records be persisted, signed, or audited?**
   - IDR-SRV-041, 050, and 051 will refine accountability and traceability.
7. **How should a separate Glaux experimental-capabilities resource be related and represented?**
   - Only if later needed; IDR-SRV-010/014. It must remain outside `conformsTo`.
8. **What is the exact recursive prerequisite closure for the full 25-class CSAPI target?**
   - IDR-SRV-008 recorded 15 directly invoked external classes, not a complete transitive closure. IDR-SRV-051 must audit the SensorML, SWE Common, model, GeoPose, and other inherited graphs before the full release declaration is fixed.

No unresolved question prevents this behavior baseline from being accepted or used by the next eligible research topic.

---

## 17. Downstream Topic Handoff Matrix

| Topic | Required handoff from IDR-SRV-009 |
|---|---|
| IDR-SRV-010 | Preserve the mandatory discovery spine; decide registered extra links and navigation from collections to all CSAPI resources; test resolvability |
| IDR-SRV-010A | Decide stable/versioned root and API-definition URLs, backward-compatible link behavior, deprecation, and build identity |
| IDR-SRV-011 | Ensure query parameters and selection semantics are represented accurately in the implementation definition |
| IDR-SRV-012 | Define `Accept`, `f`, q-values, media parameters, defaults, Content-Type, alternates, and 406/415 boundaries |
| IDR-SRV-013 | Define Problem Details, 4xx/5xx responses, stale/unavailable documentation, readiness, and method errors |
| IDR-SRV-014 | Select OAS version(s), JSON/YAML exposure, generator/source, combined definition, reference closure, renderer, operation IDs, validation, and optional OAS-class posture |
| IDR-SRV-014A–G | Deepen implementation/client lessons without changing normative authority; add named fixtures and deltas |
| IDR-SRV-023 | Validate landing/conformance schemas, OAS structure/references/dialects, link semantics, and artifact closure |
| IDR-SRV-039/046/048 | Secure public-base/proxy trust, deployment configuration, health/readiness, and observable contract failures |
| IDR-SRV-050 | Implement the 28 direct tests, cross-cutting API-definition tests, ATS adapters, semantic declaration checks, and Features ETS lane |
| IDR-SRV-051 | Recursively close every direct and transitive class dependency, then trace class → prerequisite → requirement → route/media/schema → test → released evidence → declaration string |
| IDR-SRV-056 | Exercise Explorer, generic OGC clients, browsers, generated clients, and deployment/base-path variants |
| IDR-SRV-057 | Refresh issues #23/#48/#77/#152/#186, PURL health, OAS releases, and any accepted later decisions |

---

## 18. Validation Against the Research Plan

### 18.1 Methodology-Phase Validation

| Phase | Status | Evidence |
|---|---|---|
| 1. Source collection and setup | Complete | §§3–4 |
| 2. Landing extraction | Complete | §5, §12 |
| 3. API-definition extraction | Complete | §6, §§8–10, §12 |
| 4. Conformance extraction | Complete | §7, §10, §12 |
| 5. Representation/error/interoperability/test analysis | Complete | §§8–13 |
| 6. Synthesis | Complete | §§1, 14–17, Appendices |

### 18.2 Success-Criterion Validation

| Topic-plan success criterion | Status | Evidence |
|---|---|---|
| Landing requirements/conventions identified with anchors | Met | §§3, 5, 12 |
| API-definition requirements/conventions identified with anchors | Met | §§3, 6, 10, 12 |
| Conformance-declaration requirements/conventions identified with anchors | Met | §§3, 7, 10, 12 |
| Inherited OGC behavior identified | Met | §§2.4, 4, 5–8 |
| CSAPI-specific behavior distinguished from general OGC behavior | Met | §§5.1, 5.4, 6.3, 7.4 |
| Declaration aligned to IDR-SRV-008 | Met | §§2.4, 7.3–7.6 |
| Representation, negotiation, error, OAS, validation, conformance, and testing handoffs documented | Met | §§8–9, 13, 17 |
| Incomplete, stale, and overclaim risks identified | Met | §§9–10, 16 |
| Recommendations are bounded and decision-usable | Met | §§1.3, 14–15 |
| References are explicit and reproducible | Met | §§3, 10–11, 20 |

### 18.3 Required-Content Validation

| Required content | Location | Status |
|---|---|---|
| 1. Executive summary | §1 | Present |
| 2. Scope and plan alignment | §2 | Present |
| 3. Evidence and authority | §3 | Present |
| 4. Landing findings | §5 | Present |
| 5. API-definition findings | §6 | Present |
| 6. Conformance findings | §7 | Present |
| 7. Representation/media implications | §8 | Present |
| 8. Error/failure/stale implications | §9 | Present |
| 9. Interoperability/implementation implications | §11 | Present |
| 10. Test implications | §13 | Present |
| 11. Downstream handoff matrix | §17 | Present |
| 12. Recommendations | §14 | Present |
| 13. Risks/constraints/open questions | §16 | Present |
| 14. Success validation | §18.2 | Present |
| 15. References | §20 | Present |

### 18.4 Behavior-Matrix Validation

Section 12 contains every required field: behavior area; source/anchor; summary; classification; Glaux implication; related class/requirement; related schema/OAS artifact; test implication; downstream handoff; and notes/unresolved issues.

### 18.5 Independent Review and Reconciliation

Three independent read-only audits covered: (1) exact normative requirements, ATS, inheritance, identifier closure, and defects; (2) tagged/released OAS, official issues/PRs, pins, hashes, and register updates; and (3) official ETS, CSAPI Explorer, implementations, live spot checks, interoperability fixtures, and exemplar-method alignment. Their overlapping findings agreed on the mandatory spine, dual relations, informative-example defects, evidence-generated declaration, deployment-specific OAS, and multi-lane testing. Differences in emphasis were reconciled through the authority rules in §3.4.

---

## 19. Next Steps and Handoff

### 19.1 Current Status

Research, drafting, source pinning, three independent technical audits, reconciliation, and plan validation are complete. The report is **In Review**. Plan-owner acceptance remains pending and is intentionally not recorded by the author.

### 19.2 The Next Two Actions

If the plan owner finds this report decision-usable, the next two actions are:

1. approve IDR-SRV-009, which records acceptance and makes its baseline available downstream; then
2. authorize execution of exactly one next eligible topic, IDR-SRV-010, *Collections, Resources, Links, and Navigation Behavior*.

The combined response may be:

> Approve IDR-SRV-009. Then execute exactly one Glaux Server research plan: the next one, using the standing single-topic execution instructions.

This report does not record its own acceptance and does not begin IDR-SRV-010.

---

## 20. References

### 20.1 Controlling and Inherited Standards

- [OGC API - Connected Systems - Part 1: Feature Resources, OGC 23-001](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2: Dynamic Data, OGC 23-002](https://docs.ogc.org/is/23-002/23-002.html)
- [OGC API - Common - Part 1: Core, OGC 19-072](https://docs.ogc.org/is/19-072/19-072.html)
- [OGC API - Features - Part 1: Core Corrigendum, OGC 17-069r4](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [Versioned Common landing-page schema](https://schemas.opengis.net/ogcapi/common/part1/1.0/openapi/schemas/landingPage.yaml)
- [Versioned Common conformance schema](https://schemas.opengis.net/ogcapi/common/part1/1.0/openapi/schemas/confClasses.yaml)
- [OpenAPI Specification 3.0.3](https://spec.openapis.org/oas/v3.0.3.html)
- [OpenAPI Specification 3.1.0](https://spec.openapis.org/oas/v3.1.0.html)
- [Latest published OpenAPI Specification](https://spec.openapis.org/oas/latest.html)

### 20.2 Official CSAPI Artifacts and History

- [Official CSAPI `v1.0.0` source](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)
- [Part 1 modular OAS](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/openapi-connectedsystems-1.yaml)
- [Part 2 modular OAS](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/openapi-connectedsystems-2.yaml)
- [Tagged landing example](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/examples/landingPage.json)
- [Tagged conformance example](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/examples/confClasses.json)
- [Release `v1.0.0`](https://github.com/opengeospatial/ogcapi-connected-systems/releases/tag/v1.0.0)
- [Issues #12, #13, #23, #28, #48, #77, #78, #79, #137, #152, and #186](https://github.com/opengeospatial/ogcapi-connected-systems/issues)
- [Glaux upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md)

### 20.3 Test, Client, and Implementation Evidence

- [Official OGC API - Features 1.0 ETS](https://github.com/opengeospatial/ets-ogcapi-features10/tree/a314c1e6a9278b14ab9a2ed865cfe36d202f0125)
- [Features ETS landing tests](https://github.com/opengeospatial/ets-ogcapi-features10/blob/a314c1e6a9278b14ab9a2ed865cfe36d202f0125/src/main/java/org/opengis/cite/ogcapifeatures10/conformance/core/landingpage/LandingPage.java)
- [Features ETS API-definition tests](https://github.com/opengeospatial/ets-ogcapi-features10/blob/a314c1e6a9278b14ab9a2ed865cfe36d202f0125/src/main/java/org/opengis/cite/ogcapifeatures10/conformance/core/apidefinition/ApiDefinition.java)
- [Features ETS conformance tests](https://github.com/opengeospatial/ets-ogcapi-features10/blob/a314c1e6a9278b14ab9a2ed865cfe36d202f0125/src/main/java/org/opengis/cite/ogcapifeatures10/conformance/core/conformance/Conformance.java)
- [CSAPI Explorer](https://github.com/OS4CSAPI/ogc-csapi-explorer/tree/00f1c188e05738ee03390fd95f09d351e073a9c3)
- [connected-systems-go](https://github.com/OS4CSAPI/connected-systems-go/tree/e900da88738cca92872038b703c4ad537fc0c8fd)
- [OpenSensorHub CSAPI implementation](https://github.com/OS4CSAPI/osh-core/tree/b2badae59aaa78455c5638ad73b452ccdee40207)
- [pygeoapi](https://github.com/geopython/pygeoapi/tree/bdf3b9ff70b15b4bd72e19df624d038d72c2f466)
- [SECD interoperability evidence](https://github.com/Sam-Bolling/csapi-server-interop-secd/tree/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0)
- [OS4CSAPI testing research corpus](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing)

### 20.4 Project and Governance Sources

- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)
- [IDR-SRV-009 Research Plan](../IDR%20Plans/idr-srv-009-landing-page-api-definition-and-conformance-declaration-behavior.md)
- [Research Planning Approach](../../../../../Governance/research-planning-approach.md)
- [Research Report Template](../../../../../Governance/research-report-template.md)
- [Accepted IDR-SRV-006 Report](idr-srv-006-csapi-part-1-requirement-baseline-report.md)
- [Accepted IDR-SRV-007 Report](idr-srv-007-csapi-part-2-requirement-baseline-report.md)
- [Accepted IDR-SRV-008 Report](idr-srv-008-conformance-class-and-requirement-mapping-report.md)

---

## 21. Appendices

### Appendix A - Direct Requirement and Test Families

| Lane | Exact requirements / tests |
|---|---|
| Common HTTP | `/req/core/http` / A.1 |
| Common Landing | `/req/landing-page/root-op`, `root-success`, `api-definition-op`, `api-definition-success`, `conformance-op`, `conformance-success` / A.12–A.17 |
| Common JSON | `/req/json/definition`, `/req/json/content` / A.18–A.19 |
| Features Core entry points | `/req/core/root-op`, `root-success`, `api-definition-op`, `api-definition-success`, `conformance-op`, `conformance-success`, `http` / A.1 and A.3–A.8 |
| CSAPI Part 1 runtime media support / ATS advertisement | Requirements 77, 78, 89, 90 / A.84, A.85, A.96, A.97 |
| CSAPI Part 2 runtime media support / ATS advertisement | Requirements 93, 94, 107, 108, 115, 116, 123, 124 / A.93, A.94, A.107, A.108, A.115, A.116, A.123, A.124 |

The CSAPI requirements in the last two lanes govern runtime media behavior. Eleven of their twelve tests use API-definition advertisement as evidence; Part 2 A.93 directly retrieves a JSON resource.

### Appendix B - Every Research Question Mapped

| ID | Plan question (short form) | Status | Evidence |
|---|---|---|---|
| CQ1 | Required landing behavior | Complete | §5 |
| CQ2 | Required API-definition behavior | Complete | §6 |
| CQ3 | Required conformance behavior | Complete | §7 |
| CQ4 | Honest CSAPI/SensorML/SWE/Glaux capability exposure | Complete | §§7.3–7.6, 14 |
| CQ5 | Test/validation implications | Complete | §13 |
| LP1 | Required root behavior by source | Complete | §§5.1–5.2 |
| LP2 | Required/should links | Complete | §§5.1–5.2, 12 |
| LP3 | API/service/collection/license/terms/alternate metadata | Complete | §§5.3–5.4, 8 |
| LP4 | Links to every CSAPI resource family | Complete: not required; handoff defined | §5.4, §17 |
| LP5 | Required/optional/conventional/project behaviors | Complete | §5, §12 |
| AD1 | Required/expected definition endpoint | Complete | §§6.1, 6.4 |
| AD2 | OpenAPI and machine-readable exposure | Complete | §§6.2–6.4 |
| AD3 | Definition formats/representations | Complete | §§6.2, 8 |
| AD4 | Live API/definition relationship | Complete | §§6.3–6.4, 9.3 |
| AD5 | Developers/clients/AI/conformance support | Complete | §§6.5, 11, 13 |
| AD6 | Handoff to IDR-SRV-014 | Complete | §§6.4, 17 |
| CD1 | Required conformance endpoint behavior | Complete | §7.1 |
| CD2 | Relevant identifiers | Complete | §§7.4, 7.6 |
| CD3 | Implemented/planned/deferred/experimental distinction | Complete | §§7.3, 7.5 |
| CD4 | Evidence before declaration | Complete | §§7.2–7.4, 13 |
| CD5 | Alignment to IDR-SRV-008 | Complete | §§2.4, 7.3–7.4 |
| CD6 | Avoiding staged overclaim | Complete | §§7.3, 9.5, 14 |
| RP1 | Required/expected representations | Complete | §8.1 |
| RP2 | Required/expected media types | Complete | §8.1 |
| RP3 | Alternate representation links | Complete | §8.3 |
| RP4 | Handoff to IDR-SRV-012 | Complete | §§8.2–8.3, 17 |
| RP5 | Handoff to IDR-SRV-023 | Complete | §§6.4, 13, 17 |
| EF1 | Relevant entry-point failures | Complete | §9 |
| EF2 | Unavailable/stale/incomplete/inconsistent definition | Complete | §§9.2–9.3 |
| EF3 | Unsupported representation | Complete | §§9.4, 8.2 |
| EF4 | Handoff to IDR-SRV-013 | Complete | §§9, 17 |
| EF5 | Negative tests | Complete | §13.3 |
| II1 | Existing implementation behavior | Complete | §11 |
| II2 | Explorer/client/tool needs | Complete | §§11.1, 13.4 |
| II3 | Implementation-study lessons | Complete within bounded supporting review; deep studies deferred | §§11, 17 |
| II4 | External-client test behaviors | Complete | §§13.4–13.5 |

### Appendix C - Confirmed Source Defects and Required Adapters

| Defect / ambiguity | Handling |
|---|---|
| Common vs Features conformance relation mismatch | Emit and test both exact lexical relations |
| Tagged landing `collections` relation | Negative fixture; require `data` |
| Tagged conformance invalid/stale class list | Negative fixture; use accepted registry and evidence generation |
| Features root-links recommendation has `/req/` identifier | Preserve recommendation/SHOULD classification |
| Common landing-class subject omits `-1` in one Annex URI | Use actual class identifier and normative requirements |
| Common JSON A.18 loops beyond normative JSON scope | Adapter limits mandatory JSON to landing and conformance |
| Part 2 A.115 checks binary advertisement in Text test | Adapter checks `application/swe+text` |
| Part 2 PURLs return 404 | Keep literal identifiers; separately monitor resolver |
| Released OAS bundles retain relative references | Do not publish unchanged; validate Glaux-owned definition |

### Appendix D - Report Completion Checklist

- [x] Topic ID matches the overall plan
- [x] Topic and overall plans are linked
- [x] All 5 core and 31 detailed questions are covered
- [x] All six methodology phases are complete
- [x] All ten success criteria are met
- [x] All fifteen required content areas are present
- [x] All ten minimum behavior-matrix fields are present
- [x] Standards obligations, findings, analysis, and recommendations are distinguished
- [x] Mutable sources are pinned by version, tag, commit, state, hash, and/or access date
- [x] OAS definitions and bounded official open/closed issue/PR history were reviewed
- [x] Evidence limitations and unresolved issues are explicit
- [x] Accepted prior reports are reconciled
- [x] Three independent read-only audits are reconciled
- [x] Executive summary and recommendations are independently decision-usable
- [x] No other research topic was begun
- [ ] Plan-owner acceptance recorded; pending Glaux Project Lead review

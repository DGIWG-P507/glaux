# Section 013: Error Model, HTTP Status Codes, and Failure Semantics - Research Report

**Topic ID:** IDR-SRV-013<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-013 Error Model, HTTP Status Codes, and Failure Semantics](../IDR%20Plans/idr-srv-013-error-model-http-status-codes-and-failure-semantics.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core and 47 detailed questions; all six methodology phases, nine success criteria, eighteen required content areas, and thirteen minimum error-matrix fields are validated<br>
**Methodology Used:** Reconciliation of accepted IDR-SRV-006 through IDR-SRV-012 with approved CSAPI, OGC API, HTTP, Problem Details, authentication, caching, and OpenAPI sources; mechanical review of immutable tagged OpenAPI, schema, and abstract-test artifacts; bounded refresh of official issues, pull requests, and the draft Features Part 4 dependency; and three independent read-only evidence audits<br>
**Research Time:** Approximately 6 hours of AI-assisted elapsed execution, including three parallel evidence audits, on August 2, 2026<br>
**Primary Sources:** OGC 23-001, OGC 23-002, OGC 17-069r4, OGC 19-072, RFC 9110, RFC 9111, RFC 6585, RFC 6750, RFC 8470, RFC 9457, IANA HTTP registries, and the OpenAPI Specification<br>
**Supporting Resources:** Accepted IDR-SRV-006 through IDR-SRV-012 reports; official CSAPI tag `v1.0.0`; tagged modular OAS, schemas, and ATS; a pinned current Features Part 4 draft; the shared upstream-history register; and current official issue/PR state<br>
**Document Purpose:** Establish a plain-language, implementation-usable failure contract for the Rust Glaux reference server without turning an example artifact, draft convention, or security-sensitive detail into a standards obligation<br>
**Author:** OpenAI Codex, with independent read-only Part 1/Features, Part 2/dynamic-data, and HTTP/security/interoperability audits<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** August 31, 2026<br>
**Date:** August 2, 2026<br>
**Last Updated:** August 31, 2026

---

## Reading Guide

This report uses six evidence labels:

- **N — normative CSAPI:** an explicit obligation in approved CSAPI Part 1 or Part 2, including its normative ATS;
- **I — inherited:** an applicable normative OGC API or HTTP rule;
- **A — artifact finding:** an observation about official OpenAPI, schema, example, or test material that does not override approved prose;
- **H — history:** issue, pull-request, commit, or draft evidence that explains a design or unresolved dependency but does not amend the approved standard;
- **P — project recommendation:** a bounded Glaux choice that becomes project policy only after plan-owner acceptance; and
- **U — unresolved:** a gap whose exact disposition belongs to a later topic or upstream decision.

An HTTP error describes the outcome of one HTTP request. A command or feasibility **domain status** describes the state of a durable task-like resource. Confusing those two levels is one of the most important failure modes this report prevents.

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Error Behavior Extraction Methodology
5. Standards-Based Error Behavior Inventory
6. HTTP Status-Code Mapping Matrix
7. Error Family Behavior Analysis
8. Machine-Readable Error Response Guidance
9. Security-Sensitive Error Behavior Findings
10. DDIL, Degraded-State, and Dependency-Failure Findings
11. OpenAPI and Documentation Implications
12. Interoperability and Existing-Implementation Implications
13. Test-Strategy Implications
14. Downstream Topic Handoff Matrix
15. Recommendations
16. Risks, Constraints, and Open Questions
17. Validation Against the Research Plan
18. References
Appendix A. Detailed Question Ledger
Appendix B. Official Artifact and History Audit
Appendix C. Proposed Decision Register
Appendix D. Completion and Handoff

---

## 1. Executive Summary

Glaux should expose one predictable error contract across the whole server. The approved CSAPI standards specify only a narrow set of negative outcomes directly: inherited query and lookup rules supply important `400` and `404` behavior; Part 1's normative cascade test fixes `409` for `cascade=false` even though its requirement prose says any present `cascade` parameter is accepted; and Part 2 fixes `400` for Observation/Command values that violate a parent stream schema and `409` for protected stream-schema changes or non-cascading deletion. The rest must be selected from HTTP semantics without claiming that CSAPI said more than it did.

The recommended baseline is:

1. **Use RFC 9457 Problem Details for every application-generated, non-empty 4xx or 5xx response.** Serialize it as `application/problem+json`, use stable absolute problem-type URIs, and add only a small set of documented extensions. OGC API Common recommends the older RFC 7807; RFC 9457 supersedes it and expressly demonstrates returning Problem Details even when the request's `Accept` did not list that media type.
2. **Keep status meanings narrow.** Use `400` for malformed syntax, invalid or undeclared query values, and the CSAPI-mandated parent-schema failures. Use `422` only for a well-formed, structurally valid request whose instructions are semantically unprocessable and for which CSAPI does not require `400` or `409`. Use `409` for a conflict with current resource state, not as a catch-all validation error.
3. **Separate authentication, authorization, and concealment.** Missing or invalid credentials produce `401` with `WWW-Authenticate`; an authenticated caller lacking permission normally receives `403`; Bearer `insufficient_scope` also carries a challenge; and a protected target may deliberately appear as `404` when disclosure would leak its existence. The same policy must govern links, parent lookups, count-bearing responses, and command endpoints.
4. **Make ordinary HTTP failures correct and complete.** A known target with an unsupported method returns `405` and `Allow`; failed response negotiation returns `406`; unsupported request media or content coding returns `415`; supplied conditional requests that fail return `412`; temporary overload returns `503`, normally with `Retry-After` when a useful delay is known.
5. **Do not turn valid absence into failure.** A valid collection query with no visible matches returns `200` and an empty collection. A stale but usable last-known representation is also a successful representation with explicit timestamps/status metadata, not automatically a 5xx response.
6. **Do not turn task lifecycle into transport errors.** Before a Command or Feasibility resource is durably accepted, malformed input, missing authority, missing parents, and service inability use HTTP failures. Once accepted, rejection, cancellation, execution failure, and progress are represented through CommandStatus/Feasibility status resources and successful HTTP retrievals. A feasibility analysis that completes with a negative answer has status `COMPLETED` and carries the answer in a CommandResult; infeasibility is not itself an HTTP error or failed analysis. A later actuator failure is not retroactively a `500` for the create request.
7. **Use dependency codes according to the server's role.** `502` and `504` apply only when Glaux is acting as a gateway or proxy. An origin service temporarily unable to fulfill an operation uses `503`; an unexpected local defect uses `500`. Do not borrow WebDAV's `424 Failed Dependency` or `207 Multi-Status` as generic CSAPI conventions.
8. **Make errors safe by construction.** Return correction-oriented details, public parameter names, and safe request pointers. Do not return stack traces, SQL, filesystem or internal schema paths, secrets, tokens, classification rules, hidden identifiers/counts, upstream topology, or rejected values by default. Expose an opaque trace/occurrence identifier that operators can correlate with protected logs.
9. **Drive runtime, OpenAPI, documentation, and tests from one operation-level error catalog.** A generic global list overstates many routes and omits required distinctions. Every operation should declare the statuses it can actually generate, the shared Problem schema, required headers, security scheme, and representative examples.
10. **Do not copy the tagged OAS error material.** The two root documents reach 87 operations; 81 mechanically advertise the same `400`, `401`, `403`, `404`, and `5XX` set. A raw scan finds an 88th method block in unreferenced `subsystemById.yaml`, whose root path is commented out. No root-reachable operation declares `405`, `406`, `412`, `415`, `422`, `428`, `429`, or a precise `503`. Six Part 1 GETs—the inline landing page and conformance declaration plus four Common collection/item routes—advertise only `200`. The only error-body component is unused, labels its content `application/json`, and references two nonexistent files.

These conclusions are sufficient for later planning, but four decisions remain deliberately downstream: the final hosted problem-type URI namespace and OAS dialect (IDR-SRV-014), the complete schema-validation and `400`/`422` field catalog (IDR-SRV-023), conditional/idempotency/transaction details (IDR-SRV-029), and the security, DDIL, and command-lifecycle policies owned by their focused topics.

---

## 2. Scope and Plan Alignment

### 2.1 Included

This report covers:

- standards-required, inherited, recommended, and project-selected failure behavior for Part 1 and Part 2 HTTP resources;
- lookup, navigation, query, filtering, sorting, pagination, selection, negotiation, encoding, and validation failures;
- dynamic data, command, feasibility, status, and event boundaries;
- authentication, authorization, concealment, policy, cache, retry, rate-limit, and information-leakage concerns;
- local, upstream, degraded, stale, last-known, and DDIL-informed outcomes;
- a shared RFC 9457 response structure and deterministic error precedence;
- operation-level OpenAPI, documentation, fixture, conformance, security, and interoperability implications; and
- exact handoffs to later IDR topics.

### 2.2 Excluded

This report does not finalize:

- Rust error types, framework middleware, crate choices, stack layout, logging implementation, or exception conversion;
- the final OpenAPI dialect, generator, or documentation site structure, owned by IDR-SRV-014;
- the complete field-by-field schema validator and validation code catalog, owned by IDR-SRV-023;
- transaction atomicity, conditional-update policy, idempotency keys, or concurrency algorithms, owned by IDR-SRV-029;
- detailed status, DDIL, synchronization, command, feasibility, safety, authorization, or cross-boundary models owned by IDR-SRV-020, 036–043;
- streaming protocol failure frames or broker-specific semantics, owned by IDR-SRV-035; or
- research for IDR-SRV-014 or any later topic.

### 2.3 Prerequisite State

The goal, governance sources, overall plan, report template, and accepted IDR-SRV-006 through IDR-SRV-012 reports were available. The approved standards, RFCs, tagged artifacts, IANA registries, and official repositories were reachable. No dependency exception was used. The user's current instruction accepted IDR-SRV-012 before this topic was executed.

### 2.4 Core-Question Answers

| Core question | Answer | Detail |
|---|---|---|
| CQ1. Required or expected semantics | A small explicit CSAPI floor plus inherited OGC API/HTTP rules; Glaux adds one deterministic policy where those sources permit choices. | §§5–7 |
| CQ2. Status selection | Use the condition/status map in §6; never infer a status merely because the tagged OAS lists it. | §6 |
| CQ3. Machine-readable structure | RFC 9457 `application/problem+json`, stable types, safe base members, bounded extensions, and standard response headers. | §8 |
| CQ4. Family differences | Validation, lookup, security, task lifecycle, dependency, and degraded-state outcomes are separated by whether the request failed, the target is hidden, or a durable domain resource already exists. | §§7, 9–10 |
| CQ5. Downstream implications | Generate runtime/OAS/test contracts from the same catalog and pass bounded open decisions to the named focused topics. | §§11–15 |

---

## 3. Evidence Base and Authority Classification

### 3.1 Controlling and Inherited Sources

| Source | Version or immutable pin | Authority and use | Limitation |
|---|---|---|---|
| [CSAPI Part 1](https://docs.ogc.org/is/23-001/23-001.html) | OGC 23-001, Version 1.0, approved June 2 and published July 16, 2025 | Controlling Part 1 obligations; Requirement 61 and ATS A.68 cascade behavior | Most general failures are inherited; Requirement 61 prose and ATS A.68 conflict over `cascade=false` |
| [CSAPI Part 2](https://docs.ogc.org/is/23-002/23-002.html) | OGC 23-002, Version 1.0 | Controlling Part 2 dynamic-resource obligations; Requirements 64, 65, 67, 69, 70, 72, 82, and 86 | Several lifecycle and PATCH outcomes are under-specified |
| [OGC API Features Part 1](https://docs.ogc.org/is/17-069r4/17-069r4.html) | OGC 17-069r4, Version 1.0.1 corrigendum; source [`4ff19a57`](https://github.com/opengeospatial/ogcapi-features/commit/4ff19a5734578cf1f815d03ab192e8e0dc407e9f) | Inherited HTTP, query `400`, collection/item `404`, and status guidance | Contains a `limit` requirement/prose tension |
| [OGC API Common Part 1](https://docs.ogc.org/is/19-072/19-072.html) | OGC 19-072, Version 1.0.0 | Query rules, Problem Details recommendation, and conditional OAS completeness | Problem recommendation cites superseded RFC 7807; OAS class is conditional |
| [HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html) and [Caching](https://www.rfc-editor.org/rfc/rfc9111.html) | RFC 9110/9111, June 2022 | Controlling meanings, headers, conditional behavior, gateway distinction, and caching | Often permits multiple server policies; it does not make every status mandatory |
| [Problem Details](https://www.rfc-editor.org/rfc/rfc9457.html) | RFC 9457, July 2023; obsoletes RFC 7807 | Current Problem Details members, media types, extensions, security, and type definitions | Adoption remains Glaux policy because CSAPI/Common only recommend it |
| [Additional HTTP Status Codes](https://www.rfc-editor.org/rfc/rfc6585.html) | RFC 6585 | `428`, `429`, `431`, cache and retry behavior | Codes are optional; later topics decide when policies activate them |
| [Bearer Token Usage](https://www.rfc-editor.org/rfc/rfc6750.html) | RFC 6750 | Bearer `WWW-Authenticate`, `invalid_token`, and `insufficient_scope` semantics | Applies only if the selected deployment uses OAuth bearer tokens |
| [HTTP Early Data](https://www.rfc-editor.org/rfc/rfc8470.html) | RFC 8470 | Narrow `425 Too Early` replay-risk semantics | Not a generic stale-command or concurrency code |
| [OpenAPI Specification](https://spec.openapis.org/oas/latest.html) | Current published version 3.2.0, checked August 2, 2026 | Response objects, exact/range/default response descriptions | IDR-SRV-014 chooses Glaux's actual dialect |
| [IANA status registry](https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml) and [field registry](https://www.iana.org/assignments/http-fields/http-fields.xhtml) | Live registries checked August 2, 2026 | Registered names and current protocol state | Time-sensitive; retrieval date is material |

SensorML 3.0 and SWE Common 3.0 are controlling for valid domain encodings. They define what valid payloads mean, but they do not replace the HTTP error selection above.

### 3.2 Official Artifact and History Evidence

The official `v1.0.0` tag resolves to immutable commit [`8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2). The mutable default branch was refreshed at [`3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f). The only Part 1/2 API-tree delta is one example-link correction; none of the error-response findings here has been repaired on `master`.

The [shared upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md) was refreshed after a bounded review of all 141 official issues and 58 pull requests. Directly material history includes cascade [#61](https://github.com/opengeospatial/ogcapi-connected-systems/issues/61), draft Features Part 4 [#141](https://github.com/opengeospatial/ogcapi-connected-systems/issues/141), missing PATCH/error contract [#170](https://github.com/opengeospatial/ogcapi-connected-systems/issues/170), Deployment deletion [#171](https://github.com/opengeospatial/ogcapi-connected-systems/issues/171), schema distinctions [#181](https://github.com/opengeospatial/ogcapi-connected-systems/issues/181), and bulk-artifact ambiguity [#185](https://github.com/opengeospatial/ogcapi-connected-systems/issues/185). Threads explain context; only adopted standard text controls conformance.

CSAPI normatively cites a draft OGC API Features Part 4 dependency. The current official draft was pinned at [`9ca25f56a58ed822ea8a685a7a41afa7181aaa8b`](https://github.com/opengeospatial/ogcapi-features/commit/9ca25f56a58ed822ea8a685a7a41afa7181aaa8b), July 13, 2026. Its status table, conditional-update rules, Problem examples, and open issues [#1022](https://github.com/opengeospatial/ogcapi-features/issues/1022) and [#1023](https://github.com/opengeospatial/ogcapi-features/issues/1023) are useful change-controlled context, not an approved replacement for the version CSAPI referenced.

### 3.3 Supporting Security and Operational Evidence

- [OWASP API Security Top 10 (2023)](https://owasp.org/API-Security/editions/2023/en/0x11-t10/) was used to cross-check object/property authorization, resource consumption, unsafe upstream consumption, inventory, and information-disclosure concerns. It is guidance, not a CSAPI obligation.
- The active [RateLimit header draft `-11`](https://datatracker.ietf.org/doc/draft-ietf-httpapi-ratelimit-headers/), dated May 23, 2026 and expiring November 24, 2026, was checked. Because it remains an Internet-Draft, Glaux should not baseline its proposed fields as stable standards. Registered `429` plus `Retry-After` is sufficient for this topic.
- Existing-implementation studies IDR-SRV-014A through 014G have not run. Prior reports provide limited pinned client/server observations, but no implementation is treated as authority.

### 3.4 Evidence Limitations

- No private SWG minutes, unpublished corrigendum, private maintainer communication, or proprietary implementation was used.
- No live server interoperability run or executable conformance suite was performed in this topic.
- The exact security scheme, classification policy, DDIL metadata model, command lifecycle, and Rust architecture remain intentionally downstream.
- An edge proxy, TLS stack, or HTTP parser can reject a request before application code and may not emit Glaux Problem Details. The application contract can cover only errors Glaux generates or controls.
- The tagged OAS and ATS are incomplete and sometimes contradictory. Mechanical findings identify risks; they cannot create normative requirements.
- Mutable GitHub, IANA, OAS, and Internet-Draft state is a point-in-time snapshot from August 2, 2026.

---

## 4. Error Behavior Extraction Methodology

### 4.1 Six-Phase Execution

| Plan phase | Work performed | Output |
|---|---|---|
| 1. Sources/taxonomy | Reconciled governance and accepted reports; pinned standards, RFCs, artifacts, registries, and official history; defined N/I/A/H/P/U labels. | §§3–4 source and taxonomy framework |
| 2. Standards extraction | Extracted direct CSAPI `400`/`409`, inherited query/lookup/HTTP rules, Problem Details, headers, and conditional OAS requirements. | §5 standards inventory |
| 3. Family mapping | Grouped lookup, query, negotiation, validation, dynamic/tasking, security, dependency, rate/resource, and internal failures. | §§6–7 matrices and analysis |
| 4. Error body/docs | Assessed RFC 9457, safe extensions, type/instance identity, response headers, OpenAPI components, and fixtures. | §§8, 11 |
| 5. Security/DDIL/interop/tests | Applied disclosure, cache, dependency-role, last-known-state, official-history, and test considerations. | §§9–13 |
| 6. Synthesis | Resolved what can be decided now, bounded what cannot, built handoffs, recommendations, ledgers, and validation. | §§14–17 and appendices |

### 4.2 Taxonomy Fields

Every matrix entry records: error family, example condition, HTTP outcome, exact source/anchor, authority classification, body expectation, headers/links, security sensitivity, retry/recovery guidance, OpenAPI implication, test implication, downstream handoff, and notes/unresolved points. The matrix is split into two keyed tables in §6 so all thirteen required fields remain readable.

### 4.3 Selection Rule

The selection order is:

1. apply an explicit approved CSAPI status when its condition matches;
2. otherwise apply an inherited approved OGC API requirement;
3. otherwise use the narrow RFC meaning that fits the actual failure;
4. where HTTP permits choices, apply one documented Glaux policy consistently;
5. never let an example OAS, issue comment, implementation precedent, or convenient framework default override steps 1–4.

### 4.4 Reproducibility

The artifact audit used the immutable tag, walked each root document's path references, enumerated every reachable operation and declared response key, separately identified unreferenced path files, checked response component references and file existence, compared the tag to current `master`, and refreshed the bounded upstream register. Appendix B records the mechanical results and pins.

---

## 5. Standards-Based Error Behavior Inventory

### 5.1 CSAPI Part 1

Part 1 requires conformance with applicable OGC API Common and Features behavior. Its local negative-status surface is small and includes one material prose/ATS contradiction:

| Condition | Behavior | Authority |
|---|---|---|
| System DELETE with nested resources or a Deployment association | Requirement 61 says omission of `cascade` is rejected but any request containing the parameter is accepted; normative ATS A.68 instead requires `cascade=false` to return `409` and `cascade=true` to delete | Tagged [Requirement 61](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/standard/sections/clause_16_requirements_class_create_replace_delete.adoc#L34-L46) and [ATS A.68](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/standard/sections/annex-abstract-test-suite.adoc#L1385-L1407), N/U; Glaux P-013-04 interprets false like omission and true as cascade |
| Unknown or invalid query, malformed advanced filter | `400` through inherited Common/Features rules | [Features Requirements 8–9](https://docs.ogc.org/is/17-069r4/17-069r4.html#_unknown_or_invalid_query_parameters), I |
| Standards-required `recursive` on `GET /deployments` | Must be recognized as a valid boolean parameter; tagged `deployments.yaml` omits it, so an OAS-derived strict implementation could wrongly return `400` | Tagged [recursive Deployment requirement](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/standard/requirements/subdeployment/req_recursive_search_deployments.adoc#L1-L12), tagged [OAS omission](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/paths/deployments.yaml), and open [#169](https://github.com/opengeospatial/ogcapi-connected-systems/issues/169), N/A/H |
| Missing OAF collection or item | `404` through inherited Features clauses | [Features §§7.14–7.16](https://docs.ogc.org/is/17-069r4/17-069r4.html), I |
| Canonical `/systems/{id}`-style target absent | `404` is correct HTTP/Glaux behavior; Part 1 does not independently restate the Features item rule for each canonical route | [RFC 9110 §15.5.5](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.5.5), I/P |
| `text/uri-list` custom-collection membership request malformed or unsupported | Requirement 71 fixes the valid representation, but not complete negative or atomicity semantics | [Part 1 Requirement 71](https://docs.ogc.org/is/23-001/23-001.html#_req_create-replace-delete_add-to-collection), N/U |
| PATCH/update failure | Update class requires PATCH but does not settle patch document, arrays/null, immutable fields, atomicity, conditional policy, or complete error mapping | [Part 1 Requirements 72–76](https://docs.ogc.org/is/23-001/23-001.html#_req_update), N/U |

A valid filter with no matches is a successful empty collection, normally `200`. Features Requirement 22C says a `limit` above the maximum must be clamped without error, although later explanatory prose says values outside the range produce `400`; the numbered requirement controls.

### 5.2 CSAPI Part 2

Part 2 supplies the clearest direct status obligations:

| Requirement | Condition | Required status |
|---|---|---|
| [64](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_datastream-update-schema) | REPLACE changes a DataStream observation schema after nested Observations exist | `409` |
| [65](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_datastream-delete-cascade) | DELETE a non-empty DataStream without cascade | `409` |
| [67](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_observation-schema) | Observation CREATE/REPLACE violates parent DataStream schema | `400` |
| [69](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_controlstream-update-schema) | REPLACE changes a ControlStream command schema after nested Commands exist | `409` |
| [70](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_controlstream-delete-cascade) | DELETE a non-empty ControlStream without cascade | `409` |
| [72](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_command-schema) | Command CREATE/REPLACE violates parent ControlStream schema | `400` |
| [82](https://docs.ogc.org/is/23-002/23-002.html#_req_update_observation-schema) | Observation UPDATE violates parent DataStream schema | `400` |
| [86](https://docs.ogc.org/is/23-002/23-002.html#_req_update_command-schema) | Command UPDATE violates parent ControlStream schema | `400` |

Requirements 80 and 84 require rejecting PATCH changes to populated stream schemas but omit a status. The normative ATS expects `409`; Glaux should use `409` for parity while recording the prose/ATS gap. The standard explicitly says command cancellation is not HTTP DELETE: a client posts a new CommandStatus with `CANCELED` and the Command remains. Synchronous processing returns a status report whose domain status can be `COMPLETED`, `REJECTED`, or `FAILED`; asynchronous progress and final outcomes likewise belong in status resources.

### 5.3 Inherited Features and Common Behavior

- Features Core requires HTTP conformance and makes `200`, `400`, and `404` support mandatory where called out. Its Table 3 strongly encourages `304`, `401`, `403`, `405`, `406`, and `500` and explains their intended use.
- Features Requirements 8 and 9 require `400` for undeclared and invalid query parameters. Common provides controlled tolerance permissions; a parameter genuinely declared through an allowed range is not unknown.
- Common recommends a Problem Details report in error responses and `application/problem+json` for JSON, but cites RFC 7807. RFC 9457 is its current successor.
- If Glaux claims Common's OAS 3.0 class, Requirements 25 and 26 require every operation to document all success responses and all error statuses originating from the server. Transport errors outside server control need not be enumerated.

### 5.4 HTTP and Problem Details

HTTP defines status meanings and mandatory companion fields when a code is generated: `401` needs `WWW-Authenticate`; `405` needs `Allow`; `415` can advertise accepted media types with `Accept` or accepted request codings with `Accept-Encoding`; `503` and `429` can carry `Retry-After`. `404`, `405`, `410`, `414`, and `501` are heuristically cacheable unless controlled; protected/dynamic Glaux failures therefore need explicit conservative cache policy.

RFC 9457 makes the `type` URI the machine identity, requires the body `status` to match the actual response, keeps `title` stable by type, reserves `detail` for human correction rather than parsing/debugging, and permits documented extension members. Its validation example uses an `errors` array and JSON Pointer, and its security section warns against exposing attack-relevant internals.

### 5.5 Draft Features Part 4 Context

The pinned draft lists `400`, `401`, `403`, `404`, `405`, `406`, `412`, `413`, `415`, `422`, `428`, and `500` as relevant; requires `404` for a nonexistent PUT target when PUT-create is not supported; requires `412` when `If-Match` names a nonexistent target; and defines optional optimistic-locking behavior. It also uses Problem Details examples and `Allow`. Because CSAPI references a draft and open issues still concern the PATCH representation, this evidence informs later transaction planning but cannot silently expand approved CSAPI obligations.

### 5.6 Official Artifact Inventory

The tagged OAS is an example using OAS 3.1.0 and `info.version: 0.0.1`. Its two root documents reach 87 operations—39 in Part 1 and 48 in Part 2. One additional DELETE method exists in `paths/subsystemById.yaml`, but the corresponding root path/reference is commented out, so raw all-file counts are one higher. For the 87 root-reachable operations:

- 81 declare each of `400`, `401`, `403`, `404`, and `5XX` regardless of operation-specific meaning;
- 44 declare `200`, 15 declare `201`, 29 declare `204`, and 16 declare `409`;
- six Part 1 GETs—the inline landing page and conformance declaration plus four Common collection/item routes—declare only `200`;
- no operation declares `405`, `406`, `408`, `410`, `412`, `413`, `414`, `415`, `422`, `425`, `428`, `429`, `431`, or precise `500`/`502`/`503`/`504` responses;
- `401` has no `WWW-Authenticate`, and the 4xx components have no bodies or headers;
- `409_PUT` appears only on two separate schema-operation paths, while main DataStream/ControlStream PUT operations omit the direct Requirements 64/69 conflict;
- `GET /deployments` omits the normative `recursive` parameter even though strict unknown-query handling could turn that omission into an incorrect `400`;
- tagged [`systemOrArrayOrRefs.yaml`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/requests/systemOrArrayOrRefs.yaml) advertises JSON URI arrays and GeoJSON/SensorML System arrays in addition to Requirement 71's `text/uri-list`, while unused [`batch_delete.json`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/schemas/common/batch_delete.json) and [`batch_response.json`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/schemas/common/batch_response.json) suggest per-item integer statuses/free-text errors without normative atomicity or partial-failure semantics;
- Command POST declares `200` but points to the Part 1 `resourceLinks` response—an array of created-resource URIs—rather than the synchronous Part 2 status report;
- Part 2 retains four System History operations although the corresponding standard clause is commented out of the approved document;
- `5XX` is a valid OpenAPI range key, but its prose “Only retry on 502 and 503” is not a safe complete retry contract; and
- unused `ServerError.yaml` uses `application/json` and references missing `schemas/exception.yaml` and `examples/serverError.json`.

The artifact is valuable defect evidence. It is neither the standards obligation baseline nor a server implementation blueprint.

---
## 6. HTTP Status-Code Mapping Matrix

### 6.1 Condition, Authority, and Body

| ID | Error family | Example condition | HTTP outcome | Source/anchor | Class | Body expectation |
|---|---|---|---|---|---|---|
| M01 | Request/query syntax | Malformed JSON, invalid declared query value, undeclared query name, invalid selector/token syntax | `400` | [Features Req. 8–9](https://docs.ogc.org/is/17-069r4/17-069r4.html#_unknown_or_invalid_query_parameters); [RFC 9110 §15.5.1](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.5.1) | I/P | `invalid-request` or narrower Problem type; safe correction details |
| M02 | CSAPI parent-schema validation | Observation or Command create/replace/update violates the authorized parent stream schema | `400` | Part 2 Reqs. [67](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_observation-schema), [72](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_command-schema), [82](https://docs.ogc.org/is/23-002/23-002.html#_req_update_observation-schema), [86](https://docs.ogc.org/is/23-002/23-002.html#_req_update_command-schema) | N | `stream-schema-violation`; safe pointers/codes only |
| M03 | Semantic validation | Media and syntax valid, public schema shape valid, but an instruction violates a processable semantic constraint not assigned `400`/`409` | `422` | [RFC 9110 §15.5.21](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.5.21) | P | `unprocessable-content`; documented `errors` entries |
| M04 | Authentication | Missing credentials, or an expired, revoked, malformed, or otherwise invalid token value | `401` | [RFC 9110 §15.5.2](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.5.2); [RFC 6750 §3.1](https://www.rfc-editor.org/rfc/rfc6750.html#section-3.1) when Bearer applies | I | Minimal `authentication-required`/`invalid-token`; no secret diagnostics |
| M05 | Authorization/policy | Authenticated principal may not perform operation and disclosure is allowed | `403` | [RFC 9110 §15.5.4](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.5.4) | I/P | Generic `forbidden`; safe remedy only |
| M06 | Lookup/concealment | Route, item, nested parent, or current representation absent; or protected existence deliberately hidden | `404` | Features §§7.14–7.16; [RFC 9110 §§15.5.4–15.5.5](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.5.4) | I/P | Same safe `not-found` shape for genuine and concealed absence |
| M07 | Pagination | Expired server-owned continuation target; forged token; hidden cursor context | `404`; malformed/forged may be `400` or concealed `404` per accepted IDR-SRV-011 | [Accepted IDR-SRV-011 §§8.3 and 12](idr-srv-011-query-filtering-sorting-pagination-and-selection-semantics-report.md#83-continuation-contract) | P | `continuation-expired` only when disclosure is safe; otherwise generic |
| M08 | Method | Method is known but not supported by this target | `405` | [RFC 9110 §15.5.6](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.5.6) | I | `method-not-allowed` |
| M09 | Response negotiation | No representation/coding satisfies explicit client preferences and Glaux will not ignore them | `406` | [RFC 9110 §15.5.7](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.5.7); accepted IDR-SRV-012 | P | `not-acceptable`; safe supported alternatives |
| M10 | Request transport timeout | Server did not receive a complete request within its wait period | `408` | [RFC 9110 §15.5.9](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.5.9) | I | Body when safely generated; edge may close without one |
| M11 | Explicit CSAPI state conflict | Blocked System/DataStream/ControlStream cascade or protected populated-stream schema change | `409` | Tagged Part 1 [ATS A.68](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/standard/sections/annex-abstract-test-suite.adoc#L1385-L1407); Part 2 Reqs. [64–65](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_datastream-update-schema), [69–70](https://docs.ogc.org/is/23-002/23-002.html#_req_create-replace-delete_controlstream-update-schema), and tagged [ATS for 80/84](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/annex-abstract-test-suite.adoc#L1520-L1592) | N/P, with Part 1 prose/ATS conflict and Part 2 PATCH status gap | Specific safe conflict type and corrective action |
| M12 | Other state conflict | Duplicate identity, immutable state, lifecycle transition, or current-version conflict not expressed as a failed supplied precondition | `409` | [RFC 9110 §15.5.10](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.5.10) | P/U | Stable conflict type; safe current-state hint |
| M13 | Permanent retirement | Server knows access is permanently unavailable and may disclose that fact | `410` | [RFC 9110 §15.5.11](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.5.11); accepted IDR-SRV-010A | I/P | `gone`; replacement link only if safe |
| M14 | Supplied precondition | `If-Match` or another evaluated conditional header is false | `412` | [RFC 9110 §15.5.13](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.5.13) | I | `precondition-failed`; safe current validator may accompany |
| M15 | Request resource limits | Body/decompressed body too large; URI too long; headers too large | `413`; `414`; `431` | RFC 9110 §§15.5.14–15; [RFC 6585 §5](https://www.rfc-editor.org/rfc/rfc6585.html#section-5) | I/P | Generic limit type; identify a header only when safe |
| M16 | Request representation | Missing required Content-Type on typed non-empty body, unsupported media type, or unsupported content coding | `415` | [RFC 9110 §15.5.16](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.5.16); accepted IDR-SRV-012 | I/P | `unsupported-media-type` or `unsupported-content-coding` |
| M17 | TLS early data | Unsafe/replay-sensitive request arrives in early data | `425` | [RFC 8470 §5.2](https://www.rfc-editor.org/rfc/rfc8470.html#section-5.2) | I | `too-early`; no generic stale/schedule use |
| M18 | Required concurrency guard | Policy requires a conditional request but none was supplied | `428` | [RFC 6585 §3](https://www.rfc-editor.org/rfc/rfc6585.html#section-3) | P/U | `precondition-required`; explain required header |
| M19 | Caller rate/resource policy | Principal or request key exceeded a defined rate limit | `429` | [RFC 6585 §4](https://www.rfc-editor.org/rfc/rfc6585.html#section-4) | P | `rate-limit-exceeded`; avoid sensitive quota internals |
| M20 | Local server defect | Unexpected exception, invariant failure, or response-validation failure before commit | `500` | [RFC 9110 §15.6.1](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.6.1) | I/P | Sanitized `internal-error`; opaque occurrence/trace only |
| M21 | Function unsupported generally | Method/functionality is not recognized or supported for any resource | `501` | [RFC 9110 §15.6.2](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.6.2) | I | Generic `not-implemented`; rare in documented Glaux routes |
| M22 | Gateway failure | Glaux as gateway receives invalid upstream response or no timely response | `502` or `504` | RFC 9110 [§15.6.3](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.6.3) and [§15.6.5](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.6.5) | I | Sanitized `bad-gateway`/`gateway-timeout`; hide topology |
| M23 | Temporary service/dependency outage | Origin temporarily cannot fulfill request due overload, maintenance, unavailable local dependency, or open circuit | `503` | [RFC 9110 §15.6.4](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.6.4) | I/P | `service-unavailable`; safe recovery estimate |
| M24 | Valid empty result | Valid query has no visible matches | `200` with empty collection, not an error | [Accepted IDR-SRV-011 §12](idr-srv-011-query-filtering-sorting-pagination-and-selection-semantics-report.md#12-error-failure-and-validation-implications), grounded in Features collection semantics | I/P | Normal collection representation |
| M25 | Stale/partial state | Stored last-known representation remains usable; or explicit partial-success shape exists | `200` with freshness/completeness metadata; otherwise `503` | [RFC 9111 §4.2.4](https://www.rfc-editor.org/rfc/rfc9111.html#section-4.2.4); final status/DDIL model deferred to IDR-SRV-020/042 | P/U | Domain representation, never a Problem body inside `200` |
| M26 | Command/feasibility lifecycle | Valid request is durably accepted, then rejected, canceled, fails, progresses, or completes feasibility with a positive/negative answer | Successful HTTP response plus CommandStatus/Feasibility domain state and, for completed feasibility, a CommandResult | Tagged Part 2 [Command status model](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/clause_9_requirements_class_controlstreams.adoc#L365-L414) and [Feasibility status/result model](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/clause_10_requirements_class_command_feasibility.adoc#L50-L90) | N/P | Normal status resource with safe domain message; completed feasibility answer in its result resource |
| M27 | Moved resource | Authorized safe redirect to temporary/permanent replacement | `307`/`308`; otherwise `404`/`410` | RFC 9110 [§15.4.8](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.4.8) and [§15.4.9](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.4.9); accepted IDR-SRV-010A | I/P | Short redirect note or Problem only when not redirecting |
| M28 | Cache revalidation | Conditional GET/HEAD finds selected representation unchanged | `304`, not an error | [RFC 9110 §15.4.5](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.4.5) | I | No body; required validator/cache metadata only |
| M29 | Protocol-edge cases | Length required, unsatisfied Range, misdirected request, protocol upgrade, or genuine legal block | `411`, `416`, `421`, `426`, or `451` | RFC 9110 [§15.5.12](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.5.12), [§15.5.17](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.5.17), [§15.5.20](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.5.20), [§15.5.22](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.5.22); [RFC 7725 §§3–4](https://www.rfc-editor.org/rfc/rfc7725.html#section-3) | I | Problem when origin can generate one; protocol-specific headers preserved |
| M30 | Custom membership/batch ambiguity | `text/uri-list` contains malformed, inaccessible, external, incompatible, duplicate, or mixed-validity references | `400` for malformed/invalid request; possible `404`/`409` only after later atomicity/disclosure decisions | [Part 1 Requirement 71](https://docs.ogc.org/is/23-001/23-001.html#_req_create-replace-delete_add-to-collection) and open [#185](https://github.com/opengeospatial/ogcapi-connected-systems/issues/185) | P/U | One dominant safe Problem; no invented 207 contract |
| M31 | Duplicate create/idempotent replay | Same logical request or identifier already exists | Existing artifact suggests `303`; exact replay/result/conflict policy deferred | Tagged [`303_POST` component](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/responses/StandardResponses.yaml#L12-L18); final policy deferred to IDR-SRV-029 | A/U | If used, `Location`; otherwise original success or conflict per later contract |
| M32 | Failure after response commit | Serializer, stream, or connection fails after headers/body start | No new HTTP status can be sent; abort stream/connection and record protected telemetry | [RFC 9110 §6.4](https://www.rfc-editor.org/rfc/rfc9110.html#section-6.4) and [RFC 9112 §6](https://www.rfc-editor.org/rfc/rfc9112.html#section-6) message framing; detailed handling deferred | P/U | No second Problem body; later robustness/streaming policy |
| M33 | Error-body negotiation | Original failure occurs while request `Accept` omits Problem Details | Preserve original status and use `application/problem+json` when a body is generated | [RFC 9457 example, §3](https://www.rfc-editor.org/rfc/rfc9457.html#section-3) | P | Normal Problem body; do not replace original failure with recursive `406` |

### 6.2 Headers, Security, Recovery, Documentation, Tests, and Handoffs

| ID | Required/useful headers or links | Security sensitivity | Retry/recovery | OpenAPI implication | Minimum test | Downstream handoff | Notes/unresolved |
|---|---|---|---|---|---|---|---|
| M01 | None generally | High for query names, values, pointers | Correct request | Explicit `400` per applicable operation | Unknown, duplicate, malformed, boundary, prohibited combination | 014, 023, 050–053 | Do not echo secret or oversized values |
| M02 | Problem type docs | Parent identity/schema can be sensitive | Correct against authorized schema | `400` on Observation/Command writes | Each format, create/replace/update, auth-before-schema | 023, 034, 036 | CSAPI `400` overrides a generic 422 preference |
| M03 | Problem type docs | Field/domain details can leak | Correct semantics | Reusable `422` response only on applicable writes | Structural-valid/semantic-invalid boundary | 014, 023, 029 | Keep catalog finite and documented |
| M04 | `WWW-Authenticate` mandatory; Bearer parameters when applicable | Critical | New/replaced credentials | Security scheme plus `401` header/schema | No credentials, invalid token, expired token | 039, 055 | No error code when a Bearer request supplied no auth, per RFC 6750 |
| M05 | For Bearer `insufficient_scope`, `WWW-Authenticate` is mandatory; `scope` is optional and only safe when disclosure permits | Critical | Different authority/policy, not same credentials | `403` plus conditional challenge/header schema | Object/property/function authorization and insufficient scope | 038–040, 055 | `403` must not leak rule/classification details; HTTP permits non-credential refusal too |
| M06 | None; replacement link only when safe | Critical for concealment | Correct identifier/authority | `404` on item/nested routes | Genuine vs concealed parity; missing parent/item | 014, 039–040, 055 | Default `Cache-Control: no-store` |
| M07 | Safe pagination links only | High | Restart query from stable collection URL | Document token lifecycle outcomes | Malformed, forged, expired, cross-principal token | 014, 050, 056 | Exact concealed outcome can be security-profile specific |
| M08 | `Allow` mandatory | Method list can leak protected capability | Use allowed method | Header required in response object | Every route/method pair; auth before protected Allow | 014, 050, 055 | OPTIONS advertisement must agree |
| M09 | Safe alternates or `Link`; relevant `Vary` | Medium | Change `Accept`/`Accept-Encoding`/profile | `406` per negotiated operation | q-values, exclusions, wildcard, aliases, unavailable profile | 014, 050, 056 | Supported lists should not expose hidden variants |
| M10 | Connection close may apply | Low/operational | Retry only with replay safety | Usually edge/default documentation | Incomplete request integration test | 044, 048, 052 | Not application deadline or upstream timeout |
| M11 | Safe conflict/replacement link | Relationship counts/state may leak | Reconcile or use cascade where authorized | Exact `409` on normative/accepted operations | Requirement/ATS conflicts and every rollback boundary | 014, 029–031, 050–051 | Part 1 prose accepts any present `cascade` while ATS treats false as noncascade; Part 2 PATCH 80/84 prose omits code while ATS expects 409 |
| M12 | Current validator/link only when safe | State/version sensitive | Reconcile state | Operation-specific `409` | Duplicate/immutable/lifecycle conflict | 029, 036–038 | Do not substitute for supplied-condition `412` |
| M13 | `Link`/`Location` only if safe | Reveals past existence | Use replacement or stop | Explicit `410` only on retired public contract | Known permanent vs uncertain vs concealed | 014, 016, 039–040 | Heuristically cacheable; normally `no-store` unless public policy |
| M14 | Current `ETag` may help | Validator can reveal state | Refetch/reconcile/retry | `412` on conditional writes | Strong/weak/stale/missing validators | 014, 029, 050 | Later topic decides required preconditions |
| M15 | `Retry-After` should be sent for a temporary 413; no unsafe size disclosure | High resource-exhaustion concern | Reduce request or wait if temporary | Separate responses/limits | Compressed bomb, exact boundary, URI/header limits | 039, 044, 048, 052, 055 | 431 must not be cached; 414 is heuristically cacheable |
| M16 | Response `Accept` for media failure; `Accept-Encoding` only for coding failure | Medium | Resend supported representation | `415` with correct header variants | Missing/wrong type, unsupported coding, valid type malformed body | 014, 023, 050, 056 | Never confuse with response-side 406 |
| M17 | None required | Replay/safety critical | User agent should retry automatically after the handshake, never again in early data | Document only if edge supports early data | Replay-sensitive command/write | 038–039, 055 | Generate only for a request received in early data or carrying `Early-Data: 1`; not cacheable by default; never generic “too soon” |
| M18 | Explain required conditional header | Concurrency policy | Fetch validator then retry | `428` plus schema; no-store | Missing vs failed condition | 029, 050 | Policy activation deferred |
| M19 | `Retry-After` when useful; draft RateLimit fields not baseline | Quota/identity sensitive | Wait with jitter; replay safely | `429` and header | Per-principal isolation, burst, exhaustion | 039, 044, 048, 052, 055 | Must not be cached; service-wide overload is 503 |
| M20 | Opaque trace/instance only | Critical internal-detail risk | Usually operator action; cautious safe retry | Reusable `500` | Forced invariant/serializer failure, sanitization | 014, 023, 044, 052–055 | Response-validation failure before commit is 500 |
| M21 | Capability docs/link if public | May reveal disabled capability | Configuration/version change | Rare explicit `501`; cache controls | Unknown method globally vs route-specific 405 | 014, 046, 050 | Avoid for optional class not claimed or unadvertised filter |
| M22 | No upstream topology | Critical dependency disclosure | Backoff and safe replay | Separate `502`/`504` only for gateway routes | Invalid vs timeout upstream | 041–043, 046, 052 | Local database timeout is not automatically 504 |
| M23 | `Retry-After` when estimate exists | Operational/topology sensitive | Backoff, budget, safe replay | Explicit `503` on applicable operations/readiness | Maintenance, overload, dependency circuit | 041–043, 046, 052 | Delay does not make unsafe replay safe |
| M24 | Normal links/counts subject to authorization | Counts can leak | No recovery needed | Normal `200` schema | Zero-match and zero-visible-match cases | 014, 050, 056 | Never 404 solely because result set is empty |
| M25 | `Age` for HTTP cache; explicit as-of/freshness/completeness in domain model | Staleness may reveal operations | Client decides based on age; otherwise retry 503 | Normal success schema plus later availability metadata | Usable stale vs unusable; partial contract | 020, 041–043, 046, 053 | RFC 9111 obsoletes `Warning`; 206/207 are wrong generic choices |
| M26 | `Location`/status links on acceptance where applicable | Status/reason can be classified | Poll/subscribe per later model | Success/status schemas, not error reuse | Synchronous/asynchronous rejection/failure/cancel | 034–038, 055–056 | Intake inability before durability remains HTTP failure |
| M27 | `Location` required/useful; authorization preserved | Redirect can leak target or credentials | Follow only under trusted policy | Explicit safe redirect variants | Same/cross-origin, read/write, auth, loop | 014, 016, 039–040, 056 | Writes do not auto-redirect by default |
| M28 | Required validators/cache metadata; no body | Low/representation sensitive | Use cached representation | Document `304` on conditional reads | Per-representation ETag and Vary | 014, 028, 050 | Not a Problem response |
| M29 | `Content-Range` for 416, `Upgrade` for 426, `Link rel=blocked-by` for 451 | Varies; 451/legal is sensitive | 411: add valid length; 421: new connection; others condition-specific | Edge response components where supported | Header/status pair contracts and retry behavior | 014, 039–040, 044, 050 | A proxy must not generate 421; `blocked-by` identifies the implementing entity; 451 is cacheable by default |
| M30 | None until atomicity contract; perhaps original collection link | High for referenced-resource existence | Correct whole request | Do not document batch semantics not implemented | External, incompatible, duplicate, mixed-validity, rollback | 023, 029–031, 050, 053 | No standards-defined per-item response model |
| M31 | `Location` if redirect/result reference used | Identity/existence sensitive | Use original result or reconcile | Later operation-specific response | Same key/same body vs key/different body | 029, 050 | Tagged 303 is artifact evidence only |
| M32 | Connection/stream signaling only | Internal failure sensitive | Client detects truncation and safely retries | Streaming callbacks/extensions later | Mid-serialization and mid-stream fault injection | 035, 044, 048, 052–053 | HTTP status is already committed |
| M33 | Original status-specific headers still required | Same as original error | Recover from original condition | Problem media on every application error response | Accept excludes Problem; HEAD; lower-layer rejection | 014, 050, 056 | Error representation does not replace original semantics |

### 6.3 Codes and Patterns to Avoid

Glaux should explicitly reject these common misuses:

- `200` with an embedded error envelope, or `202` when work was not accepted;
- `204` with a body, `206` for partial domain data, or `207` for an invented batch model;
- WebDAV `423`, `424`, `507`, or `508` without adopting their protocols;
- `400` for every failure, `401` without a challenge, or `405` without `Allow`;
- `409` for authentication, content type, failed supplied preconditions, or all validation;
- `418`, private `499`/`598`/`599`, or nonstandard 6xx/7xx wire contracts;
- `425` for scheduling or stale state;
- `429` for service-wide overload or `503` for caller-specific quota;
- `501` for a feature flag, an optional class Glaux does not claim, a temporary outage, or a particular target's method rejection;
- `502`/`504` when Glaux is not acting as a gateway/proxy; and
- origin-generated `407` or captive-portal `511`.

### 6.4 Protocol-Edge Details

These codes are not routine CSAPI application outcomes, but an implementation or deployment that generates them must preserve their exact protocol meaning:

| Status | Exact boundary | Recovery, header, and cache rule |
|---|---|---|
| `411` | The server refuses a request without a defined `Content-Length`. | The client may retry after adding a valid `Content-Length`; do not require it where HTTP permits framing without it. |
| `421` | The request was directed to a server unable or unwilling to produce an authoritative response for the target scheme/authority. | A proxy must not generate it. The client may retry over a different connection, even for a non-idempotent method. |
| `425` | The server is unwilling to risk processing a request received in TLS early data or forwarded with `Early-Data: 1`. | Do not generate it outside that boundary. The user agent should retry automatically after the handshake and must not send the retry in early data. It is not cacheable by default. |
| Temporary `413` | The content-size condition is temporary rather than a fixed request limit. | RFC 9110 says the server should send `Retry-After`; the client still needs replay-safe semantics. |
| `451` | Access is unavailable because of a legal demand. | The response should carry `Link: <...>; rel="blocked-by"`; that target identifies the entity implementing the block, not the authority demanding it. The status is cacheable by default, so Glaux must set deliberate cache policy. |

---

## 7. Error Family Behavior Analysis

### 7.1 Application Error Precedence

Glaux needs deterministic externally observable precedence, even though an HTTP stack or gateway can reject framing before the Rust application runs. For requests that reach application control, use this order:

1. **Trusted edge/protocol checks:** framing, header/URI/body limits, request completion, TLS early-data policy.
2. **Route and coarse capability:** distinguish an unknown route from a known operation without exposing protected capability details.
3. **Authentication and coarse authorization/concealment:** authenticate, identify the authorized resource scope, and decide whether forbidden targets are disclosed before emitting parent-specific facts.
4. **Method and response negotiation:** enforce method and `Accept`/profile/coding selection; on protected routes, do not leak `Allow` or alternatives before authorization.
5. **Request representation:** decode supported content coding, verify Content-Type, parse syntax, then validate public schema.
6. **Authorized domain validation:** resolve the authorized parent, apply parent stream schema and business rules, then evaluate preconditions/conflicts in the order HTTP and the later transaction policy require.
7. **Execute or durably accept:** after commitment, represent task progress/failure as domain state and preserve atomicity.

Some authentication or message-signature schemes require untouched request bytes. That processing occurs before decoding without changing the disclosure policy. When several failures exist, return the first safe problem selected by this pipeline; do not aggregate unrelated problems into a misleading batch type.

### 7.2 Resources, Links, and Navigation

- Unknown routes, absent items, invalid path identifiers, and missing nested parents return `404`.
- A valid public resource known to be permanently retired may return `410`; uncertain removal remains `404`.
- A moved resource may use a method-preserving `307`/`308` only under the accepted versioning/security policy. Automatic redirects for writes are disabled by default.
- A broken optional related link does not automatically make the containing resource erroneous. Return a contract-valid parent and represent relationship availability only if the model defines it. If completeness is promised and cannot be produced, fail the request rather than silently omit required content.
- An inaccessible linked target follows its own authentication/concealment policy. A parent representation must not reveal whether a target is merely absent or restricted when that distinction is protected.
- Valid stale links remain links; dereferencing them yields the actual current `404`, `410`, redirect, or authorization outcome. Glaux does not rewrite history silently.

### 7.3 Query, Filtering, Sorting, Pagination, and Selection

- Truly undeclared names, malformed values, invalid temporal ranges/bboxes/WKT, invalid sort/projection expressions, incompatible supported-filter combinations, malformed/forged continuation tokens, and invalid `f`/schema selectors use `400`.
- A semantically unprocessable instruction can use `422` only after it has passed the declared syntax/schema and no inherited/CSAPI `400` rule controls. Ordinary filter failures should remain `400` for interoperability.
- An optional filter or sort that Glaux does not advertise is an unknown parameter (`400`), not `501`.
- A requested limit above the advertised maximum is clamped where inherited Features Requirement 22C applies; it is not an error.
- A valid query with no visible matches returns an empty `200` collection. Count, paging, error detail, and timing must not expose hidden matches.
- Correctable details may identify a public parameter/field and constraint code but should not echo its full rejected value or reveal hidden schema/capability branches.

### 7.4 Content Negotiation and Encoding

- `406` is response-side: the explicit `Accept`, supported profile, or accepted response coding cannot be satisfied.
- `415` is request-side: the media type or content coding is missing where required or unsupported. A 415 caused by media type can advertise safe accepted request media with `Accept`; one caused by coding should use `Accept-Encoding` and must not use that header for unrelated causes.
- A supported media type containing malformed JSON/GeoJSON/SensorML/SWE syntax is `400`. A valid syntactic/structural payload with a non-CSAPI semantic violation may be `422`.
- Parent-specific format/schema details are resolved only after authority to address that parent is established.
- The response to an error can be `application/problem+json` even if the request did not list it. Preserve the original `400`/`406`/`415`/other status rather than recursively producing a new `406`.
- Suggestions, alternates, and profile/schema links are emitted only when they do not expose unavailable or unauthorized capabilities.

### 7.5 Validation and Schemas

Validation occurs in distinct layers: transport/content-coding, media type, parser syntax, public representation schema, authorized parent logical schema, domain invariant, precondition, and persistence/response invariant. Recording the failed layer is more useful than one generic “validation failed” bucket.

Request failures produce 4xx as mapped above. If Glaux's own response cannot satisfy the promised schema before committing the response, the client did nothing wrong: return sanitized `500` and log the full internal defect. After a response or stream is committed, Glaux cannot replace it with a new status; it must terminate safely, record telemetry, and let clients detect truncation.

Public validation detail may include a stable rule code, `query`/`path`/`header`/`body` location, a public parameter name, and a JSON Pointer into the submitted document. It must not expose local filesystem paths, internal schema graph URIs, Rust types, SQL, secret values, authorization predicates, or parent schema data that the caller cannot read. IDR-SRV-023 owns the final validation catalog and format-specific mapping.

### 7.6 Dynamic Data, Commands, and Feasibility

Observation, DataStream, event-history, and status-query transport failures use the same lookup/query/negotiation/security rules as other resources. The explicit Part 2 `400`/`409` rules then apply at the domain layer.

For commands and feasibility, use this boundary:

| Moment | Examples | Representation |
|---|---|---|
| Before durable acceptance | Malformed parameters, invalid media, unauthorized principal, hidden/missing ControlStream, schema violation, temporary inability to record work | Appropriate HTTP 4xx/5xx; no Command is claimed to exist |
| At synchronous accepted processing | Command completes, is rejected by the receiving system, or fails during execution | Successful HTTP response containing the Part 2 status report |
| After asynchronous acceptance | Command: pending, accepted, scheduled, executing, updated, rejected, canceled, failed, completed. Feasibility does not use `SCHEDULED` or `UPDATED`; a completed yes/no answer is in CommandResult. | CommandStatus/Feasibility status and result resources retrieved with normal HTTP success |
| Later retrieval/update | Missing/hidden status, invalid update, failed precondition, current lifecycle conflict | HTTP lookup/validation/security/conflict response for that new request |

An authorized caller blocked by access policy receives `403`/concealed `404`. An authorized command rejected by a safety or feasibility rule is normally durable domain `REJECTED`, with a safe status message and protected audit evidence. A downstream actuator failure after acceptance is domain `FAILED`, not a delayed reinterpretation of the original HTTP response. Duplicate commands, idempotency, stale updates, and exact lifecycle conflicts remain with IDR-SRV-029 and 036–038.

Asynchronous execution alone does not imply HTTP `202`. If the Command resource is created immediately, the inherited creation result is normally `201` with `Location`, while later execution remains domain state. `202` is appropriate only if the HTTP creation/mutation itself was accepted for later processing. Part 2 does not settle the exact synchronous response status/body shape; IDR-SRV-037 must do so without erasing the required status report.

### 7.7 Partial and Multiple Failures

RFC 9457 recommends representing the most relevant or urgent problem when multiple unrelated problems exist. An `errors` array is suitable for multiple occurrences of the **same validation problem type**, not for mixing authorization, outage, and validation failures.

Glaux should not invent a `207 Multi-Status` batch contract. It should expose only atomic collection/batch mutations unless a later explicit ordered per-item result contract is designed. Until IDR-SRV-029/031 settles custom membership and bulk behavior, reject an invalid request as a whole and do not expose partial side effects.

---

## 8. Machine-Readable Error Response Guidance

### 8.1 Adopt RFC 9457 as the Wire Baseline

For every application-generated 4xx/5xx response that can carry a body, Glaux should emit `Content-Type: application/problem+json`. This is a project profile of RFC 9457, not a claim that CSAPI mandates it.

Glaux should require these members locally:

| Member | Local rule |
|---|---|
| `type` | Stable absolute URI controlled by the project; the primary machine identifier; normally resolves to human-readable documentation |
| `title` | Short stable summary for that type; may be localized but is not a programmatic key |
| `status` | Actual HTTP status as a JSON number; must match the response status |
| `detail` | Optional, occurrence-specific, correction-oriented, and safe; clients must not parse it |
| `instance` | Optional opaque absolute URI/URN identifying this occurrence; not automatically dereferenceable |

RFC 9457 makes standard members optional, but requiring the first three in Glaux gives generated clients and offline logs a stable minimum. `about:blank` is reserved for genuinely generic status-only failures; it must not conceal a useful stable type merely to avoid maintaining documentation.

### 8.2 Bounded Extensions

Use only these initial extensions:

| Extension | Purpose | Rule |
|---|---|---|
| `code` | Stable short project code within the problem type | Machine-readable; documented; not a Rust enum name or localized text |
| `traceId` | Opaque correlation key for authorized support/telemetry | Never a token, hostname, stack key, user identifier, or promise of public log access |
| `errors` | Multiple occurrences of one validation problem type | Array of safe entries; not a mixture of unrelated problem types |

Each `errors` entry may contain `code`, `detail`, `location` (`query`, `path`, `header`, or `body`), a public `name`, and a JSON Pointer `pointer`. Omit rejected values by default. Do not expose an internal schema path merely because it is available from a validator.

Illustrative response—not a final hosted URI decision:

```http
HTTP/1.1 400 Bad Request
Content-Type: application/problem+json
Cache-Control: no-store

{
  "type": "https://dgiwg-p507.github.io/glaux/problems/stream-schema-violation",
  "title": "The request does not satisfy the stream schema",
  "status": 400,
  "detail": "Two request fields need correction.",
  "instance": "urn:uuid:41386c5f-8b6f-4622-87fe-614f306ee43d",
  "code": "stream_schema_violation",
  "traceId": "5d6a8e4c173a4d17",
  "errors": [
    {
      "code": "required",
      "location": "body",
      "pointer": "/resultTime",
      "detail": "A result time is required."
    }
  ]
}
```

IDR-SRV-014 owns the final stable URI origin and OAS schema. IDR-SRV-023 owns the final validation entry catalog. Those topics may refine naming, but should not remove the base interoperability rules accepted here.

### 8.3 Type Catalog Design

Problem types should identify actionable semantics, not every route/status combination. Useful initial families include `invalid-request`, `invalid-query-parameter`, `stream-schema-violation`, `unprocessable-content`, `authentication-required`, `invalid-token`, `forbidden`, `not-found`, `method-not-allowed`, `not-acceptable`, `unsupported-media-type`, `conflict`, `precondition-failed`, `precondition-required`, `content-too-large`, `rate-limit-exceeded`, `service-unavailable`, and `internal-error`.

Do not mint one type per Rust error, database constraint, resource identifier, or localized sentence. New types require a stable title, expected status, client handling, extensions, security review, examples, and compatibility policy. Clients branch on `type` or documented `code`, never `detail`.

### 8.4 Headers and Links Remain Authoritative

Problem Details supplements HTTP; it does not replace:

| Status/context | Header/link rule |
|---|---|
| `401` | `WWW-Authenticate` is mandatory |
| Bearer `403 insufficient_scope` | `WWW-Authenticate` is mandatory; `scope` is optional and disclosure-sensitive |
| `405` | `Allow` is mandatory and authorization-sensitive |
| `415` caused by request coding | `Accept-Encoding` should advertise acceptable request codings |
| `415` caused by media type | response `Accept` may advertise acceptable request media types |
| `416` byte range | `Content-Range: bytes */length` should be sent |
| `426` | `Upgrade` is mandatory |
| Temporary `413` | `Retry-After` should be sent |
| `429`/`503` | `Retry-After` may be sent when a meaningful estimate exists |
| Redirect/replacement/documentation | `Location` or typed `Link` only when safe and semantically applicable |

The problem `type` URI is the canonical documentation identity. An additional help link should use a registered/documented relation rather than an ad hoc field.

### 8.5 Body, Negotiation, HEAD, and Lower Layers

- Preserve the original failure status and generate Problem Details even when `Accept` omitted `application/problem+json`, as RFC 9457's validation example permits. Do not recursively turn every error into `406`.
- A `HEAD` response carries the same status and applicable headers as GET but no body.
- A 204 or 304 never carries a Problem body.
- An HTTP parser, proxy, TLS endpoint, or overload shedder may be unable to generate the application body. Glaux documentation must distinguish application-controlled responses from infrastructure failures.
- If multiple unrelated problems exist, report the most relevant safe one. Use `errors` only for multiple instances of the same validation type.

### 8.6 Caching

Default all Glaux application Problem responses to `Cache-Control: no-store`. A public endpoint may deliberately allow negative caching only after a reviewed operation-specific policy supplies explicit freshness and proves the response is not caller-specific or concealment-sensitive. Shared caches must not reuse a response to a request containing `Authorization` unless the response carries an explicit directive that RFC 9111 permits for shared caching. This matters because HTTP makes `404`, `405`, `410`, `414`, and `501` heuristically cacheable; RFC 6585 independently forbids caching `428`, `429`, and `431`.

---

## 9. Security-Sensitive Error Behavior Findings

### 9.1 Authentication and Authorization

For any chosen authentication scheme, `401` means credentials are missing or invalid and must include a challenge. HTTP `403` more broadly means the server understood the request but refuses it; the reason need not concern credentials. Glaux should normally use it for authenticated authorization/policy denial when concealment is not required, but that narrower usage is project policy. If OAuth bearer tokens are used:

- no credentials: `401` plus a Bearer challenge, with no detailed error code or diagnosis;
- malformed token transmission or multiple bearer delivery methods: `invalid_request`, normally `400`;
- expired, revoked, malformed, or otherwise invalid token value: `invalid_token`, normally `401`;
- valid token with insufficient scope: `insufficient_scope`, normally `403`, with a mandatory Bearer challenge and optional required-scope information only when safe.

RFC 9700 supplies current OAuth security best practice, while RFC 6750 continues to define these HTTP mappings. The challenge remains authoritative even when a Problem body is present.

### 9.2 Concealment and Enumeration Resistance

RFC 9110 permits a forbidden target to appear as `404`. The later security/policy topics must identify which resources and operations require that concealment. The policy must apply consistently to:

- direct item and nested-parent lookup;
- link visibility and link dereferencing;
- method/capability discovery and `Allow`;
- query counts, page totals, timing, and continuation tokens;
- parent stream schema/capability detail;
- Command, status, result, feasibility, event, and audit resources; and
- cache behavior and Problem body size/type/detail.

A concealed response should not say “forbidden,” identify a policy rule, expose a different problem type, or be cached differently from an ordinary protected `404`.

### 9.3 Public Versus Protected Detail

Public responses may contain stable types/codes, safe parameter names/pointers, an opaque occurrence key, and a correction that does not disclose protected state. Protected telemetry may contain stack traces, dependency diagnostics, transaction state, and authorization context after redaction and access control.

Never expose:

- bearer tokens, authorization headers, session material, signatures, or secret request values;
- stack traces, Rust type/module names, SQL, filesystem paths, internal schema paths, or configuration;
- internal hostnames, queues, database/broker topology, or raw upstream responses;
- policy expressions, classification/releasability labels the caller cannot see, or exact denial predicates;
- hidden identifiers, parent existence, filtered counts, or alternative representations unavailable to the caller; or
- raw request bodies or rejected values by default.

### 9.4 Command and Control Safety

Authorization to call a command endpoint is checked before exposing ControlStream schema or state. A caller lacking authority receives `403` or concealed `404`. A caller who is authorized to submit but whose valid command is rejected by a safety/mission rule normally receives a durable `REJECTED` domain status; the public message remains safe and the detailed reason belongs in the protected audit trail. The exact control ownership, confirmation, unsafe-command, cancellation, and audit rules remain with IDR-SRV-038 through 040.

### 9.5 Resource Exhaustion and Abuse

Apply explicit URI, header, decoded-body, nesting, array, query-complexity, pagination, decompression, concurrency, and timeout limits. Select `413`, `414`, `431`, `429`, or `503` by the actual condition. Under attack, the edge may drop connections rather than spend more resources constructing a Problem body; the application contract cannot guarantee otherwise.

---

## 10. DDIL, Degraded-State, and Dependency-Failure Findings

### 10.1 Decision Table

| Situation | Response model | Reason |
|---|---|---|
| Current stored representation is available and meets the operation contract | `200` normal representation | Upstream reachability is irrelevant to this successful read |
| Stored representation is usable but known stale/last-known | `200` plus explicit domain `as-of`, freshness, source, and completeness metadata once defined | Staleness is state, not automatically transport failure |
| Part 2 DataStream or ControlStream has `live=false` | Return the resource normally; decide request-specific behavior separately | `live` describes channel state and does not itself require 503 |
| Optional related resource is unavailable but parent representation remains complete by contract | `200` parent with only contract-valid links/availability metadata | Avoid turning adjacency failure into parent failure |
| Required complete representation cannot be assembled | `503` when temporary; otherwise condition-specific 4xx/5xx | Do not silently return ambiguous partial data |
| Glaux origin cannot fulfill because a local dependency/database/broker is temporarily unavailable | `503`, optionally `Retry-After` | Glaux is the origin, not necessarily a gateway |
| Glaux HTTP gateway receives invalid upstream HTTP | `502` | Exact gateway semantics |
| Glaux HTTP gateway times out waiting for upstream HTTP | `504` | Exact gateway semantics |
| Unexpected local bug/invariant failure | `500` | Not a dependency classification shortcut |
| Caller-specific quota is exhausted | `429`; `Retry-After` if useful | Distinct from service-wide overload |
| Service-wide overload/maintenance/circuit breaker | `503`; `Retry-After` if useful | Temporary service inability |
| Accepted Command later loses actuator/connectivity | Domain `FAILED` or other later-defined status/event | The HTTP request already succeeded and created durable state |
| Feasibility analysis completes with a negative answer | Domain `COMPLETED` plus negative feasibility result | “Infeasible” is the result, not analysis failure or HTTP failure |

### 10.2 Stale and Cached State

HTTP cache reuse must obey RFC 9111 and expose correct `Age`; “usable stale” means stale reuse is actually allowed by the governing request/response directives. Directives such as `no-cache` or `must-revalidate` can prohibit reuse. Application-level last-known data needs explicit domain timestamps and completeness. RFC 9111 obsoletes the old `Warning` response header, so Glaux should not build a stale-data contract around `Warning: 110`. Optional `stale-if-error` is informative RFC 5861 behavior and belongs to later deployment/cache planning, not the CSAPI baseline.

Never return `200` with a Problem object to represent an outage. If a usable normal representation exists, return it as the declared resource media type with state metadata. If it does not, return the appropriate Problem response.

### 10.3 Retry Guidance

`Retry-After` is an HTTP date or nonnegative delay in seconds. Prefer delay-seconds for short operational events to reduce clock-skew ambiguity. It is a scheduling hint, not a guarantee and not permission to replay an unsafe write.

Clients should use jittered bounded backoff, an overall deadline/budget, and cancellation. Automatic retry remains gated by method/idempotency semantics or an explicit later idempotency contract:

- `401`: acquire/replace credentials;
- `403`: do not repeat automatically with the same credentials;
- `400`, `406`, `413`, `415`, `422`: correct the request;
- `409`, `412`, `428`: reconcile state or validators;
- `425`: retry after the handshake, never again in early data;
- `429`, `502`, `503`, `504`: possibly transient, but replay only when safe;
- `410`, `501`: normally stop until capability/configuration changes.

The active RateLimit header work remains an Internet-Draft. Glaux may study it later but should baseline only `429`, Problem Details, and optional `Retry-After` now.

### 10.4 Status Resources and Events

Availability, source freshness, connectivity, and degraded-mode facts belong in the status/availability model and, where standardized, SystemEvent or operational event resources. Part 2's current predefined SystemEvent types are not a generic HTTP error bus. IDR-SRV-020, 041, and 042 must decide the actual vocabulary and how DDIL transitions are recorded without making every degraded state an HTTP error.

---

## 11. OpenAPI and Documentation Implications

### 11.1 One Error Catalog, Four Projections

Maintain one operation-level error catalog as structured project data. Generate or validate four projections from it:

1. Rust runtime mapping and middleware behavior;
2. OpenAPI response objects and security/header declarations;
3. human documentation/problem-type pages; and
4. negative, conformance, security, and golden-file tests.

Hand-maintained copies will drift. The catalog records condition, operation, status, problem type/code, headers, disclosure/cache profile, retry class, and test IDs.

### 11.2 OpenAPI Shape

Regardless of the dialect chosen by IDR-SRV-014:

- define one extensible RFC 9457 Problem schema and the bounded extension schemas;
- define status-specific reusable responses because `401`, `405`, `415`, `429`, `503`, and other cases need different headers and semantics;
- list known operation-specific statuses explicitly; a `default` or `4XX`/`5XX` range may be a safety net, not a substitute for known behavior;
- model `application/problem+json` under response `content` and never as a response header entry;
- document `WWW-Authenticate`, `Allow`, `Accept`, `Accept-Encoding`, `Retry-After`, `Content-Range`, `Upgrade`, `Location`, and `Link` where applicable;
- document security schemes and operation requirements accurately;
- model Bearer authentication as `type: http` and `scheme: bearer`; treat `bearerFormat` only as a documentation hint; and remember that schemes inside one Security Requirement Object are conjunctive while separate array entries are alternatives;
- include representative safe examples for authentication, query, validation, negotiation, conflict/precondition, rate, dependency, and internal errors; and
- validate runtime/OAS parity in CI.

If Glaux claims OGC API Common's OAS 3.0 conformance class, Common Requirements 25 and 26 make complete server-originated success/error documentation normative. The current latest OpenAPI publication is 3.2.0, while the official CSAPI example uses 3.1.0; the chosen Glaux dialect remains an IDR-SRV-014 decision.

### 11.3 Documentation

Each problem-type page should state identity, title, normal status, applicability, safe client action, extensions, headers, retry/cache behavior, examples, security notes, and compatibility history. Documentation must explicitly distinguish:

- CSAPI-required outcomes from Glaux policy;
- an HTTP failure from Command/Feasibility domain state;
- request-side 415 from response-side 406;
- current-state 409 from supplied-precondition 412 and missing-precondition 428;
- caller quota 429 from service outage 503; and
- origin 503 from gateway 502/504.

### 11.4 Tagged OAS Repair Implications

Do not mechanically fork the tagged response components. Rebuild them from the accepted catalog. At minimum, repair the missing Problem body/schema, required headers, operation-specific status sets, main stream PUT `409` omissions, six Part 1 GET error-response omissions, absent PATCH/feasibility operations, and incorrect generic retry language. The broader OAS strategy and all non-error defects remain with IDR-SRV-014.

---

## 12. Interoperability and Existing-Implementation Implications

### 12.1 What Can Be Concluded Now

- Clients can rely on standard HTTP status and headers before understanding Glaux extensions.
- Stable Problem `type`/`code` values let clients branch without parsing English.
- Unknown Problem extensions must be ignored, allowing additive evolution.
- A valid empty collection, a retrieved failed CommandStatus, and a negative completed feasibility result remain ordinary successful representations.
- Explicit operation-level documentation avoids generated clients treating every route as able to return the same generic 400/401/403/404/5XX set.
- Status-specific headers are as important as the JSON body; clients that only deserialize the body will mishandle authentication, method discovery, retry, and negotiation.

### 12.2 Official History and Draft Context

The official issue history confirms unresolved PATCH, bulk, schema, cross-encoding, and cascade questions, but no adopted universal error object. The current Features Part 4 draft has useful Problem/conditional examples yet remains change-controlled, and open Features issues still debate the PATCH representation. Glaux should monitor and pin it through the register, then run a delta review when it becomes an approved dependency.

The CSAPI tagged OAS is not evidence of widespread client expectation: it is labeled an example, reports `0.0.1`, and contains broken/unused error material. Compatibility should target approved standard semantics and observed clients, not preserve artifact defects as intentional behavior.

### 12.3 Evidence Still Needed

Focused studies IDR-SRV-014A through 014G must test how OpenSensorHub, connected-systems-go, pygeoapi, SECD, CSAPI Explorer, and OS4CSAPI clients actually handle error bodies, status codes, redirects, synchronous command status, and omitted responses. Those later results may justify compatibility aliases or additional examples; they may not weaken approved requirements or safe disclosure.

---

## 13. Test-Strategy Implications

### 13.1 Required Test Families

| Family | Minimum coverage |
|---|---|
| Standards/conformance | Every direct Part 1/2 `400`/`409`; inherited query `400`, lookup `404`, and official ATS; record upstream ATS defects separately |
| Status boundary | 400/415/422, 403/404 concealment, 405/501, 409/412/428, 408/504, 429/503, 500/502/503/504 |
| Problem contract | HTTP/body status equality; stable type/title/code; correct media; unknown-extension tolerance; `about:blank`; safe occurrence IDs |
| Headers | 401 challenge, Bearer 403 `insufficient_scope` challenge, 405 Allow, 415 Accept/Accept-Encoding distinction, 416 Content-Range, 426 Upgrade, Retry-After, redirect/replacement links |
| Query/navigation | Unknown/duplicate/malformed/boundary parameters; zero results; missing item/parent; forged/expired continuation; hidden counts |
| Negotiation/encoding | q-values, exclusions, profiles, f/selectors, unsupported type/coding, malformed supported body, error media when Accept excludes it |
| Validation | Syntax vs structural vs parent schema vs semantic vs conflict; every supported representation; safe pointers; no rejected secrets |
| Command/feasibility | Pre-acceptance HTTP failure; synchronous rejected/failed status; asynchronous lifecycle; cancellation as status; negative completed feasibility |
| Security | No/malformed/expired token, insufficient scope, object/property/function authorization, concealment parity, disclosure/redaction, cache controls |
| DDIL/dependency | Usable current/stale data, unusable completeness, live=false, local dependency 503, gateway 502/504, safe retry/no duplicate side effect |
| Limits/robustness | URI/header/body/decompressed-body/nesting/query limits, mid-serialization/stream failure, overload shedding, HEAD/no-body behavior |
| OAS parity | Every runtime status/type/header is documented; every documented response is reachable or deliberately synthetic; security schemes match |

### 13.2 Golden Files and Schemas

Maintain canonical Problem fixtures per stable type and important status/header variant. Normalize or pattern-match nondeterministic `instance`, `traceId`, time, and Retry-After fields. Validate fixtures against the Glaux schema and round-trip them through the Rust serializer once implementation starts.

Golden files should catch accidental public contract changes, not freeze unsafe detail strings. Tests must assert stable type/code/status/headers and safe field structure while allowing localized or improved human `detail` text where the contract permits it.

### 13.3 Negative and Security Properties

In addition to examples, generate boundary/property tests for parser limits, unknown extensions, duplicate parameters, content-coding expansion, JSON Pointer escaping, arbitrary Unicode, header injection, oversized detail, and error-path panics. Scan every public 4xx/5xx fixture and fault-injected response for stack traces, secrets, authorization headers, SQL, local paths, internal hosts, and hidden identifiers/counts.

### 13.4 Retry and Side-Effect Tests

Fault injection must prove that `Retry-After` does not cause duplicate Command or mutation side effects, that non-idempotent writes are not automatically retried without an explicit contract, and that post-commit failures are detectable rather than converted into a false second status. IDR-SRV-029 and the later resilience topics will supply the exact idempotency and retry machinery.

### 13.5 Official Versus Supplemental Suites

Run the official ATS unchanged to support conformance claims, but do not mistake its omissions or known copy errors for desired behavior. Maintain a supplemental Glaux suite linked to the exact N/I/P authority for each test. Upstream discrepancies belong in the evidence register and should be reported without silently modifying what a normative test asserts.

---

## 14. Downstream Topic Handoff Matrix

This report fixes the common failure vocabulary and the boundaries between HTTP failure and domain state. It does not pre-decide the detailed designs owned by later topics.

| Downstream topic | What this report hands off | What that topic must decide or produce |
|---|---|---|
| IDR-SRV-014 — OpenAPI and documentation | RFC 9457 baseline, operation-specific response principle, required headers, tagged-OAS defects—including the Command POST `200`/`resourceLinks` mismatch—and the proposed type catalog | OpenAPI dialect/version, reusable Problem schemas and responses, hosted problem-type URI namespace, generation/validation workflow, correct synchronous Command status response, and repaired operation coverage |
| IDR-SRV-020 — Status and availability | `live=false`, stale data, dependency state, and HTTP/domain-state separation | Public status and availability properties, freshness/completeness vocabulary, and how operational state is linked from resources |
| IDR-SRV-023 — Validation | Explicit CSAPI `400` obligations; project boundary among `400`, `415`, `422`, and `409`; safe validation entries | Validation layers by representation/schema, complete error-code catalog, JSON Pointer rules, limits, and cross-encoding equivalence |
| IDR-SRV-029 — Transactions and idempotency | `409`/`412`/`428` boundary, unsafe-retry prohibition, cascade ambiguity, and duplicate Command risk | Atomicity, rollback, ETags, preconditions, idempotency keys, deduplication windows, replay results, and post-commit failure handling |
| IDR-SRV-031 — Batch and collection mutation | Rejection of generic `207`/WebDAV semantics and unresolved `text/uri-list`/bulk failure behavior | Whether batch operations exist, atomic versus partial behavior, per-member reporting, and authorization rules |
| IDR-SRV-034 — Observations, status, and events | Successful empty query, stale/last-known behavior, validation, and domain-status guidance | Resource-specific state/event representations and query/result semantics |
| IDR-SRV-035 — Streaming and subscriptions | HTTP Problem Details applies to the HTTP setup plane; Part 2 supplies no streaming failure contract | In-stream error frames, disconnects, acknowledgements, retry/resume, backpressure, and Part 3 alignment |
| IDR-SRV-036 — Command lifecycle | Accepted Command outcomes are domain status; cancellation is a posted `CANCELED` status; invalid transitions are unresolved | Legal transition graph, terminal states, status creation/update authority, timeouts, cancellation, and error mapping for illegal transitions |
| IDR-SRV-037 — Feasibility and asynchronous tasking | Infeasible can be `COMPLETED` plus a negative result; HTTP `202` concerns queued HTTP work, not merely asynchronous actuation | Feasibility result model, synchronous response, polling, readiness, timeout, retention, and task resource behavior |
| IDR-SRV-038 — Command authorization and safety | Pre-acceptance authority failures use HTTP; accepted rejection/failure is domain state; no unsafe-command code is standardized | Safety-policy evaluation, control ownership, authorized cancellation, denial detail, audit, and replay protection |
| IDR-SRV-039 / 039A — API security and zero trust | `401`/`403`/concealed `404`, Bearer challenge rules, no-store default, and disclosure controls | Authentication mechanisms, trust boundaries, subject/context propagation, enforcement points, and detailed threat mitigations |
| IDR-SRV-040 — Policy and releasability | Consistent concealment across status, body, links, counts, timing, and caching | Classification/releasability policy model, cross-boundary decisions, public denial codes, and redaction/aggregation rules |
| IDR-SRV-041 — External dependencies | Role-based `500`/`502`/`503`/`504` distinction and sanitized dependency details | Dependency inventory, health model, circuit breakers, failure isolation, and operator-visible diagnostics |
| IDR-SRV-042 — DDIL semantics | Serve usable stale state as success with freshness/completeness; use `503` when no usable representation exists; retry remains replay-safe | Disconnection modes, offline read/write policy, sync queues, freshness thresholds, backoff budgets, and recovery behavior |
| IDR-SRV-043 — Synchronization and conflict | Current-state conflicts versus failed conditions, safe reconciliation, and opaque conflict detail | Merge/conflict model, tombstones, version vectors or alternatives, stale-write policy, and cross-node reconciliation |
| IDR-SRV-044 / 048 / 052 — Limits, reliability, observability | `413`/`414`/`429`/`431`/`503`, no-store, trace correlation, fault-injection, and retry constraints | Concrete budgets, overload control, graceful degradation, telemetry schema, SLOs, and incident diagnostics |
| IDR-SRV-046 — Deployment | Origin/gateway distinction and the requirement for edge/runtime Problem parity | Proxy topology, forwarded-error trust, timeout ownership, health endpoints, TLS termination, and deployment-specific response controls |
| IDR-SRV-050 / 051 — Conformance and traceability | Normative/inherited/project authority labels, official-ATS discrepancies, and required negative tests | Executable conformance harness and requirement-to-test trace matrix that keeps official and supplemental assertions distinct |
| IDR-SRV-053 — Fixtures | Stable Problem fields, nondeterministic-field normalization, and minimum fixture families | Canonical fixture corpus, generation/review rules, redaction scans, versioning, and golden-file maintenance |
| IDR-SRV-055 — Security and command-control tests | Authentication/authorization/concealment cases, command two-plane boundary, unsafe replay, and disclosure properties | Adversarial cases, policy fixtures, timing/cache checks, control-safety tests, and audit verification |
| IDR-SRV-056 — Client interoperability | Stable machine identity, response media, required headers, status boundaries, and documented deviations | Cross-client matrix, tolerance expectations, real implementation evidence, and regression corpus |

---

## 15. Recommendations

These are proposed Glaux decisions. They are not accepted project policy until the project lead accepts this report.

| ID | Recommendation | Basis and boundary |
|---|---|---|
| P-013-01 | Use RFC 9457 `application/problem+json` for every application-generated, body-capable 4xx/5xx response. | P; modernizes Common's RFC 7807 recommendation without changing HTTP status semantics. |
| P-013-02 | Publish a small, stable, documented problem-type registry; require `type`, `title`, and matching numeric `status`. | P; IDR-SRV-014 owns the URI origin and OpenAPI representation. |
| P-013-03 | Apply one deterministic selection order: explicit CSAPI, inherited OGC API, narrow HTTP meaning, then documented project policy. | N/I/P; examples and framework defaults never outrank approved requirements. |
| P-013-04 | Implement every direct CSAPI `400` and `409`; interpret Part 1 `cascade=false` like omission and `cascade=true` as cascade to follow ATS A.68; use ATS `409` for prohibited populated-stream PATCH; and preserve both prose/ATS conflicts in traceability. | N/P; resolves implementation behavior without pretending the published contradictions do not exist. |
| P-013-05 | Use `400` for malformed/structural requests and explicit CSAPI schema failures; reserve `422` for well-formed semantic failures where no controlling rule requires another status. | I/P; IDR-SRV-023 must finish the per-rule catalog. |
| P-013-06 | Use `401` plus `WWW-Authenticate` for absent/invalid credentials, `403` for ordinary authenticated denial, and policy-selected `404` only for deliberate concealment. | I/P; detailed identity and policy design remains downstream. |
| P-013-07 | Return `200` with an empty collection for a valid zero-match query; do not use `404` for selection emptiness. | I. |
| P-013-08 | Preserve status-specific headers and use `Cache-Control: no-store` by default on dynamic, caller-specific, security-sensitive, and Problem responses. | I/P; deliberate public negative caching requires an explicit exception. |
| P-013-09 | Separate HTTP-request outcome from Command/Feasibility domain state. Retrieval of `REJECTED`, `FAILED`, `CANCELED`, or negative completed feasibility remains a successful HTTP retrieval. | N/P; lifecycle topics own the transition model. |
| P-013-10 | Use `502`/`504` only while Glaux is acting as an HTTP gateway/proxy, `503` for temporary origin/dependency unavailability, and `500` for unexpected local failure. | I/P. |
| P-013-11 | Never infer replay safety from `Retry-After`; automatically retry unsafe writes only under an explicit idempotency/replay contract. | I/P; IDR-SRV-029 owns that contract. |
| P-013-12 | Use `429` for caller-specific quota/rate policy and `503` for service-wide overload; do not adopt draft `RateLimit` fields as a baseline requirement. | I/P; monitor the active IETF draft. |
| P-013-13 | Maintain one reviewed operation-level error catalog that generates or validates runtime mapping, OpenAPI, public documentation, fixtures, and tests. | P; exact tooling belongs to implementation/OpenAPI planning. |
| P-013-14 | Rebuild—not copy—the tagged OAS error responses, adding correct bodies, headers, operation coverage, PATCH/feasibility paths where warranted, and explicit known statuses. | A/P; IDR-SRV-014 owns execution. |
| P-013-15 | Maintain canonical Problem fixtures plus negative, boundary, property, disclosure, retry, and fault-injection tests; run official and supplemental conformance suites separately. | P. |
| P-013-16 | Continue bounded upstream-history refreshes and treat draft/issue/PR evidence as context unless incorporated into an approved standard or project decision. | H/P. |
| P-013-17 | Do not invent generic batch/partial-failure semantics with `207`, `424`, or an error-in-`200` envelope. | I/P; any bulk extension needs its own explicit contract. |
| P-013-18 | Keep public errors correction-oriented and sanitized; correlate opaque occurrence/trace identifiers with protected internal diagnostics. | I/P; security topics own detailed logging and access controls. |

---

## 16. Risks, Constraints, and Open Questions

### 16.1 Constraints

- The approved CSAPI standards define only part of the failure surface. A complete server contract necessarily includes documented project choices.
- The official tag's OAS and ATS contain omissions and defects. They are evidence and conformance inputs, not a safe implementation blueprint.
- Part 1 Requirement 61 and ATS A.68 conflict over `cascade=false`; P-013-04 supplies a local implementation profile but cannot repair the published standard.
- CSAPI's dependency on a draft Features Part 4 is mutable. Draft behavior can guide planning but cannot be represented as frozen approved-CSAPI text.
- Focused implementation and interoperability studies IDR-SRV-014A through IDR-SRV-014G have not yet been performed. This report therefore avoids claims about current server/client behavior that the standards and official repository cannot prove.
- Problem responses generated below the application layer—TLS, front proxy, request parser, overload shedder—may have less detail. Deployment planning must still align their status, headers, media, and disclosure posture as far as the layer permits.

### 16.2 Open Questions

| ID | Open question | Owning topic(s) |
|---|---|---|
| U-013-01 | What stable HTTPS origin and exact OpenAPI dialect/version will Glaux use for Problem schemas and type documentation? | 014 |
| U-013-02 | Which representation/schema failures are structural `400`, semantic `422`, or current-state `409`, and what public field codes are stable? | 023 |
| U-013-03 | Which resources and operations require concealed `404`, and how will timing, size, links, counts, and cache behavior be equalized? | 039, 039A, 040 |
| U-013-04 | What PATCH media type, null/array semantics, immutable-field rules, and error mapping apply under the unresolved Features Part 4 dependency? | 014, 023, 029 |
| U-013-05 | Are custom collection membership and any future bulk operations atomic; if not, what per-member result contract applies? | 029, 031 |
| U-013-06 | What ETag/precondition/idempotency contract controls stale writes, duplicate creates, and replayed Commands? | 029, 043 |
| U-013-07 | What exact HTTP response/body is returned for synchronous Command processing, and where is the durable acceptance boundary? | 036, 037 |
| U-013-08 | What happens when a Command is submitted to a `live=false` ControlStream: HTTP rejection, accepted-then-rejected domain state, or another policy? | 036, 038, 042 |
| U-013-09 | Which freshness, completeness, and provenance fields make stale or last-known data safe and interoperable? | 020, 034, 042 |
| U-013-10 | Will the IETF RateLimit draft mature into an RFC before implementation freezes its public rate-limit fields? | 014, 044, 048 |
| U-013-11 | Which moves, replacements, deprecations, or legal/policy blocks justify redirects, `410`, `451`, or link metadata? | 010A, 039, 040 |
| U-013-12 | How will reverse proxies, gateways, and streaming endpoints preserve the same safe error identity and trace correlation? | 035, 046, 052 |
| U-013-13 | Will an upstream corrigendum reconcile Part 1 Requirement 61's “parameter present” rule with ATS A.68's true/false behavior? | 014, 050, 057 |

### 16.3 Material Risks

| Risk | Consequence | Mitigation |
|---|---|---|
| A generic framework exception handler controls the contract | Wrong codes, missing headers, leaking details, and OAS drift | Central catalog plus operation tests and generated/validated documentation |
| Command lifecycle is collapsed into HTTP success/failure | Clients retry dangerous work or misread accepted tasks | Enforce the two-plane rule and test acceptance/replay boundaries |
| Hidden-resource policy is inconsistent | Enumeration through status, body, timing, counts, links, or caches | One policy decision applied at every lookup and serialization layer |
| Tagged OAS is copied as authoritative | Required cases omitted; generic cases overstated; broken schema references retained | Rebuild from approved requirements and record every artifact deviation |
| Retry hints are mistaken for replay permission | Duplicate mutation or actuation | Gate retries on method/idempotency state; fault-inject post-commit failures |
| Draft behavior is frozen accidentally | Future incompatibility or false conformance claim | Pin draft evidence, label it H/U, and revisit only in its assigned downstream topic |

---

## 17. Validation Against the Research Plan

### 17.1 Methodology Phases

| Phase | Validation | Result |
|---|---|---|
| 1 — sources and taxonomy | Sources, pins, labels, and thirteen matrix fields are documented in §§3–4 and Appendix B. | Satisfied |
| 2 — standards extraction | Direct CSAPI, inherited OGC API/HTTP, OAS, schema, ATS, and problem guidance appear in §5. | Satisfied |
| 3 — family mapping | Thirty-three conditions and all required families are mapped in §§6–7. | Satisfied |
| 4 — body and documentation | RFC 9457 profile, extensions, safe fields, headers, OAS, docs, and fixtures are covered in §§8 and 11. | Satisfied |
| 5 — security, DDIL, interop, tests | §§9–13 cover disclosure, degraded state, bounded implementation evidence, and test implications. | Satisfied |
| 6 — synthesis | §§14–16 and Appendices C–D provide handoffs, proposed decisions, risks, open questions, and acceptance state. | Satisfied |

### 17.2 Success Criteria

| Plan criterion | Evidence | Result |
|---|---|---|
| Requirements identified with anchors | §§3, 5–7, 18; Appendix B | Satisfied |
| CSAPI and inherited behavior distinguished | Reading Guide; §§3, 5–7 | Satisfied |
| Families mapped to statuses | §6 two-part keyed matrix; §7 | Satisfied |
| Machine-readable guidance documented | §8 | Satisfied |
| Required failure areas assessed | §§5–10; Appendix A D11–D41 | Satisfied |
| Security and leakage risks identified | §9; §§6, 16 | Satisfied |
| OpenAPI/documentation/validation/conformance/test handoffs | §§11, 13–14 | Satisfied |
| Recommendations decision-usable and bounded | §15; Appendix C | Satisfied; owner acceptance pending |
| References explicit and reproducible | §18; Appendix B pins and audit method | Satisfied |

### 17.3 Deliverable Content and Matrix Fields

All eighteen required report areas are present as numbered Sections 1–18. Appendices A–D provide question-level traceability, artifact/history evidence, decision status, and completion/transition control.

The §6 matrix is deliberately split into two tables keyed by row ID. Together each row contains all thirteen required fields: family, condition, status, source/anchor, authority, body, headers/links, security, recovery, OpenAPI implication, test implication, handoff, and unresolved notes.

### 17.4 Research-Question Validation

All five core questions are answered directly in §2.4. Appendix A maps every one of the forty-seven detailed questions to the report evidence and its present disposition. No question is silently deferred: unresolved design details have an explicit owner in §14 or §16.

---

## 18. References

### 18.1 Controlling and Inherited Standards

- [OGC API — Connected Systems — Part 1: Feature Resources, OGC 23-001](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API — Connected Systems — Part 2: Dynamic Data, OGC 23-002](https://docs.ogc.org/is/23-002/23-002.html)
- [OGC API — Features — Part 1: Core, OGC 17-069r4](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [OGC API — Common — Part 1: Core, OGC 19-072](https://docs.ogc.org/is/19-072/19-072.html)
- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)
- [RFC 9111 — HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111.html)
- [RFC 6585 — Additional HTTP Status Codes](https://www.rfc-editor.org/rfc/rfc6585.html)
- [RFC 6750 — OAuth 2.0 Bearer Token Usage](https://www.rfc-editor.org/rfc/rfc6750.html)
- [RFC 7725 — An HTTP Status Code to Report Legal Obstacles](https://www.rfc-editor.org/rfc/rfc7725.html)
- [RFC 8470 — Using Early Data in HTTP](https://www.rfc-editor.org/rfc/rfc8470.html)
- [RFC 9457 — Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html)
- [IANA HTTP Status Code Registry](https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml)
- [IANA HTTP Field Name Registry](https://www.iana.org/assignments/http-fields/http-fields.xhtml)
- [IANA HTTP Problem Types Registry](https://www.iana.org/assignments/http-problem-types/http-problem-types.xhtml)
- [OpenAPI Specification 3.2.0](https://spec.openapis.org/oas/v3.2.0.html)

### 18.2 Official Artifacts and Maintenance Evidence

- [Official CSAPI repository, Version 1.0.0 tag](https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0)
- [Pinned Version 1.0.0 commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)
- [Official tagged API artifacts](https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api)
- [Official CSAPI issues](https://github.com/opengeospatial/ogcapi-connected-systems/issues)
- [Official CSAPI pull requests](https://github.com/opengeospatial/ogcapi-connected-systems/pulls)
- [Current OGC API — Features Part 4 draft](https://docs.ogc.org/DRAFTS/20-002r1.html)
- [Pinned Features repository commit `9ca25f56a58ed822ea8a685a7a41afa7181aaa8b`](https://github.com/opengeospatial/ogcapi-features/commit/9ca25f56a58ed822ea8a685a7a41afa7181aaa8b)
- [Glaux shared upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md)

### 18.3 Supporting Security and Operational Guidance

- [RFC 9700 — Best Current Practice for OAuth 2.0 Security](https://www.rfc-editor.org/rfc/rfc9700.html)
- [RFC 5861 — HTTP Cache-Control Extensions for Stale Content](https://www.rfc-editor.org/rfc/rfc5861.html)
- [OWASP API Security Top 10 — 2023](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
- [OWASP Error Handling Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html)
- [IETF RateLimit Fields draft, revision 11](https://datatracker.ietf.org/doc/draft-ietf-httpapi-ratelimit-headers/)

### 18.4 Project Sources

- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)
- [IDR-SRV-013 Research Plan](../IDR%20Plans/idr-srv-013-error-model-http-status-codes-and-failure-semantics.md)
- [Research Report Template](../../../../../Governance/research-report-template.md)
- Accepted IDR-SRV-006 through IDR-SRV-012 reports in this report directory

---

## Appendix A. Detailed Question Ledger

| ID | Detailed plan question | Answer/evidence | Disposition |
|---|---|---|---|
| D01 | Error behavior in CSAPI Part 1 | §5.1; Part 1-local negative status is chiefly cascade `409`, with inherited query/lookup behavior. | Answered |
| D02 | Error behavior in CSAPI Part 2 | §§5.2, 7.6; direct `400`/`409` plus task-domain outcomes. | Answered |
| D03 | Inherited Features/Common behavior | §5.3 and family analysis. | Answered |
| D04 | Applicable HTTP status guidance | §§5.4, 6–7. | Answered |
| D05 | Relevant machine-readable standard | §8; RFC 9457 selected, RFC 7807 superseded. | Answered; P-013-01 pending |
| D06 | Behavior reflected in OAS/schemas | §§5.6, 11.4; Appendix B. | Answered; defects recorded |
| D07 | Handling named and other status codes | §6 includes required/recommended core and protocol-edge cases. | Answered |
| D08 | Codes for unsupported/invalid/conflict/stale/rate/dependency cases | §§6–7 and §10. | Answered; detailed policies handed off |
| D09 | Inappropriate codes | §6.3. | Answered |
| D10 | Headers, links, and metadata | §§6.2, 8.4, 10. | Answered |
| D11 | Missing/deleted/moved/stale/broken/inaccessible/invalid/unsupported resources | §7.2. | Answered; move/deletion policy U-013-11 |
| D12 | Nonexistence versus unauthorized existence | §9.2. | Answered; exact policy U-013-03 |
| D13 | Partial related-resource failure | §§7.2, 7.7, 10. | Bounded; batch/partial policy U-013-05 |
| D14 | Stale/cached/last-known references | §10.2. | Answered; metadata U-013-09 |
| D15 | Query/filter/sort/time/bbox/limit/pagination/selection failures | §§7.3, 6. | Answered; accepted IDR-SRV-011 controls details |
| D16 | `400` versus `422` versus `501` | §§6, 7.5. | Baseline answered; detailed catalog U-013-02 |
| D17 | Correctable machine details | §§8.1–8.3. | Answered; validation codes to IDR-SRV-023 |
| D18 | Avoiding restricted counts/existence leakage | §§7.3, 9.2–9.3. | Answered; policy U-013-03 |
| D19 | Accept/Content-Type/profile/encoding/payload/representation failures | §§7.4, 6. | Answered |
| D20 | Suggesting alternates | §8.4; use authoritative headers/links only when safe and applicable. | Answered |
| D21 | Negotiation documentation/tests | §§11, 13. | Answered |
| D22 | Schema validation behavior | §§5.2, 7.5. | Baseline answered; U-013-02 |
| D23 | Response versus request validation | §7.5; bad server output is an internal defect, not client `422`. | Answered |
| D24 | Safe validation detail | §§8.2, 9.3. | Answered |
| D25 | Identifying fields/constraints/schemas/profiles | §8.2. | Bounded; final catalog to IDR-SRV-023 |
| D26 | Handoff to IDR-SRV-023 | §14. | Complete |
| D27 | Observation/datastream/event/status failures | §§5.2, 7.6, 10.4. | Answered at common-contract level |
| D28 | Command/feasibility/unavailable/duplicate/unsafe/stale failures | §§7.6, 9.4, 10. | Bounded; lifecycle/safety issues explicit |
| D29 | HTTP error versus domain state | §§7.6, 10.4; P-013-09. | Answered |
| D30 | Feasibility versus execution failure | §§7.6, 10.4. | Answered; result model to IDR-SRV-037 |
| D31 | Handoffs to 036/037/038 | §14. | Complete |
| D32 | Authentication versus authorization | §9.1. | Answered |
| D33 | Policy/releasability/classification/cross-boundary/need-to-know failure | §§9.2–9.3. | Bounded; no sensitive policy invented |
| D34 | Avoid confirming restricted resources | §9.2. | Answered; exact profile U-013-03 |
| D35 | Command/control authorization | §9.4. | Baseline answered; detailed handoff complete |
| D36 | Handoffs to 039/040/055 | §14. | Complete |
| D37 | Publisher/system/sensor/database/broker/storage outage | §10.1. | Answered by Glaux role and usable-output state |
| D38 | Server/dependency/degraded/stale/last-known/unavailable distinctions | §§10.1–10.2. | Answered |
| D39 | DDIL response semantics | §§10.1–10.4. | Baseline answered; metadata/policy handed off |
| D40 | HTTP versus status/event/warning/metadata/retry | §§10.2–10.4; obsolete `Warning` noted. | Answered |
| D41 | Handoffs to 020/035/041/042/043/046 | §14. | Complete |
| D42 | OpenAPI representation | §11.2. | Answered; concrete document to IDR-SRV-014 |
| D43 | Reusable error schemas | §§8, 11.2. | Baseline answered |
| D44 | Negative tests per family | §13.1. | Answered |
| D45 | Conformance/security/command/interoperability tests | §§13.1, 13.5. | Answered; detailed topics retain ownership |
| D46 | Golden files/fixtures | §§13.2–13.3. | Answered |
| D47 | Handoffs to 014/050/051/053/055/056 | §14. | Complete |

---

## Appendix B. Official Artifact and History Audit

### B.1 Reproducibility Pins

| Evidence | Pin/state used |
|---|---|
| Approved CSAPI repository | Tag `v1.0.0`, commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2` |
| Current CSAPI repository comparison | `master` commit `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f` observed August 2, 2026 |
| Features Part 4 context | `ogcapi-features` commit `9ca25f56a58ed822ea8a685a7a41afa7181aaa8b`, observed August 2, 2026 |
| Official repository counts | 141 issues and 58 pull requests visible through the official repository/API at refresh time |
| Shared history | Upstream-history register refreshed with IDR-SRV-013 routing in the same change set |

### B.2 Mechanical OAS Review

The audit walked each tagged root OAS, enumerated all reachable HTTP methods and response keys, separately scanned every path file for unreachable method blocks, checked response components and referenced files, and compared the tagged API tree with current `master`.

| Check | Reproducible result |
|---|---|
| Root-reachable operations | 87 (Part 1: 39; Part 2: 48) |
| Unreferenced method block | One additional DELETE in tagged [`part1/openapi/paths/subsystemById.yaml`](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/paths/subsystemById.yaml); its root path/reference is commented out, producing raw all-file count 88 |
| Generic error declarations | 81 root-reachable operations each declare `400`, `401`, `403`, `404`, and `5XX`; the unreferenced DELETE would make each raw count 82 |
| Success declarations | Root-reachable `200`: 44; `201`: 15; `204`: 29; the unreferenced DELETE makes raw `204` count 30 |
| Conflict declarations | Root-reachable `409`: 16; the unreferenced DELETE makes raw count 17 |
| Part 1 GET exceptions | Six operations declare only `200`: inline `/` and `/conformance`, plus four Common collection/item routes |
| Individually absent statuses | No `405`, `406`, `408`, `410`, `412`, `413`, `414`, `415`, `422`, `425`, `428`, `429`, `431`, `500`, `502`, `503`, or `504` |
| Error bodies/headers | Shared 4xx components are description-only; `401` lacks `WWW-Authenticate`; generic `5XX` has no body/header contract |
| Broken unused body component | `ServerError.yaml` says `application/json` and references missing `schemas/exception.yaml` and `examples/serverError.json` |
| Direct CSAPI conflict alignment | DataStream/ControlStream main PUT operations omit Requirements 64/69 `409`; `409_PUT` is attached to two separate schema-operation paths |
| Required query/OAS alignment | Part 1 requires `recursive` on `GET /deployments`; the tagged path omits it, creating a potential false unknown-parameter `400` |
| Bulk/partial-failure artifacts | `systemOrArrayOrRefs.yaml` advertises JSON URI arrays and System arrays beyond Requirement 71's `text/uri-list`; unused `batch_delete.json`/`batch_response.json` imply per-item statuses/free-text errors without normative atomicity or failure semantics |
| Command synchronous response | Command POST declares `200` but references Part 1 `resourceLinks.yaml`, an array-of-created-resource-URI response, rather than the Part 2 synchronous status report |
| Non-approved Part 2 paths | Four System History operations remain although the associated standard clause is commented out of the approved document |
| Tag-to-master API-tree difference | Only one Part 1 example-link change; no material error-contract repair |

The Part 2 bundle separately contains 48 operations—23 GET, 7 POST, 10 PUT, and 8 DELETE—with no PATCH and no Feasibility path. This supports the root-reachable inventory and explains several normative/OAS gaps; it is not an additional standard obligation.

### B.3 Normative ATS and Prose Review

- Part 1 Requirement 61 says any request containing `cascade` is accepted, while ATS A.68 treats `cascade=false` as non-cascading and expects `409`; Glaux's proposed interpretation follows the ATS and ordinary boolean meaning while recording the normative conflict.
- Part 2 Requirements 64, 65, 69, and 70 explicitly require `409`; Requirements 67, 72, 82, and 86 explicitly require `400`.
- Part 2 Requirements 80 and 84 require rejection of populated-stream schema PATCH but omit the code; their normative ATS expects `409`.
- The ATS and artifacts have additional copy/path/schema defects, including Feasibility test/path mismatches and missing PATCH/Feasibility OAS coverage. The report uses the applicable requirement while preserving each discrepancy for conformance planning.
- The tagged [Feasibility `ref-from-controlstream` ATS](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/annex-abstract-test-suite.adoc#L649-L676) exercises `/commands` rather than `/feasibility`; the [CommandResult ATS](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/annex-abstract-test-suite.adoc#L1963-L1974) validates nonexistent `result` against `resultSchema` although the [JSON schema](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/schemas/json/commandResult.json#L21-L31) uses `data` and feasibility uses `feasibilityResultSchema`; and the [Command schema ATS](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/standard/sections/annex-abstract-test-suite.adoc#L1363-L1375) checks only CREATE while describing the wrong “result structure” field. These tests must remain identifiable in the official-versus-supplemental harness ledger.

### B.4 Bounded Upstream History

The refresh concentrated on issues and pull requests that could explain or change the failure contract. Material examples include:

| Issue | State at refresh | Relevance |
|---|---|---|
| [#23](https://github.com/opengeospatial/ogcapi-connected-systems/issues/23) | Open | Advertising write-encoding support affects `415` and capabilities. |
| [#61](https://github.com/opengeospatial/ogcapi-connected-systems/issues/61) | Closed | Cascade/delete design history; does not replace the approved `409` requirement. |
| [#141](https://github.com/opengeospatial/ogcapi-connected-systems/issues/141) | Open | Parts 1/2 still depend on draft Features Part 4 CRUD/update behavior; publication requires an eventual delta review. |
| [#164](https://github.com/opengeospatial/ogcapi-connected-systems/issues/164) | Open | Client-receivable versus server-generated association links during POST/PUT remain unclear, affecting invalid-write handling. |
| [#166](https://github.com/opengeospatial/ogcapi-connected-systems/issues/166) | Open | Cross-encoding PUT lossiness and validation remain unresolved. |
| [#169](https://github.com/opengeospatial/ogcapi-connected-systems/issues/169) | Open | Required Deployment `recursive` query support is omitted from the tagged OAS; approved PR #196 remains unmerged. |
| [#170](https://github.com/opengeospatial/ogcapi-connected-systems/issues/170) | Open | Published OAS lacks PATCH and the referenced Features Part 4 dependency is problematic. |
| [#171](https://github.com/opengeospatial/ogcapi-connected-systems/issues/171) | Open | Cascade/OAS context relevant to conflict coverage. |
| [#178](https://github.com/opengeospatial/ogcapi-connected-systems/issues/178) | Open | Stream-schema interpretation affects validation and conflict behavior. |
| [#181](https://github.com/opengeospatial/ogcapi-connected-systems/issues/181) | Open | Observation validation/schema context. |
| [#185](https://github.com/opengeospatial/ogcapi-connected-systems/issues/185) | Open | Bulk mutation and partial-failure semantics are not settled. |

Issue and pull-request discussion is H evidence. It becomes controlling only through an approved standards revision, corrigendum, or explicit accepted Glaux project decision.

### B.5 Evidence Limitations

- Mutable web state was captured on August 2, 2026. The immutable CSAPI tag remains the approved anchor.
- Existing implementation/server/client behavior was not generalized from anecdotes; the focused IDR-SRV-014A–014G studies remain the proper evidence source.
- No official CSAPI Problem Details schema or complete error catalog exists in the reviewed tag.
- RateLimit fields remain an active draft, not a stable standards-track RFC baseline.
- Some lower-layer failures cannot carry the full Problem body; their behavior depends on deployment components not yet selected.

---

## Appendix C. Proposed Decision Register

The Glaux Project Lead accepted every entry on August 31, 2026.

| Decision | Short label | Status | Acceptance owner |
|---|---|---|---|
| P-013-01 | RFC 9457 response baseline | Accepted | Glaux Project Lead |
| P-013-02 | Stable problem-type registry | Accepted | Glaux Project Lead |
| P-013-03 | Deterministic authority/selection order | Accepted | Glaux Project Lead |
| P-013-04 | Direct CSAPI `400`/`409` mapping | Accepted | Glaux Project Lead |
| P-013-05 | `400`/`415`/`422` boundary | Accepted | Glaux Project Lead |
| P-013-06 | `401`/`403`/concealed `404` boundary | Accepted | Glaux Project Lead |
| P-013-07 | Empty query is successful | Accepted | Glaux Project Lead |
| P-013-08 | Headers and conservative caching | Accepted | Glaux Project Lead |
| P-013-09 | HTTP/domain-task two-plane rule | Accepted | Glaux Project Lead |
| P-013-10 | Origin/gateway dependency mapping | Accepted | Glaux Project Lead |
| P-013-11 | Retry never implies unsafe replay | Accepted | Glaux Project Lead |
| P-013-12 | `429`/`503` limit boundary | Accepted | Glaux Project Lead |
| P-013-13 | Single operation-level error catalog | Accepted | Glaux Project Lead |
| P-013-14 | Rebuild tagged OAS error material | Accepted | Glaux Project Lead |
| P-013-15 | Fixtures and negative/fault tests | Accepted | Glaux Project Lead |
| P-013-16 | Bounded upstream-history monitoring | Accepted | Glaux Project Lead |
| P-013-17 | No invented generic partial-failure codes | Accepted | Glaux Project Lead |
| P-013-18 | Sanitized public detail and opaque correlation | Accepted | Glaux Project Lead |

The acceptance recorded here authorizes these recommendations as the Glaux planning baseline while preserving every downstream ownership boundary and unresolved item stated in this report.

---

## Appendix D. Completion and Handoff

- [x] Exactly one research topic executed: IDR-SRV-013.
- [x] All five core questions answered.
- [x] All forty-seven detailed questions traced in Appendix A.
- [x] All six methodology phases executed and validated.
- [x] All nine success criteria satisfied.
- [x] All eighteen required content areas present.
- [x] All thirteen minimum matrix fields present for every keyed condition.
- [x] Direct standards obligations, inherited rules, artifact/history evidence, analysis, project recommendations, and unresolved issues remain distinguishable.
- [x] Official tagged OAS, schema/reference integrity, normative ATS, current draft dependency, and bounded issue/PR history reviewed.
- [x] Three independent read-only evidence reviews reconciled.
- [x] Report reviewed for scope, coverage, source authority, disclosure risk, and downstream ownership.
- [x] Plan-owner acceptance of IDR-SRV-013 — accepted by the Glaux Project Lead on August 31, 2026.

**Research execution completed:** August 2, 2026
**Plan-owner acceptance:** August 31, 2026
**Next authorized topic:** IDR-SRV-014 — OpenAPI Description and API Documentation Strategy

The Glaux Project Lead accepted IDR-SRV-013 and authorized execution of exactly one next research topic, IDR-SRV-014, on August 31, 2026. This records the completed transition; IDR-SRV-014 remains governed by its own report and acceptance gate.

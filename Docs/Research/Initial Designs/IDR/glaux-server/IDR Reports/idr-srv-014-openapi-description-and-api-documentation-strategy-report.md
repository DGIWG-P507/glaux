# Section 014: OpenAPI Description and API Documentation Strategy - Research Report

**Topic ID:** IDR-SRV-014<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-014 OpenAPI Description and API Documentation Strategy](../IDR%20Plans/idr-srv-014-openapi-description-and-api-documentation-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core and 36 detailed questions; all six methodology phases, ten success criteria, eighteen required content areas, and thirteen minimum OpenAPI/documentation-matrix fields are validated<br>
**Methodology Used:** Reconciliation of accepted IDR-SRV-006 through IDR-SRV-013 with approved CSAPI, OGC API, OpenAPI, JSON Schema, HTTP, and Problem Details sources; byte-normalized comparison of immutable tagged and published artifacts; mechanical lint, parse, bundle, and render probes; bounded refresh of routed official issues, pull requests, releases, and current repository state; and current primary-source tooling review<br>
**Research Time:** Approximately 5 hours of AI-assisted elapsed execution on August 31, 2026<br>
**Primary Sources:** OGC 23-001, OGC 23-002, OGC 17-069r4, OGC 19-072, OGC 23-000, OGC 24-014, OpenAPI 3.1.2 and 3.2.0, JSON Schema Draft 2020-12, RFC 9110, RFC 8288, RFC 9457, and IANA registries<br>
**Supporting Resources:** Accepted IDR-SRV-006 through IDR-SRV-013 reports; official CSAPI tag `v1.0.0`; published modular packages and release bundles; the shared upstream-history register; current official issue/PR/release state; and official Redocly, Swagger UI, Scalar, Stoplight Elements, and OpenAPI Generator documentation<br>
**Document Purpose:** Establish an implementation-usable machine-contract and human-documentation baseline for the Rust Glaux reference server without copying an incomplete example, overstating conformance, or binding the contract to one renderer or framework<br>
**Author:** OpenAI Codex<br>
**Accepted By:** Pending Glaux Project Lead review<br>
**Acceptance Date:** Pending<br>
**Date:** August 31, 2026<br>
**Last Updated:** August 31, 2026

---

## Reading Guide

This report uses six evidence labels:

- **N — normative CSAPI:** an explicit obligation in approved CSAPI Part 1 or Part 2, including its normative ATS;
- **I — inherited:** an applicable normative OGC API, OpenAPI, JSON Schema, HTTP, or Web Linking rule;
- **A — artifact finding:** an observation about an official OpenAPI, schema, example, package, or rendering that does not override approved prose;
- **H — history:** issue, pull-request, commit, release, or maintenance evidence that explains an outcome but does not amend the approved standard;
- **P — project recommendation:** a bounded Glaux choice that becomes project policy only after plan-owner acceptance; and
- **U — unresolved:** a question assigned to a later topic, implementation spike, or upstream decision.

“OpenAPI description” or **OAD** means the complete machine-readable description, which may be a multi-document graph. “Bundle” means a distribution form whose required resources are closed over the published package; it does not mean recursively expanding every `$ref`, which is unsafe and sometimes impossible for recursive schemas. “Rendered documentation” is a view of the contract, not a second contract.

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Official CSAPI OpenAPI Artifact Comparison
5. Upstream Maintenance History and Known-Defect Disposition
6. Glaux OpenAPI Contract Scope and Structure
7. Schema and Component Strategy
8. Human-Readable Documentation and Examples Strategy
9. Publishing and Discovery
10. Security and Policy Documentation
11. Validation, Contract Drift, and Generated Clients
12. Interoperability and Existing-Implementation Implications
13. Test-Strategy Implications
14. Downstream Topic Handoff Matrix
15. Recommendations
16. Risks, Constraints, and Open Questions
17. Validation Against the Research Plan
18. References
Appendix A. Detailed Question Ledger
Appendix B. Reproducible Artifact and Tool Audit
Appendix C. Proposed Decision Register
Appendix D. Completion and Handoff

---

## 1. Executive Summary

Glaux should treat OpenAPI as a generated, release-governed projection of the same behavior registries that drive routing, validation, conformance declarations, representations, errors, and tests. It should not hand-maintain an OAD beside the Rust implementation, generate a contract only from framework annotations, or copy the official CSAPI example files. Those three approaches all create an avoidable second source of truth.

The recommended baseline is:

1. **Author and publish OpenAPI 3.1.2.** It preserves the CSAPI artifacts' JSON Schema Draft 2020-12 alignment while taking the current 3.1 patch level. OpenAPI 3.2.0 is the latest specification, but the CSAPI release is 3.1.0, OpenAPI Generator still labels 3.1 support beta, and the renderer ecosystem has uneven 3.2 completion. Reassess 3.2 at a release boundary using the same compatibility suite.
2. **Do not claim an OAS 3.0 conformance class initially.** OGC API Common's OAS 3.0 class conditionally requires a JSON OAS 3.0 representation, HTML documentation, implementation parity, complete response coverage, and security-scheme coverage. A 3.1 OAD does not satisfy that class. Generate a separate 3.0.4 compatibility view only if a named consumer requires it and automated loss/parity tests pass.
3. **Publish one complete implementation-specific deployment contract.** It must combine every enabled Part 1, Part 2, inherited, and Glaux extension operation exposed at that API root. Split modular source files by domain, not by public contract. Do not publish Part 2 as if it were independently complete.
4. **Use a registry-first hybrid workflow.** Typed capability, route, parameter, representation, schema, error, security, maturity, and conformance metadata are authoritative inputs. Generate the OAD and route/test projections from those inputs; retain reviewed prose and examples as versioned source. Fail CI when the projections disagree.
5. **Distribute three synchronized forms.** Serve a JSON `service-desc` as the canonical machine representation, a YAML alternate, and a self-hosted HTML `service-doc`. Keep modular sources for maintenance, publish a reference-closed bundle for tools, and publish an offline archive containing the modular graph, vendored schemas, examples, manifest, hashes, and licenses.
6. **Pin schemas and tools.** Vendor exact approved OGC/SensorML/SWE Common artifacts without editing them, record origin and SHA-256, preserve their dialect declarations, and place Glaux-owned wrappers/components under stable `$id` URIs. Pin parser, linter, bundler, renderer, and generator versions in the build.
7. **Keep documentation renderer-neutral.** Provisionally use self-hosted Redoc CE 3.x for the production reference page if it passes the Glaux corpus and resource-budget gate. Use Swagger UI 5.32+ as an independent render/interaction smoke test and local developer console. Scalar is the preferred fallback candidate. A renderer result never substitutes for structural, semantic, runtime, or client tests.
8. **Make drift release-blocking.** Validate syntax, reference closure, schemas, examples, operation identifiers, routes, parameters, media types, errors, security, conformance, server URLs, semantic change classification, documentation rendering, and selected client generation. Compare both directions: every advertised behavior exists at runtime, and every public runtime behavior is documented.

The official CSAPI artifacts are useful evidence but not reusable Glaux contracts. The tag contains modular Part 1 and Part 2 examples with `openapi: 3.1.0` and `info.version: 0.0.1`; neither has `operationId` values or security schemes, both contain demonstration/local servers, and together they omit required operations while retaining removed or contradictory material. The release bundles retain 32 Part 1 and 51 Part 2 relative external references. Current Redocly CLI 2.43.2 linting found substantial errors and warnings; Part 2 release-bundle linting hit a maximum call-stack failure, and all four tested Redocly static builds terminated with exit 134. Those results demonstrate the need for dependency closure, bounded component selection, cyclic-schema tests, and tool diversity.

No new user decision is required to begin the downstream implementation studies. Acceptance of this report would fix the proposed P-014 decisions in Appendix C and authorize `IDR-SRV-014A`; it would not select the Rust web framework, implement the pipeline, or claim OAS 3.0 conformance.

## 2. Scope and Plan Alignment

### 2.1 Included

This topic determines:

- the canonical OAS line and conditional OAS 3.0 compatibility policy;
- contract coverage, composition, modularity, bundle, and artifact forms;
- the boundary between metadata registries, generated material, and reviewed prose/examples;
- schema pinning, vendoring, `$id`, dialect, component, and offline-package policy;
- stable operation identifiers, tags, links, errors, security, maturity, and conformance traceability;
- live, immutable-release, local, test, static-site, and DDIL publication behavior;
- renderer selection criteria and a provisional production/development split;
- validation, drift, semantic-diff, generated-client, rendering, and test gates; and
- explicit handoffs to implementation, resource-model, schema, security, conformance, test, and interoperability topics.

### 2.2 Excluded

This topic does not:

- choose or implement the Rust HTTP/OpenAPI framework;
- repair upstream CSAPI artifacts;
- define the canonical resource model, authentication mechanism, policy vocabulary, persistence design, streaming protocol, or deployment platform;
- claim that a documentation renderer or linter proves standards conformance;
- commit Glaux to a particular generated-client language set before interoperability studies; or
- start any work from `IDR-SRV-014A` through `IDR-SRV-014G`.

### 2.3 Prerequisite State

IDR-SRV-006 through IDR-SRV-013 are accepted. Their registries and decisions supply the behavior this OAD must project: applicable conformance classes, entry points, routes and links, compatibility identities, query semantics, representations, and RFC 9457 error behavior. This report does not reopen them. It records later work only where OpenAPI needs a projection or verification hook.

### 2.4 Core-Question Answers

| Core question | Answer |
|---|---|
| Required or expected behavior | Every instance needs a retrievable definition that accurately describes its capabilities. If Glaux claims Common OAS 3.0, the additional OAS 3.0 representation, completeness, parity, error, and security requirements become normative. Independently, OpenAPI syntax and reference rules govern any OAD Glaux publishes. |
| Representation of server behavior | One combined, capability-accurate 3.1.2 deployment OAD maps all public routes, parameters, representations, schemas, headers, links, RFC 9457 errors, and security requirements. Optional behavior appears only when enabled; experimental behavior is clearly marked and separately gated. |
| Publication, versioning, validation, synchronization | Registry-first generation produces live JSON/YAML, immutable release artifacts, renderer input, tests, and a manifest. CI and startup/runtime parity checks block drift. Contract version, software version, standards versions, schema versions, and artifact digest remain distinct. |
| Human documentation relationship | `service-doc` is a self-hosted view of the same immutable semantic contract, supplemented by versioned guides and scenario examples. Renderer configuration may change presentation but not behavior. |
| Downstream implications | Schema closure, semantic diff, selected client generation, render smoke tests, runtime parity, conformance traceability, security review, and interoperability clients become required downstream gates. |

## 3. Evidence Base and Authority Classification

### 3.1 Controlling Standards

| Source | Authority | Material result |
|---|---|---|
| OGC 23-001, Clause 8.3 | N | Every CSAPI implementation instance provides a definition describing that instance and complies with OGC API Common definition requirements. |
| OGC 23-002 | N | Part 2 extends the Part 1 API and contributes dynamic-data routes, representations, schemas, and tasking behavior; it is not a standalone server definition. |
| OGC 19-072, Common Core and OAS 3.0 class | I | Landing-page `service-desc`/`service-doc` links must dereference. The optional OAS 3.0 class adds exact JSON/HTML, validity, parity, completeness, and security obligations. |
| OGC 17-069r4 | I | Features entry-point, collection, feature, query, representation, and conditional OAS behavior apply where inherited. |
| OpenAPI 3.1.2 | I | Defines the selected OAD syntax, multi-document references, Schema Object dialect, paths, operations, components, security, and extension behavior. |
| OpenAPI 3.2.0 | I/context | Latest OAS as of this report; supplies the monitored upgrade target and explicit security guidance for external resources, reference cycles, security filtering, and Markdown sanitization. |
| JSON Schema Draft 2020-12 | I | Governs external JSON schemas and recursive/dynamic reference behavior when declared. |
| RFC 9110, RFC 8288, RFC 9457 | I | Govern HTTP semantics, typed discovery links, and Glaux's accepted error representation. |

### 3.2 Artifact and History Evidence

The immutable artifact baseline is tag `v1.0.0` at commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`, the two published `schemas.opengis.net` ZIP packages, and the two `v1.0.0` release assets. The mutable comparison baseline remains `master` commit `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f`; the August 31 refresh found no later commit on `master`.

The bounded history review covered the register-routed issues #12, #13, #48, #57, #77–#79, #81, #114, #137, #146, #148, #169–#170, #177, #185–#186 and directly cross-cutting new issue #200. Issue states and PR #196 remained materially unchanged. The repository now contains 142 issues (61 open, 81 closed) and 58 pull requests (3 open, 52 merged, 3 closed unmerged).

### 3.3 Tooling Evidence

Current official documentation establishes capability, not fitness for Glaux:

- Redoc CE 3.x documents OAS 3.1 support and standalone HTML generation; Redocly CLI documents structural support through OAS 3.2.
- Swagger UI 5.32.0 documents compatibility through OAS 3.2.0 but also documents lack of relative external-file support.
- Scalar's parser documents 3.0/3.1/3.2 support, while its open 3.2 tracking issue still lists API-reference/client work; its documented URL-loading pattern is appropriate for 3.1 evaluation.
- Stoplight Elements documents OAS 3.1 and self-hosted interactive documentation.
- OpenAPI Generator 7.22.0 documents OAS 3.1 as beta support and no 3.2 compatibility claim.

Therefore no single tool is the authority, and “supported version” is not evidence that CSAPI's recursive schema graph will render or generate safely.

### 3.4 Evidence Limitations

- Tool probes were performed on Windows with Node.js 26.8.1 and Redocly CLI 2.43.2. They establish reproducible failures in that environment, not universal impossibility.
- No browser automation, Rust generator, or external live implementation was required by this topic. Those belong to 014A–014G, 044, and 056.
- The official release remains marked prerelease in GitHub metadata although the OGC standards are approved. The flag does not downgrade their authority.
- Open issue labels and comments are maintenance evidence, not adopted standards text.

## 4. Official CSAPI OpenAPI Artifact Comparison

### 4.1 Provenance and Equality

| Artifact | Pin / digest | Inventory | Comparison result | Authority/use |
|---|---|---|---|---|
| Tagged Part 1 modular OAS | `v1.0.0`, `8e03b236` | 153 files under `openapi/`, including README; 20 paths, 39 operations | Source baseline | A/PB; reference and negative fixture |
| Published Part 1 ZIP | SHA-256 `43FCAB8FB079B153E1DC01559C9395A51E8FAB0E7C87C709FDAE6A28B1983F12` | 153 total: 152 OAS files plus package README | All 152 OAS files equal tag after CRLF/LF normalization; tag README omitted/replaced | A/PB; published package provenance |
| Part 1 release bundle | SHA-256 `69DA631D5D05F01716381CCA7B7EE6311402F2752A8FD79A9B72B663539555AA` | 196,226 bytes | Retains 32 `../examples/...` references | A/PB; non-standalone fixture |
| Tagged Part 2 modular OAS | `v1.0.0`, `8e03b236` | 160 files under `openapi/`, including README; 23 paths, 48 operations | Source baseline | A/PB; reference and negative fixture |
| Published Part 2 ZIP | SHA-256 `02ACCC4DD11A197F029A9A65D6E9EB3724EF3E7DC8A7E6E82BC05504844100A9` | 165 total: 159 OAS, 5 AsyncAPI, package README | All 159 OAS files equal tag after CRLF/LF normalization; tag README omitted/replaced | A/PB; published package provenance |
| Part 2 release bundle | SHA-256 `86ED005F9E7CF176264D6DEB72581A0B521A227CD7A198B6CB1BD32B39D83667` | 419,139 bytes | Retains 51 relative references: 45 examples and 6 schemas | A/PB; non-standalone fixture |

The published modular OAS content is therefore traceable to the tag. The package and bundle are not interchangeable: the ZIPs contain co-located dependency graphs; the release bundles do not close all dependencies.

### 4.2 Root Metadata and Contract Fitness

Both tagged roots declare `openapi: 3.1.0` and `info.version: 0.0.1`. They are titled as Parts 1 and 2, include demonstration or localhost server URLs, omit security schemes, and assign no `operationId` to any of the 87 root-reachable operations. Part 1's release bundle contains 39 operations: 20 GET, 8 POST, 5 PUT, and 6 DELETE. Part 2 contributes 48 operations. The absence of stable operation identifiers is a direct generated-client and traceability defect for an implementation contract.

The artifact split also conflicts with deployment-contract use. A Part 2 implementation depends on Part 1 and inherited behavior, while a client needs one coherent route and component graph for the selected API root. Glaux must compose a complete OAD, not ask consumers to merge standard-part examples.

### 4.3 Coverage and Correctness Gaps

Accepted reports 009–013 and the current audit establish these material examples:

- required deployment-scoped DataStream and ControlStream endpoints are absent from Part 2 OAS;
- required Feasibility routes and the canonical SystemEvent item are absent;
- removed System History routes remain;
- the required Part 1 Deployment `recursive` parameter is absent; PR #196 is open and unmerged;
- optional PATCH behavior is absent from both parts and its media/atomicity semantics remain unresolved;
- bulk/array request components exist even though interoperable bulk semantics are not standardized;
- response sets are incomplete relative to Common's OAS completeness principle and the accepted Glaux error catalog;
- command POST response material contains a status/body mismatch identified in IDR-SRV-013;
- representation, parameter, and path defects identified by reports 011–012 cannot be repaired by bundling; and
- external example references keep the released bundles dependent on their original directory layout.

The official files are informative examples and valuable defect fixtures. They are not implementation-specific descriptions of Glaux and cannot be copied unchanged.

### 4.4 Tool and Resource Behavior

Redocly CLI 2.43.2 produced the following default-recommended lint results:

| Target | Result |
|---|---|
| Tagged modular Part 1 root | 45 errors, 235 warnings |
| Tagged modular Part 2 root | 59 errors, 517 warnings |
| Part 1 release bundle | 44 errors, 299 warnings |
| Part 2 release bundle | Aborted: `Maximum call stack size exceeded` |

The counts mix structural, content, example, and opinionated policy rules and must not be called 100%-normative defect counts. They are still a reproducible compatibility signal. A generic YAML conversion emitted numerous alias/structure diagnostics and aborted on the Part 2 bundle with an “excessive alias count” resource-exhaustion guard. Redocly `build-docs` terminated with exit 134 for both modular roots and both release bundles in this environment. Upstream issues #78, #79, and #137 independently record circular-reference memory and rendering failures in Spectral, Swagger UI, and ReDoc-era tools.

Glaux must therefore:

- preserve recursive `$ref` graphs rather than fully dereference them;
- limit its OAD to enabled, reachable components;
- close required distribution references without YAML alias expansion;
- set parser/render time, memory, depth, alias, and external-fetch budgets;
- test at least two independent parsers/renderers; and
- classify rule failures so a style warning cannot hide a structural or runtime mismatch.

## 5. Upstream Maintenance History and Known-Defect Disposition

| History cluster | Verified disposition | Glaux consequence |
|---|---|---|
| #12, #13 | Closed after adding Part 1/2 OpenAPI examples | Treat them as examples, not normative or implementation-complete contracts. |
| #48, #77 | Open; published primary artifacts are 3.1, earlier dual 3.0/3.1 intent remains unfulfilled | Use 3.1.2; do not claim OAS 3.0; retain an explicit optional compatibility lane. |
| #57 | Closed; tagged fix corrected specific path parameters | Preserve the published corrected result, but validate every Glaux path parameter independently. |
| #78, #79, #137 | Closed; bundling was automated after documented recursion/resource failures | Bundle in CI with pinned tools and resource budgets; “bundle generated” is not an acceptance condition. |
| #81, #114 | Closed after reference/example fixes | Retain regression fixtures; closure does not prove the full graph or every example valid. |
| #146, #148 | HTML publication lag fixed; clause-number rendering remains open | Generated HTML needs source/version provenance, link checks, and stable semantic anchors not fragile clause fragments alone. |
| #169 / PR #196 | Issue and clean PR remain open/unmerged | Implement the normative `recursive` behavior and document it from Glaux's accepted query registry; track upstream separately. |
| #170 | Open; PATCH is required only for selected Update classes, exact patch representation remains unsettled | Omit PATCH unless Glaux enables the class and later topics define a complete contract; never advertise a placeholder. |
| #177 | Open; required deployment-scoped stream routes omitted | Include the routes when the corresponding Part 2 classes are enabled. |
| #185 | Open; individual transactions are published while reusable common bulk design is future work | Do not advertise array/bulk mutation as CSAPI interoperability behavior. Mark any later Glaux extension explicitly. |
| #186 | Open; proposes bundled OAS for ReDoc but release bundles retain relative dependencies | Publish a genuinely reference-closed Glaux bundle and verify renderer behavior; do not reproduce the proposed shortcut. |
| #200 | New August 20 issue; official overview ReDoc links return 404 | Link checking and deployment-path tests are release gates. Glaux static and live documentation URLs must be derived from the same manifest. |

No routed issue changed the accepted findings of IDR-SRV-006 through IDR-SRV-013. The new #200 evidence strengthens, rather than changes, the requirement for automated publication-link checks.

## 6. Glaux OpenAPI Contract Scope and Structure

### 6.1 Canonical Version and Compatibility Views

The canonical Glaux OAD is OpenAPI **3.1.2**. Use its base dialect for embedded Schema Objects and preserve explicit Draft 2020-12 declarations on external JSON Schemas. The `openapi` value identifies OAS syntax; `info.version` identifies the Glaux API contract release. Neither is the server software build, standards profile, schema package version, nor deployment identifier.

OAS 3.0.4 is an optional generated compatibility view. It may exist only when:

1. a consumer requirement is recorded;
2. conversion rules and known losses are documented;
3. the view is generated from the canonical registry/OAD, never edited independently;
4. route, parameter, media, response, error, and security parity tests pass;
5. selected clients generate and execute; and
6. any OAS 3.0 conformance declaration passes the applicable OGC ATS.

OpenAPI 3.2.0 is the monitored next line. A version bump requires parser, linter, renderer, bundle, semantic-diff, generator, and live parity evidence and follows the accepted IDR-SRV-010A compatibility process.

### 6.2 One Public Contract, Modular Sources

Each API root publishes one complete public deployment OAD. Internally, source modules may be organized as:

- entry points and conformance;
- collections and shared feature behavior;
- systems, deployments, procedures, sampling features, and properties;
- datastreams, observations, control streams, commands, feasibility, status, and events;
- shared parameters, headers, links, representations, problems, security, and schemas; and
- explicitly named Glaux profiles/extensions.

Composition must fail on duplicate or conflicting paths, operation IDs, parameter keys, component keys, or schema identities. The generated public description contains only enabled and reachable operations/components. A build-time full-profile contract may exist for analysis, but it must be labeled as a design artifact and never exposed as a deployment's `service-desc`.

### 6.3 Registry-First Hybrid Source of Truth

The authoritative input is a typed contract registry, not the emitted YAML. At minimum it records:

- capability/conformance class and maturity;
- route, method, stable operation ID, tags, and summary;
- parameters and serialization rules;
- request/response media maps and schema selectors;
- success and RFC 9457 error outcomes, headers, and links;
- security requirements and public documentation text;
- schema/example identifiers and provenance; and
- requirement/ATS traceability URIs.

Rust routes, request extractors, validators, response metadata, the OAD, conformance output, documentation navigation, and tests should consume or be checked against this registry. Long descriptions, tutorials, and examples remain reviewed source assets referenced by stable IDs. Framework annotations may be a projection mechanism but cannot become an unreviewed competing authority.

### 6.4 Operation and Component Conventions

- Assign every operation a stable, unique lower-camel-case `operationId` based on domain action, not framework function name.
- Use flat, stable domain tags in 3.1. Renderer-specific grouping belongs in renderer configuration, not canonical contract extensions.
- Give every operation a short summary and behavior-focused description; link to the controlling standard and guide where useful.
- Represent every path/query/header parameter once through registry-backed reusable components, including exact style, explode, schema, default, and examples.
- Use operation-specific request and response maps. Do not apply one generic response set to all routes.
- Reference reusable RFC 9457 Problem schemas and typed responses while preserving operation-specific status/header applicability from IDR-SRV-013.
- Use OpenAPI Link Objects only where their expressions are accurate and tested; otherwise document ordinary RFC 8288 links in headers/bodies.
- Use standard fields before extensions. The only baseline Glaux extensions are `x-glaux-requirements` (URI array) and `x-glaux-maturity` (`stable`, `experimental`, or `deprecated`). `deprecated: true` remains authoritative for OAS-aware tools.
- Never embed a digest of the complete document in the document itself. Publish contract fingerprints through `ETag`, `Content-Digest`/manifest metadata, release metadata, and the accepted version registry.

### 6.5 Optional, Conditional, Experimental, and Profile Behavior

| Behavior state | OAD treatment |
|---|---|
| Enabled stable capability | Include normally and trace to declared conformance/profile. |
| Standard capability not implemented | Omit from live OAD and conformance declaration. It may remain in a clearly labeled design/full-profile artifact. |
| Conditional representation or method | Include only when enabled for that deployment and describe the condition precisely. |
| Experimental Glaux extension | Include only behind an explicit feature/profile, set `x-glaux-maturity: experimental`, use non-conflicting paths/media types/relations, and exclude from standards conformance claims. |
| Deprecated behavior | Retain during its announced compatibility window, set `deprecated: true` and `x-glaux-maturity: deprecated`, link migration guidance, and test old clients. |
| Future idea | Exclude. Document in roadmap/design material, not the live OAD. |
| Policy-hidden/classified route | Exclude from that published profile entirely; do not leak it through descriptions, examples, tags, or schemas. |

Do not generate a personalized OAD per ordinary user. That creates cache, fingerprint, client-generation, and support instability. Publish a small number of named deployment/policy profiles only when the security architecture explicitly requires different visible surfaces.

## 7. Schema and Component Strategy

### 7.1 Source Classes

| Schema class | Treatment |
|---|---|
| Approved OGC/SensorML/SWE Common schema | Vendor exact bytes under a versioned third-party tree; record source URL, standard/version, SHA-256, retrieval date, license, and dialect; never silently patch. |
| Published CSAPI OpenAPI-local schema | Treat as artifact evidence and candidate input; repair or wrap only in a Glaux-owned namespace with traceability. |
| Glaux domain schema | Author in Draft 2020-12 with stable absolute `$id`, semantic version/release association, tests, and ownership. |
| OAS-only parameter/response wrapper | Generate as a named component from contract registries. |
| Example | Store by stable ID with provenance and expected schema/media/operation; embed or package during generation. |

### 7.2 Reference and Identity Rules

Glaux-owned external schemas use stable HTTPS `$id` URIs that resolve in connected deployments and map to the offline package. Relative `$ref` values are allowed inside a coherent modular package but must resolve against declared base identities. Vendor schemas retain their original `$schema` and structure. If an upstream schema lacks `$id`, the manifest supplies package identity; Glaux must not change the vendored file merely to add one.

The source graph may reference immutable vendored files. The live deployment serves every referenced resource at a controlled, same-origin or explicitly allowlisted location. The distribution bundle internalizes required external schemas and examples as named components or a compound package while preserving cycles as references. Remote mutable GitHub branches, unversioned CDNs, local filesystem paths, and demo servers are forbidden release dependencies.

### 7.3 Reachability, Recursion, and DDIL

Generation begins from enabled operations and includes only transitively reachable components. It detects reference cycles but does not expand them. Parser and validator configurations use explicit depth, alias, memory, time, and fetch limits. The offline archive contains:

- canonical JSON and YAML entry documents;
- the modular source graph;
- the reference-closed consumer bundle;
- vendored schemas and examples;
- provenance, hashes, licenses, dialects, and tool versions;
- documentation assets and guides; and
- validation results or a reproducible validation command manifest.

This package is the DDIL/test fixture. Runtime correctness must not depend on fetching `schemas.opengis.net`, GitHub, a CDN, or the documentation renderer.

### 7.4 Handoff to IDR-SRV-023

IDR-SRV-023 owns validator selection and the normative resolution of schema/prose contradictions, including reversed array/object flags, missing identities, recursive behavior, format assertion policy, and instance-validation profiles. IDR-SRV-014 fixes the publication constraints: dialects are explicit, inputs are pinned, vendor bytes are preserved, references resolve offline, and validation output is classified rather than silently normalized.

## 8. Human-Readable Documentation and Examples Strategy

### 8.1 Documentation Layers

| Layer | Content | Source/synchronization rule |
|---|---|---|
| OAD descriptions | Operation purpose, parameters, media, responses, errors, security, deprecation, links | Registry and reviewed Markdown fragments; release-blocking parity |
| Generated API reference (`service-doc`) | Navigable operations and schemas for the exact deployment contract | Render the same pinned OAD/build manifest |
| Task guides | Discovery, traversal, ingestion, observation retrieval, commands, feasibility, polling, errors, auth | Versioned Markdown with executable links/examples |
| Standards/profile guide | Claimed classes, optional profiles, Glaux extensions, known limitations | Generated tables plus reviewed explanations |
| Example corpus | Minimal, typical, edge, invalid, and workflow examples | Stable IDs; validate against exact operation/media/schema |
| Release/migration guide | Contract changes, deprecations, replacement paths, compatibility | Semantic diff plus reviewed release notes |

The API reference must serve developers and generators; guides must serve DGIWG/NATO users, open-source implementers, external clients, conformance testers, and operators. Terminology follows the accepted crosswalk and preserves standards identifiers alongside plain language.

### 8.2 Required Scenario Examples

Provide complete examples for:

- landing-page discovery through `service-desc`, `service-doc`, conformance, and data links;
- collection/resource traversal and pagination;
- each enabled query family, including encoding and boundary cases;
- every request and response representation actually supported;
- SensorML, GeoJSON, SWE Common, JSON, binary, and selector-driven cases where enabled;
- Observation creation/retrieval and schema association;
- Command submission, status polling, completion, rejection, conflict, and result retrieval;
- Feasibility request/lifecycle if enabled;
- security challenge/authorization without real secrets;
- each public RFC 9457 problem family and recovery header; and
- deprecated and experimental behavior only in separately labeled fixtures.

Examples must use reserved documentation domains, synthetic identifiers, non-sensitive coordinates/content, and deterministic timestamps where tests require them. Never copy a live token, internal hostname, classified label, real platform location, or production response into the public corpus.

### 8.3 Renderer Decision

The production `service-doc` should be a self-hosted static Redoc CE 3.x build **only after** it passes the representative Glaux corpus, reference-cycle, link, accessibility, content-security-policy, offline, and resource-budget gates. Pin the exact version and assets; disable interactive requests on production unless the security design explicitly enables them.

Swagger UI 5.32+ is the independent local/test renderer and “try it” console because its published compatibility reaches OAS 3.2 and it exercises a different implementation. It must consume the reference-closed bundle because its official documentation notes that relative external-file support is absent. Scalar is the preferred production fallback if the Redoc spike fails; Stoplight Elements remains a comparative candidate. Renderer choice is configuration, not contract policy, so replacement does not change API semantics or `info.version`.

The official CSAPI bundle failures mean the implementation spike must occur early in IDR-SRV-014A/044. A failed spike changes the renderer, bundling projection, or reachable component set—not the accepted server behavior.

## 9. Publishing and Discovery

### 9.1 Live Endpoints

Recommended default paths are:

- `{api_root}/api` — canonical JSON OAS 3.1 `service-desc` for the active deployment;
- `{api_root}/api.yaml` — YAML alternate of the same semantic contract;
- `{api_root}/api.html` — self-hosted HTML `service-doc`;
- `{api_root}/api/releases/{contract_version}/openapi.json` and `.yaml` — immutable release descriptions; and
- `{api_root}/api/releases/{contract_version}/package.zip` — offline package.

The paths are project conventions, not standards obligations; landing-page typed links remain authoritative. The JSON `service-desc` uses the accepted OGC-ecosystem convention `application/vnd.oai.openapi+json;version=3.1`; unlike Common's mandated 3.0 use, that 3.1 token is a Glaux convention and is not currently registered by IANA. The YAML alternate uses the registered `application/yaml` media type from RFC 9512; Glaux may accept the conventional `application/vnd.oai.openapi;version=3.1` as a compatibility alias but must not describe it as IANA-registered. HTML uses `text/html`. Every advertised URI must support GET and return a representation consistent with its declared type.

### 9.2 Server URLs and Deployment Projection

The live OAD contains the externally reachable API root derived from trusted deployment configuration. It must not infer public authority from untrusted forwarding headers or expose internal service names, cluster addresses, test URLs, or credentials. Immutable generic release artifacts may use a documented server variable or omit a deployment-specific server only when OAS permits and guides explain client configuration.

### 9.3 Versioning and Caching

The active alias changes only through a deployment/release action and emits strong or appropriately constructed validators. Immutable release artifacts receive long-lived immutable caching; the active description uses conditional requests and a shorter policy. The manifest records:

- OAS version;
- contract/API version (`info.version`);
- software/build version;
- enabled standards/profile versions;
- schema package/dialects;
- toolchain versions;
- artifact SHA-256 values; and
- build time and source commit.

These values follow IDR-SRV-010A and must not be collapsed into one string.

### 9.4 Static Project Site and Documentation Build

The project site publishes immutable reference contracts, guides, migration notes, and examples. It must not masquerade as a live deployment contract or contain a `servers` value pointing to a demonstration service unless that service is actively tested and supported. Link checking covers HTML anchors, OAD/schema/example references, download links, landing links, and cross-version links. Issue #200 is the regression fixture for wrong deployment-relative ReDoc URLs.

### 9.5 Synchronization

One build invocation creates JSON, YAML, bundle, manifest, reference HTML, generated tables, and validation results. CI compares a clean regeneration with the committed/generated release tree. Runtime startup verifies that compiled routes and enabled capabilities match the embedded/served manifest. A deployment health check follows the public landing-page links and validates content types and fingerprints from outside the trusted proxy boundary.

## 10. Security and Policy Documentation

### 10.1 Security Schemes and Operations

Define reusable OpenAPI security schemes only after IDR-SRV-039 selects mechanisms. Apply a secure root default when most operations share it and explicit operation overrides where public discovery or stronger command authorization differs. Document scheme type, token location, relevant scopes/audiences at an interoperable level, challenge behavior, and the RFC 9457/headers clients can expect. Do not encode internal RBAC rules, policy-engine structure, key locations, bypass paths, or sensitive authorization rationale.

### 10.2 Policy Profiles and Information Exposure

The public OAD is itself information disclosure. Descriptions, examples, schema names, tags, server URLs, extensions, external documentation, and error examples pass the same releasability review as code and guides. Named cross-boundary/profile contracts may exclude non-releasable routes and schemas, but the project must avoid per-user dynamic descriptions. A user receiving `403` for a public contract operation can still see that operation; a route whose existence is sensitive does not appear in that profile.

### 10.3 Interactive Documentation

Production reference pages default to non-interactive rendering. If “try it” is enabled later:

- use same-origin HTTPS and an explicit allowlist;
- prohibit storing or logging bearer credentials in documentation assets;
- apply CSP, frame, referrer, CORS, and CSRF controls appropriate to the deployment;
- require additional confirmation and policy for mutation/command requests;
- prevent generated sample commands from embedding secrets; and
- test logout, token expiry, scope failure, and browser-history behavior.

### 10.4 Untrusted Documents and External Resources

Build tools process OAD text, Markdown, YAML aliases, JSON Schemas, examples, and remote references as untrusted inputs. CI uses pinned dependencies, sandbox/resource limits, no unrestricted network fetch, an allowlisted vendor cache, Markdown/HTML sanitization, dependency scanning, and artifact signing/provenance where the deployment topic adopts it. The upstream recursion failures are security and availability evidence, not only developer inconvenience.

### 10.5 Handoffs

IDR-SRV-039 selects auth schemes and threat controls; 040 selects releasability/profile behavior; 055 turns them into negative/security tests. This report fixes the documentation rule: publish enough interoperable behavior for clients and tests, never confidential enforcement detail, and never advertise a security scheme that runtime does not enforce.

## 11. Validation, Contract Drift, and Generated Clients

### 11.1 Layered Validation Pipeline

| Gate | Required check | Failure meaning |
|---|---|---|
| Parse/structure | Strict JSON and YAML parse; selected OAS 3.1.2 schema/structural rules; unique keys | Artifact cannot be trusted |
| Reference closure | Resolve every `$ref`, example, externalDocs, and link under offline/no-network conditions; preserve legal cycles | Package is incomplete or unsafe |
| Policy lint | Stable operation IDs, tags, summaries, no demo/internal servers, allowed extensions, descriptions, security defaults | Glaux contract-policy defect |
| Schema/example | Validate every example against exact schema/media/direction; validate reusable schemas under declared dialect | Payload documentation defect |
| Registry parity | Compare routes, methods, parameters, media, selectors, headers, errors, links, security, maturity, and conformance both directions | Runtime/OAD drift |
| Semantic diff | Classify breaking, additive, corrective, deprecating, and documentation-only changes | Version/release decision required |
| Bundle/package | Reproducible hashes, manifest, licenses, zero uncontrolled references, bounded resource use | Distribution defect |
| Render/link | Build primary and smoke renderer, load representative deep schemas, crawl links/anchors/assets | Human-doc publication defect |
| Generated client | Generate selected clients, compile, run discovery/read/write/error scenarios against test server | Practical interoperability defect |
| External conformance | Applicable OGC ATS and Glaux supplemental tests | Conformance evidence missing |

Lint output is triaged into structural error, semantic/runtime mismatch, example/schema error, security finding, Glaux policy violation, and advisory style warning. A checked-in, reviewed suppression needs rule ID, exact scope, rationale, owner, and expiry/review trigger.

### 11.2 Contract Drift

The build derives a normalized operation inventory from both the OAD and runtime route registry. It rejects:

- an OAD operation with no reachable route;
- a public route absent from the OAD;
- differences in path parameter names or requiredness;
- undocumented accepted request or emitted response media types/statuses;
- missing security requirements or public overrides;
- conformance classes without complete operation/schema/test evidence;
- undocumented feature-flag changes; and
- stale server URLs, examples, or external documentation.

Runtime telemetry may record an anonymized “undocumented response status/media type” counter in non-sensitive form, but telemetry is a detection aid, not a replacement for deterministic tests.

### 11.3 Semantic Compatibility

Semantic diff operates on resolved canonical models, not line-oriented YAML. Removing or renaming a path/operation/parameter/media/schema member, tightening input constraints, weakening guaranteed output, changing security, or changing operation IDs is breaking unless an accepted compatibility rule says otherwise. Adding an optional field or new operation is normally additive but can still affect closed enums, generated clients, pagination, or security review. Correcting an OAD to match longstanding runtime behavior is a documentation correction and a client-impact event even when the HTTP API did not change.

### 11.4 Generated Clients

OpenAPI Generator's current 3.1 beta label prevents using “generation succeeded” as the validity definition. Select at least two materially different clients after 014A–014G/056—provisionally Rust and TypeScript—for release smoke generation. Pin generator, templates, options, and runtime libraries. Compile generated code and execute representative discovery, query, negotiation, error, Observation, and Command scenarios. Recursive SensorML/SWE types may require documented generic JSON/value escape hatches; such adaptations must not change the wire schema.

No generated client source needs to be committed if reproducible generation and build artifacts are retained. Golden API surface snapshots should detect unstable naming caused by operation/component changes.

## 12. Interoperability and Existing-Implementation Implications

### 12.1 What This Topic Establishes

Existing implementations are evaluated against a fixed rubric rather than copied:

- discovery links resolve and types agree;
- the OAD matches the deployed capability/conformance set;
- Part 1/Part 2/inherited operations compose coherently;
- parameters, representations, errors, security, and dynamic routes are complete;
- schemas/examples resolve offline and under bounded resources;
- stable operation IDs support generators;
- release and live artifacts are distinguishable; and
- rendered documentation is accessible, link-clean, and derived from the same contract.

### 12.2 Deferred Empirical Findings

IDR-SRV-014A through 014G remain necessary. They may reveal framework patterns, client expectations, practical operation naming, renderer behavior, or schema workarounds. They may refine implementation choices but cannot silently weaken the accepted accuracy, closure, security, or parity rules. A conflicting finding is recorded as a delta and returned to the project lead at the next approval boundary.

### 12.3 CSAPI Explorer and External Clients

CSAPI Explorer, OS4CSAPI clients, SECD results, and other clients should consume the public `service-desc` rather than a repository file or standard-part example. Test matrices must record which OAS features and schema constructs each client actually uses. Client-specific compatibility views are generated products with named owners and retirement criteria, not changes to the canonical contract.

## 13. Test-Strategy Implications

### 13.1 Required Test Families

1. **Artifact reproducibility:** regenerate byte/semantic-equivalent JSON, YAML, bundle, archive, HTML, and manifest from a clean checkout.
2. **Strict parsing:** JSON duplicate-key checks, YAML strict mode and alias limits, OAS structural validation.
3. **Reference closure:** online allowlisted and fully offline resolution, cycle detection, missing/redirected/hash-changed targets.
4. **Operation inventory:** route/method/operationId uniqueness and bidirectional runtime parity.
5. **Parameter serialization:** every accepted query/path/header/cookie form, including invalid and default behavior.
6. **Representation maps:** request/response media types, selectors, `406`, `415`, binary schemas, and alternate links.
7. **Error maps:** all operation-applicable RFC 9457 statuses, headers, problem types, concealment, and retry semantics.
8. **Schema/example corpus:** request/read variants, minimal/typical/edge/invalid examples, recursive schemas, and directionality.
9. **Security projection:** public/protected overrides, challenges, scopes, policy profiles, and absence of secrets/internal endpoints.
10. **Conformance projection:** every declared class maps to complete operations, schemas, and tests; undeclared/disabled classes are absent.
11. **Semantic diff/versioning:** breaking/additive/corrective/deprecation cases and old-client regression.
12. **Render and link:** primary plus independent renderer, deep schema navigation, anchors, downloads, CSP/offline/accessibility smoke.
13. **Generated clients:** pinned generation, compile, and live scenarios in selected languages.
14. **Resource exhaustion/security:** cycles, aliases, large examples, deep nesting, external fetch, unsafe Markdown/HTML, and time/memory budgets.
15. **Deployment:** trusted external server URL, proxy behavior, content types, cache validators, immutable release URLs, and outside-in link crawl.

### 13.2 Negative Fixtures from Official Artifacts

Retain immutable tests for residual external references, missing operation IDs/security, required-route omissions, retained removed routes, `recursive` omission, PATCH/bulk ambiguity, response incompleteness, recursive-resource failures, and broken documentation links. These fixtures prove Glaux's pipeline detects known failure classes; they do not assert that every upstream file is globally invalid.

### 13.3 Acceptance Evidence

Each release retains machine-readable validation summaries, tool versions/config hashes, semantic diff, generated-client results, OGC ATS results, link report, and artifact manifest. Raw warning counts alone are not acceptance evidence.

## 14. Downstream Topic Handoff Matrix

| Downstream topic | Fixed input from IDR-SRV-014 | Remaining decision/work |
|---|---|---|
| 014A–014G implementation studies | 3.1.2 combined-contract rubric, closure/parity/render/client gates | Evaluate real implementations and interoperability evidence; report deltas |
| 015 canonical resource model | Stable schema/component identities and one deployment contract | Define canonical resource structures and associations |
| 023 schema validation | Explicit dialects, vendored immutable inputs, offline resolution, cycle/resource constraints | Select validators and resolve schema contradictions |
| 039 authentication/security | Security-scheme projection and no-secret/no-internal-detail boundary | Select mechanisms, threats, scopes, challenges |
| 040 policy/releasability | Named policy-profile model; no per-user OAD baseline | Define cross-boundary visibility and review policy |
| 044 Rust framework | Registry-first generation and renderer/tool spike requirements | Select crates/framework and implement prototypes |
| 046/047 deployment/configuration | Live/immutable/static/offline artifact forms; trusted server URL | Deployment topology, configuration, secrets, publication automation |
| 050 conformance harness | OAD/conformance/runtime parity and conditional OAS 3.0 rule | Implement ATS orchestration and evidence packaging |
| 051 traceability | `x-glaux-requirements`, operation IDs, manifest inputs | Final requirement-to-code/test graph |
| 052 Rust test architecture | Layered gates and deterministic generated projections | Assign unit/component/integration/system test placement |
| 053 fixtures | Stable example IDs, official negative fixtures, offline package | Build and govern scenario corpus |
| 055 security tests | Security projection and interactive-doc safeguards | Implement authorization, abuse, disclosure, browser tests |
| 056 interoperability | Canonical versus compatibility views and selected-client criteria | Choose clients/languages and execute matrix |
| 057 synthesis | P-014 decisions, evidence pins, unresolved upgrade/compatibility triggers | Reconcile later evidence and final dispositions |

## 15. Recommendations

1. Accept the P-014 decision set in Appendix C as the Glaux planning baseline.
2. Standardize the canonical deployment OAD on OpenAPI 3.1.2 and monitor 3.2.0 through release-gated compatibility tests.
3. Withhold Common/Features OAS 3.0 conformance claims unless a separate generated 3.0.4 view passes the complete applicable ATS and parity suite.
4. Implement one complete capability-filtered OAD per API root; retain domain modularity only as source/build organization.
5. Adopt the registry-first hybrid workflow and prohibit independent hand edits to emitted release OADs.
6. Publish canonical JSON, YAML alternate, self-hosted HTML, immutable releases, and a reference-closed offline package from one build manifest.
7. Vendor and hash exact approved external schemas; give Glaux-owned schemas stable `$id` values; never mutate vendor bytes silently.
8. Preserve cycles as references, include only reachable components, and enforce resource budgets and network allowlists in all tools.
9. Assign stable operation IDs and require complete operation-specific parameters, media, errors, headers, links, security, maturity, and requirements traceability.
10. Use only `x-glaux-requirements` and `x-glaux-maturity` as baseline contract extensions; keep renderer presentation outside the canonical OAD.
11. Provisionally use pinned self-hosted Redoc CE 3.x for production reference docs after corpus acceptance; use Swagger UI 5.32+ as the independent test/dev renderer and Scalar as fallback.
12. Disable production interactive mutation by default and subject all OAD/document assets to security and releasability review.
13. Make strict parsing, closure, registry/runtime parity, semantic diff, example/schema validation, render/link, selected client generation, and applicable OGC ATS release-blocking.
14. Retain official CSAPI defects and issue #200 as immutable negative/regression fixtures.
15. Begin IDR-SRV-014A only after plan-owner acceptance; do not start it as part of this report review.

## 16. Risks, Constraints, and Open Questions

### 16.1 Material Risks

| Risk | Consequence | Mitigation |
|---|---|---|
| Recursive/large SensorML and SWE graphs exhaust tools | Build, docs, or clients fail | Reachability pruning, preserved refs, budgets, two tools, offline tests |
| Code, registry, and prose become three authorities | Silent runtime/OAD drift | Typed registry, generation, bidirectional parity, reviewed prose IDs |
| Optional behavior is advertised globally | False conformance/client failures | Capability-filter live projection and full-profile separation |
| OAS 3.0 compatibility conversion loses 2020-12 meaning | Incorrect schemas/clients | No initial claim; explicit loss/parity suite and named consumer trigger |
| Renderer-specific extensions leak into contract | Vendor lock and inconsistent views | Minimal extensions and external renderer config |
| Dynamic/personalized OAD fragments contract identity | Cache/client/support explosion | Few named policy profiles; no ordinary per-user generation |
| Public docs expose sensitive details or enable dangerous calls | Security/policy harm | Releasability review, non-interactive production default, security tests |
| Published links or assets move | Broken discovery like #200 | Immutable URLs, same manifest, outside-in link crawl |
| Linter warning volume masks real defects | False confidence or alert fatigue | Severity taxonomy and scoped reviewed suppressions |

### 16.2 Constraints

- Approved standards and accepted reports control behavior; the OAD documents rather than invents it.
- The canonical contract must function connected and offline/DDIL.
- The server is a future Rust reference implementation, but this topic remains framework-neutral.
- OGC-owned artifacts and links may change outside Glaux control; releases therefore pin and vendor dependencies.
- Generated clients cannot represent every recursive/polymorphic schema equally; interoperability evidence determines supported client profiles.

### 16.3 Open Questions with Owners

| Question | Owner / decision trigger |
|---|---|
| Which Rust OpenAPI/route metadata approach best implements the registry-first model? | IDR-SRV-044 prototype |
| Does the representative Glaux schema graph pass Redoc CE within budgets, or should Scalar become primary? | 014A/044 renderer spike |
| Which two generated-client languages and generators become release gates? | 014A–014G and 056 |
| Is a named consumer willing to fund and test OAS 3.0.4 compatibility? | Product/interoperability evidence; 050/056 |
| When does OAS 3.2 tool support meet Glaux's adoption threshold? | Each contract release under 010A/057 |
| Which routes/schemas require distinct releasability profiles? | 039/040 |
| How are exact OGC schema contradictions resolved without altering vendor bytes? | 023 |

None blocks the next research topic.

## 17. Validation Against the Research Plan

### 17.1 Methodology Phases

| Phase | Evidence of completion |
|---|---|
| 1 — sources/framework | Standards, prior reports, register, OAS/tool sources, authority labels, matrix fields established |
| 2 — official artifacts | Tag, ZIPs, bundles, hashes, inventories, equality, references, lint/parse/render behavior, and routed history audited |
| 3 — Glaux contract scope | Version, composition, registry-first workflow, operations, optionality, components, extensions, and compatibility defined |
| 4 — documentation/examples | Audiences, layers, scenario corpus, renderer policy, and stale/sensitive-content controls defined |
| 5 — publishing/validation/security/tests | Live/release/offline publication, security, layered gates, semantic diff, clients, and handoffs defined |
| 6 — synthesis | Recommendations, risks, question ledger, decisions, register/plan updates, and handoff prepared |

### 17.2 Success Criteria

- [x] Modular, tag, ZIP, and bundled artifacts compared with immutable pins and digests.
- [x] Routed issue/PR/commit/release history refreshed and authority-classified.
- [x] OpenAPI requirements and conventions identified.
- [x] OAD/runtime/schema/example/human-documentation relationships documented.
- [x] Schema reference and component reuse implications identified.
- [x] Query, representation, error, security, conformance, and dynamic-data needs mapped.
- [x] Publishing, versioning, deprecation, and synchronization documented.
- [x] Validation, drift, generated-client, conformance, and interoperability implications identified.
- [x] Recommendations are decision-usable and bounded to Glaux Server.
- [x] References and reproducibility anchors are explicit.

### 17.3 Required Content and Matrix Fields

All eighteen required sections are present. Section 4 and Appendix B contain artifact provenance/version/digest and issue disposition. Sections 6–13 and the following paired matrix cover all thirteen required fields.

| Area ID | Documentation/contract area | Related server behavior | Source anchor | OAD representation | Schema/component/artifact | Variant/provenance/authority |
|---|---|---|---|---|---|---|
| M1 | Discovery/definition | Landing links and GET | CSAPI P1 8.3; Common core | Complete JSON `service-desc`, YAML alternate, HTML `service-doc` | Entry OAD and manifest | Live generated + immutable release; N/I/P |
| M2 | Conformance/capabilities | Enabled classes only | IDR-008/009 | Capability-filtered operations plus requirements URIs | Capability registry | Deployment projection; P |
| M3 | Routes/query | All public operations/parameters | IDR-010/011; CSAPI/ATS | Stable operation IDs and exact parameter serialization | Route/parameter registries | Generated; N/I/P |
| M4 | Representations | Request/response media and selectors | IDR-012 | Operation-specific content maps and headers | Representation registry/schemas | Generated; N/I/P |
| M5 | Errors | Status, headers, RFC 9457 | IDR-013 | Reusable Problem plus operation-specific responses | Error registry | Generated; I/P |
| M6 | Dynamic/tasking | Streams, observations, commands, feasibility, events | CSAPI P2; IDR-007 | Include enabled required routes/lifecycles only | Domain modules | Combined deployment OAD; N/P |
| M7 | Security/policy | Authentication, authorization, visibility | CSAPI 7.10; future 039/040 | Reusable schemes and operation overrides; named profiles | Security registry | Generated after security decisions; N/P/U |
| M8 | Schemas/examples | Payload structure and illustrations | OGC schemas; OAS 3.1.2 | Pinned refs, stable Glaux `$id`, validated examples | Vendor tree + Glaux tree | Hash-manifested; I/A/P |
| M9 | Human docs | Reference and task guidance | Common links; tool docs | Renderer-neutral OAD + guides | HTML/build assets | Pinned generated view; P |
| M10 | Version/deprecation | Contract evolution | IDR-010A | `info.version`, `deprecated`, maturity extension | Version registry/manifest | Live + immutable releases; P |

| Area ID | Upstream history/disposition | Human documentation need | Generated-client implication | Validation implication | Test implication | Downstream handoff / unresolved notes |
|---|---|---|---|---|---|---|
| M1 | #12/#13 examples; #200 broken links | Explain canonical/alternate/release forms | One discovery entry | Link/type/fingerprint parity | Outside-in crawl | 046/050; paths conventional |
| M2 | #48/#77 version distinction | Explain claims versus available behavior | Avoid unsupported methods | Conformance/OAD/runtime parity | ATS + supplemental | 050/051 |
| M3 | #57/#169/#170/#177 | Exact required/optional routes and query use | Stable method names/signatures | Route/parameter diff | Positive/negative inventory | 014A–G/015/052 |
| M4 | #114/#185 plus IDR-012 defects | Media examples and selector rules | Correct encoders/decoders | Example/media/schema parity | `406`/`415` and round trip | 023/053/056 |
| M5 | IDR-013 mechanical defects | Problem catalog/recovery | Typed exceptions/results | Status/header/body map | Every problem family | 050/052/055 |
| M6 | #177 omissions; #170/#185 unresolved | Workflow diagrams and lifecycle examples | Async/status polling behavior | Capability and route closure | Command/feasibility scenarios | 031–038/056 |
| M7 | OAS files omit security | Safe auth guide, no enforcement internals | Auth injection/config | Scheme/runtime parity and releasability | 401/403/concealment/browser | 039/040/055 |
| M8 | #78/#79/#81/#137/#186 recursion/closure | Dialect/provenance and example meaning | Recursive/polymorphic limits | Strict/offline/budgeted validation | Valid/invalid/cycle fixtures | 023/053; generator adaptation open |
| M9 | #146/#148/#186/#200 | Version banner, guides, accessibility | None directly; code samples help | Renderer/link/CSP/offline | Two-renderer smoke | 014A/044/046 |
| M10 | #48/#77 and current tool maturity | Migration and support policy | Old-client regression | Semantic diff and compatibility view | Old/new client matrix | 010A/056/057; 3.2 trigger open |

### 17.4 Research-Question Validation

Appendix A maps each detailed question group to a substantive report section. No question is left unanswered; later-owner questions have a concrete handoff and trigger rather than an implicit deferral.

## 18. References

### 18.1 Controlling and Inherited Standards

- [OGC API — Connected Systems — Part 1: Feature Resources 1.0](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API — Connected Systems — Part 2: Dynamic Data 1.0](https://docs.ogc.org/is/23-002/23-002.html)
- [OGC API — Common — Part 1: Core](https://docs.ogc.org/is/19-072/19-072.html)
- [OGC API — Features — Part 1: Core corrigendum](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [OGC SensorML Encoding Standard 3.0](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC SWE Common Data Model Encoding Standard 3.0](https://docs.ogc.org/is/24-014/24-014.html)
- [OpenAPI Specification 3.1.2](https://spec.openapis.org/oas/v3.1.2.html)
- [OpenAPI Specification 3.2.0](https://spec.openapis.org/oas/v3.2.0.html)
- [JSON Schema Draft 2020-12](https://json-schema.org/draft/2020-12)
- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)
- [RFC 8288 — Web Linking](https://www.rfc-editor.org/rfc/rfc8288)
- [RFC 9457 — Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457)
- [IANA Media Types Registry](https://www.iana.org/assignments/media-types/media-types.xhtml)

### 18.2 Official CSAPI Artifacts and History

- [`v1.0.0` source and release](https://github.com/opengeospatial/ogcapi-connected-systems/releases/tag/v1.0.0)
- [Published Part 1 package](https://schemas.opengis.net/ogcapi/connected-systems/part1/1.0/)
- [Published Part 2 package](https://schemas.opengis.net/ogcapi/connected-systems/part2/1.0/)
- [Part 1 bundled OAS 3.1](https://github.com/opengeospatial/ogcapi-connected-systems/releases/download/v1.0.0/ogcapi-connectedsystems-1.bundled.oas31.yaml)
- [Part 2 bundled OAS 3.1](https://github.com/opengeospatial/ogcapi-connected-systems/releases/download/v1.0.0/ogcapi-connectedsystems-2.bundled.oas31.yaml)
- [Official issue tracker](https://github.com/opengeospatial/ogcapi-connected-systems/issues)
- [Official pull requests](https://github.com/opengeospatial/ogcapi-connected-systems/pulls)
- [New broken-ReDoc-link issue #200](https://github.com/opengeospatial/ogcapi-connected-systems/issues/200)

### 18.3 Tooling Sources

- [Redoc CE 3.x documentation](https://redocly.com/docs/redoc/v3.x)
- [Redocly structural-rule documentation](https://redocly.com/docs/cli/rules/common/struct)
- [Swagger UI compatibility](https://github.com/swagger-api/swagger-ui/blob/main/README.md)
- [Scalar OpenAPI parser](https://github.com/scalar/scalar/blob/main/packages/openapi-parser/README.md)
- [Scalar configuration](https://github.com/scalar/scalar/blob/main/documentation/configuration.md)
- [Stoplight Elements](https://stoplight.io/open-source/elements)
- [OpenAPI Generator compatibility](https://github.com/OpenAPITools/openapi-generator)

### 18.4 Project Sources

- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)
- [IDR-SRV-014 Research Plan](../IDR%20Plans/idr-srv-014-openapi-description-and-api-documentation-strategy.md)
- [Shared upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md)
- [Glaux Server goal and definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- Accepted IDR-SRV-006 through IDR-SRV-013 reports in this directory

## Appendix A. Detailed Question Ledger

| Question group | Resolution |
|---|---|
| Standards and documentation sources | Sections 3–5 separate CSAPI/Common obligations, OAS rules, immutable artifacts, history, and tool evidence. |
| Official version and artifact structure | Sections 4.1–4.4 record 3.1.0/0.0.1 roots, tag/package equality, bundle hashes/references, inventory, defects, and tool behavior. |
| Artifact reuse | Sections 4–7 classify official files as reference/negative fixtures and require a Glaux-generated implementation contract. |
| One versus multiple descriptions | Section 6 requires one complete public deployment OAD with modular sources and separately labeled design/compatibility artifacts. |
| Complete behavior mapping | Sections 6.3–6.5 and matrix M1–M7 cover entry points, collections, resources, queries, media, schemas, errors, streams, commands, feasibility, status, events, and links. |
| Optional/conditional/experimental/profile behavior | Section 6.5 defines inclusion and marking rules and prohibits future placeholders. |
| Schema reference/reuse/version/offline | Section 7 defines vendor, Glaux, wrapper, example, `$id`, dialect, closure, cycle, and DDIL rules; 023 owns validator resolution. |
| Human documentation | Section 8 defines audiences, layers, guides, examples, renderer choice, stale/sensitive controls, and standards/profile explanations. |
| Publishing/discovery | Section 9 defines paths, link types, live/release/static/offline forms, servers, caching, fingerprints, and synchronization. |
| Security/authorization/policy | Section 10 defines scheme projection, named policy profiles, interactive controls, releasability, and handoffs to 039/040/055. |
| Validation/testing/interoperability | Sections 11–13 define layered validation, bidirectional drift, semantic diff, generated clients, official negative fixtures, and release evidence. |
| Existing implementations | Section 12 fixes the rubric and defers empirical findings to 014A–014G without weakening the baseline. |

## Appendix B. Reproducible Artifact and Tool Audit

### B.1 Immutable Pins

| Item | Value |
|---|---|
| Research date | 2026-08-31 |
| Official tag commit | `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2` |
| Current `master` | `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f` |
| Part 1 ZIP hash | `43FCAB8FB079B153E1DC01559C9395A51E8FAB0E7C87C709FDAE6A28B1983F12` |
| Part 2 ZIP hash | `02ACCC4DD11A197F029A9A65D6E9EB3724EF3E7DC8A7E6E82BC05504844100A9` |
| Part 1 bundle hash | `69DA631D5D05F01716381CCA7B7EE6311402F2752A8FD79A9B72B663539555AA` |
| Part 2 bundle hash | `86ED005F9E7CF176264D6DEB72581A0B521A227CD7A198B6CB1BD32B39D83667` |
| Probe environment | Windows; Node.js 26.8.1; Redocly CLI 2.43.2 |

### B.2 Comparison Procedure

1. Clone the official repository and checkout `v1.0.0`.
2. Download the two published ZIPs and the two release assets from their official HTTPS locations.
3. Verify SHA-256 and file counts.
4. Compare each published `openapi/` relative path with its tagged counterpart after normalizing CRLF/LF only.
5. Inventory root paths/methods, metadata, operation IDs, security, components, and external references.
6. Run pinned Redocly lint and static-build probes on both roots and both bundles.
7. Run an independent generic YAML conversion to expose parser/alias behavior.
8. Refresh routed issue/PR/release/master facts through the official GitHub API.

### B.3 Mechanical Results

- Part 1: 152 published OAS files equal tag; published package substitutes a package README for the tag's `openapi/README.md`.
- Part 2: 159 published OAS files equal tag; package additionally contains five AsyncAPI files and a package README.
- Roots: 20 + 23 paths; 39 + 48 operations; zero operation IDs and zero operation-level security requirements.
- Part 1 bundle: 32 residual relative example references.
- Part 2 bundle: 51 residual relative references (45 examples, 6 schemas).
- Redocly lint: 45/235 and 59/517 errors/warnings for modular roots; 44/299 for Part 1 bundle; Part 2 bundle stack overflow.
- Redocly static build: exit 134 for all four targets in the probe environment.
- Generic YAML conversion: diagnostics on generated alias-heavy structures; Part 2 aborted at alias resource-exhaustion protection.

These commands and counts are evidence snapshots. Future reports should pin tools and compare deltas rather than assuming counts remain stable.

### B.4 Mutable Refresh

As of August 31, 2026:

- official `master` remains `3fd86c73`;
- the release asset sizes and digests are unchanged;
- routed issue states are unchanged from register Version 1.7;
- PR #196 remains open, non-draft, clean/mergeable, and unmerged at head `e8e0cb4e3825aa83d72414d8ff8c3db8beb5ab88`;
- one new issue, #200, directly affects documentation publication and is added to the register; and
- the complete public counts are 142 issues and 58 pull requests.

## Appendix C. Proposed Decision Register

| ID | Proposed decision | Status pending plan-owner review |
|---|---|---|
| P-014-001 | Canonical Glaux OAD uses OpenAPI 3.1.2. | Proposed |
| P-014-002 | OAS 3.2 is monitored and adopted only through release-gated tool/runtime/client evidence. | Proposed |
| P-014-003 | Glaux initially makes no Common/Features OAS 3.0 conformance claim. | Proposed |
| P-014-004 | Any OAS 3.0.4 view is generated, consumer-driven, parity-tested, and non-authoritative. | Proposed |
| P-014-005 | Each API root publishes one complete capability-filtered deployment OAD; modularity is internal/source organization. | Proposed |
| P-014-006 | A typed contract registry drives or verifies routes, OAD, conformance, docs, and tests. | Proposed |
| P-014-007 | Canonical release OAD output is generated and not independently hand-edited. | Proposed |
| P-014-008 | Publish JSON `service-desc`, YAML alternate, HTML `service-doc`, immutable releases, and an offline package from one manifest. | Proposed |
| P-014-009 | Distribution bundles close required dependencies while preserving cycles as references. | Proposed |
| P-014-010 | Approved external schemas are vendored byte-for-byte with provenance/hashes; Glaux schemas use stable `$id`. | Proposed |
| P-014-011 | Tools are pinned, network-allowlisted, resource-bounded, and independently cross-checked. | Proposed |
| P-014-012 | Every operation has a stable unique `operationId` and complete operation-specific contract metadata. | Proposed |
| P-014-013 | Baseline custom extensions are limited to `x-glaux-requirements` and `x-glaux-maturity`. | Proposed |
| P-014-014 | Future/unimplemented behavior is absent; experimental and deprecated behavior follows explicit marking/profile rules. | Proposed |
| P-014-015 | Production docs provisionally use pinned self-hosted Redoc CE 3.x after corpus acceptance; Swagger UI is the independent test/dev renderer; Scalar is fallback. | Proposed |
| P-014-016 | Production interactive mutation is disabled by default and subject to later security authorization. | Proposed |
| P-014-017 | Strict parsing, reference closure, bidirectional runtime parity, semantic diff, schema/examples, render/link, clients, and ATS are layered release gates. | Proposed |
| P-014-018 | No ordinary per-user OAD is generated; materially different visibility uses a small set of named policy profiles. | Proposed |
| P-014-019 | Official CSAPI artifacts and #200 are retained as provenance and negative/regression fixtures, not copied as Glaux contracts. | Proposed |
| P-014-020 | IDR-SRV-014A begins only after explicit plan-owner acceptance of this report. | Proposed |

## Appendix D. Completion and Handoff

All six research phases, all ten success criteria, all required content areas, all five core questions, all thirty-six detailed questions, and all thirteen matrix fields are complete. The report, plan checklist, overall progress table, and upstream-history register have been prepared for project-lead review.

**Acceptance state:** Pending Glaux Project Lead review.

**Next authorized topic after acceptance:** `IDR-SRV-014A — OSH CSAPI Server Implementation Study`.

**Iteration boundary:** Do not begin IDR-SRV-014A until the Glaux Project Lead explicitly accepts IDR-SRV-014 and says `proceed`.

# Section 010A: API Versioning, Backward Compatibility, and Deprecation Strategy - Research Plan

**Status:** In Progress<br>
**Last Updated:** August 1, 2026<br>
**Estimated Research Time:** 11-15 hours<br>
**Actual Research Time:** Approximately 5 hours of AI-assisted elapsed execution time on August 1, 2026<br>
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-010a-api-versioning-backward-compatibility-and-deprecation-strategy-report.md`

---

## Usage Instructions

Before executing this plan, review the full exemplar corpus and align the report to the proven standards for content nature, type, level, and detail:

- Full research-plans corpus:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans
- Early exemplar (blueprint-first depth):
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/01-pr114-blueprint-analysis.md
- Mid-stream exemplar (inventory + sourcing rigor):
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/15-fixture-sourcing-organization.md
- End-state synthesis exemplar:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/38-testing-playbook-synthesis.md

This plan is intentionally limited to research planning. It does not execute the research and does not produce the research report.

---

## 1. Research Objective

This topic research must define the Glaux Server planning baseline for **API versioning, backward compatibility, and deprecation strategy**.

The research must answer:

- How should Glaux Server version its API surface while remaining aligned with OGC API - Connected Systems Part 1, OGC API - Connected Systems Part 2, inherited OGC API behavior, and AEP-4789 adoption context?
- What constitutes a breaking change for Glaux Server API behavior, resource representations, links, schemas, OpenAPI descriptions, query parameters, media types, conformance declarations, and dynamic-data/tasking behavior?
- How should Glaux Server preserve backward compatibility for clients, external tools, conformance tests, and future ecosystem components?
- How should Glaux Server communicate deprecation, retirement, experimental behavior, staged conformance, and future evolution without misleading clients?
- What versioning and deprecation practices are supported by current authoritative API guidance, HTTP standards, OpenAPI guidance, and observed OGC/CSAPI implementation patterns?

The output must be an API versioning, compatibility, and deprecation baseline with source anchors, Glaux Server policy recommendations, downstream handoffs, and test implications.

### Why This Topic Order

This topic follows `IDR-SRV-010: Collections, Resources, Links, and Navigation Behavior`.

`IDR-SRV-009` establishes API entry-point behavior, and `IDR-SRV-010` establishes resource/navigation behavior. This topic then researches how those external contracts should evolve over time without breaking clients or invalidating conformance claims.

This topic must precede query behavior, content negotiation, error handling, OpenAPI documentation strategy, canonical resource modeling, identifier strategy, test-strategy planning, and interoperability testing because API evolution rules affect all externally visible server behavior.

### Critical Constraint(s)

- Treat OGC API - Connected Systems, OGC API - Features, HTTP, OpenAPI, and adopted standards-package behavior as controlling where they define or constrain API surface behavior.
- Do not invent project-specific versioning mechanisms that conflict with OGC API conventions or CSAPI resource patterns.
- Do not use versioning to avoid conformance obligations.
- Do not use deprecation labels to hide incomplete implementation.
- Clearly distinguish:
  - standards version,
  - server software version,
  - API contract version,
  - resource representation version,
  - schema version,
  - OpenAPI document version,
  - media type/profile version,
  - experimental or extension behavior,
  - conformance declaration status.
- Keep the research bounded to Glaux Server behavior and server-side contracts.
- Do not design release-management operations, product-roadmap governance, or enterprise change-management processes beyond what is needed for server API compatibility.
- Distinguish the published `v1.0.0` baseline from pre-publication rationale, post-publication merged maintenance, and unresolved upstream proposals. Repository activity does not silently revise the standard or Glaux compatibility contract.

---

## 2. Research Questions

### Core Questions

1. What API versioning strategy should Glaux Server use for standards-aligned CSAPI behavior?
2. What changes should be treated as breaking, non-breaking, additive, deprecated, experimental, or implementation-specific?
3. How should Glaux Server preserve backward compatibility while standards, schemas, implementation capabilities, and client expectations evolve?
4. How should Glaux Server communicate deprecation, retirement, staged conformance, and experimental behavior?
5. What test, documentation, interoperability, and implementation implications follow from the chosen versioning and deprecation baseline?

### Detailed Questions

#### Standards and Versioning Context

- How do CSAPI Part 1 and Part 2 identify their own version, requirement classes, conformance classes, schemas, and OpenAPI artifacts?
- Does CSAPI define or imply how servers should expose standards version, API version, or schema version?
- How does OGC API - Features or common OGC API practice handle API definitions, conformance declarations, versioned standards, and evolving APIs?
- How should Glaux Server distinguish implementation version from standards version?
- How should AEP-4789 adoption context affect versioning and compatibility expectations?
- What do official CSAPI releases, tags, issue resolutions, linked pull requests, and commit history establish about how the standard and its supporting artifacts have evolved, and which changes are or are not part of the published baseline?

#### API Versioning Strategy

- Should Glaux Server use URI path versioning, media-type/profile versioning, OpenAPI document versioning, semantic server release versioning, or another approach?
- Which versioning approaches are compatible or incompatible with OGC API and CSAPI conventions?
- How should versioning interact with landing page links, collections, item URLs, conformance endpoints, API definitions, and schemas?
- Should Glaux Server expose one versioned API surface, multiple concurrent API versions, or one standards-aligned surface with versioned server software?
- What evidence supports the recommended approach?

#### Backward Compatibility and Breaking Changes

- What constitutes a breaking change for Glaux Server clients?
- Which changes are likely non-breaking, such as additive links, optional fields, new collections, new conformance classes, or new query parameters?
- Which changes are likely breaking, such as removing fields, changing identifiers, changing URI patterns, changing semantics, changing required properties, or altering command lifecycle behavior?
- How should backward compatibility apply to:
  - landing page behavior,
  - conformance declarations,
  - collections and resource links,
  - schemas and representations,
  - query parameters,
  - media types and content negotiation,
  - OpenAPI descriptions,
  - datastreams and observations,
  - control streams and commands,
  - status/events,
  - error responses?
- Which compatibility findings should be handed to downstream resource-model, validation, and testing topics?

#### Deprecation and Retirement Behavior

- How should Glaux Server communicate deprecated endpoints, fields, links, query parameters, schemas, conformance classes, or implementation-specific extensions?
- What role should documentation, response headers, links, OpenAPI descriptions, changelogs, or conformance metadata play in deprecation communication?
- What deprecation and retirement windows should be researched or recommended for open-source reference-server planning?
- How should clients discover replacement resources or successor behavior?
- How should Glaux Server avoid misleading clients when features are planned, experimental, staged, or not yet conformant?

#### Experimental, Extension, and Profile Behavior

- How should Glaux Server represent experimental extensions without polluting the standards-conformant API surface?
- How should implementation-specific resources, extensions, or profiles be named, linked, documented, and tested?
- How should extension behavior avoid conflict with future CSAPI, SensorML, SWE Common, or AEP updates?
- How should profile-specific behavior be separated from core server behavior?
- Which findings should be handed to OpenAPI documentation, schema validation, and interoperability testing topics?

#### Documentation, OpenAPI, and Client Implications

- How should API versions, server software versions, schemas, conformance claims, and deprecations appear in the OpenAPI description?
- How should documentation distinguish stable, experimental, deprecated, and removed behavior?
- How should generated clients handle versioned or evolving API contracts?
- Which documentation questions should be handed to `IDR-SRV-014`?
- Which compatibility concerns should be tested against CSAPI Explorer, OS4CSAPI client work, and other external clients?

#### Testing and Verification

- What tests are needed to protect backward compatibility?
- What tests are needed for deprecation headers, deprecation metadata, replacement links, and documentation consistency?
- What tests are needed to detect accidental breaking changes in OpenAPI descriptions, schemas, response shapes, link relations, and error responses?
- What golden-file, contract-test, snapshot-test, and external-client test strategies are implied?
- Which findings should be handed to `IDR-SRV-050`, `IDR-SRV-051`, `IDR-SRV-053`, and `IDR-SRV-056`?

---

## 3. Primary Resources

The future research report must analyze these sources directly.

### Project and Governance Sources

- Glaux Server Overall IDR Research Plan:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Creation Prompt:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-creation-prompt.md
- Research Plan Template:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Overall Research Report Template:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/overall-research-report-template.md
- Research Planning Approach:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md

### Prior IDR Topic Sources

- `IDR-SRV-006` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-006-csapi-part-1-requirement-baseline-report.md`
- `IDR-SRV-007` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-007-csapi-part-2-requirement-baseline-report.md`
- `IDR-SRV-008` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-008-conformance-class-and-requirement-mapping-report.md`
- `IDR-SRV-009` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-009-landing-page-api-definition-and-conformance-declaration-behavior-report.md`
- `IDR-SRV-010` research report, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-010-collections-resources-links-and-navigation-behavior-report.md`
- Shared OGC API - Connected Systems upstream-history register:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Evidence/ogc-connected-systems-upstream-history-register.md`
- Earlier baseline reports, if needed for scope and terminology:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-001-stanag-4789-aep-4789-server-obligation-baseline-report.md`
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-002-aep-4789-volume-i-functional-mapping-to-server-responsibilities-report.md`
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-003-aep-4789-volume-ii-standards-package-implementation-baseline-report.md`
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-004-terminology-and-concept-crosswalk-report.md`
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-005-related-nato-standards-boundary-review-report.md`

### Controlling OGC and API Sources

- OGC API - Connected Systems landing page:
  - https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources:
  - https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data:
  - https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository:
  - https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems releases, tags, issues, and pull requests:
  - https://github.com/opengeospatial/ogcapi-connected-systems/releases
  - https://github.com/opengeospatial/ogcapi-connected-systems/tags
  - https://github.com/opengeospatial/ogcapi-connected-systems/issues
  - https://github.com/opengeospatial/ogcapi-connected-systems/pulls
- OGC API - Connected Systems OpenAPI artifacts:
  - https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api
- OGC schemas:
  - https://schemas.opengis.net/
- OGC API - Features Part 1 standard:
  - https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC API standards family:
  - https://ogcapi.ogc.org/
- OpenAPI Specification:
  - https://spec.openapis.org/oas/latest.html
- OpenAPI Initiative:
  - https://www.openapis.org/
- Semantic Versioning:
  - https://semver.org/

### HTTP, Deprecation, and Web Compatibility Sources

Use these for HTTP-level compatibility, cache, header, link, and deprecation behavior:

- RFC 9110 - HTTP Semantics:
  - https://www.rfc-editor.org/rfc/rfc9110
- RFC 9111 - HTTP Caching:
  - https://www.rfc-editor.org/rfc/rfc9111
- RFC 8288 - Web Linking:
  - https://www.rfc-editor.org/rfc/rfc8288
- RFC 8594 - The Sunset HTTP Header Field:
  - https://www.rfc-editor.org/rfc/rfc8594
- IANA HTTP Field Name Registry:
  - https://www.iana.org/assignments/http-fields/http-fields.xhtml
- IANA Link Relation Types:
  - https://www.iana.org/assignments/link-relations/link-relations.xhtml

### API Design Guideline Sources

Use these to compare current API versioning and deprecation best practices. Treat them as supporting guidance, not as standards that override OGC/CSAPI requirements.

- Microsoft REST API Guidelines:
  - https://github.com/microsoft/api-guidelines
- Microsoft Azure REST API Guidelines:
  - https://github.com/microsoft/api-guidelines/blob/vNext/azure/Guidelines.md
- Google API Improvement Proposals:
  - https://google.aip.dev/
- Google API Improvement Proposal AIP-181: Stability levels:
  - https://google.aip.dev/181
- Google API Improvement Proposal AIP-180: Backwards compatibility:
  - https://google.aip.dev/180
- Zalando RESTful API Guidelines:
  - https://opensource.zalando.com/restful-api-guidelines/
- Heroku HTTP API Design Guide:
  - https://github.com/interagent/http-api-design

### Existing Implementation and Interoperability Context

Use these as supporting evidence once the corresponding focused studies are available:

- `IDR-SRV-014A through IDR-SRV-014G`
- OS4CSAPI organization:
  - https://github.com/OS4CSAPI
- OS4CSAPI client work:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability findings repository:
  - https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer:
  - https://ogc-csapi-explorer.pages.dev/

---

## 4. Supporting Resources

Use these sources to interpret project context, downstream dependencies, expected research depth, and report style.

- DGIWG P5.07 Glaux main repository:
  - https://github.com/DGIWG-P507/glaux
- Glaux project website:
  - https://dgiwg-p507.github.io/glaux/
- DGIWG P5.07 GitHub organization:
  - https://github.com/DGIWG-P507
- Glaux Server repository, if available or created:
  - https://github.com/DGIWG-P507/glaux-server
- OS4CSAPI testing research-plan corpus:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans
- Blueprint-first exemplar:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/01-pr114-blueprint-analysis.md
- Inventory/sourcing rigor exemplar:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/15-fixture-sourcing-organization.md
- End-state synthesis exemplar:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/38-testing-playbook-synthesis.md
- OS4CSAPI discussions, for background only:
  - https://github.com/orgs/OS4CSAPI/discussions

---

## 5. Research Methodology

### Phase 1: Source Collection and Versioning Vocabulary Setup (1.5-2 hours)

**Objective:** Establish the evidence base and define the vocabulary for API versioning, compatibility, and deprecation analysis.

**Tasks:**

1. Gather CSAPI, OGC API - Features, OpenAPI, HTTP, deprecation, and API-design guideline sources listed in this plan.
2. Gather reports from `IDR-SRV-006` through `IDR-SRV-010`, if available.
3. Consult the shared upstream-history register and refresh only entries relevant to standards releases, artifact evolution, compatibility, deprecation, or corrigenda.
4. Identify where CSAPI, OGC API, schemas, and OpenAPI artifacts expose standards version, API version, schema version, conformance class, or implementation-version information.
5. Define terms for the report:
   - standards version,
   - server software version,
   - API contract version,
   - representation/schema version,
   - media type/profile version,
   - deprecation,
   - sunset/retirement,
   - breaking change,
   - non-breaking change,
   - experimental behavior.
6. Define the evaluation matrix fields for versioning and compatibility findings.

**Expected Output:** Source inventory and versioning/compatibility vocabulary framework.

### Phase 2: OGC / CSAPI Versioning and Evolution Constraint Review (2-3 hours)

**Objective:** Identify versioning and compatibility constraints from OGC API - Connected Systems, OGC API - Features, schemas, and OpenAPI artifacts.

**Tasks:**

1. Review CSAPI Part 1 and Part 2 for version, conformance, schema, and extension/evolution language.
2. Review OGC API - Features for API entry-point, collections, conformance, and API definition behavior that may constrain versioning choices.
3. Review OpenAPI artifacts for explicit version fields, server URLs, schema identifiers, descriptions, tags, and operation patterns.
4. Compare the `v1.0.0` tag/release with later official repository maintenance relevant to Glaux; trace material differences through issues, pull requests, commits, and release inclusion.
5. Identify where OGC conventions appear to prefer stable resource URLs, links, conformance classes, profiles, or alternate representations over URI path versioning.
6. Identify ambiguity or gaps that require comparison with broader API guidance.

**Expected Output:** OGC/CSAPI versioning constraint and gap inventory.

### Phase 3: General API Versioning and Deprecation Best-Practice Review (2.5-3.5 hours)

**Objective:** Compare authoritative API guidance for compatibility and deprecation strategies applicable to Glaux Server.

**Tasks:**

1. Review selected API guidelines for URI versioning, header/media-type versioning, semantic versioning, additive change rules, and compatibility definitions.
2. Review HTTP and IANA resources for headers, links, caching, deprecation/sunset behavior, and replacement-link practices.
3. Identify practices that align with OGC API/CSAPI conventions.
4. Identify practices that conflict with or are inappropriate for OGC API/CSAPI conventions.
5. Identify best-practice patterns for documenting breaking changes, deprecation windows, experimental behavior, and replacement resources.

**Expected Output:** API best-practice comparison table with applicability notes for Glaux Server.

### Phase 4: Glaux Server Compatibility and Breaking-Change Analysis (2-3 hours)

**Objective:** Define what should be treated as breaking, non-breaking, additive, deprecated, or experimental for Glaux Server planning.

**Tasks:**

1. Analyze externally visible Glaux Server contract areas:
   - landing page,
   - API definition,
   - conformance declaration,
   - collections,
   - resources,
   - links,
   - schemas,
   - media types,
   - query parameters,
   - error responses,
   - observations,
   - streams,
   - commands,
   - status/events.
2. Classify example changes as breaking, non-breaking, conditional, additive, or uncertain.
3. Identify compatibility risks for clients, conformance tests, OpenAPI-generated clients, CSAPI Explorer, and external integrations.
4. Identify how backward compatibility should interact with server-side contracts for Publisher, Simulator, Web App, Mobile, and external clients.
5. Identify unresolved project decisions.

**Expected Output:** Glaux Server breaking-change and compatibility classification matrix.

### Phase 5: Deprecation, Sunset, Documentation, and Test Implication Analysis (2-2.5 hours)

**Objective:** Define how Glaux Server should communicate evolution and how compatibility should be tested.

**Tasks:**

1. Identify recommended deprecation communication mechanisms:
   - documentation,
   - OpenAPI descriptions,
   - response headers,
   - link relations,
   - changelog entries,
   - conformance metadata,
   - schema annotations where available.
2. Identify how Glaux Server should expose replacements or successor behavior.
3. Identify how to communicate experimental, planned, deferred, or staged-conformance features.
4. Identify compatibility and deprecation tests:
   - OpenAPI contract comparison,
   - schema/golden-file comparison,
   - response-shape compatibility,
   - link-relation stability,
   - deprecated-resource warnings,
   - sunset/replacement behavior,
   - external-client compatibility.
5. Map handoffs to downstream documentation, validation, testing, and interoperability topics.

**Expected Output:** Deprecation/sunset communication and compatibility-test implication matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert research output into a decision-usable versioning, compatibility, and deprecation baseline.

**Tasks:**

1. Consolidate OGC/CSAPI constraints, API-guideline comparisons, breaking-change classifications, deprecation patterns, and test implications.
2. Resolve conflicts and ambiguities where possible.
3. Produce findings grouped by versioning strategy, compatibility rules, deprecation communication, and test implications.
4. Produce recommendations for Glaux Server planning and downstream IDR topic use.
5. Update the shared register only where this topic establishes a newer state, release relationship, or compatibility handoff.
6. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [x] CSAPI, OGC API - Features, OpenAPI, HTTP, and relevant API-guideline sources have been reviewed.
- [x] Standards version, server version, API contract version, schema version, media/profile version, deprecation, and sunset concepts are distinguished.
- [x] OGC/CSAPI constraints on versioning and evolution are identified with source anchors.
- [x] Material official release/tag/issue/PR/commit history is reconciled to the published baseline and authority-classified.
- [x] API versioning approaches are compared for compatibility with OGC/CSAPI conventions.
- [x] Breaking, non-breaking, additive, deprecated, experimental, and uncertain change categories are defined for Glaux Server.
- [x] Deprecation, replacement, and retirement communication mechanisms are identified.
- [x] Documentation, OpenAPI, conformance, validation, testing, and interoperability implications are documented.
- [x] Recommendations are decision-usable and bounded to Glaux Server.
- [x] Risks of over-versioning, under-versioning, overclaiming, and breaking client compatibility are identified.
- [x] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** API Versioning, Backward Compatibility, and Deprecation Strategy Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-010a-api-versioning-backward-compatibility-and-deprecation-strategy-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Versioning terminology and concept distinctions
5. OGC/CSAPI versioning and evolution constraints
6. General API versioning and deprecation best-practice comparison
7. Glaux Server breaking-change and compatibility classification matrix
8. Deprecation, sunset, replacement, and experimental-behavior communication findings
9. Documentation, OpenAPI, conformance, validation, and test implications
10. Downstream topic handoff matrix
11. Recommendations
12. Risks, constraints, and open questions
13. Validation against this plan's success criteria
14. References

The compatibility matrix should include, at minimum:

- API surface area
- Example change
- Change classification
- Source guidance / source anchor
- Client impact
- Conformance impact
- Documentation impact
- Test implication
- Recommended handling
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-006` through `IDR-SRV-010` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, OGC API - Features, OpenAPI, HTTP, and selected API-design guidance sources must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-011: Query, Filtering, Sorting, Pagination, and Selection Semantics`
- `IDR-SRV-012: Content Negotiation, Media Types, and Encoding Selection`
- `IDR-SRV-013: Error Model, HTTP Status Codes, and Failure Semantics`
- `IDR-SRV-014: OpenAPI Description and API Documentation Strategy`
- `IDR-SRV-015: Canonical Glaux Server Resource Model`
- `IDR-SRV-016: Identifier, URI, and Resource Lifecycle Strategy`
- `IDR-SRV-023: Schema and Encoding Validation Strategy`
- `IDR-SRV-050: Conformance Harness Strategy`
- `IDR-SRV-051: Requirement-to-Test Traceability Strategy`
- `IDR-SRV-053: Test Data, Fixtures, Golden Files, and Scenario Corpus Strategy`
- `IDR-SRV-056: Interoperability Test Matrix for External CSAPI Clients`
- Final overall IDR synthesis report

---

## 9. Research Status Checklist

Update this section as work progresses.

- [x] Phase 1 complete
- [x] Phase 2 complete
- [x] Phase 3 complete
- [x] Phase 4 complete
- [x] Phase 5 complete
- [x] Phase 6 synthesis complete
- [x] Deliverable draft complete
- [x] Deliverable reviewed
- [ ] Deliverable accepted

**Actual Research Time:** Approximately 5 hours of AI-assisted elapsed execution time on August 1, 2026<br>
**Research Execution Completed:** August 1, 2026<br>
**Completion Date:** TBD until plan-owner acceptance

---

## 10. Notes and Open Questions

- This topic should recommend an API evolution baseline, not design the entire release-management process.
- The report must not recommend versioning approaches that undermine OGC API discoverability, stable links, or CSAPI conformance.
- API versioning should be treated as a compatibility and interoperability concern, not as a way to reduce required standards behavior.
- Existing implementation studies may later refine practical expectations for versioning and deprecation behavior.
- Open question: Does Glaux Server need explicit API versioning beyond standards version, server release version, OpenAPI version, and schema/conformance declarations?
- Open question: Should Glaux Server avoid URI path versioning unless a future standards or profile requirement demands it?
- Open question: How should Glaux Server handle experimental extensions without making them appear part of core CSAPI conformance?
- Open question: What compatibility guarantees are realistic for a reference implementation before its first stable release?
- Risk: Over-versioning could fragment clients and make CSAPI interoperability harder.
- Risk: Under-versioning could make breaking changes hard to communicate.
- Risk: Deprecation without replacement links, documentation, and tests could harm client trust and interoperability.
- Risk: Staged conformance could be misunderstood as completed conformance if not communicated carefully.

---

## References

- Glaux Server Overall IDR Research Plan:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Creation Prompt:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-creation-prompt.md
- Research Plan Template:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Overall Research Report Template:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/overall-research-report-template.md
- Research Planning Approach:
  - https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md
- OGC API - Connected Systems landing page:
  - https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources:
  - https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data:
  - https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository:
  - https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems v1.0.0 tagged API artifacts:
  - https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api
- OGC API - Connected Systems v1.0.0 release and bundled OpenAPI 3.1 artifacts:
  - https://github.com/opengeospatial/ogcapi-connected-systems/releases/tag/v1.0.0
- OGC schemas:
  - https://schemas.opengis.net/
- OGC API - Features Part 1:
  - https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC API standards family:
  - https://ogcapi.ogc.org/
- OpenAPI Specification:
  - https://spec.openapis.org/oas/latest.html
- OpenAPI Initiative:
  - https://www.openapis.org/
- Semantic Versioning:
  - https://semver.org/
- RFC 9110 - HTTP Semantics:
  - https://www.rfc-editor.org/rfc/rfc9110
- RFC 9111 - HTTP Caching:
  - https://www.rfc-editor.org/rfc/rfc9111
- RFC 8288 - Web Linking:
  - https://www.rfc-editor.org/rfc/rfc8288
- RFC 8594 - The Sunset HTTP Header Field:
  - https://www.rfc-editor.org/rfc/rfc8594
- IANA HTTP Field Name Registry:
  - https://www.iana.org/assignments/http-fields/http-fields.xhtml
- IANA Link Relation Types:
  - https://www.iana.org/assignments/link-relations/link-relations.xhtml
- Microsoft REST API Guidelines:
  - https://github.com/microsoft/api-guidelines
- Microsoft Azure REST API Guidelines:
  - https://github.com/microsoft/api-guidelines/blob/vNext/azure/Guidelines.md
- Google API Improvement Proposals:
  - https://google.aip.dev/
- Google AIP-180 - Backwards Compatibility:
  - https://google.aip.dev/180
- Google AIP-181 - Stability Levels:
  - https://google.aip.dev/181
- Zalando RESTful API Guidelines:
  - https://opensource.zalando.com/restful-api-guidelines/
- Heroku HTTP API Design Guide:
  - https://github.com/interagent/http-api-design
- OS4CSAPI testing research-plan corpus:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans

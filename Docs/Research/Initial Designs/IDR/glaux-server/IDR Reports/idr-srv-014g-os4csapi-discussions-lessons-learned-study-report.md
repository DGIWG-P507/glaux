# Section 014G: OS4CSAPI Discussions Lessons-Learned Study - Research Report

**Topic ID:** IDR-SRV-014G<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-014G OS4CSAPI Discussions Lessons-Learned Study](../IDR%20Plans/idr-srv-014g-os4csapi-discussions-lessons-learned-study.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core and 34 detailed questions; all six methodology phases, nine success criteria, sixteen required content areas, and twelve minimum lessons-matrix fields are validated<br>
**Methodology Used:** Complete organization-discussion inventory through GitHub GraphQL; exact body, comment, reply, date, participant, linked-issue, pull-request, and artifact review; current disposition checks; comparison with accepted IDR-SRV-006 through IDR-SRV-014F and upstream-history register version 1.8; evidence-strength and ownership classification; bounded synthesis<br>
**Research Time:** Approximately 5 hours of AI-assisted elapsed execution on August 31, 2026<br>
**Discussion Baseline:** `OS4CSAPI/os4csapi-meta` organization discussions, 14 threads created October 19, 2025 through July 20, 2026 and current through August 31, 2026<br>
**Supporting Baseline:** Accepted Glaux reports IDR-SRV-006 through IDR-SRV-014F; OS4CSAPI phase-9 client at `754411897173c2ec4debaa9bcf4ed9e0f8a9e230`; official CSAPI `v1.0.0`; upstream-history register version 1.8<br>
**Document Purpose:** Convert informal OS4CSAPI experience into evidence-qualified Glaux design, validation, documentation, fixture, conformance, and interoperability handoffs without treating discussion, popularity, prototype claims, or proposed architecture as normative or implemented fact<br>
**Author:** OpenAI Codex<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** August 31, 2026<br>
**Date:** August 31, 2026<br>
**Last Updated:** August 31, 2026

---

## Reading Guide

| Label | Meaning |
|---|---|
| N | Normative OGC standard or accepted Glaux standards-baseline decision |
| D | OS4CSAPI discussion body, comment, or reply |
| K | Linked code, pull request, issue, release, or immutable repository artifact |
| T | Reproduced test or implementation evidence from accepted IDR-SRV-014A through 014F |
| P | Proposal, recommendation, or analytical design construct not yet implemented |
| H | Historical or superseded statement retained for correction history |
| X | Analyst synthesis or Glaux recommendation; never a standards claim |

Evidence-strength terms are **high** (independent reproduction plus primary artifact or accepted test evidence), **moderate** (specific multi-party observation with traceable evidence), **bounded** (single-party experience or synthesis), and **proposal-only** (reasoned design without implementation or independent review). Thread status “open” means the GitHub discussion is not closed; it does not mean every statement is unresolved or endorsed.

The controlling rule is: **discussion may identify a question, risk, reproduction path, or candidate solution; standards and accepted Glaux decisions determine the server contract.**

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Discussion Source Inventory and Evidence Classification
4. Standards Interpretation and Ambiguity Findings
5. Implementation Lesson Findings
6. Interoperability Lesson Findings
7. API Behavior and Resource-Model Findings
8. Testing, Validation, Fixture, and Conformance Findings
9. Documentation and Developer-Experience Findings
10. Risk and Prioritization Findings
11. Lessons for Glaux Server
12. Downstream Topic Handoff Matrix
13. Recommendations
14. Risks, Constraints, and Open Questions
15. Validation Against the Research Plan
16. References
Appendix A. Detailed Question Ledger
Appendix B. Reproducible Discussion Inventory Record
Appendix C. Completion and Handoff

---

## 1. Executive Summary

The OS4CSAPI discussion corpus is valuable but narrower than the word “community” can imply. It contains fourteen open threads, all initiated by the Glaux project lead. Four threads received comments; five unique people participated across sixteen top-level comments and nine replies. Seven threads are 2025 code-sprint prompts or reports, three are 2026 demo/coordination threads, and four are long July 2026 architecture proposals with no comments [D]. The corpus therefore supplies implementer experience and candidate designs, not a representative poll, consensus, or standards interpretation authority.

The strongest discussion evidence is concentrated in three independently engaged threads. Discussion #1 records concrete OpenAPI packaging and recursive-schema failures, links the release-bundle automation that partly resolved them, and later distinguishes bundle existence from bundle discoverability. Accepted IDR-SRV-014 mechanically refined that lesson: the published bundles exist but still contain residual relative references and fail common lint/static-generation paths, while issue #186 remains open. Glaux must produce and validate an implementation-specific, dependency-closed API definition rather than publish an upstream bundle unchanged [D,K,T].

Discussion #2 shows the conditional value of OGC API - Features compatibility. QGIS can discover CSAPI resources through landing, conformance, API definition, collections, schemas, GeoJSON items, pagination, and bounding boxes. It also exposes real limits: nested arrays may render as null; content negotiation and default-media behavior can route a client to empty data or a different envelope; internal hostnames in links break traversal; and a noncanonical URL-format workaround may be needed on one deployment [D,T]. The lesson is not to flatten the CSAPI model or silently tolerate every foreign query. Glaux should keep its canonical contract, test generic clients black-box, document known limitations, and put compatibility transformations outside the normative server surface.

Discussion #37 supplies a particularly clean deployment lesson. A reverse-proxied OpenSensorHub landing page emitted internal `http://...:8181` links; configuring the public base URL fixed the client immediately and the reporter confirmed success. That result corroborates IDR-SRV-009 and 014A: public origin, forwarded-header trust, link generation, OpenAPI servers, redirects, and authentication scope must be tested as one deployed identity graph [D,T].

Several sprint recommendations are sound but require narrowing. Multi-encoding must be designed early, recursive SensorML/SWE schemas require bounded tooling, live-server CI must be replaced by pinned captures, and cross-client tests are valuable. Conversely, “ignore `/collections` first,” “copy and paste the other entities,” and “tolerate weird unsupported parameters” conflict with accepted Glaux rules when read as server-contract advice. Collections is central to generic-client discovery; repeated handler copying invites semantic drift; and accepted IDR-SRV-011 requires `400` for undeclared parameters. Such statements remain useful records of prototype pressure, not Glaux decisions [D,N,X].

The July architecture discussions #40–#43 are a substantial, explicitly synthetic three-paper proposal covering composition, OGC API - Records discovery, harvesting, identity, governance, materialization, failure isolation, and layered interfaces. Their most durable insights are boundary disciplines: CSAPI servers remain systems of record; cross-resource composition is often a client/federation concern rather than a server defect; multi-server identity must be source-qualified and reconciled rather than merged by string coincidence; conformance is not completeness; declared capabilities must be tested against observed behavior; stale and withheld data must not look absent; and failure/freshness must be explicit [P]. They are not evidence that the architecture works. The authors state that nothing has been built, and no one commented on the threads. Glaux should route their questions to later topics, not adopt the proposed federation stack as server scope.

The discussions confirm rather than overturn the accepted implementation-study sequence. They reinforce OpenAPI dependency closure, negotiation truthfulness, canonical discovery links, public-URL correctness, recursive schema validation, semantic client testing, isolated fixtures, and explicit capability/freshness states. The principal refinement is evidentiary: a compelling discussion can explain why a test matters, but only the standard, code, raw wire behavior, or a controlled experiment can establish the result.

---

## 2. Scope and Plan Alignment

### 2.1 Included Scope

- All fourteen OS4CSAPI organization discussions visible through `OS4CSAPI/os4csapi-meta` as of August 31, 2026.
- Every body, top-level comment, reply, date, author, category, and linked artifact material to Glaux Server.
- Current disposition checks for the principal linked standards issues, implementation pull requests, client contribution, and tooling issues.
- The 2025 code-sprint reports, 2026 demo/coordination threads, and July 2026 composition/discovery/architecture proposal series.
- Comparison with accepted Glaux query, negotiation, error, OpenAPI, implementation, client, and interoperability findings.
- Glaux Server handoffs for model, validation, documentation, security, test data, conformance, and interoperability topics.

### 2.2 Excluded Scope

- Treating discussion views, reactions, silence, or the author’s role as consensus.
- Re-executing every historical live demo, accepting attachments as vetted code, or promoting personal testimony to implementation proof.
- Designing the proposed multi-server federation, OGC API - Records profile, backend-for-frontend interfaces, or policy engine as part of Glaux Server.
- Deciding future Part 3 Pub/Sub or Part 5 binary behavior before their owning research topics.
- Changing accepted normative Glaux decisions solely to accommodate one generic client.
- Converting lessons directly into issues or engineering tickets.

### 2.3 Plan Coverage

All five core and 34 detailed questions are answered. The work completed the six plan phases: complete source inventory; standards-ambiguity classification; implementation/interoperability extraction; test/fixture/documentation analysis; risk/handoff analysis; and synthesis. The report contains all sixteen required sections and the twelve-field lessons matrix. Appendix A maps every detailed question.

---

## 3. Discussion Source Inventory and Evidence Classification

### 3.1 Complete Inventory

| # | Discussion | Category | Created / last updated | Top-level comments | Evidence role / freshness |
|---:|---|---|---|---:|---|
| 1 | [OpenAPI YAML feedback](https://github.com/orgs/OS4CSAPI/discussions/1) | General | 2025-10-19 / 2026-06-19 | 3 | Multi-party tooling evidence; later update narrows resolution |
| 2 | [Features clients to CSAPI servers](https://github.com/orgs/OS4CSAPI/discussions/2) | General | 2025-10-19 / 2026-02-15 | 5 | Multi-party QGIS/live-server evidence; historical deployment state |
| 7 | [Adding CSAPI to API - Features](https://github.com/orgs/OS4CSAPI/discussions/7) | General | 2025-10-21 | 0 | Single implementer notes copied from email; bounded advice |
| 8 | [Code Sprint Day 1](https://github.com/orgs/OS4CSAPI/discussions/8) | General | 2025-10-21 / 2025-10-22 | 0 | Secondary synthesis of linked work |
| 9 | [Code Sprint Day 2](https://github.com/orgs/OS4CSAPI/discussions/9) | General | 2025-10-22 | 0 | Secondary progress synthesis; includes a stale thread-number link |
| 11 | [Code Sprint Day 3](https://github.com/orgs/OS4CSAPI/discussions/11) | General | 2025-10-24 | 0 | Secondary demo/progress synthesis; claims require linked evidence |
| 13 | [Sprint lessons and recommendations](https://github.com/orgs/OS4CSAPI/discussions/13) | General | 2025-10-25 | 0 | Author synthesis; useful agenda, not independent validation |
| 37 | [OGC 134 demo feedback](https://github.com/orgs/OS4CSAPI/discussions/37) | Polls | 2026-03-01 / 2026-03-05 | 5 | Multi-party deployment reproduction and adoption anecdotes |
| 38 | [52North P1/P2 validation](https://github.com/orgs/OS4CSAPI/discussions/38) | General | 2026-04-02 | 0 | Coordination invitation only; no validation response |
| 39 | [Builder Days coordination](https://github.com/orgs/OS4CSAPI/discussions/39) | General | 2026-05-10 / 2026-05-11 | 3 | Branch/focus coordination; future-work proposals |
| 40 | [Layered joins and discovery](https://github.com/orgs/OS4CSAPI/discussions/40) | Ideas | 2026-07-04 / 2026-07-07 | 0 | Long analytical proposal; refined by #41–#43 |
| 41 | [Cross-resource composition](https://github.com/orgs/OS4CSAPI/discussions/41) | Ideas | 2026-07-20 | 0 | Synthetic design paper; audit/proposal, no implementation |
| 42 | [Discovery with OGC API - Records](https://github.com/orgs/OS4CSAPI/discussions/42) | Ideas | 2026-07-20 | 0 | Candidate profile/harvester proposal, no adoption |
| 43 | [Layered implementation architecture](https://github.com/orgs/OS4CSAPI/discussions/43) | Ideas | 2026-07-20 | 0 | Reference-architecture proposal; explicitly not built |

All fourteen threads remained open, none had a chosen answer, and all were authored by `Sam-Bolling`. Four threads had comments. Across discussion bodies, comments, and replies, the unique participants were `Sam-Bolling`, `EricLo-417`, `SpeckiJ`, `doublebyte1`, and `nsnarayanam`. GitHub reported sixteen top-level comments and nine replies [D]. These counts establish corpus shape, not importance.

### 3.2 Evidence Classes

| Class | Admission rule | Example |
|---|---|---|
| Confirmed linked result | Discussion links primary code/test evidence and current disposition agrees | OSH JSON naming PR #318 merged |
| Independently reproduced observation | Another participant reproduces a bounded behavior with conditions | QGIS connection or proxy-base correction |
| Implementation-supported lesson | Accepted IDR or raw artifact corroborates the discussion | Negotiation routes to different stores/shapes |
| Standards-interpretation question | Discussion raises a requirement or ambiguity needing normative comparison | default media type, schema support, UID scope |
| Proposal/recommendation | Candidate architecture, workaround, or future direction | Records federation or materialized composition tier |
| Opinion/anecdote | Specific experience without reproducible evidence | perceived value of `q` or copy/paste ease |
| Stale/superseded | Later code, standard, or discussion materially narrows it | “bundles do not exist”; phase-7 branch review |
| Coordination only | Invitation, ownership, or work planning without a finding | Discussion #38 |

### 3.3 Source Precedence and Freshness

Published standards and accepted Glaux decisions control. Linked immutable code and raw tests can corroborate. Discussion comments provide context and reproduction conditions. Sprint summaries are secondary to the threads and artifacts they summarize. The July proposal series is current as a proposal but not as implementation evidence.

Current checks on August 31 found: official bundle issues #78, #79, and #137 closed; discoverability issue #186 open; Camptocamp client PR #136 open; OSH JSON naming PR #318 merged; OWSLib failing-test issue #1013 closed; Hakunapi CSAPI issue #153 open; onboarding PR #158 merged; OSH Part 3 MQTT PR #194 open and draft; and QGIS mixed-type GeoJSON issue #51911 open [K].

---

## 4. Standards Interpretation and Ambiguity Findings

### 4.1 Lessons-Learned Matrix

Each row contains the plan’s twelve required fields: identifier; source; evidence type; topic; lesson; standard/prior finding; server relevance; confidence; Glaux implication; test/documentation recommendation; handoff; notes.

| ID | Source anchor | Evidence | Topic | Lesson | Standard / prior IDR | Server relevance | Strength | Glaux implication | Test / documentation recommendation | Handoff | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|
| LL-01 | [#1 comment](https://github.com/OS4CSAPI/os4csapi-meta/discussions/1#discussioncomment-14731675), [follow-up](https://github.com/OS4CSAPI/os4csapi-meta/discussions/1#discussioncomment-17359150) | D/K/T | OpenAPI packaging | Modular files are hard for common tools; released bundles exist but are not a proven standalone implementation contract | IDR-014; issues 78, 79, 137, 186 | Direct documentation/build concern | High | Generate a Glaux-specific closed definition from the real route contract | Dependency closure, lint, render, codegen, hash, and download test | 023, 050, 051 | #186 remains open; upstream bundles retain residual refs |
| LL-02 | [#1](https://github.com/orgs/OS4CSAPI/discussions/1), [#7](https://github.com/orgs/OS4CSAPI/discussions/7) | D/K/T | Recursive schemas | Recursive SensorML/SWE components exceed assumptions in some OAS/schema tooling | IDR-014, 021–023 | Direct schema/toolchain concern | High | Bound and pin recursive tooling; never flatten the normative model just to satisfy one generator | Recursive valid/invalid corpus, stack/alias budget, generator compatibility table | 021, 022, 023, 050 | Tool failure is not schema invalidity |
| LL-03 | [#2 first comment](https://github.com/OS4CSAPI/os4csapi-meta/discussions/2#discussioncomment-14730482) | D/T | Generic Features discovery | Generic clients use root, conformance, API definition, collections, schemas, items, limit, bbox, and GeoJSON | Features/CSAPI inheritance; IDR-009–012 | Direct interoperability surface | High | Keep inherited feature-resource surface coherent and complete | QGIS/other OAPIF black-box discovery script | 015, 017, 050, 056 | Client request sequence is historical QGIS behavior, not normative |
| LL-04 | [#2 request trace](https://github.com/OS4CSAPI/os4csapi-meta/discussions/2#discussioncomment-14730482) | D/N | Unknown parameters | QGIS sends WFS-like parameters; discussion recommends tolerance | Accepted IDR-011 P-011-13 | Direct conflict with accepted server policy | High boundary | Do not weaken canonical `400` policy; compatibility belongs in an explicit adapter/profile | Negative unknown-parameter test plus optional adapter test | 050, 056 | Discussion advice rejected for core Glaux API |
| LL-05 | [#2 schema comments](https://github.com/OS4CSAPI/os4csapi-meta/discussions/2#discussioncomment-14732752), [QGIS issue](https://github.com/qgis/QGIS/issues/51911) | D/K | Nested JSON | A generic client may display arrays/objects as null despite valid JSON and schema links | IDR-015/017 future model; IDR-014E | Direct consumer limitation, not server defect by itself | Moderate | Preserve canonical nested relationships; document and test client loss | Nested arrays, mixed types, empty arrays, link arrays across clients | 015, 017, 023, 056 | Flattening is optional compatibility output, not canonical shape |
| LL-06 | [#2 February follow-up](https://github.com/OS4CSAPI/os4csapi-meta/discussions/2#discussioncomment-15814425) | D/T | Negotiation | Format selection can expose different datasets and envelopes, not merely encodings | IDR-012, 014C, 014E | Direct server correctness concern | High | One logical resource set must not silently change with representation | Accept and `f` cross-product with semantic equality assertions | 012, 023, 050, 053 | Historical 52North deployment observation |
| LL-07 | [#7](https://github.com/orgs/OS4CSAPI/discussions/7) | D/X | Implementation reuse | Once one entity exists, shared mechanics recur; literal copy/paste risks divergent semantics | IDR-009–014 accepted behavior | Direct architecture concern | Bounded | Reuse typed route/validation abstractions, not copied handlers | Parameterized resource-family contract tests | 015, 025–030, 050 | “Copy and paste” is anecdotal shortcut advice |
| LL-08 | [#7](https://github.com/orgs/OS4CSAPI/discussions/7) | D/N | Free text query | An implementer found `q` low value and expensive | Part 1 advanced filtering; IDR-011 | Direct required/optional class planning | Bounded | Standards/profile decisions control; stage indexing after correctness and realistic data tests | Correctness tests first, workload benchmark before optimization | 011, 025, 054 | No client usage survey supports generalization |
| LL-09 | [#7](https://github.com/orgs/OS4CSAPI/discussions/7), [#13](https://github.com/orgs/OS4CSAPI/discussions/13) | D/T | Multi-encoding | Encoding architecture added late creates conversion and round-trip failures | IDR-012, 014A–014F | Direct representation architecture concern | High | Define canonical domain model and explicit codecs before endpoint proliferation | Cross-encoding semantic/round-trip golden suite | 015, 021–023, 050 | Confirmed across implementation studies |
| LL-10 | [#7](https://github.com/orgs/OS4CSAPI/discussions/7) | D/N/T | Collections | Prototype advice says ignore `/collections` initially | OGC API inheritance; IDR-009/010/014E/014F | Direct conflict with discovery needs | High boundary | Reject as Glaux sequencing advice; collections is an early vertical slice | Landing→collections→items traversal test from first milestone | 009/010 accepted, 050, 056 | Historical prototype shortcut only |
| LL-11 | [#8](https://github.com/orgs/OS4CSAPI/discussions/8), [OWSLib #1013](https://github.com/geopython/OWSLib/issues/1013) | D/K/T | CI fixtures | Tests tied to retired live servers fail for reasons unrelated to client correctness | IDR-014E, 014F | Direct test-strategy concern | High | Use captured/pinned fixtures and keep live tests observational | Mocked raw responses plus separate live smoke lane | 050, 053, 056 | OWSLib issue closed; principle remains current |
| LL-12 | [#8](https://github.com/orgs/OS4CSAPI/discussions/8), [OSH PR 318](https://github.com/opensensorhub/osh-core/pull/318) | D/K | Schema evolution | ControlStream/Command property renaming required implementation alignment | IDR-010A, 014A | Direct version/compatibility concern | High | Pin standard package and trace every model name to schema/version | Compatibility diff and old/new fixture tests | 010A, 015, 036, 051 | PR merged October 22, 2025 |
| LL-13 | [#11](https://github.com/orgs/OS4CSAPI/discussions/11), [#2 replies](https://github.com/OS4CSAPI/os4csapi-meta/discussions/2#discussioncomment-14795489) | D/T | Demo claims | QGIS success required a format workaround and did not prove universal client compatibility | IDR-014C/014E | Direct evidence-quality concern | High | Publish exact URL, version, credentials, selectors, and limitations with every demo | Scripted demo reproduction and raw request log | 049, 053, 056 | Day-3 synthesis overstates “without configuration changes” |
| LL-14 | [#37 proxy comment](https://github.com/OS4CSAPI/os4csapi-meta/discussions/37#discussioncomment-15999342), [fix confirmation](https://github.com/OS4CSAPI/os4csapi-meta/discussions/37#discussioncomment-16000593) | D/T | Public URL identity | Internal proxy URLs made protected resources appear inaccessible; public-base configuration fixed traversal | IDR-009, 014A | Direct deployment correctness concern | High | Treat external origin and link graph as security/correctness configuration | Reverse-proxy matrix for links, OAS, redirects, auth, HTTPS | 041, 045, 049, 050 | Independently confirmed pass after fix |
| LL-15 | [#37 analytics comment](https://github.com/OS4CSAPI/os4csapi-meta/discussions/37#discussioncomment-16006208) | D | Adoption/use cases | A newcomer built discovery, live map, logging, anomaly, and prediction workflows from the demo | Goal/demo readiness | Indirect server relevance | Bounded | Usable examples and stable read APIs enable unplanned value | Reproducible tutorial with bounded dataset and attribution | 047, 049, 053 | Anecdote; attachment not vetted as Glaux code |
| LL-16 | [#38](https://github.com/orgs/OS4CSAPI/discussions/38) | D | Coordination | A request for 3-client/3-server validation links smoke notes but contains no result | IDR-014C/014E | None as a result | Bounded | Do not count invitations or targets as completed validation | Require outcome artifact and version matrix | 056 | Coordination-only record |
| LL-17 | [#39](https://github.com/orgs/OS4CSAPI/discussions/39) | D/K | Branch freshness | Review was initially aimed at an obsolete phase-7 branch; phase-9 became controlling | IDR-014E pinning | Direct evidence-governance concern | High | Every review/test record must pin repository, branch, commit, package, and date | Provenance schema enforced in findings | 050, 051, 053, 056 | Current client PR #136 remains open |
| LL-18 | [#39](https://github.com/orgs/OS4CSAPI/discussions/39), [Part 3 PR](https://github.com/opensensorhub/osh-addons/pull/194) | D/K/N | Future standards | Part 3 Pub/Sub and Part 5 binary work were active sprint interests | Upstream register 1.8; Part 2 future bindings | Future, not v1 server obligation | High boundary | Preserve extension points; make no conformance claim or implementation commitment here | Draft-version compatibility experiments in owning topics | 035, 037 | Part 3 PR open/draft; proposals remain unsettled |
| LL-19 | [#40](https://github.com/orgs/OS4CSAPI/discussions/40) | P/N | Composition boundary | Cross-resource enrichment, discovery, and selection may be additive integration concerns rather than defects in each source server | Parts 1/2 links; IDR-017/056 future | Mostly client/federation relevance | Proposal-only | Keep Glaux resource graph canonical; do not add speculative joined endpoints | Link-traversal and N+1 cost scenarios | 017, 026, 034, 036, 056 | Broad paper refined by #41–#43 |
| LL-20 | [#41](https://github.com/orgs/OS4CSAPI/discussions/41) | P/N | Association mechanics | Proposal classifies direct, multi-hop, descriptive, temporal, correlation, and closure mechanics | Parts 1/2; accepted link/query baselines | Direct graph/query test inspiration | Proposal-only | Use mechanics as scenario vocabulary, not proof of exhaustive standard semantics | One canonical scenario per mechanic and failure state | 015–018, 034, 036, 053 | Claimed 47-association audit was not independently re-performed here |
| LL-21 | [#40](https://github.com/orgs/OS4CSAPI/discussions/40), [#41](https://github.com/orgs/OS4CSAPI/discussions/41), [#42](https://github.com/orgs/OS4CSAPI/discussions/42) | P/N | Multi-server identity | Earlier global-UID shortcut is refined: source and canonical URI identify a representation; UID is evidence, not proof of same asset | Part 1 identity; IDR-016 future | Direct identifier design relevance | Moderate analytical | Separate local ID, canonical URL, UID, source, and reconciliation assertion | Collision, rehost, alias, duplicate-asset fixtures | 015, 016, 017, 053 | Later trilogy supersedes broad #40 wording |
| LL-22 | [#41 boundaries](https://github.com/orgs/OS4CSAPI/discussions/41) | P/X | Consistency/auth visibility | Multi-request answers are mixed-version and missing can mean absent, withheld, or failed | IDR-013; future security/freshness topics | Direct response and client-state relevance | Moderate analytical | Expose retrieval/freshness context and uniform concealed/failed states where applicable | Concurrent-change and authorization-relative link tests | 018, 020, 039–043, 050, 055 | Federation semantics exceed base-server scope |
| LL-23 | [#42](https://github.com/orgs/OS4CSAPI/discussions/42) | P/N | Records discovery | Candidate OGC API - Records profile proposes multi-server governed discovery | Records is separate standard; Glaux server scope rule | Optional adjacent-service relevance | Proposal-only | Do not absorb federation catalog into canonical Glaux server scope | Keep a clean adapter/export boundary | 015, 047, 057 | No adoption, comments, executable schema, or tests |
| LL-24 | [#42 §4.9](https://github.com/orgs/OS4CSAPI/discussions/42) | P/X | Completeness | Schema/profile conformance does not prove harvest completeness | IDR-008/050 distinction | Direct general lesson | Moderate analytical | Separate validity, conformance, completeness, freshness, and authorization | Independent completeness/freshness assertions | 018, 020, 050, 051 | Useful beyond proposed Records profile |
| LL-25 | [#42 §7](https://github.com/orgs/OS4CSAPI/discussions/42) | P/T | Capability probing | Declared conformance should be compared with sampled observed behavior; full scan is legitimate fallback | IDR-008/014A–F | Direct interoperability harness relevance | Moderate, corroborated | Glaux must make declarations true; external harness should test both | Declaration-to-behavior matrix and drift alarm | 050, 051, 056 | Harvester design itself is unimplemented |
| LL-26 | [#42 §7.4](https://github.com/orgs/OS4CSAPI/discussions/42) | P/X | Deletion evidence | One absence is not deletion; auth, paging, filtering, and failures can mimic it | IDR-013; lifecycle topics | Direct lifecycle/client relevance | Moderate analytical | Define delete/tombstone semantics; clients must not infer deletion from one scan | Complete-scan, permission-change, reappearance scenarios | 016, 030, 050, 053 | Federation inference policy, not base CSAPI requirement |
| LL-27 | [#43](https://github.com/orgs/OS4CSAPI/discussions/43) | P | Layered architecture | Proposal isolates source servers, ingestion, catalog, composition, and persona interfaces | Glaux goal/scope; IDR-025+ future | Server boundary clarity | Proposal-only | Keep Glaux focused as canonical source server with extension interfaces | Architecture fitness tests in owning topics | 025–033, 057 | Explicitly not built or empirically cost-validated |
| LL-28 | [#43 §9](https://github.com/orgs/OS4CSAPI/discussions/43) | P/X | Failure/freshness | Stale, denied, unavailable, and empty should be typed, visible states; failure should isolate by source | IDR-013; later robustness topics | Direct error/freshness inspiration | Moderate analytical | Avoid null/silent gaps; expose bounded freshness and machine errors | Fault injection, stale-source, fail-closed scenarios | 018, 020, 042–044, 050, 054–056 | Materialized federation tier is out of Glaux scope |
| LL-29 | [#43 §7](https://github.com/orgs/OS4CSAPI/discussions/43) | P/N | Authorization | Distributed enforcement with centralized policy evaluation and auth-context caches is proposed | Future security topics | Direct security design prompt | Proposal-only | Route to threat/policy research; never treat cached auth decisions as globally shareable | Principal-scoped cache and PDP outage tests | 039–043, 055 | Product selection and detailed policy intentionally open |
| LL-30 | [#43 §11](https://github.com/orgs/OS4CSAPI/discussions/43) | P/N | Evolution | Tolerant readers and concurrently served additive contract versions reduce upgrade risk | IDR-010A accepted | Direct compatibility relevance | Moderate analytical | Follow accepted version/deprecation rules; test old clients against additive changes | Consumer contract and schema-diff suite | 015, 023, 048, 050 | Proposal notes consumer-driven contracts are aspirational |

### 4.2 Ambiguity Disposition

| Question raised in discussion | Disposition for Glaux |
|---|---|
| Which OpenAPI artifact is authoritative for implementation? | Published standard and tagged artifacts establish the source baseline; Glaux must generate/validate its own runtime-complete definition. An upstream bundle is not the deployed contract. |
| Must a default representation be GeoJSON? | No discussion preference creates a mandate. Accepted IDR-SRV-012 controls selection and truthfulness. |
| Should unknown WFS-like parameters be tolerated? | No for the canonical Glaux surface: accepted IDR-SRV-011 requires `400`. A separate adapter may intentionally emulate another protocol. |
| Should nested structures be flattened for QGIS? | Not canonically. Preserve the standard model and document/tool explicit compatibility views if later authorized. |
| Is a UID globally unique enough for federation merging? | No mandatory guarantee supports that inference. Use source-qualified representation identity and explicit reconciliation evidence. |
| Does `DataChoice` or recursion make schemas invalid? | No. Tool limitations and schema validity are distinct. Validate with pinned recursive-capable tooling. |
| Do the July papers define required federation architecture? | No. They are synthetic, proposal-only, and unimplemented. |

### 4.3 Potential Standards-Maintenance Follow-up

- Continue tracking issue #186 for the official developer gateway to the correct versioned artifacts.
- Preserve reproducible recursive-schema and bundle-closure failures for official tooling discussions; do not describe a tool failure as a normative defect without the exact schema basis.
- Route Part 3 Pub/Sub and binary-encoding questions only through their owning research topics and current draft pins.
- Treat any proposed Records profile, federation identity assertion, or composition API as a separate adoption artifact requiring its own venue, schema, tests, and authority.

No new SWG escalation is required to complete this report. Existing official issues and later Glaux topics cover the actionable questions.

---

## 5. Implementation Lesson Findings

### 5.1 Build One Vertical Slice, Then Generalize

Discussion #7 correctly observes that resource families share mechanics. The safe Glaux translation is a vertical slice—landing, conformance, collections, one resource family, schema, query, errors, OpenAPI, persistence, and tests—followed by typed reusable abstractions. Literal copying is rejected because the accepted standards baseline contains family-specific relationships, filters, lifecycle rules, encodings, and conformance dependencies [D,N,X].

### 5.2 Design Representations Before Endpoint Multiplication

The discussion warning about late multi-encoding conversion is confirmed across OSH, Connected Systems Go, pygeoapi, SECD, and client studies. Glaux needs a canonical domain model with codecs, loss policy, content negotiation, validation, and round-trip invariants before large route expansion. “Convert on push” may be an implementation optimization but cannot silently make representations semantically unequal [D,T].

### 5.3 Pin Every Input

Discussion #39 records a reviewer working from an inactive phase-7 branch. The correction to phase 9 reinforces a project-wide rule: branch names are mutable; findings must identify commit, package version, environment, route, and date. Current upstream contribution PR #136 is still open, so neither its final merge state nor upstream release inclusion can be assumed [D,K].

### 5.4 Deployment Is Part of API Correctness

The proxy-base correction in #37 shows that a conformant handler behind an incorrectly configured proxy is not an interoperable deployment. Glaux must generate one public identity consistently across payload links, pagination, OAS servers, redirects, Location headers, authentication callbacks, and documentation. Forwarded headers require an explicit trusted-proxy policy; blindly trusting them introduces host-header/security risk [D,T,X].

### 5.5 Separate Prototype Advice From Production Requirements

Ignoring Collections, flattening nested structures, accepting foreign parameters, converting everything after ingestion, or copying resource handlers may accelerate a demonstration. None should enter production architecture without standards, security, loss, and maintenance analysis. The discussions are most useful when they expose where prototype pressure will recur.

---

## 6. Interoperability Lesson Findings

### 6.1 Generic-Client Compatibility Is Conditional

QGIS proved that a generic Features client can reach part of a CSAPI surface. It did not prove that every CSAPI resource, representation, schema, relationship, or deployment works in QGIS. The demo depended on collections, GeoJSON-compatible item responses, bbox, limit, schemas, and eventually an explicit format selector. Nested objects remained lossy and one collection contained internal links [D].

Glaux should state compatibility as a versioned matrix: client version; connection URL; selectors; claimed classes; resource families; spatial/nonspatial handling; nested-field behavior; pass/fail; and raw evidence. “Works with QGIS” is too broad.

### 6.2 Negotiation Must Not Change Logical Holdings

The 52North follow-up is one of the corpus’s strongest lessons because it compares the same endpoint under selectors: SensorML returned populated `items`; JSON/GeoJSON returned an empty `FeatureCollection`. That is a provider-routing implementation behavior, not a normal encoding difference. Glaux representation tests must assert semantic equality of resource identity and count where two encodings represent the same logical collection [D,T].

### 6.3 Client Success Must Include Semantic Completeness

QGIS can connect while hiding nested arrays; the OS4CSAPI TypeScript client can parse while dropping fields, as IDR-SRV-014E/F showed. External-client results therefore need three states: transport/discovery success, structural parse success, and semantic completeness. Only the third can support an unrestricted interoperability claim [D,T].

### 6.4 Cross-Server Comparison Needs Immutable Fixtures

Discussion #38’s three-client/three-server goal is worthwhile, but the thread contains no outcome. Live endpoints change or disappear; the prior studies recorded both. IDR-SRV-056 should pair each live run with an immutable raw-wire fixture and record which result is observational versus deterministic.

### 6.5 Cross-Resource Composition Is a Boundary, Not an Excuse

The July proposal usefully distinguishes source-resource correctness from federation composition. Glaux need not precompute every cross-resource analytical view. It must still emit canonical identifiers, links, temporal semantics, filters, validators, and authorization behavior so a client can compose correctly. Avoiding speculative joins does not excuse a broken resource graph [P,N,X].

---

## 7. API Behavior and Resource-Model Findings

### 7.1 Discovery Graph

The discussions and prior reports converge on an early invariant: landing page, API definition, conformance declaration, Collections, collection schemas, item links, and public origin form one graph. A generic client does not infer the server’s intent; it follows what is emitted. Glaux should test the graph from outside its reverse proxy and under every supported base-path configuration.

### 7.2 Identifiers and Relationships

The broad #40 paper initially leaned too heavily on global UID uniqueness. The later #41/#42 papers correct this: a source-qualified canonical URI identifies a harvested representation; UID is evidence toward semantic equivalence, not mandatory proof. This refinement should inform IDR-SRV-016. Glaux itself must clearly distinguish local identifier, canonical resource URI, globally oriented UID, external identifier, alias, and relationship target [P,N].

### 7.3 Temporal and Hierarchical Composition

The proposal’s framework—direct dereference plus five mechanics: multi-hop traversal, descriptive selection, time-qualified association, uncertain correlation, and hierarchical closure—offers useful scenario language. It does not establish new endpoints or requirements. Later model/query topics should use it to test whether canonical resources and filters support composition without inventing undocumented paths [P].

### 7.4 Missing, Hidden, Failed, and Stale

Both the client evidence and architecture proposals argue against collapsing all absence to null. For Glaux, exact wire behavior remains governed by IDR-SRV-013 and later security topics, but the model should be capable of distinguishing: no resource; relationship not applicable; optional value unknown; access concealed/denied; upstream dependency failed; representation omitted; and value stale. Those distinctions require careful disclosure design so the server does not leak protected existence [N,P,X].

### 7.5 External Catalog and Composition Boundaries

OGC API - Records discovery and a multi-server composition tier may be valuable adjacent services. They are not implicit obligations of Glaux Server. Glaux should expose clean source APIs and optional adapter interfaces; it should not import the candidate governance profile, materialized federation views, or persona BFFs into the core server without a separately approved scope decision.

---

## 8. Testing, Validation, Fixture, and Conformance Findings

### 8.1 Required Test Layers

1. **Normative requirement tests:** exact obligations and conformance dependencies.
2. **OpenAPI/schema artifact tests:** closure, lint, recursion, render, generation, example validity, runtime parity.
3. **Raw-wire API tests:** statuses, headers, bodies, links, errors, and selectors.
4. **Semantic representation tests:** identifiers/counts/meaning preserved across encodings and round trips.
5. **Deployment tests:** public origin, proxy paths, HTTPS, redirects, Location, auth, and OAS servers.
6. **External-client tests:** QGIS, OWSLib, OS4CSAPI client, and later selected clients, with semantic-completeness assertions.
7. **Observational live smoke:** non-gating and date-stamped.

### 8.2 Discussion-Derived Fixture Families

| Family | Minimum variants |
|---|---|
| OpenAPI packaging | Modular tree; generated bundle; missing ref; wrong case; recursive valid schema; cyclic alias failure |
| Features clients | Spatial/nonspatial collection; nested object; array of objects; empty array; mixed type; schema link |
| Negotiation | Default; JSON; GeoJSON; SensorML; invalid Accept; `f`; same IDs/counts across encodings |
| Discovery | Complete graph; missing Collections; wrong conformance; internal-host link; base path; reordered relations |
| Identity | Same local ID on two sources; same UID/different asset; same asset/different UID; rehost/alias |
| Composition | Direct; two-hop; temporal; hierarchy; uncertain correlation; source failure; pagination drift |
| Evidence | Mutable branch; pinned commit; retired endpoint; captured fixture; current live comparison |
| Security/freshness | absent; concealed; denied; stale; failed; empty; auth-context cache separation |

### 8.3 CI Versus Live Testing

The OWSLib incident and the accepted client study make the split explicit. CI uses local deterministic servers and raw fixtures. A scheduled live lane checks ecosystem drift and produces evidence but does not fail a Glaux merge merely because a third-party endpoint, certificate, credential, dataset, or route changed.

### 8.4 Conformance Versus Interoperability

A generic client workaround can improve interoperability without becoming a conformance rule. Conversely, a conformance test may pass while a popular client loses nested semantics. Both suites are necessary, labeled separately, and traced to different authorities.

### 8.5 Findings Record Schema

Every discussion-derived test record should capture: source/thread/comment; claim type; standard anchor; server/client versions; commit; request; response; fixture digest; parsed output; semantic assertions; environment; date; ownership classification; freshness; and downstream disposition.

---

## 9. Documentation and Developer-Experience Findings

### 9.1 Developer Gateway

Discussion #1 shows that artifact existence is insufficient. A developer must be able to identify the correct version, view it, download the exact displayed file, understand whether it is modular or closed, and reproduce its validation. Glaux documentation should place the runtime API definition, source package, generated artifact digest, and supported-tool matrix together.

### 9.2 Recursive-Schema Guidance

Documentation should explain recursion as a model property, identify tested tools and versions, set safe recursion/alias limits, provide small valid recursive examples, and distinguish renderer/code-generator limits from server validation. A flattened example may teach but must not masquerade as the canonical schema.

### 9.3 Generic-Client Guide

A QGIS/Features-client guide should include the exact connection URL, supported client versions, selectors, spatial/nonspatial behavior, nested-property limitations, schema endpoints, bbox/limit behavior, authentication, screenshots, and known failures. It should not promise full CSAPI behavior through a client that only understands Features.

### 9.4 Demo Reproducibility

Every demo needs a seed revision, server build, base URL, credentials policy, expected resource counts/IDs, scripted reset, and raw smoke script. Community stories and tutorials are valuable adoption material when separated from conformance evidence and when third-party contributions are attributed.

### 9.5 Documentation as a Tested Surface

Links, examples, curl commands, ReDoc/Swagger rendering, downloadable API files, reverse-proxy URLs, and authentication steps should be exercised in CI or release validation. The Day-2 stale link to discussion #3—when the relevant thread was #2—is a small example of why human-readable synthesis also needs link checking [D].

---

## 10. Risk and Prioritization Findings

| Priority | Risk | Evidence | Required control |
|---|---|---|---|
| Critical | API returns success but wrong/empty logical data under a representation selector | #2 plus IDR-012/014C/F | Semantic equality and selector-negative tests |
| Critical | Public link graph points inside the deployment or across the wrong auth/origin boundary | #37 plus IDR-009/014A | External-origin deployment test matrix |
| High | Published API definition cannot be consumed or does not match runtime | #1 plus IDR-014/register 1.8 | Dependency closure, pinned tools, runtime parity |
| High | Recursive schemas break tooling or trigger resource exhaustion | #1/#7 plus IDR-014 | Tool bounds, validation budgets, curated corpus |
| High | Generic client connects but silently loses fields | #2 plus IDR-014E/F | Semantic-completeness assertions |
| High | Mutable live endpoint/branch invalidates evidence | #8/#38/#39 | Immutable pins and fixtures; observational live lane |
| High | Prototype shortcut becomes permanent contract divergence | #7 | Vertical slice and architecture review gates |
| High | Identity strings cause false merges across sources | #40–#42 refinement | Source-qualified identity and explicit reconciliation |
| Medium | Future draft work is treated as v1 obligation | #39 | Topic/version authority labels |
| Medium | Compelling architecture proposal expands Glaux scope | #40–#43 | Server admission rule and downstream routing |
| Medium | Documentation exists but developers cannot find/use the correct artifact | #1/#13 | Tested developer gateway |

The first implementation priorities remain canonical model and links, truthful negotiation, uniform validation/errors, OAS/runtime parity, and deterministic fixtures. Federation catalog/materialization proposals must not displace those prerequisites.

---

## 11. Lessons for Glaux Server

### 11.1 Adopt

- Complete, typed discovery and relationship graphs.
- Implementation-specific OpenAPI generation with dependency closure and pinned validation.
- Canonical domain modeling plus explicit encoders/decoders designed early.
- External-origin correctness as part of deployment acceptance.
- Deterministic raw-wire fixtures and separate live observational tests.
- Semantic-completeness checks for every external client.
- Source, version, commit, date, and ownership fields in every finding.
- Explicit states for failure, freshness, validation, and authorization where applicable.

### 11.2 Avoid

- Publishing upstream modular/bundled artifacts unchanged as the Glaux contract.
- Flattening or renaming canonical structures solely for one client.
- Accepting undeclared parameters silently to accommodate protocol probes.
- Treating representation selection as permission to route to different holdings.
- Copy/paste endpoint proliferation.
- CI that depends on mutable public endpoints.
- Claims of community consensus based on one author, reactions, or no replies.
- Importing the proposed federation stack into Glaux Server scope.

### 11.3 Investigate in Owning Topics

- Exact identifier/alias/reconciliation semantics.
- Recursive SensorML/SWE component implementation and validation limits.
- Temporal/hierarchical composition and lifecycle boundaries.
- Status/event/freshness representation.
- Future Pub/Sub and binary encodings.
- Federation/adaptor interfaces and optional Records integration.
- Security enforcement, trusted proxy policy, and principal-scoped caches.

### 11.4 Escalate Only With Evidence

When a discussion suggests standards maintenance, produce the minimal reproducer, exact published clause/schema, affected tool/version, observed/expected result, and scope of impact. Existing issue #186 and upstream-history entries should be updated rather than duplicated where they already own the question.

---

## 12. Downstream Topic Handoff Matrix

| Topic | Handoff |
|---|---|
| IDR-SRV-015 | Canonical nested resource fields, feature/type distinctions, semantic completeness, and no client-driven flattening |
| IDR-SRV-016 | Local ID, canonical URI, UID, source, alias/rehost, collision, and reconciliation distinctions |
| IDR-SRV-017 | Typed links, traversal, public targets, reverse relations, hierarchy, and composition scenarios |
| IDR-SRV-018 | Temporal-as-of relationships, mixed-version reads, freshness, and staleness states |
| IDR-SRV-020 | Status/event versus absent/failed/stale/hidden distinctions |
| IDR-SRV-021/022 | Recursive SensorML/SWE schemas, tool bounds, examples, and valid/invalid fixture families |
| IDR-SRV-023 | One schema/OAS/runtime contract, recursive validation, cross-encoding equivalence, read/write round trips |
| IDR-SRV-024 | URI semantic bindings, observed/controlled properties, and client-friendly documentation without lossy flattening |
| IDR-SRV-034/036 | Composition-ready observation/command relationships, temporal filtering, status/result/event lifecycle |
| IDR-SRV-035/037 | Treat Part 3 Pub/Sub and binary work as version-pinned proposals until published/accepted |
| IDR-SRV-039–043 | Public-origin security, concealed/denied states, policy enforcement, auth-context caching, fail closed |
| IDR-SRV-045/049 | Reverse-proxy identity, HTTPS links, deployable demo seed, tested tutorials, and public documentation |
| IDR-SRV-050 | Requirement, artifact, raw-wire, deployment, external-client, fault, and live-smoke test layers |
| IDR-SRV-051 | Trace discussion lessons to tests as informative provenance, never as requirement authority |
| IDR-SRV-053 | Add OpenAPI, QGIS, negotiation, identity, composition, proxy, freshness, and evidence-drift fixtures |
| IDR-SRV-054 | Bound recursive tooling, fan-out, live streams, and composition cost with realistic data |
| IDR-SRV-055 | Test principal-scoped caches, concealed resources, command authorization, and PDP/dependency failure |
| IDR-SRV-056 | Versioned multi-client/multi-server matrix with raw fixtures and semantic-completeness levels |
| IDR-SRV-057 | Preserve the evidence hierarchy and server-versus-federation boundary in final synthesis |

---

## 13. Recommendations

1. Treat the accepted Glaux contract as canonical and add client-specific compatibility only through explicit, separately tested adapters.
2. Make Collections and the complete discovery graph part of the first end-to-end server slice.
3. Generate Glaux OpenAPI from the implementation contract and require dependency closure, lint, render, code generation, example validation, and runtime parity before release.
4. Build representation codecs and semantic-equivalence tests before broad endpoint implementation.
5. Add an external reverse-proxy acceptance environment that verifies every emitted URL and authentication boundary.
6. Define interoperability success at transport, structural, and semantic levels.
7. Pair all live-client/server evidence with immutable captures and exact version metadata.
8. Use the July composition mechanics as fixture-scenario prompts, not as adopted server architecture.
9. Route OGC API - Records federation, materialization, and BFF proposals outside the core Glaux Server scope unless separately approved.
10. Publish tested onboarding and generic-client guides beside the exact downloadable Glaux API definition.
11. Maintain a standards-maintenance evidence template and update existing upstream issues rather than multiplying informal claims.
12. Preserve the OS4CSAPI discussions as dated rationale and pain-point evidence, while deriving requirements only from controlling sources.

---

## 14. Risks, Constraints, and Open Questions

### 14.1 Constraints

- All top-level discussions were authored by one person; breadth of participation is limited.
- Ten threads have no top-level comments; the four architecture papers have no comments.
- Discussion status and edits are mutable; GraphQL inventory is date-stamped rather than immutable.
- Several historical live endpoints, branches, and client versions have drifted.
- Attachments, screenshots, and community notebooks were not security- or correctness-audited.
- Architecture papers are synthetic exercises and explicitly lack implementation evidence.

### 14.2 Resolved Plan Questions

- **Broad versus one-off:** OpenAPI closure, negotiation, public links, fixture pinning, and semantic parsing recur across independent artifacts; `q` value, copy/paste ease, and specific UI preferences remain one-off opinions.
- **Formal follow-up:** Existing #186 and upstream-history entries cover the current OpenAPI/Part 3 questions. No new escalation blocks Glaux.
- **Documentation versus implementation:** Artifact discovery, tutorials, exact demo steps, and known-client limits are documentation; canonical links, representation equality, schema validity, errors, and public origin are implementation behavior.
- **CI candidates:** artifact closure, recursive schema, discovery graph, negotiation semantics, proxy origin, semantic field preservation, and immutable-fixture tests belong in CI; third-party live availability does not.

### 14.3 Open but Non-Blocking

- Which external clients and versions will be selected for the final IDR-SRV-056 matrix?
- Will Glaux provide any explicitly noncanonical compatibility adapter, and under what namespace/profile?
- Which recursive-schema tools meet Glaux’s performance and safety budgets?
- Is an optional Records export or federation connector desirable after the core server exists?
- Which future Part 3/Part 5 artifacts will be mature when their owning topics execute?

These questions belong to indexed downstream research and do not prevent acceptance of this evidence baseline.

---

## 15. Validation Against the Research Plan

### 15.1 Success Criteria

| Criterion | Result | Evidence |
|---|---|---|
| Exact discussions and dates | Met | Complete fourteen-thread inventory and exact links |
| Evidence ownership distinguished | Met | Reading guide, classes, matrix evidence/strength |
| Standards questions classified | Met | Section 4.2–4.3 |
| Implementation/interoperability mapped | Met | Sections 5–7 and matrix |
| Prior studies compared | Met | Executive synthesis and per-row prior-IDR field |
| Test/documentation/DX implications | Met | Sections 8–9 |
| Risks and unresolved questions | Met | Sections 10 and 14 |
| Downstream handoffs | Met | Section 12 and matrix |
| Reproducible references | Met | Section 16 and Appendix B |

### 15.2 Deliverable Structure

All sixteen required numbered sections are present. The lessons matrix contains thirty findings and all twelve required fields. Appendix A covers all 34 detailed questions. Acceptance remains open while the report is In Review.

### 15.3 Quality Checks

- All current discussion threads, comments, replies, categories, authors, and dates were queried directly.
- The narrow participant distribution is disclosed.
- Sprint summaries are not double-counted as independent evidence.
- Current linked issue/PR state is distinguished from historical prose.
- #40 is treated as refined by #41–#43; later identity nuance controls.
- No architecture proposal is described as implemented or normative.
- Conflicting prototype advice is adjudicated against accepted Glaux rules.
- No engineering tickets or later-topic decisions were created.

---

## 16. References

### 16.1 Discussion Corpus

- [OS4CSAPI organization discussions](https://github.com/orgs/OS4CSAPI/discussions)
- [OS4CSAPI meta repository](https://github.com/OS4CSAPI/os4csapi-meta)
- [Discussion #1: OpenAPI YAML feedback](https://github.com/orgs/OS4CSAPI/discussions/1)
- [Discussion #2: Features clients to CSAPI](https://github.com/orgs/OS4CSAPI/discussions/2)
- [Discussion #7: Adding CSAPI to API - Features](https://github.com/orgs/OS4CSAPI/discussions/7)
- [Discussion #8: Code Sprint Day 1](https://github.com/orgs/OS4CSAPI/discussions/8)
- [Discussion #9: Code Sprint Day 2](https://github.com/orgs/OS4CSAPI/discussions/9)
- [Discussion #11: Code Sprint Day 3](https://github.com/orgs/OS4CSAPI/discussions/11)
- [Discussion #13: Sprint lessons](https://github.com/orgs/OS4CSAPI/discussions/13)
- [Discussion #37: OGC 134 demo feedback](https://github.com/orgs/OS4CSAPI/discussions/37)
- [Discussion #38: 52North validation coordination](https://github.com/orgs/OS4CSAPI/discussions/38)
- [Discussion #39: Builder Days coordination](https://github.com/orgs/OS4CSAPI/discussions/39)
- [Discussion #40: Layered joins and discovery](https://github.com/orgs/OS4CSAPI/discussions/40)
- [Discussion #41: Cross-resource composition](https://github.com/orgs/OS4CSAPI/discussions/41)
- [Discussion #42: Records discovery](https://github.com/orgs/OS4CSAPI/discussions/42)
- [Discussion #43: Layered implementation architecture](https://github.com/orgs/OS4CSAPI/discussions/43)

### 16.2 Linked Primary Artifacts

- [Official CSAPI v1.0.0 release](https://github.com/opengeospatial/ogcapi-connected-systems/releases/tag/v1.0.0)
- [Bundle issues #78](https://github.com/opengeospatial/ogcapi-connected-systems/issues/78), [#79](https://github.com/opengeospatial/ogcapi-connected-systems/issues/79), [#137](https://github.com/opengeospatial/ogcapi-connected-systems/issues/137), and [developer gateway #186](https://github.com/opengeospatial/ogcapi-connected-systems/issues/186)
- [Camptocamp OGC client PR #136](https://github.com/camptocamp/ogc-client/pull/136)
- [OSH property-name PR #318](https://github.com/opensensorhub/osh-core/pull/318)
- [OWSLib failing-test issue #1013](https://github.com/geopython/OWSLib/issues/1013)
- [Hakunapi CSAPI issue #153](https://github.com/nlsfi/hakunapi/issues/153) and [onboarding PR #158](https://github.com/nlsfi/hakunapi/pull/158)
- [QGIS mixed-type GeoJSON issue #51911](https://github.com/qgis/QGIS/issues/51911)
- [OSH Part 3 MQTT draft PR #194](https://github.com/opensensorhub/osh-addons/pull/194)
- [OS4CSAPI phase-9 client](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230)
- [CSAPI Explorer](https://ogc-csapi-explorer.pages.dev/)

### 16.3 Standards and Glaux Baseline

- [OGC API - Connected Systems Part 1](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems Part 2](https://docs.ogc.org/is/23-002/23-002.html)
- [OGC API - Features Part 1](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html)
- [SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html)
- [Overall Glaux Server IDR plan](../IDR%20Plans/overall-idr-research-plan.md)
- [Shared upstream-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md)
- [IDR-SRV-014 OpenAPI strategy](idr-srv-014-openapi-description-and-api-documentation-strategy-report.md)
- [IDR-SRV-014E client smoke findings](idr-srv-014e-os4csapi-client-smoke-test-findings-study-report.md)
- [IDR-SRV-014F SECD interoperability findings](idr-srv-014f-secd-interoperability-findings-study-report.md)

---

## Appendix A. Detailed Question Ledger

| # | Detailed question | Answer location |
|---:|---|---|
| 1 | Relevant discussions, comments, issues, sprint/meeting notes, PRs, artifacts | 3.1, 16.1–16.2 |
| 2 | Server/client/OAS/schema/example/fixture/conformance topics | Matrix and Sections 4–9 |
| 3 | Dates, participants, repositories, links | 3.1, Appendix B |
| 4 | Current, stale, superseded, unresolved, historical | 3.2–3.3 and matrix notes |
| 5 | Exact evidence | Matrix source anchors and Section 16 |
| 6 | Standards interpretation questions | Section 4.2 |
| 7 | Part 1/2, Features, SensorML, SWE, OAS, media, query, tasking, streaming | LL-01–LL-18 and 4.3 |
| 8 | Ambiguity versus misunderstanding | 4.2 |
| 9 | Resource/schema/dynamic/tasking/conformance handoffs | Section 12 |
| 10 | Questions requiring SWG/maintenance follow-up | 4.3 |
| 11 | Recurring implementation challenges | Sections 5–6 |
| 12 | Server pitfalls to avoid | 11.2 |
| 13 | Patterns/shortcuts to consider carefully | 5.1–5.5 |
| 14 | Cross-server issues | 6.1–6.5 |
| 15 | External-client expectations | 6.1–6.3 |
| 16 | Landing/OAS/conformance/collections/paths/links/IDs/query/media/error/OAS | LL-01, LL-03–06, LL-14, Section 7 |
| 17 | Canonical resource-model effects | 7.2–7.5 |
| 18 | Lifecycle/ID/time/status/property/SensorML/SWE/semantics effects | LL-09, LL-12, LL-20–30 |
| 19 | Category C/D carry-forward | Section 12 |
| 20 | Conformance/smoke/schema/client/golden/fixture gaps | Section 8 |
| 21 | Derived test cases/fixtures | 8.2 |
| 22 | Independently supported tests | 8.1–8.4 |
| 23 | Implementation-specific test risks | 8.4 |
| 24 | IDR-050/051/053/056 handoffs | Section 12 |
| 25 | Developer pain points | Sections 9.1–9.4 |
| 26 | Documentation gaps | Section 9 |
| 27 | Needed examples/tutorials/diagrams/API docs/guides | 9.1–9.4 |
| 28 | Demo/onboarding/NATO adoption relevance | 9.3–9.4, LL-15 |
| 29 | Documentation/deployment/demo handoffs | Section 12 |
| 30 | High-risk areas | Section 10 |
| 31 | Sequencing/test priority | Section 10 and Recommendations |
| 32 | Remaining questions after implementation studies | 14.3 |
| 33 | Clarification/research/prototype/community needs | 4.3, 11.3–11.4 |
| 34 | Important out-of-scope findings | 2.2, 7.5, LL-18/23/27–29 |

---

## Appendix B. Reproducible Discussion Inventory Record

### B.1 Query Boundary

```text
source: GitHub GraphQL API, Repository.discussions
repository: OS4CSAPI/os4csapi-meta
query date: 2026-08-31
discussion order: created ascending
threads returned: 14
thread number range: 1 through 43 (non-contiguous)
created range: 2025-10-19T16:25:50Z through 2026-07-20T22:59:09Z
all thread states: open
chosen answers: 0
```

### B.2 Participation Shape

```text
top-level discussion authors: Sam-Bolling
threads with comments: 4 of 14
top-level comments: 16
replies: 9
unique participants across bodies/comments/replies: 5
participants:
  doublebyte1
  EricLo-417
  nsnarayanam
  Sam-Bolling
  SpeckiJ
```

### B.3 Current Linked-State Check

```text
opengeospatial issues 78, 79, 137: closed
opengeospatial issue 186: open
camptocamp/ogc-client PR 136: open, not draft, not merged
opensensorhub/osh-core PR 318: merged
geopython/OWSLib issue 1013: closed
nlsfi/hakunapi issue 153: open
nlsfi/hakunapi PR 158: merged
opensensorhub/osh-addons PR 194: open, draft, not merged
qgis/QGIS issue 51911: open
```

### B.4 Proposal-Series Boundary

Discussions #41–#43 total approximately 268,000 characters and explicitly use a fictional exercise. They describe analytical audits, candidate profiles, and a reference architecture. Discussion #43 states that the design has not been built and that implementation experience remains open work. No discussion in the series has comments. This report therefore extracts bounded server lessons and routes the proposals; it does not validate or adopt the architecture.

---

## Appendix C. Completion and Handoff

IDR-SRV-014G is complete and was accepted by the Glaux Project Lead on August 31, 2026. Its evidence remains authoritative for downstream Glaux planning within the source and scope limits stated in this report.

The next two actions are:

1. Plan-owner acceptance of the planning amendment that inserts `IDR-SRV-014H` and revises the later Part 3 decision boundary.
2. Authorization to execute exactly one next eligible topic, `IDR-SRV-014H`.

Per the established iterative workflow, the project lead may perform both actions by replying **`proceed`** after the planning amendment is placed in review. Part 3 research and `IDR-SRV-015` both remain unstarted.

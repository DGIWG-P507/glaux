# Section 066: Aleph (Alephex) Connected Systems UI Client Study - Research Report

**Topic ID:** IDR-SRV-066<br>
**Report Status:** In Review — research complete; project-lead acceptance pending<br>
**Research Plan:** [IDR-SRV-066 plan](../IDR%20Plans/idr-srv-066-aleph-connected-systems-ui-client-study.md)<br>
**Overall Research Plan:** [Controlling overall IDR plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** Q1–Q6, through source inspection and representative test analysis. No application, identity-provider, broker or Glaux interoperability run was performed.<br>
**Methodology Used:** Pinned application and exact npm-package inspection; route-to-library workflow tracing; representative unit/component/browser tests and stories; targeted published-standard and draft-history checks; comparison with the approved Guide/Roadmap and implemented Phase 1 documentation.<br>
**Research Time:** One AI-assisted research/report iteration, October 3, 2026. Source retrieval began at 15:29 UTC; drafting and separate review followed in the same iteration. This records elapsed-session provenance, not human effort or executed tests.<br>
**Primary Sources:**
- [Alephex `7af6c076a4edec1959fc138b11d13e309baf5e67`][App]
- [Exact `cs-api-client` 0.1.3 npm archive][NpmArchive], with [closest committed library source][Lib]
- [CSAPI Part 1][P1], [Part 2][P2], and the two explicitly distinguished [selected][P3Selected] / [current inspected][P3Current] Part 3 draft snapshots

**Supporting Resources:**
- Accepted client reports [IDR-063][R063], [IDR-064][R064] and [IDR-065][R065]
- [IDR-056 external-client matrix][R056], [Guide v1.21][Guide], [Roadmap v1.40][Roadmap]
- [Glaux Server `27955c1b9260cd811ad6bc08f85feab43ad65028`][Server], [upstream-history register][History]

**Document Purpose:** Establish what the Aleph application adds to its library's expectations, which lessons inform Phase 1 and later gates, and whether to consider it for the external-client matrix. This report neither changes requirements nor authorizes client-code reuse.<br>
**Author(s):** Codex, with separate source-analysis agents; AI-assisted research<br>
**Accepted By:** TBD — the project lead's merge of this report PR constitutes acceptance under the September 29, 2026 decision<br>
**Acceptance Date:** TBD<br>
**Date:** October 3, 2026<br>
**Last Updated:** October 3, 2026

---

## Table of Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base
4. Findings by Research Question
5. Decision Analysis
6. Key Recommendations
7. Implementation Implications and Estimates
8. Risks, Constraints, and Open Questions
9. Validation Against Plan Success Criteria
10. Next Steps and Handoff
11. References
12. Appendices

---

## 1. Executive Summary

**Aleph is a useful later-stage application to test with Glaux, but it is not an unchanged Phase 1 smoke-test client.** Its library can explicitly create/read our minimal System as GeoJSON. The application instead creates Systems using SensorML, reads individual Systems using the library's SensorML default, and browses collections that Phase 1 has not implemented. Those capabilities already have later Roadmap owners. This is a difference in development stage, not evidence that Phase 1 is defective.

Three findings matter most:

- **A library that fits does not mean its whole application fits.** Aleph makes additional choices about representations, required editor fields, lists, maps and navigation. Check the actual user workflow, not merely whether its library accepts a hand-selected response.
- **Having sign-in and MQTT dependencies does not establish an authenticated streaming workflow.** Aleph disables its MQTT transport when HTTP authentication is configured. Its discovery relation, fallback topic, parent-field spelling and System Event media choice also differ from Glaux's selected experiment.
- **Its tests provide useful examples, not independent conformance proof.** Several verify exact identities and discriminating failures. Browser tests intercept HTTP with fixtures; neither those tests nor this study demonstrate a live Glaux/broker/identity-provider integration.

The upstream Part 3 working draft has also moved since Glaux's selected pin. That newer draft uses different event-type and parent-field names. This report records the change; **it does not migrate Glaux to it**.

**Recommendation:** complete this fourth client study, then resume the bounded Phase 1 test-source audit on a new `proceed` after acceptance. Consider Aleph for a later, pinned, explicitly qualified application check. Keep the approved implementation plan unchanged. No new research plan is needed to act on this report.

---

## 2. Scope and Plan Alignment

IDR-SRV-066 studies the **application above** `cs-api-client`, not the library again. IDR-065 was accepted when the project lead merged [PR #110](https://github.com/DGIWG-P507/glaux/pull/110) on September 30, 2026; that acceptance is recorded alongside this report. The October 3 `proceed` authorized this study, not the next review step or implementation.

Completed: identity/history inventory; representative browser, editor, paging, map, stream-schema, value-send, sign-in and MQTT paths; failure handling; test-source inspection; standards classification; Phase 1 and later-task comparison; four-study synthesis.

Excluded: a full application security/code audit, all presentational components, live deployment, package installation, performance measurement, maintainer contact, upstream changes, downstream planning changes and implementation. No secrets or deployment-specific configuration values are reproduced.

| Plan question | Coverage | Evidence |
|---|---|---|
| Q1 — identity, workflows and independence | Complete within source-based scope | §§3, 4.1 |
| Q2 — server capabilities and application/library attribution | Complete for selected material workflows | §§4.2–4.3 |
| Q3 — response dependencies and failure handling | Complete for selected paths; runtime outcomes unverified | §§4.2–4.3, 4.5 |
| Q4 — standards alignment | Classified against published rules, explicit Glaux choices or outside-CSAPI behavior; live compatibility unresolved | §4.4 |
| Q5 — tests, expected values and assumed server | Representative coverage, not every test; nothing executed | §4.5, Appendix A |
| Q6 — Phase 1, later gates and cross-study transfer | Complete; recommendations not adopted | §4.6, §§5–10 |

---

## 3. Evidence Base

### 3.1 Primary sources reviewed

All retrievals/checks below were on **2026-10-03**, unless an accepted report's earlier date is explicitly reused.

| Source | Version / status | Authority | Stable anchor / inspected scope | Limit |
|---|---|---|---|---|
| Alephex | `7af6c076a4edec1959fc138b11d13e309baf5e67`; package version `0.0.0` | Informative application source | [package][Package], [main data layer][Connection], router, editors, auth, live events, map/picker paths and selected tests | No build or runtime observation |
| Published client package | `cs-api-client` **0.1.3** | Informative shipped dependency | Exact archive; `dist/pubsub/events.js`, `codecs.js`, `subscription.js`, MQTT transport; IDR-065 for HTTP/model findings | Archive inspected, not installed/executed |
| CSAPI Part 1 | OGC 23-001, 1.0, published July 16, 2025 | Normative for applicable classes | Requirements 39–46, 60–61, 77–82, 89–92; linked published text | Only implicated rules, not a new standards audit |
| CSAPI Part 2 | OGC 23-002, 1.0, published July 16, 2025 | Normative for applicable classes | Schema operations, Command issue-time Table 11, JSON requirements 93–106 | Implicated rules and existing Guide interpretations |
| Published source schemas | `v1.0.0`, `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2` | Published supporting artifacts, subject to recorded contradictions | [Official tagged API tree][Schemas]; IDR-065's documented fixture/schema comparison reused | Examples and peer agreement are not independent normative rules |
| Part 3 selected baseline | `6f529a15bfa63259febc3620378d3e5a06305333` | Draft; authority only through Glaux's selected experimental choices | Resource Events and Resource Data Messages, Guide §4.8 | Not an approved OGC Part 3 |
| Part 3 current inspected branch | `72b8ec0519806688438bde0ed1cd3fe3cfd09103` | Later draft, informative here | [Comparison with selected pin][P3Compare], event-property/type changes | Not adopted into Glaux |
| OpenID Connect Core | 1.0, errata set 2 | Protocol reference, outside CSAPI resource requirements | [§3.1 Authorization Code Flow][OIDC] | Does not prescribe Glaux deployment scopes or prove this client's sign-in |

### 3.2 Supporting sources reviewed

| Source | Baseline | Use and limit |
|---|---|---|
| Glaux planning | `2105f60d7992335971c5fc8be73132160db92067`; Goal v1.10, Guide v1.21, Roadmap v1.40 | Current approved design and existing task ownership, not proof those later tasks are implemented |
| Glaux Server | `27955c1b9260cd811ad6bc08f85feab43ad65028` | Pinned [System create][ServerCreate], [System read][ServerRead], [discovery][ServerDiscovery], [authentication][ServerAuth] documentation; no server tests rerun |
| IDR-063 / 064 / 065 | Accepted reports at planning baseline, with 065 acceptance recorded in this iteration | Reused findings, especially library HTTP behavior and fixture origins; no prior study reopened |
| IDR-056 | Accepted external-client matrix | Candidate inclusion and evidence limits only; matrix unchanged |
| Upstream history | Existing register plus bounded October 3 refresh | Issues #149, #175, #201; event/discovery issues and Part 3 branch changes implicated by this application; no complete history recensus |
| Repository metadata/history | Dated GitHub API responses and full local Git history | Counts in §4.1; absence claims limited to inspected public repository |

### 3.3 Evidence quality notes

- Published standards control standards claims. Guide choices control Glaux-specific implementation decisions. Application code, tests, issue comments and drafts remain separately labelled.
- The exact npm archive's SHA-512 matches Aleph's lockfile and IDR-065: `sha512-aN54cvRElSuQ/vuEwfI57K0Rbz+2sEp1Lae9YrQyMmrCace77zK/tAuckEeS+fBqTa/r2kTUREHsFl3J9dBOZQ==`. Repository source at `a632798` is a navigable comparison, not a substitute for checking the shipped package.
- The same developer wrote Aleph, its library and CS-GO. The project lead reports AI assistance; which portions were assisted is unknown. Agreement among these projects is **not independent confirmation**.
- “Source-supported” below means what the inspected code expresses. Predictions about interoperability are **inferences**, not observations of passing/failing live requests.
- No application software was installed or run. Read-only cloning, archive download/extraction, text inspection and metadata/hash checks were performed. Node was found; `pnpm` and Docker were not found in the local command lookup. No approved isolated browser/broker/IdP run was configured for this iteration; none was required by the plan.
- No licence file or package licence field was found; GitHub reported no detected licence. Do not infer permission to copy source, fixtures or assets.

---

## 4. Findings by Research Question

### 4.1 Q1 — Identity, workflows and independence

**Source-supported finding.** The execution pin matches the plan. Its full, non-shallow Git history has **nine reachable commits**, one distinct author, July 23–September 14, 2026. The public API returned one branch, no tags, no issues and no pull requests, exhausting each collection. The repository was created August 22, 2026. “Shallow history” in the plan means limited development evidence, **not** a shallow clone.

The tracked inventory contains **526 files, 58 unit/component spec files, one Playwright spec file with 12 test declarations, and 57 Storybook files**. Four commits touch tests. That is insufficient to establish test-driven development or red-before-green practice. No tracked GitHub workflow was found in the inspected tree/history; outside CI, if any, is unknown. [Package][Package], [test configuration][Vitest], [Playwright configuration][Playwright].

The [browser route inventory][Routes] supports browsing Systems, Procedures, Properties, Deployments and Sampling Features; streams, Observations and Commands; System Events/history; Command status/results; resource editing, maps and connection/sign-in management. No Feasibility workflow appears in that route inventory. Browser route names are **not** proof that matching server endpoints are published CSAPI routes.

**Application versus library.** Aleph selects formats, scopes, editor restrictions and call sequences. The library constructs HTTP requests, handles authentication/response decoding and exposes typed resource services. Aleph also calls the library's lower-level HTTP interface for description discovery and delete previews. MQTT uses the library transport/codecs, with application-selected topics and UI event handling.

**Packaging limit.** [README][AppReadme] and [Dockerfile][Dockerfile] still describe/build a named local library context, while package/lockfile resolve npm 0.1.3. An isolated build must record both inputs and the actually resolved dependency. This source discrepancy is not proof that the build fails. Caddy serves the SPA and callback route; it does not supply a CSAPI server, broker or identity provider.

### 4.2 Q2 and Q3 — HTTP workflows and response dependencies

In the table, **A** denotes application choices; **L** denotes library request/decoding behavior. Unless stated otherwise, L is exact 0.1.3, reusing IDR-065.

| User workflow | Request / response dependency and attribution | Missing, extra or failing behavior; interpretation |
|---|---|---|
| Open a connection and browse | A eagerly loads ten resource families with `Promise.allSettled`, not a conformance-gated REST startup. A asks for GeoJSON System lists, SensorML Deployment lists; L supplies routes/headers/decoders. [Connection lines 139–150, 954–977, 1543–1608][Connection] | Successful families survive failed ones with a warning. Unsupported families can warn without proving a violation of what the server advertises. Live-event discovery is a separate path. |
| Read/create a System | A's single-System get leaves L's SensorML default; A editor requires a position and submits `format: "sml"`. After save A reloads collections, rather than navigating the created `Location` itself. [Connection lines 1777–1784, 1862–1902, 2516–2562][Connection] | A valid position-less minimal GeoJSON System is not enough to complete this editor workflow. L's ability to extract an ID from creation headers does not make A a standalone Phase 1 create/read smoke test. |
| Native filtered browsing | A maps IDs, keyword, parent/procedure/FOI, bbox, WKT, temporal and recursive controls to L parameters. Observation `dataStream` and Command `controlStream` retain spelling. Observation/Command free-text search is local filtering, not server keyword search. [Connection lines 335–745, 992–1001][Connection] | Optional control visibility does not prove server support. No application CQL2 or Observation-to-sampling-feature geometry join was found in these paths. Do not infer that a locally filtered loaded page represents all matching server data. |
| Continue a main list | A stores L's opaque `page.next` and optional `numberMatched`; explicit load-more invokes it. Search generations prevent stale responses replacing current results. [Connection lines 921–939, 1617–1735][Connection] | Failed continuation preserves current rows and reports its own error; retry is possible. Do not replace server links with manufactured offsets. |
| Select a related resource | A's [resource picker][Picker] takes first-page `items` and filters locally (lines 108–127, 170–182). Systems/Procedures/Deployments inherit L's SensorML default. | Unlike the main list, this path does **not** follow `next`. An item beyond page one can be unavailable to the picker without any server paging defect. |
| Inspect the map | A [map sources][MapSources] force GeoJSON for relevant feature families and bbox cells. [Cell query][MapQuery] probes limit 1, then bounded details; counts are optional, with `next`/threshold used for a cluster label. [Geometry mapping][Geometry] selects supported geometry/position shapes. | Does not exhaust every page. Null/unrecognized geometry is not plotted, although the resource can remain in lists. Map display is neither authoritative geometry validation nor indirect spatial filtering of Observations. |
| Preview/delete hierarchy | A [delete preview][DeletePreview] follows advertised links using their media type, traverses continuations and adds recursive subsystem selection. Preview errors disable confirmation. A deletes Deployment descendants sequentially; a System delete uses cascade. [Connection lines 2597–2658][Connection] | A later failed descendant DELETE does not undo earlier successful HTTP requests. This is not a demand for a cross-request server transaction. Link helper accepts bare / `ogc-rel:` subdeployment relations, not every full-URI spelling. |
| View/edit a stream schema | A [schema dialog][StreamSchemas] loads every advertised format with `allSettled`; successful formats remain and failed formats are named. It offers schema PUT through L. | The editor does not know all committed-data state. Showing “save” is not permission to mutate a populated stream's schema; server admission remains controlling. |
| Send an Observation or Command | A [send dialog][StreamSend] loads the chosen schema, supplies graphical JSON/SWE-JSON values or raw text, and invokes L. A generates browser-time `resultTime` / `issueTime` and may supply a sender. [Connection lines 2210–2298][Connection] | Does not establish binary/Protobuf interoperability, producer authority or physical execution. Browser-supplied Command time/sender must not override Glaux's receipt-time/identity rules. |
| See failures | A reads L's Problem Details detail/title or validation paths, then ordinary error text. [Connection lines 224–236, 1847–1858][Connection] | More informative than success-only rendering; not a guarantee safe server errors contain client-desired detail. No automatic application write retry was found in the selected save/delete paths; inherited auth behavior is IDR-065's separate finding. |

**Phase 1 comparison, source-based.** At the pinned server commit, System POST and item GET support the documented minimal GeoJSON path; unsupported SensorML writes/read requests yield 415/406, and collection GET is not implemented (405). Discovery advertises the implemented subset, not full Part 1 conformance. These are the [documented current boundaries][ServerCreate], [not a regression introduced by Aleph][ServerRead].

Consequently an unchanged Aleph workflow is expected to stop at representation/list dependencies. The underlying library with explicit GeoJSON and suitable credentials remains a different, narrower candidate. No runtime outcome was measured.

### 4.3 Q2 and Q3 — Sign-in, MQTT and their limits

#### Sign-in

**Source-supported.** [`useOidcAuth.ts`][Auth] configures authorization-code response type, a same-origin callback, configurable issuer/client/scopes, default `openid profile`, and session storage for OIDC state/user material. Saved endpoint profiles use local storage. It passes the **access token**, not the ID token, to L's OAuth provider. Automatic silent renewal is disabled, but the provider supplies an explicit refresh callback, with a 60-second refresh-before-expiry setting. With no refresh token, refresh removes the local user and requests sign-in again. Sign-out removes local state; it is not observed provider-wide logout/revocation.

**Inference / Glaux disposition.** This can fit Guide §4.10's externally issued access-token approach if the issuer, audience, token profile and permission scopes match. `openid profile` alone establishes no Glaux read/write authority. Client-accepted HTTP URLs do not relax Glaux's deployment/TLS policy. No IdP login, renewal, CORS/preflight or forbidden-resource test was run. These are outside CSAPI resource conformance, not missing CSAPI classes.

#### MQTT discovery, messages and delivery

**Source-supported.** [Client creation][Connection] (lines 250–267) constructs MQTT only when **`!auth`** and a broker URL are configured. Configured OIDC authentication therefore leaves `client.pubsub` absent. Broker URL and optional username/password come from application configuration, not AsyncAPI server selection. No endpoint, credential or example-deployment value is reproduced here.

| Dependency | Application/library evidence | Comparison with Glaux Guide §4.8 |
|---|---|---|
| Discover live channels | A [live-event module][Live] loads landing JSON, follows only `service-desc`, parses JSON/YAML AsyncAPI and classifies channels/messages heuristically (lines 244–272). | Glaux advertises its protected AsyncAPI with `urn:glaux:rel:experimental-asyncapi`. An OpenAPI `service-desc` does not make that description discoverable by A's path. Do not replace Glaux's approved relation merely to suit A. |
| Choose topics | A normalizes channel parameters to MQTT wildcards; failure can fall back to `systems/+/events`. Broker connection remains separately configured. [Live][Live] | Neither that fallback nor wildcard expansion establishes Glaux's versioned prefix/topic or an authorized audience. Generated AsyncAPI and broker-captured messages remain the authority for our local binding. |
| Decode lifecycle notifications | L accepts a finite `org.ogc.api.consys.*` set and optional `parentId`; A's [target resolver][EventTarget] also slices that prefix and reads `parentId`. Exact archive `dist/pubsub/events.js`, lines 21–46. | Same historical prefix as the selected draft, **not complete Glaux compatibility**. Glaux emits lowercase `parentid` and additional named tokens. A will not obtain the parent from that differently named field; L rejects tokens outside its set. |
| Decode native System Events | A selects `systemEvent.smlJson` ([Live][Live], lines 353–355). L expects `application/sml+json`; an explicit different content type is rejected (`dist/pubsub/codecs.js`, lines 113–117; `subscription.js`, lines 7–10). | Glaux's native System Event data uses `application/json`. Structural similarity is not media compatibility. Omitting media metadata to bypass validation would not be an acceptable fix. |
| Batch notifications | A and L support batch resource-event decoding and UI display. | Glaux deliberately excludes Batch Resource Events from its initial binding. An unavailable batch channel is not a server defect. |
| MQTT protocol / QoS | A passes URL/credentials without explicit protocol version, subscribe QoS or session policy. L's transport uses default subscribe QoS 0 and underlying MQTT-library reconnection behavior. | Does not establish an MQTT 5 / QoS 1 / zero-session-expiry subscriber matching our examples. Reconnection is not gap-free replay or proof the application processed every publication. |
| UI and buffering | A keeps a bounded visible record list and source-specific errors. L enqueues decoded messages before invoking callbacks; A uses callbacks, not the async iterator that drains that queue. | The visible-list cap is not proof of bounded subscription memory. Source suggests possible accumulation; no runtime leak measurement or server defect is claimed. |

**Important limit:** neither a discovered channel nor a successful subscription callback proves protected delivery, token revocation, recipient receipt, replay completeness or exact payload interoperability. Resource notifications and native System Event data are separate paths; successful decoding of one does not establish the other.

### 4.4 Q4 — Standards classification and upstream history

The following classifies **material dependencies**, not the entire application. Published requirement names are relative to the applicable CSAPI part's identifier base.

| Dependency | Controlling anchor / classification | Glaux consequence |
|---|---|---|
| GeoJSON versus SensorML requests | P1 Req 77–78 `/req/geojson/mediatype-read` / `mediatype-write`; Req 89–90 SensorML equivalents. **Conforming format choices with capability prerequisites.** | Preserve later SensorML work; do not call an unsupported Phase 1 representation a conformance failure. |
| Position-required editor | P1 Req 81 `/req/geojson/system-schema`, and the published schemas; A imposes its own editor precondition. **Stricter UI workflow**, not a requirement that every Glaux System be located. | Keep minimal/unlocated standards fixtures. |
| Lists, identifiers, links and counts | P1 §7.7 and inherited Features paging, including Features Part 1 `/req/core/fc-links`. A follows links in main lists but not pickers; counts remain optional in its map path. **Mixed client strategies**, not new server rules. | Opaque continuation and independent selected-ID checks already planned. |
| Native filter parameters | P1 Req 39–41 `/req/advanced-filtering/resource-by-id`, `resource-by-keyword`, `feature-by-geom`; association Req 42–46. **Aligned parameter construction**, not proven predicate semantics. | No inferred CQL2 or spatial-observation join coverage. |
| System hierarchy writes/deletes | P1 Req 60–61 `/req/create-replace-delete/system`, `system-delete-cascade`; associations under applicable resource classes. **Standard operations; client sequencing is extra.** | Per-request authority/atomicity, not transactional guarantees across A's separate requests. |
| Schema lookup and value formats | P2 Req 11 `/req/datastream/schema-op`, Req 25 `/req/controlstream/schema-op`; JSON Req 96–102. **Aligned API selection, encoding correctness unproved.** | Continue existing schema/value and wire-format tests; a graphical form is not an encoding oracle. |
| Client Command time / sender | P2 Table 11 and Guide §4.9's documented ownership interpretation. **Client-supplied values are not authoritative receipt or identity.** | Test distinct sender/receipt times and refused impersonation; no new clock policy. |
| System history view | Implicated official issue #149 and published route baseline; library exposure of legacy history is already qualified in IDR-065. **Legacy client capability**, not an inherited server obligation. | Do not add `/history` to satisfy a route in the application. |
| OIDC, browser storage, CORS | OIDC Core §3.1; Guide §4.10. **Outside CSAPI resource requirements / deployment-specific.** | Controlled provider/access-token tests, not an embedded server login system. |
| AsyncAPI relation, topic/QoS and parent spelling | Guide §4.8; selected draft Resource Events / Resource Data Messages. **Experimental binding differences**, not approved-standard defects. | Keep explicitly pinned client adaptations separate from conformance tests. |
| Batch events and newer draft event names | Two Part 3 snapshots below. **Draft and post-baseline changes.** | Neither existing client support nor a later merge changes the selected Glaux scope. |

**Normative precision:** these anchors classify the dependency; they do not certify all encoders, editor values or client models. Incorporated SWE/SensorML content validation stays with the published schema-bound rules and the existing codec owners. No new standalone SWE/SensorML obligation is inferred from an editor widget.

**Bounded history refresh.** Issues [#149](https://github.com/opengeospatial/ogcapi-connected-systems/issues/149), [#175](https://github.com/opengeospatial/ogcapi-connected-systems/issues/175) and [#201](https://github.com/opengeospatial/ogcapi-connected-systems/issues/201) remain open. The first explains removed history subresources; the second discusses future sorting; the third leaves System Event mapping unsettled. They do not supply adopted replacements for the published text or Guide §13.

The Part 3 working branch is now **12 commits ahead**, with 26 changed files against Glaux's selected `6f529a15` baseline. At `72b8ec05`, the draft:
- changes event names from `org.ogc.api.consys.*` to `org.ogc.api.csapi.*`;
- replaces `parentId` with `resourceparent`;
- incorporates representation-offering/selection work and permits the actual JSON representation media type for embedded data.

These are visible in the [pinned comparison][P3Compare]. The source still identifies itself as an SWG draft; no completed MQTT binding or AsyncAPI contract is supplied by this change. Related #190/#191/#194 are closed and PR #205 is merged; discovery discussions #14/#68/#189 remain open. Closure is informative history, not publication.

The exact npm decoder still requires the old event prefix and `application/json` for inline resource-event data; the application still reads the old parent name. **Thus “supports draft Part 3” requires a pin and a subset, not a blanket compatibility claim.** The [register][History] records this bounded change without advancing the Guide.

### 4.5 Q5 — What the tests establish, and what they do not

**Source-supported configuration:** Vitest uses jsdom and excludes the browser directory. Playwright configures three desktop browser engines and starts local Vite, not a CSAPI server, broker or identity provider. Stories exercise synthetic presentation states; no `play:` / `expect(` assertions were found in the 57 story files. Commands in package/configuration are not observed passes.

Representative tests were inspected for **setup, expected answer and discriminating assertion**:

| Source / setup | Assertion and mistake it would catch | Limit |
|---|---|---|
| [Connection tests][ConnectionTests], lines 50–111: rejected fetch / mixed synthetic HTTP | Keeps a successful System's literal ID/name while a failing family produces a warning; catches all-or-nothing loading and silent failure. | HTTP is stubbed, not a live server. |
| Same, lines 232–262: parent System create with mocked library | Root create must not run; subsystem method receives the parent and SensorML payload; catches wrong application routing. | Does not check wire encoding, 201 or `Location`. |
| Same, lines 322–406: two pages, failed continuation, selected-detail cache | Normal paging asserts ordered literal IDs; failure preserves first-page state; bbox results must not be polluted by a cached selected resource. | Retry test is less discriminating about second-page identity. No actual server cursor/access check. |
| Same, lines 439–482: populated filter controls | Checks exact library parameter arguments, including numeric bbox and WKT; catches omitted/misnamed application filters. | Does not prove server-selected IDs or geometry semantics. |
| [Map-query tests][MapTests], lines 28–83; [map-source tests][MapSourceTests], lines 77–97 | Literal sparse IDs, bounded requests, missing-count `10+` behavior; exact bbox intersection and no request for a disjoint cell. | Mock data source; not spatial server correctness. |
| [Schema tests][SchemaTests], lines 15–45 | One format succeeds while another fails; successful schema survives with format-labelled error; new format is PUT. | One expected schema is taken from current application state, not an independent complete-schema oracle. |
| [Send tests][SendTests], lines 15–40 | Numeric Observation wrapper and SWE-JSON Command method/format; catches wrong submission path. | Partial object matching; synthetic schema; no independent bytes or physical-effect proof. |
| [Browser tests][BrowserTests], lines 73–151 | Intercepts 100+5 Features, scrolls, expects continuation request and 105 cards. | Real UI/library path against canned HTTP; no real server continuation. |
| Same, lines 232–330 | Map-cell reuse and bounded requests; drawn filters appear in outgoing requests. | Nonempty bbox / `POINT(` prefix is weaker than exact intended geometry and selected-ID assertions. |
| [OIDC tests][AuthTests] and [live-event tests][LiveTests] | Mocked OIDC manager and synthetic descriptions/transports exercise profile/token callbacks, discovery and failure state. | No real authorization-code exchange, broker session, revocation or controlled identity-provider outage. |

**Expected-value independence.** Literal IDs and deliberately conflicting cache/filter fixtures provide useful regression discrimination. However, many tests mock typed `CSApiClient` methods; they bypass wire validation. Other tests run real client parsing against application-authored fixtures. Neither proves that the fixture itself conforms.

For example, the raw HTTP System fixtures in connection/browser tests use `properties.featureType: "PhysicalSystem"`, whereas the System story uses `featureType: "system"` and `processType: "PhysicalSystem"`. They cannot all be treated as interchangeable normative examples merely because the UI renders them. The Phase 1 audit should derive its expected representation from the published schema, not select whichever fixture agrees with a parser.

**Inference:** use the test techniques—exact identity checks, preserved partial success, excluded cached rows and failed-continuation recovery—as inspiration. Do not copy fixtures without licence clearance or claim test counts, configured browsers, peer agreement or round-trip success establish conformance. No test was executed in this study.

### 4.6 Q6 — Transfer to Glaux and four-study comparison

The following are **recommendations for existing review/implementation owners**, not newly authorized requirements or tasks.

| Material lesson | Existing Guide / Roadmap home | Disposition and bounded verification use |
|---|---|---|
| Minimal GeoJSON library use differs from whole Aleph flow | Guide §§4.2–4.3, 6.2; Phase 1 tasks 1.5.1–1.5.2; later 2.2.3, 2.3.1, 2.3.9 | **Already covered.** Phase 1 step 2 checks schema-derived create/read expectations; do not add list/SensorML work early. |
| Main list, picker and map have different completeness behavior | Guide §§4.1, 4.4, 8.1; tasks 2.5.7–2.5.11 | **Already covered server behavior.** Later client checks seed matching resources beyond page one and distinguish list completeness from picker/map presentation. |
| Link spelling and multi-request deletion | Guide §§4.1.1, 4.6, 13; tasks 2.4.3–2.4.4, 2.6 | **Already covered.** Check canonical links and failed descendant deletion; no requirement for cross-request rollback or client-specific output aliases. |
| Schema edits and value submission | Guide §§4.3–4.4, 4.9; tasks 3.1.6–3.1.7, 3.2, 5.1–5.2 | **Already covered.** Seed empty/populated streams, denied edits, incompatible values and distinct client/server times. No new schema-mutation permission. |
| Real issuer/token/CORS behavior | Guide §4.10; tasks 1.4.3–1.4.5 and later integration; CORS ownership question already recorded in IDR-063 | **Conditional application check.** Synthetic issuer/users, expected audience/scopes and explicit origin policy; no relaxation for UI convenience. No new CORS ownership claim here. |
| Authenticated MQTT unavailable in unchanged A | Guide §§4.8, 4.10; tasks 6.3.1, 6.3.6 | **Not evidence of a server gap.** Candidate integration needs a separately approved client adaptation or another subscriber; do not weaken authentication. |
| AsyncAPI/topic/parent/media mismatches | Guide §§4.8, 13; tasks 6.3.2–6.3.3, 6.3.5–6.3.7 | **Bounded change worth discussing only for the client harness.** Pin every adaptation and retain independent broker/message oracles. No Guide change recommended. |
| Newer Part 3 draft differs from both selected profile and client | Guide §4.8; task 6.3 and review before its implementation | **Record drift; no automatic adoption.** Recheck the selected pin deliberately at that gate; a profile migration needs its own approved change. |
| HTTP dynamic data is not live MQTT/native-codec proof | Guide §§4.3, 4.8, 8.1; tasks 3.5, 4.4–4.5, 6.3–6.5 | **Insufficient evidence for an Aleph compatibility claim.** Preserve separate binary/Protobuf, replay and audience tests; no such tests were supplied by this study. |
| External-client matrix inclusion | IDR-056; Guide §8.1; Roadmap 2.6.6, 3.5.3 and later integration owners | **Conditional recommendation.** Add as a pinned application candidate only by separate decision; do not silently replace the named OS4CSAPI check or make Aleph an oracle. |
| Test and licence limits | Guide §8.1/§8.1.1; Phase 1 review step 2 | **Already covered principle.** Standards-derived expectations and explicit unrun results; no source/fixture reuse assumed. |

#### Short cross-study note — no earlier report reopened

| Study | Reused finding / contribution | What it means for this review |
|---|---|---|
| IDR-063 — OSH Viewer / Toolkit | Mature OSH ecosystem; older SensorWeb/CSAPI assumptions; browser/CORS and stream dependencies. | Useful human-developed precedent, not a drop-in published-CSAPI oracle or unqualified Phase 1 application pass. |
| IDR-064 — OSCAR | Closer resource browsing, with OSH-specific auth/query/stream expectations and command-response differences. | Judge deviations against the standard; command request acceptance is not execution success. |
| IDR-065 — cs-client-ts | Most direct programmable path for explicit GeoJSON create/read; SensorML defaults, qualified model tolerance, fixture origins and shared authorship recorded. | Source-consistent minimal Glaux path, not executed interoperability and not independently authored across CS-GO/library/application. |
| IDR-066 — Aleph | Application chooses SensorML editing/detail, collections, bounded maps/pickers and experimental MQTT; HTTP auth disables MQTT. | Explains why a compatible library path can coexist with an incompatible current whole-UI path. Adds later workflow checks, not a Phase 1 scope expansion. |

**Reconciliation:** no accepted report is overturned. The apparently conflicting library/application conclusions concern different formats and call sequences. The four studies reinforce the need to distinguish published obligations, implementation-specific conveniences and experiments. They do **not** justify a vote among clients on what the standard means.

For Phase 1 step 2, the actionable input is small: trace expected System fields/media/creation identity to the official source; distinguish format-not-yet-supported from invalid-resource handling; preserve independent authentication/permission expectations. SensorML collection browsing and experimental streaming belong later.

---

## 5. Decision Analysis

| Option | Benefit | Cost / risk | Recommendation |
|---|---|---|---|
| Require unchanged Aleph to pass at Phase 1 | Familiar whole application | Would drag later capabilities into Phase 1 and confuse unsupported scope with defects | Reject |
| Use the library's explicit GeoJSON path as a narrow candidate | Small, inspectable request sequence already studied | Shared-author/model limitations; not whole-application evidence | Keep qualified, alongside independent standards-derived tests |
| Add pinned Aleph to a later application matrix | Exercises real UI choices, pagination, maps, edits and failures | Controlled build/browser/IdP/broker; likely client adaptations; no licence assumed | Conditional recommendation |
| Change Glaux's experiment to match Aleph or current upstream draft now | Might reduce one client's mismatches | Unreviewed scope/pin change, does not resolve all auth/media/discovery differences | Do not adopt in this study |
| Resume the agreed Phase 1 review after accepting this report | Uses completed research without reopening it | Runtime comparisons still require their approved environment | Recommended next bounded step |

---

## 6. Key Recommendations

1. **R-066-01 — Keep Phase 1 bounded; distinguish library and application claims.** High priority for the next test-source audit. Use the approved minimal GeoJSON contract; label Aleph's unmet list/SensorML expectations as later work unless the corresponding advertised behavior actually fails.
2. **R-066-02 — Consider Aleph for a later pinned application check, not as a conformance oracle.** Medium priority, conditional on a separate IDR-056/planning decision, acceptable use terms and approved isolated execution. Record app, package, browser and service pins, real server responses and every adaptation.
3. **R-066-03 — Keep authenticated HTTP, resource notifications and native System Event streaming as separate test claims.** High priority when Phase 6 is reviewed. Resolve client-side auth/discovery/topic/parent/media differences explicitly; do not disable protection or falsify content types to produce a pass.
4. **R-066-04 — Preserve the chosen Part 3 pin until a deliberate adoption decision.** Record the newer draft now; evaluate migration only within an authorized later design decision. This report does not declare the old or new draft a published standard.
5. **R-066-05 — Reuse test techniques, not unverified expected values or unlicensed fixtures.** Medium priority for review and later client checks. Prefer exact expected IDs/bytes and negative cases; explain mocked versus real dependencies and unexecuted checks.

Acceptance of this research does not automatically adopt recommendations 2–4 as new implementation work.

---

## 7. Implementation Implications and Estimates

### 7.1 Implications

No server architecture, API, task definition, issue or conformance declaration changes in this iteration. The Guide/Roadmap already own the server-side behaviors that Aleph requires later. New information mainly improves **how to select and interpret client checks**.

A later executable Aleph trial needs a controlled application build, browser, synthetic data, appropriate completed Glaux capabilities, an explicitly configured identity provider for sign-in, and a browser-reachable isolated broker only for MQTT. A runtime configuration file is browser-visible; treat any client-held credential as such. Do not use production endpoints or secrets from repository examples.

### 7.2 Effort / complexity estimate

| Work item | Relative complexity | Estimate / assumptions |
|---|---|---|
| Feed this report into Phase 1 step 2 | Low incremental preparation | Existing bounded review; not a new research plan |
| Unauthenticated feature browsing/editing trial after Phase 2 | Moderate | No hour estimate: build recipe and resolved dependency must first be verified in the approved environment |
| OIDC-protected application trial | Moderate, environment-dependent | Requires synthetic provider/access-policy/CORS configuration; source inspection is not setup completion |
| Glaux experimental MQTT application trial | Higher than HTTP-only | Multiple known compatibility differences and authenticated-pubsub exclusion; adaptation scope must be approved before estimating |
| Guide/Roadmap rewrite | Not proposed | No time assigned; no new server behavior justified by this study |

---

## 8. Risks, Constraints, and Open Questions

### 8.1 Risks and constraints

- A new application with limited public history is not mature-process evidence; passing its own fixtures could preserve a shared misconception.
- Public visibility and lack of a detected licence do not establish reuse rights.
- Source-based integration predictions may be affected by actual build/runtime behavior. The named local library build context is an additional provenance input.
- Client browser storage, tokens, runtime credentials and arbitrary configured services need a deployment-specific security review before real use; this was not such an audit.
- The selected experimental profile, current upstream draft and client's decoder are three different contracts. Keep their names and limitations visible.
- A bounded UI record list does not prove bounded lower-layer subscription buffering.
- Runtime compatibility, current upstream fixes after the pins, and conformance are not established by this report.

### 8.2 Open questions

| Question | Why unresolved / next use |
|---|---|
| Does the pinned application build reproducibly with the recorded npm dependency despite its local-context packaging? | Not executed. Resolve only for an authorized isolated client trial. |
| Which exact issuer/audience/scopes and origin policy will an Aleph test deployment use? | Deployment choice, not a CSAPI schema question. Use synthetic settings at the existing authentication/integration owner. |
| Is a small client adaptation sufficient for authenticated Glaux MQTT, including media and event naming? | Source identifies differences, not an executed solution. Decide before selecting this client as a Phase 6 proof. |
| Should Glaux later migrate its experimental Part 3 profile to the newer draft? | Requires deliberate scope/version review, not an incidental change during this application study. |
| Should Aleph be added to IDR-056, and should any existing named client check change? | Project-lead decision after research acceptance; no matrix or task changed here. |

These questions do not prevent the source-based study from completing or require another broad research cycle.

---

## 9. Validation Against Plan Success Criteria

| Plan criterion | Status | Evidence |
|---|---|---|
| Q1–Q6 answered or explicit limits | Met | §§2, 4; runtime limits explicit |
| Application/library pinned | Met | §3, exact archive integrity |
| Every request finding attributed | Met | §§4.1–4.3, A/L distinction |
| Workflows and material dependencies mapped/classified | Met | §§4.2–4.4; no whole-application certification |
| MQTT/OIDC classified; no secrets reproduced | Met | §4.3; only field/configuration roles |
| Shared author, AI-assistance and limited-history caveats | Met | §§3.3, 4.1 |
| Representative tests, server assumptions and independence | Met | §4.5, Appendix A; none executed |
| Phase 1, later gates and cross-study note | Met | §§4.2, 4.6 |
| Material lessons have dispositions and existing owners | Met | §4.6; recommendations remain conditional |
| Implicated official history checked/classified | Met | §4.4, bounded register refresh |
| Full report-template sections and success validation | Met | §§1–12 and completion checklist |

---

## 10. Next Steps and Handoff

1. **Separate reviewer:** inspect this actual research diff, source anchors, plan coverage and validation evidence before publication; record the reviewed commit and outcome in the PR. This is separate assistant review, not human expert review or research acceptance.
2. **Project lead:** review/merge the report PR if satisfied. Under the September 29 decision, that merge accepts IDR-066. Acceptance fields remain pending until the merge is recorded.
3. **On the next `proceed` after acceptance:** record acceptance and resume **Phase 1 review step 2, the test-source audit**, under its [existing charter][Phase1]. Do not start step 3, Phase 2 coding or a new research topic automatically.
4. **Later existing gate owners:** reuse this report's bounded candidates when relevant. Changes to IDR-056, the selected experimental profile, Guide or Roadmap require separate authorization.

The four-client research sequence is **researched but not all accepted** until this report's merge. Review gate 1 remains open; only the project lead closes it. No action-list handoff, implementation issue, archived completed-review evidence or server file is changed by this study.

---

## 11. References

Application links below all resolve to the same studied commit; line ranges used in the text refer to that snapshot. Library repository links are navigational companions to the integrity-checked npm archive.

- [Application tree][App], [package][Package], [lockfile][Lock], [README][AppReadme], [Dockerfile][Dockerfile].
- [Data layer][Connection], [routes][Routes], [OIDC][Auth], [live events][Live], [event targets][EventTarget].
- [Map sources][MapSources], [cell query][MapQuery], [geometry][Geometry], [picker][Picker], [delete preview][DeletePreview], [stream schema][StreamSchemas], [stream send][StreamSend].
- [Library source][Lib], [exact npm archive][NpmArchive]; shipped paths named in §3 / §4.3.
- [Published Part 1][P1], [Part 2][P2], [Features Part 1][Features], [official tagged schemas][Schemas], [OIDC Core][OIDC].
- [Selected Part 3][P3Selected], [current inspected draft][P3Current], [pinned comparison][P3Compare], [history register][History].
- [Glaux server][Server], [create][ServerCreate], [read][ServerRead], [discovery][ServerDiscovery], [authentication][ServerAuth].
- [Guide][Guide], [Roadmap][Roadmap], [Phase 1 charter][Phase1], [IDR-056][R056], [IDR-063][R063], [IDR-064][R064], [IDR-065][R065].

---

## 12. Appendices

### A. Read-depth and reproducibility record

**Inventory-wide checks:** tracked paths; spec/story counts; full reachable Git history/author count; public branches/tags/issues/PR metadata; tracked workflow/licence paths. Counts are snapshot inventory, not quality scores.

**Detailed selected source:** main data layer and its server-facing paths; route inventory; auth and live-event composables; resource event targeting; stream schema/send; map cell/query/source/geometry; picker and delete preview; representative System editor/link behavior. Presentational components outside those paths were inventoried, not deeply audited.

**Tests/configuration:** full Vitest/Playwright configuration and browser spec; connection, auth/live-event, stream-schema/send and map tests; selected filter/editor tests; three complete representative stories (System, DataStream and map). Not every one of 58 unit/component files or 57 story files received a deep read. Test execution, coverage measurement and mutation testing were not performed.

**Independent checks:** Git pin verification; archive SHA-512 versus lock/IDR-065; explicit source comparisons rather than application agreement as an oracle; requirement/Guide/task cross-checks; exact draft-to-draft comparison. No peer server was rerun, and no inference about its handling of tied latest Observations was made.

**Validation boundary:** document structure, relative links, cited source paths/ranges and diff scope are checked for publication. These checks are not server CI or application interoperability results. The reviewed commit and outcome belong in the PR delivery record.

### B. Candidate later trial, not an authorized test run

A useful future trial would record one successful and one deliberately unsuccessful case at each relevant boundary: a schema-derived resource; wrong media; a denied source; a later-page match; null geometry; stale/expired credentials; a prohibited stream-schema edit; a precise dynamic value; and separately an explicitly supported live-event channel/media combination. Seed independent expected answers first. Do not use the application's own reserialized output to define what should have happened.

This list refines existing coverage; it does not require a new test service, another research plan or an unchanged Aleph pass before Phase 1 can be reviewed.

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
- [ ] Plan-owner acceptance and acceptance date recorded — pending project-lead merge
- [x] Next steps are assigned

[App]: https://github.com/SomethingCreativeStudios/Alephex/tree/7af6c076a4edec1959fc138b11d13e309baf5e67
[Package]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/package.json#L2-L36
[Lock]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/pnpm-lock.yaml#L2022-L2023
[AppReadme]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/README.md
[Dockerfile]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/Dockerfile
[Connection]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/src/composables/useCsConnection.ts
[Routes]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/src/router/consoleRoutes.ts#L4-L49
[Auth]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/src/composables/useOidcAuth.ts
[Live]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/src/composables/useLiveEvents.ts
[EventTarget]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/src/composables/useResourceEventTarget.ts#L125-L160
[Picker]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/src/components/resource-picker/composable/useResourcePicker.ts#L108-L182
[MapSources]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/src/composables/useCsResourceMapSources.ts#L61-L196
[MapQuery]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/src/components/resource-map/resourceMapQuery.ts
[Geometry]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/src/utils/csGeometry.ts
[DeletePreview]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/src/composables/useResourceDeletePreview.ts
[StreamSchemas]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/src/components/stream-schema-dialog/composable/useStreamSchemas.ts#L13-L46
[StreamSend]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/src/components/stream-send-dialog/composable/useStreamSend.ts#L16-L48
[ConnectionTests]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/src/__tests__/useCsConnection.spec.ts
[MapTests]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/src/__tests__/resourceMapQuery.spec.ts
[MapSourceTests]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/src/__tests__/csResourceMapSources.spec.ts
[SchemaTests]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/src/__tests__/streamSchemas.spec.ts
[SendTests]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/src/__tests__/streamSend.spec.ts
[BrowserTests]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/e2e/vue.spec.ts
[AuthTests]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/src/__tests__/useOidcAuth.spec.ts
[LiveTests]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/src/__tests__/useLiveEvents.spec.ts
[Vitest]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/vitest.config.ts
[Playwright]: https://github.com/SomethingCreativeStudios/Alephex/blob/7af6c076a4edec1959fc138b11d13e309baf5e67/playwright.config.ts
[Lib]: https://github.com/SomethingCreativeStudios/cs-client-ts/tree/a6327989f5cec3c36422a121a2fe676e8a6bbe6d
[NpmArchive]: https://registry.npmjs.org/cs-api-client/-/cs-api-client-0.1.3.tgz
[P1]: https://docs.ogc.org/is/23-001/23-001.html
[P2]: https://docs.ogc.org/is/23-002/23-002.html
[Features]: https://docs.ogc.org/is/17-069r4/17-069r4.html
[Schemas]: https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api
[OIDC]: https://openid.net/specs/openid-connect-core-1_0.html#CodeFlowAuth
[P3Selected]: https://github.com/opengeospatial/ogcapi-connected-systems/tree/6f529a15bfa63259febc3620378d3e5a06305333/api/part3
[P3Current]: https://github.com/opengeospatial/ogcapi-connected-systems/tree/72b8ec0519806688438bde0ed1cd3fe3cfd09103/api/part3
[P3Compare]: https://github.com/opengeospatial/ogcapi-connected-systems/compare/6f529a15bfa63259febc3620378d3e5a06305333...72b8ec0519806688438bde0ed1cd3fe3cfd09103
[History]: ../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md
[Server]: https://github.com/DGIWG-P507/glaux-server/tree/27955c1b9260cd811ad6bc08f85feab43ad65028
[ServerCreate]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/system-create.md
[ServerRead]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/system-read.md
[ServerDiscovery]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/discovery.md
[ServerAuth]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/authentication.md
[Guide]: ../../../../../Plans/glaux-server/glaux-server-implementation-guide.md
[Roadmap]: ../../../../../Plans/glaux-server/glaux-server-roadmap.md
[Phase1]: ../../../../../Plans/glaux-server/Implementation-Reviews/Phase-1/README.md
[R056]: idr-srv-056-interoperability-test-matrix-for-external-csapi-clients-report.md
[R063]: idr-srv-063-osh-viewer-and-osh-js-client-study-report.md
[R064]: idr-srv-064-oscar-viewer-client-study-report.md
[R065]: idr-srv-065-cs-client-ts-client-library-study-report.md

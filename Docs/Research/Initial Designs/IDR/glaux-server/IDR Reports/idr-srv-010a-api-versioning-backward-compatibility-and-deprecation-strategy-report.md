# Section 010A: API Versioning, Backward Compatibility, and Deprecation Strategy - Research Report

**Topic ID:** IDR-SRV-010A<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-010A API Versioning, Backward Compatibility, and Deprecation Strategy](../IDR%20Plans/idr-srv-010a-api-versioning-backward-compatibility-and-deprecation-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All 5 core and 36 top-level detailed questions, including all 11 enumerated compatibility subareas; all six methodology phases, eleven success criteria, fourteen required content areas, and eleven minimum compatibility-matrix fields are validated<br>
**Methodology Used:** Reconciliation of accepted IDR-SRV-006 through IDR-SRV-010 with approved OGC standards; tagged OpenAPI, JSON Schema, release, issue, pull-request, and commit evidence; current HTTP, OpenAPI, and OGC policy authorities; pinned API-design guidance; and three independent read-only OGC/history, HTTP/guidance, and client/test audits<br>
**Research Time:** Approximately 5 hours of AI-assisted elapsed execution time, including three parallel independent read-only audits, on August 1, 2026<br>
**Primary Sources:**

- OGC 23-001, *OGC API - Connected Systems - Part 1: Feature Resources*, Version 1.0
- OGC 23-002, *OGC API - Connected Systems - Part 2: Dynamic Data*, Version 1.0
- OGC 17-069r4, *OGC API - Features - Part 1: Core corrigendum*, Version 1.0.1
- RFC 9745, *The Deprecation HTTP Response Header Field*
- RFC 8594, *The Sunset HTTP Header Field*
- OpenAPI Specification 3.2.0 and Semantic Versioning 2.0.0

**Supporting Resources:** Accepted IDR-SRV-001 through IDR-SRV-010 reports; official CSAPI tag `v1.0.0`; versioned OGC schema package; pinned official issue/PR history; RFCs 9110, 9111, 8288, 6906, and 5829; IANA registries; pinned Microsoft, Google, Zalando, and Heroku guidance; and pinned CSAPI Explorer evidence<br>
**Document Purpose:** Establish a decision-usable evolution policy for the Rust Glaux reference server so that standards conformance, stable resource identity, client compatibility, deprecation, extensions, generated documentation, and release testing remain coherent over time<br>
**Author:** OpenAI Codex, with independent read-only OGC/OAS/history, HTTP/guidance, and compatibility/client/test audits<br>
**Accepted By:** TBD pending Glaux Project Lead review<br>
**Acceptance Date:** TBD pending acceptance<br>
**Date:** August 1, 2026<br>
**Last Updated:** August 1, 2026

---

## 1. Executive Decision Summary

Glaux should keep one stable CSAPI 1.0 resource surface whose canonical URLs do not change with routine software releases. The Rust server and its declared public contract should use Semantic Versioning, but the Glaux release, OGC standards versions, conformance URIs, OpenAPI syntax, OpenAPI document revision, JSON Schema dialect, schema package, deployment profile, and experimental maturity must remain separate identities. A second complete API root is a last-resort migration mechanism for a genuinely unavoidable major incompatibility—not a routine `/v1` label.

Compatible releases preserve old requests, responses, URLs, link meanings, defaults, generated clients, conformance facts, and operational semantics. Additions are allowed only after old-client evidence; standards corrections can still be client-breaking. Stable deprecations preserve behavior, use exact RFC 9745 metadata and migration documentation, and retire no sooner than two stable minor releases and 12 months, whichever is longer, normally at the next major. Known safe route defects receive bounded read adapters; unsafe tasking/write aliases and redirects wait for idempotency and lifecycle research. The release gate proves the policy by running prior requests and generated/external clients against the candidate server.

The matrix and lifecycle immediately below are the actionable decision packet. In this report, **OpenAPI description (OAD)** means the machine-readable OpenAPI document for a Glaux deployment or released contract. Sections 7–13 then supply the research basis and detailed reasoning.

## 2. Glaux Breaking-Change and Compatibility Matrix

The matrix is evaluated against a specific released contract/profile, not source code in the abstract. **N** anchors are standards obligations; **G** anchors are comparative guidance; **P** handling is proposed Glaux policy. Change classes are: **A** additive, **CB** conditionally breaking, **B** breaking, **D** deprecated, **E** experimental, **I** internal, **U** uncertain pending evidence, **X** prohibited, and **NB** non-breaking metadata.

| API surface area | Example change | Class | Source guidance / anchor | Client impact | Conformance impact | Documentation impact | Test implication | Recommended handling | Downstream handoff | Notes / unresolved issues |
|---|---|---|---|---|---|---|---|---|---|---|
| Landing page | Add an accurately typed optional link or metadata member | A/CB | Features §7.2; IDR-009 | Generic clients should ignore it; strict snapshots may fail | None if required spine remains | Define relation and maturity | Old-client bootstrap and unknown-link tolerance | Minor release after compatibility lane | 014, 056 | Prevent relation collision |
| Landing page | Remove/change `data` or conformance discovery; remove both `service-desc` and `service-doc`; or break a promised link's `href`/`type` | B | Features Reqs 1–4; IDR-009 §§5–6 | Discovery/bootstrap fails | Can violate inherited classes | Migration and replacement required | Replay previous landing traversal | New major/parallel transition | 012–014, 050, 056 | The standard requires at least one API-definition form; the accepted Glaux project contract promises both |
| API root | Insert/remove `/v1` or move the deployed root | B | CSAPI `{api_root}` canonical requirements; IDR-010 | Every stored URL/link may fail | New root may conform while old contract breaks | Full root migration | Old-root traversal and canonical-ID corpus | Do not path-version routine releases | 016, 046 | Deployment prefix is contract identity |
| Conformance | Add a newly proved exact class URI | A/CB | Features Reqs 5–6; IDR-008/009 | Closed enum clients may fail | Truthfully expands claims | Release note and evidence reference | Unknown-class tolerance; prerequisite closure | Minor only after evidence gate | 050, 051, 056 | Update registry/OAD atomically |
| Conformance | Remove/change a previously declared class URI | B; X if capability simply regressed | IDR-008 §11; IDR-009 §7 | Clients lose relied-on capability | Direct conformance posture change | Major migration or emergency notice | Prior-build capability tests | Major/profile change; false claims removed immediately | 050, 051 | Truth cannot be preserved by lying |
| Conformance | List planned, partial, or experimental capability | X | Features Req 6; IDR-009 §7 | Misleads generic clients | False declaration | Must be separated | Negative leakage test | Prohibit | 014, 051 | Use Glaux extension metadata/docs |
| Collections | Add a collection or optional Part 2 family | A/CB | CSAPI collection requirements; IDR-010 | Closed type sets may fail | May accompany a new proved class | State type and maturity | Old client walks unknown collection | Minor; preserve existing IDs/defaults | 014, 015, 056 | Ordinary data members are not API changes |
| Collections | Remove/rename collection ID; change `itemType`, `featureType`, or membership semantics | B | IDR-010 §§5, 10 | Stored URLs/interpretation fail | May violate family requirements | Migration mapping | Old URL/type/membership replay | Major or coherent long-lived alias | 011, 015, 016 | Data membership changes follow separate policy |
| Routes | Add canonical/nested route without changing old routes | A/CB | CSAPI endpoint requirements; IDR-010 | Usually safe; codegen surface expands | May complete a class | Add to OAD/docs | Old client plus new route | Minor after registry/evidence gate | 014, 050, 056 | Same underlying resource identity |
| Routes | Remove, rename, recase, or reparent stable route | B | Canonical-route requirements; AIP-180 | Direct/generated clients fail | Could remain conformant at replacement yet break clients | Migration and replacement | Previous-route suite | Major; adapter if coherent | 013, 014, 016 | No silent “correction” |
| Defect aliases | Add safe GET/HEAD alias for a published route conflict | D compatibility adapter from first exposure | IDR-010 §6.5/App C; upstream defects | Helps literal clients | No new conformance class | Mark deprecated and noncanonical; identify canonical target | Strict and adapter lanes | Direct canonical handler; no ordinary advertising | 013, 014, 016, 050 | Same state/auth/ETag/canonical link; lifecycle clock starts when Glaux first exposes the alias |
| Defect aliases | Retire an alias that users could call | B for adapter clients | RFC 9745/8594; IDR-010 | Literal clients fail | Canonical conformance can remain | Advance notice and migration | Sunset/retirement suite | Normally next major; longer for standards defects | 013, 016, 056 | No Sunset merely because upstream closes issue |
| Unsafe aliases | Redirect or accept POST/PUT/PATCH/DELETE on singular tasking paths | U/high risk | IDR-007 tasking; RFC 9110 redirects | Redirect/retry can duplicate effects | Depends on transaction classes | Exact method policy required | Idempotency, auth, audit, duplicate execution | Disabled until lifecycle topics prove direct routing safe | 029, 031, 036–038 | Never blindly redirect Commands/Feasibility |
| Links | Add precise optional registered or extension link | A/CB | Features §5.2; RFC 8288; IDR-010 | Hypermedia clients benefit; closed relation sets may fail | Usually none | Define relation/cardinality/maturity | Unknown-relation tolerance | Minor; do not overload old relation | 012, 017, 056 | Stable deterministic spelling |
| Links | Remove required link or change relation/target meaning | B | CSAPI/Features link requirements | Traversal fails or reaches wrong resource | May violate selected class | Replacement and migration | Prior graph comparison | Major/deprecation transition | 014, 017, 050 | Canonical relation is identity-critical |
| Relation lexical case | Change only case of an `ogc-rel:` extension URI | NB semantically; CB snapshots | RFC 8288 comparison; IDR-010 | Correct comparison survives; brittle snapshots may not | None | Note deterministic output | Case-equivalence and golden test | Avoid churn; preserve one lexical output | 017, 056 | Case-insensitive comparison remains required |
| Legacy bare relations | Stop accepting/emitting a promised bare/`ogc-cs:` form | B if promised; otherwise adapter change | IDR-010 §7; Explorer evidence | Heuristic clients may fail | Prefixed output remains baseline | Compatibility note | Pinned-client/adversarial fixtures | Never emit as normal; input policy after write-link study | 017, 031, 056 | Client precedent is not authority |
| Response schema | Add optional response property | A/CB | AIP-180; JSON Schema/OAD | Strict decoder or closed schema may fail | Must remain valid under selected class | Version/release note | Decode with previous generated clients | Minor only when tolerance is proved | 015, 021–023, 056 | “Optional” is not automatically safe |
| Request schema | Add optional request property with unchanged omission default | A/CB | AIP-180 | Old requests survive; new clients opt in | None unless new class | Define default/behavior | Replay old payloads; omission invariant | Minor after validation | 021–024, 029 | Server must keep old accepted shape |
| Request/schema | Add required field, narrow constraints, remove accepted value, change type/presence | B | AIP-180; Azure schema guidance | Previously valid writes fail | May be correction or violation | Migration examples | Old-valid-payload replay | Major/parallel representation | 021–024, 029, 031 | Standards correction still needs migration |
| Enums/status values | Add response enum, Command status, or event type | CB; often B for closed state machines | AIP-180; Explorer closed unions | Exhaustive clients/control logic may fail | Depends on normative openness | Forward-compatible handling | Unknown-value client and state-machine tests | Treat lifecycle enums as breaking unless expressly open | 020–024, 036, 056 | Request-only extensible enums differ |
| Defaults/serialization | Change default, omit/present behavior, nullability, or field construction | B | AIP-180 semantic guidance | Same payload/request has new meaning | May still validate syntactically | Migration required | Old expectation replay | Major | 015, 021–023 | Wire-valid is not semantic-compatible |
| Schema identity | Mutate incompatible schema at same stable URL or resolver target | B | OGC package pin; IDR-006/007 | Cached validators/codegen become nondeterministic | Reproducibility harmed | Publish immutable mapping | Offline old/new closure | Version identifiers/digests; never silent replace | 023, 049, 053 | Official CSAPI files lack `$id` |
| JSON Schema dialect | Draft-07 → 2020-12 or future dialect | CB/B tooling event | Issue #87/PR #124; `$schema` semantics | Validators/generators differ | Payload may remain semantically same | Dialect/migration note | Parallel validators and tool matrix | Parallel proof before replacement | 014, 023, 056 | `$schema` is not package release |
| Query | Add optional parameter with identical absent behavior | A/CB | Features vendor-parameter rule; AIP-180 | Old calls remain valid; codegen expands | May activate optional class | Define default/interactions | Omitted-param regression | Minor release | 011, 014, 056 | Unknown-param 400 becoming accepted is additive |
| Query | Change grammar, default, max, sorting, result membership, pagination, or `latest` meaning | B | IDR-006/007; #64 | Same request returns different result/ordering | May violate class semantics | Migration required | Request/result replay, page invariants | Major unless adapter preserves old contract | 011, 018, 025, 027 | Pagination default changes are dangerous |
| Media negotiation | Add media type/alternate while retaining defaults | A/CB | Features/API-definition; RFC 9110 | Usually safe; codegen expands | May support new encoding class | Advertise exact media | Old Accept matrix; parser tests | Minor after evidence | 012, 014, 056 | Enforce `Vary: Accept` when selected by Accept |
| Media negotiation | Remove/rename media type; require profile/version; change default or precedence | B | RFC 9110; IDR-009 §8 | Existing requests/decoders fail | Can invalidate encoding claim | Deprecation/migration | Prior negotiation matrix | Major/parallel representation | 012–014 | `f` behavior owned by IDR-012 |
| Cache behavior | Omit/change `Vary` for negotiated representations | B/defect | RFC 9110 §12.5.5; RFC 9111 §4.1 | Wrong representation may be served | Interoperability failure | Document cache invariant | Proxy/cache cross-variant test | `Vary: Accept` Glaux invariant | 012, 046, 056 | Deprecation/Sunset do not alter freshness |
| OpenAPI | Add accurate operation/optional parameter/schema | A/CB | OAS; IDR-009 | Codegen methods/names may collide | None unless claim changes | Regenerate docs | Generate/compile previous and candidate clients | Minor after semantic diff | 014, 023, 056 | Text diff alone is insufficient |
| OpenAPI | Remove/change `operationId`, path, required input, schema, security, status, or media | B for generated contract | OAS; IDR-009 §§6, 13 | Old generated clients fail | OAD/runtime parity affected | Major migration | Old generated-client runtime lane | Major unless behavior truly unchanged and proven | 013, 014, 039, 056 | OAD correction may expose old runtime mismatch |
| OAS dialect | Replace OAS 3.1 with 3.2 or 3.0 only | CB/B tooling | OAS version rules; #48/#77 | Parser/generator compatibility varies | OAS3.0 class differs | Publish dialect clearly | Two parsers and old codegen | Offer alternate first; do not silently replace | 014, 023, 056 | OAS version != API contract |
| OpenAPI `info.version` | Increment document revision | NB metadata; Glaux policy requires increment on OAD change | OAS Info Object for meaning; P for increment policy | Helps traceability; no runtime selection | None alone | Changelog/snapshot | Digest and manifest consistency | Bind to immutable OAD snapshot | 014, 049 | OAS requires the field but does not prescribe Glaux's per-change increment rule; never use it alone as compatibility proof |
| Dynamic data | Add ordinary observations/events under unchanged schema/semantics | I/data evolution | IDR-007 | Expected runtime data | None | No API version note | Scenario tests | No API bump | 018, 027, 034 | Retention/order-policy change differs |
| Dynamic data | Change result schema in place, temporal meaning, default history window, ordering, or `latest` | B | IDR-007; #149/#181 | Decoders/replay fail | May violate selected classes | New stream/schema or migration | Old decoder/replay corpus | New stream identity or major | 018, 022, 023, 027, 034 | Never silently mutate live schema |
| Tasking | Add optional Command parameter without lifecycle change | CB | IDR-007; AIP-180 | Closed encoders may fail | Schema/class implications | Stability note | Prior encoder and unknown-field tests | Minor only after compatibility evidence | 022, 023, 036, 056 | Safety validation remains mandatory |
| Tasking | Change 201/202, states, terminal meaning, idempotency, result binding, cancellation, timeout | B/high safety impact | IDR-007 §§8, 11 | Control logic or physical action can fail/duplicate | May alter class fulfillment | Major safety migration | Complete prior state-machine/retry traces | Major; never hide as bug fix | 013, 029, 036–038, 055 | Highest-risk surface |
| Errors | Add optional structured extension member | A/CB | IDR-006/007 | Strict parser may fail | Usually none | Define member | Old error decoder | Minor if schema permits | 013, 023, 056 | Stable machine code preserved |
| Errors | Change status, media type, code, retryability, or meaning for same failure | B | HTTP/OAD; Explorer heuristics | Client control flow changes | Definition/runtime parity changes | Migration | Failure replay matrix | Major unless emergency | 013, 014, 048, 056 | 400/404 differences already affect Explorer |
| Security/config | Require auth, hide formerly visible links, or disable a claimed class | B for affected profile | Goal; IDR-009 | Existing client loses access/capability | Declaration may need change | Operator/client migration | Profile-specific contract diff | New profile/major or recorded emergency | 039, 040, 046, 051 | Security may justify expedited process |
| Features Part 4 | Adopt final publication over pinned draft | U until delta review | CSAPI normative reference; #141 | Mutation behavior may change | Four qualified direct classes affected | Delta ledger | Old draft vs final ATS/replay | No silent uptake; classify every delta | 013, 029, 050, 057 | Could require major |
| Experimental transport | Add SSE/WebSocket/MQTT/Part 3-like behavior | E or implementation-specific A | IDR-008 boundary; current Part 3 status | Opt-in clients only | Must not imply Parts 1/2 streaming conformance | Namespaced preview docs | Separate transport lane | Opt-in and outside core `conformsTo` | 035, 054 | Promotion needs full compatibility decision |
| Server internals | Refactor Rust, database, cache, or dependencies with identical external behavior | I | SemVer | None | None | Release note as useful | Full regression/compatibility suite | Patch normally | 044–049 | Performance/SLO promises still matter if defined |

### 2.1 Classification Rules That Prevent Common Mistakes

- “The schema validates” does not prove semantic or generated-client compatibility.
- “The field is optional” does not prove a prior decoder accepts it.
- “The standard corrected it” does not make the old-client impact disappear.
- “The endpoint is still conformant” does not make a route rename compatible.
- “The server binary did not change” does not make a disabling configuration compatible.
- “No client should depend on that” is not evidence; exercise old clients and fixtures.
- Normal resource data changes are not API-version changes unless they alter a promised schema, membership, retention, ordering, temporal, or lifecycle rule.

---

## 3. Deprecation, Sunset, Replacement, and Experimental Behavior

### 3.1 Lifecycle State Model

| State | Contract rule | Required evidence/communication | Removal rule |
|---|---|---|---|
| Experimental | Opt-in, visibly outside stable promise and core conformance | Namespaced/profiled OAD/docs, maturity, known risks, tests | May change under its stated preview policy; never silently promoted |
| Stable | Registered in the released contract/profile | OAD, compatibility manifest, generated outputs, prior-release gates | No breaking change within the major |
| Deprecated | Old behavior remains unchanged; a replacement is identified and tested when one exists, otherwise the no-replacement rationale and mitigation are explicit | Decision/date/version, OAD/schema markers, migration or containment docs, changelog, applicable RFC 9745 metadata | No removal merely because deprecated |
| Sunset scheduled | Actual retirement approved for a not-before date | All deprecation material plus RFC 8594 date/policy, support scope, tests | Sunset cannot precede deprecation; shortening is exceptional |
| Retired | Old stable entry is no longer ordinarily usable/discoverable | Documented 3xx/410/other policy, applicable replacement/migration link or explicit no-replacement rationale, frozen tests | Normally next contract major after support floor |
| Removed from current code | Compatibility implementation absent | Decision/migration record and frozen old-release fixtures retained | History remains indefinitely |

### 3.2 Exact HTTP Semantics

**RFC 9745:**

- `Deprecation` is a response field whose value **MUST** be a Structured Fields Date.
- Correct grammar: `Deprecation: @<unix-seconds>`. A real response must replace the placeholder with an integer epoch timestamp, as in the concrete example below.
- A future date says when deprecation will begin; a past date says when it began.
- Deprecation does not change the resource's behavior.
- Default scope is the resource identified by the response.
- `rel="deprecation"` points to deprecation information, which may have human-readable or machine-readable representations, and may be advertised before deprecation; the link alone does not mean the resource is deprecated.

**RFC 8594:**

- `Sunset` is a single HTTP-date indicating when a URI is expected to become unresponsive.
- It is appropriate for retirement, not merely because a resource is no longer preferred.
- It is a hint, not a promise of availability before or unavailability after the timestamp.
- It does not set cache freshness or prescribe post-Sunset 3xx/4xx behavior.
- `rel="sunset"` identifies retirement-policy information.

Example structure, with project-controlled real dates and HTTPS documentation:

```http
Deprecation: @1788134400
Sunset: Wed, 01 Dec 2027 00:00:00 GMT
Link: <https://example.org/glaux/deprecations/route-alias-x>; rel="deprecation"; type="text/html"
Link: <https://example.org/glaux/retirement-policy>; rel="sunset"; type="text/html"
```

RFC 9745 requires that a Sunset timestamp **MUST NOT** be earlier than the Deprecation timestamp. Glaux must test that standards invariant at configuration load and runtime serialization.

### 3.3 Scope and Communication Bundle

| Deprecated element | Primary machine signal | Runtime signal | Human signal | Replacement discovery |
|---|---|---|---|---|
| Resource URI/operation | OAD operation `deprecated: true` | RFC 9745 on affected responses | Changelog + migration or containment page | `rel="deprecation"`; canonical/replacement explained when applicable, otherwise no-replacement rationale |
| Parameter/header | OAD `deprecated: true` + description | No RFC 9745 field for the element alone; emit it only if the responding resource itself is deprecated | Parameter migration examples | Description/docs |
| Schema/property | JSON Schema/OAD `deprecated` annotation + description | No RFC 9745 field for the element alone; emit it only if the responding resource itself is deprecated | Representation migration guide | Description/profile/schema docs |
| Media representation | OAD media entry/description and compatibility manifest | No RFC 9745 field for the representation alone; if the resource is deprecated, emit it on the response; retain `Vary: Accept` whenever Accept selects the representation | Media migration guide | Deprecation doc; alternate type where appropriate |
| Conformance class support | Capability registry and release/profile metadata | Do not leave false URI in `/conformance` after capability disappears | Major migration/operational notice | Replacement class/profile if real |
| Compatibility alias | Alias registry, compatibility OAD/view if published | Deprecation on alias responses; canonical link | Known-defect/migration page | Canonical route |
| Experimental extension | Maturity metadata, separate OAD/profile | No stable-deprecation implication unless its preview policy promises one | Preview docs/changelog | Promotion/replacement path if any |

`successor-version` is used only when the target is genuinely the next version in a version history as defined by RFC 5829. It is not a generic “replacement endpoint” relation. For an alias or semantic migration, the deprecation documentation names the replacement; `canonical`, `alternate`, or another established relation is used only when its real semantics fit.

### 3.4 Support Window

**P-010A-03:** For a stable Glaux contract, retirement must occur no earlier than **two stable minor releases and 12 months after deprecation, whichever is longer**, and normally at the next contract major. Any applicable replacement must be available and tested before deprecation begins; when no replacement exists, the lifecycle record must state that fact and the containment or mitigation explicitly.

This is a conservative reference-server floor supported by Heroku's 12-month production precedent and bracketed by Microsoft's longer public-API evidence. It remains feasible for an open-source project that cannot identify or obtain consent from every downstream user. Published-standard-defect read aliases should remain for the full first stable major unless the project lead later approves a stronger reason to remove them.

An emergency security, safety, legal, regulatory, or standards-truth issue may shorten the window only when the project records:

1. the reason ordinary compatibility cannot be preserved;
2. affected clients and conformance facts;
3. the safest available replacement or containment;
4. communication and release dates;
5. test evidence; and
6. explicit project-lead approval.

### 3.5 Retirement Responses and Caching

- During deprecation, the old operation remains behaviorally compatible.
- A direct compatibility response is preferred while the adapter is supported.
- A 308 redirect preserves method but can still trigger client-specific follow/retry behavior and is heuristically cacheable; it is not safe by default for Commands or other writes.
- `410 Gone` is appropriate after a known permanent retirement when project/error/security policy permits acknowledging the old URI. Exact Problem Details, 3xx/404/405/410 selection, and tombstone duration belong to IDR-SRV-013/016.
- Deprecation and Sunset do not change cache freshness. Lifecycle documents and affected responses need deliberate validators/`Cache-Control` when metadata must update promptly.
- Accept-selected variants must carry `Vary: Accept` under the Glaux project invariant and use distinct validators as appropriate.
- A mutation invalidates its target URI under HTTP caching rules; it does not automatically invalidate every alias or parallel version root. Each extra surface multiplies invalidation work.

### 3.6 Known Route-Defect Adapter Policy

Canonical and advertised output remains the IDR-SRV-010 plural route baseline. Subject to implementation verification, Glaux should provide direct GET/HEAD compatibility handling for the bounded published conflicts:

- `GET`/`HEAD /controls/{id}` as an alias to `/controlstreams/{id}`;
- `GET`/`HEAD` on the published singular-parent `/controlstream/{id}/commands` and `/controlstream/{id}/feasibility` paths, mapped to their plural-parent Command-resource endpoints;
- `GET`/`HEAD` on the published singular-parent `/command/{id}/status` and `/command/{id}/result` requirement paths, mapped to their canonical plural-parent equivalents; and
- `/systems/{id}/systemEvents` as an alias to the primary nested events route.

These adapters cover only the required safe reads (and corresponding HEAD behavior). POST, PUT, PATCH, DELETE, redirects, retries, or any other mutation handling on singular tasking paths remain disabled until the downstream tasking/idempotency topics establish a safe direct-handling policy.

Each supported alias is a deprecated compatibility adapter from its first Glaux exposure. Its support-window clock begins with that exposure. Each alias must:

- resolve to the same domain resource/state, never a copy;
- enforce the same authorization and return consistent validators;
- include the canonical link to the primary plural route;
- stay out of ordinary landing/collection/navigation output and the primary OAD;
- have a named evidence defect, lifecycle record, tests, and optional privacy-reviewed telemetry;
- carry RFC 9745 deprecation metadata on every applicable alias response from first exposure; and
- never broaden a conformance claim.

Glaux should not implement stale `/history`, nonstandard `/cancel`, bare/`ogc-cs:` output, or other Explorer-specific behavior merely to make one client pass. Unsafe method aliases remain off until mutation/tasking topics prove direct routing, idempotency, authorization, audit, retry, and duplicate-execution behavior. Automatic redirects for unsafe methods are prohibited by this baseline.

### 3.7 Experimental and Profile Rules

1. Experimental behavior is explicit, opt-in where it can change semantics, and absent from core `conformsTo`.
2. Experimental routes/fields/relations use a collision-resistant Glaux namespace selected by downstream URI/OAD research.
3. Experimental OAD material is visibly labeled and, when practical, published as a separate overlay/view rather than silently mixed into a core generated-client contract.
4. Security, safety, isolation, and negative conformance tests still apply; “experimental” is not an excuse for unsafe behavior.
5. Promotion to stable requires source authority, compatibility classification, migration from the preview form, OAD/schema/runtime parity, external-client evidence, and project approval.
6. A profile must remain processable under base media semantics by an unaware client. A profile cannot change canonical identity, command state meaning, required methods, or base representation semantics invisibly.
7. Future CSAPI/SensorML/SWE/AEP work triggers a collision and delta review before an extension is renamed, adopted, or retired.

---

## 4. Documentation, OpenAPI, Conformance, and Client Implications

### 4.1 Required Metadata Separation

| Information | Required representation | Downstream exact-design owner |
|---|---|---|
| OAS syntax version | Root `openapi` | IDR-SRV-014 |
| OAD revision | `info.version`, immutable OAD URL/digest | IDR-SRV-014/049 |
| Glaux software/build version | Reproducible build/release metadata; optional OAD extension/diagnostic surface | IDR-SRV-014, 044–049 |
| Glaux contract/profile fingerprint | Compatibility manifest and generated build metadata | IDR-SRV-014, 044–051 |
| OGC standards versions | Exact conformance URIs, standards/source manifest, documentation links | IDR-SRV-014/051 |
| Schema package/dialect | Immutable URI/source/digest and `$schema` | IDR-SRV-014/023/049 |
| Deprecation | OAD/schema markers, descriptions, RFC 9745 where scoped, migration docs/changelog | IDR-SRV-012–014 |
| Experimental maturity | Separate extension/profile metadata and OAD labeling | IDR-SRV-014/035 |

Exact `x-glaux-*` extension names are intentionally not invented here. IDR-SRV-014 should select the smallest stable set, document their schemas, and keep them optional to generic tooling.

### 4.2 OpenAPI Publication Rules

1. Generate the instance OAD from the reconciled Glaux registry, not the defective official example/bundle.
2. Publish an immutable OAD for every released contract/profile and a discoverable description of the running deployment.
3. Treat operation IDs, paths, parameters, schemas, media, errors, security, and deprecation markers as a generated-client contract.
4. Mark deprecated operations, parameters, headers, and schema objects/properties where the selected OAS/JSON Schema dialect supports it; descriptions contain the effective date and, when applicable, replacement and migration reference. Native `Security Scheme Object.deprecated` exists in OAS 3.2 but not OAS 3.1, so a 3.1 view must use clear description text and any project metadata selected by IDR-SRV-014.
5. A boolean `deprecated` annotation supplies neither date nor removal policy. Documentation and applicable runtime metadata remain required.
6. Preserve deprecation state across every published OAD dialect/view; a compatibility view cannot disagree with runtime behavior.
7. Do not silently replace an OAS 3.1 OAD with 3.2 or 3.0. Offer and test an alternate first, then make a later default choice through IDR-SRV-014.
8. Increment `info.version` when the OAD snapshot changes under the selected policy, but never treat the string alone as proof of compatibility.
9. Run semantic OAD diffing and old-client generation; textual or schema-only diffs are insufficient.

### 4.3 Conformance and Staged Capability

`/conformance` lists only classes that the exact released deployment satisfies with complete prerequisites and evidence. An additive class is announced only when handlers, media/schema support, OAD, tests, and traceability agree. Deprecating a Glaux convenience feature does not alter conformance unless an actual class capability is being removed. If a runtime regression makes a declared class false, Glaux must restore the capability or stop the false claim immediately; release compatibility policy cannot justify a misleading declaration.

Planned, experimental, deferred, partially implemented, and future-standard behavior belongs in roadmap/extension metadata, never in `conformsTo`.

### 4.4 Generated Clients and External Clients

Compatibility testing must cover at least:

- clients generated from the previous stable Glaux OAD in more than one language/tool family;
- hand-written HTTP/hypermedia clients that ignore unknown additions;
- strict validators and closed enum clients that reveal risky additions;
- CSAPI Explorer pinned at its researched commit;
- OS4CSAPI client work and later implementation-study findings;
- future Glaux Publisher, Simulator, Web App, Mobile, and integration clients; and
- proxy/base-path/cache deployments.

CSAPI Explorer is valuable adversarial evidence. Its pinned version recognizes bare and `ogc-cs:` relations rather than the published `ogc-rel:` vocabulary, has closed top-level/status models, synthesizes nonstandard history/cancel paths, does not automatically exhaust pagination, and uses a server-specific Command fallback. Glaux must measure those limitations without copying them into the normative contract. A client failure caused by a closed nonstandard assumption is recorded as a client limitation; a failure caused by a Glaux regression remains a server defect.

### 4.5 Changelog and Migration Record

Every externally visible release should publish a human-readable and machine-traceable change ledger containing:

- added, compatibility-sensitive, deprecated, breaking, experimental, and implementation-only items;
- standards source/tag/overlay changes;
- contract/OAD/schema/profile versions and digests;
- deprecation effective dates, any Sunset date, applicable replacements or explicit no-replacement rationale/mitigation, and support floor;
- known external-client effects and mitigation;
- conformance classes added, removed, or still qualified; and
- test evidence or limitations.

The ledger supplements the OAD; it never substitutes for runtime truth or exact conformance declarations.

---

## 5. Compatibility and Deprecation Test Strategy

### 5.1 Release-Gate Layers

| Gate | Required proof | Failure disposition |
|---|---|---|
| Contract-registry diff | Every external registry change is detected and classified; breaking change fails patch/minor | Reclassify release or remove/adapter the change |
| Generated-output parity | Registry, router, landing, `/conformance`, OAD, schemas, docs, aliases, and tests agree | Release blocked |
| Prior stable request replay | Old valid requests retain status, media, shape, links, identifiers, defaults, and defined semantics | Breaking unless approved major/adapter |
| Previous generated clients | Old clients compile unchanged and run against candidate | Investigate operation/type/enum/media/auth/error break |
| Strict standards lane | Canonical routes, exact relations, schemas, classes, ATS overlays pass without alias dependence | Release/conformance claim blocked |
| Compatibility-adapter lane | Each alias is isolated, nonadvertised, same-state/auth/validator, canonically linked, and monitored/tested | Adapter disabled or corrected |
| Schema/dialect lane | Old requests accepted; old decoders accept responses; resolver graph immutable/closed; enum openness explicit | Release blocked or classified major |
| Query/media/cache lane | Omitted defaults invariant; page traversal stable; Accept/q/profile/media/error behavior replayed; `Vary` correct | Release blocked or major |
| Dynamic/tasking lane | Stream decoders and complete command lifecycle/retry/idempotency traces remain safe | Release blocked; high-risk break requires major/emergency review |
| Deprecation/Sunset lane | Applicable replacement works, or no-replacement rationale/mitigation is recorded; syntax/scope/date/docs/OAD agree; old behavior works before retirement | Lifecycle announcement/retirement blocked |
| External-client lane | Explorer, OS4CSAPI, generated clients, Glaux components, proxies/base paths exercised | Record server defect vs client limitation |
| Configured-profile lane | Each supported profile's contract fingerprint matches routes/classes/media/security/aliases | Profile unsupported or release blocked |

### 5.2 Required Concrete Tests

1. Parse `Deprecation: @<integer>` successfully; reject boolean and HTTP-date forms from Glaux configuration/generation.
2. Parse exactly one valid `Sunset` HTTP-date; reject Sunset earlier than Deprecation.
3. Assert deprecation does not alter old status, body, validators, link meaning, authorization, or side effects.
4. Assert resource/operation deprecations emit consistent metadata on every affected response and unaffected routes do not.
5. For field-, parameter-, or representation-only deprecation, assert OAD/schema markers and descriptions are present and no RFC 9745 response field is emitted unless the responding resource itself is also deprecated.
6. Resolve HTTPS deprecation/sunset documentation and validate RFC 8288 Link syntax/context.
7. Assert `successor-version` occurs only in an actual version chain.
8. Compare OAD `deprecated` inventory with runtime lifecycle registry, docs, changelog, aliases, and headers.
9. Drive a fake clock before deprecation, between deprecation and Sunset, at boundaries, after Sunset, and after retirement.
10. Test cache freshness of lifecycle documents and affected responses; verify Accept-selected responses have `Vary: Accept` and cannot cross-serve representations.
11. Replay every previous stable path, parameter/default, media selection, response/error shape, relation, enum, and schema fixture.
12. Generate/compile/run previous OAD clients; include unknown field, link, conformance class, collection, enum, and error-extension cases.
13. Verify every alias shares domain identity, ETag/Last-Modified policy, authorization, and canonical link; assert aliases never leak into normal navigation.
14. Verify no unsafe alias redirect or retry can execute a Command/Feasibility/mutation twice.
15. Verify deprecated behavior disappears from ordinary discovery after retirement while the documented tombstone/redirect/error remains correct.
16. Diff exact conformance URI sets and prerequisite closure for every profile/release.
17. Diff official standards/tag/schema/OAS/ATS inputs at update time and preserve overlay decisions as fixtures.
18. Retain retired and pre-correction contracts as immutable negative/compatibility fixtures.

### 5.3 Compatibility Matrix for Glaux Components

| Consumer | Highest-risk assumptions to test |
|---|---|
| Publisher | Write schemas, media, IDs, retries, transaction/status codes, requiredness |
| Simulator | Stream schema, encoding, temporal defaults, lifecycle/event values |
| Web App | Discovery links, collections, pagination, unknown additions, auth-sensitive visibility |
| Mobile | Cached contracts, offline/slow updates, Sunset warnings, compact error handling |
| External generic client | Landing/conformance/OAD, standard links/media, no proprietary selector |
| Generated client | `operationId`, types, enums, nullable/required, security, errors, content types |
| CSAPI Explorer | Relation vocabulary, closed resource/status types, pagination, Command fallback, nonstandard routes |
| Conformance harness | Exact normative paths/classes/ATS overlays without dependence on convenience aliases |

---

## 6. Downstream Topic Handoff Matrix

| Topic(s) | Required handoff from IDR-SRV-010A |
|---|---|
| IDR-SRV-011 | Classify query additions, defaults, grammar, ordering, paging tokens, result sets, `latest`, and unknown-parameter behavior; freeze omitted behavior |
| IDR-SRV-012 | Define negotiation precedence, media/profile evolution, `Vary: Accept`, lifecycle Link/header behavior, validators, and cache interaction |
| IDR-SRV-013 | Choose deprecated/retired route status/body, 3xx/404/405/410 rules, Problem Details stability, method-safe behavior, and tombstone links |
| IDR-SRV-014 | Define OAD dialect(s), immutable snapshots, `info.version`, build/contract extensions, stable `operationId`, deprecation markers, experimental overlays, and semantic diff gate |
| IDR-SRV-014A–014G | Measure real implementation/client compatibility expectations without promoting precedent to obligation |
| IDR-SRV-015 | Classify domain-model evolution; decide open versus closed values and stable semantic identities |
| IDR-SRV-016 | Define canonical URI lifetime, major-root coexistence, direct aliases versus redirects, tombstones, moved/deleted behavior, and exact metadata surface |
| IDR-SRV-017 | Freeze relation direction/cardinality/meaning; decide legacy input acceptance and valid use of canonical/alternate/successor relations |
| IDR-SRV-018/020 | Freeze temporal, history, freshness, whole-system status, Command Status, event-type, and event-identifier compatibility |
| IDR-SRV-021–024 | Define representation/schema/profile identifiers, enum openness, dialect/codec migration, and immutable resolver graph |
| IDR-SRV-029/031 | Resolve unsafe mutation aliases, idempotency, concurrency, atomicity, direct routing, data migration, and write compatibility |
| IDR-SRV-034/036/037 | Freeze dynamic data, Command, Feasibility, status/result, retry, and lifecycle semantics |
| IDR-SRV-035 | Define experimental streaming namespace/profile, coexistence, promotion, and future Part 3 collision rules |
| IDR-SRV-038–041 | Define security/safety emergency exception, authorization-sensitive visibility, audit, and client-impact recording |
| IDR-SRV-044–049 | Implement SemVer, registry/module boundaries, build fingerprint, immutable artifacts, deployment profiles, and upgrade/migration enforcement |
| IDR-SRV-050/051 | Turn classification into executable release gates and a requirement/change/test trace graph |
| IDR-SRV-052/053 | Preserve prior-release requests, generated clients, schemas, contracts, aliases, and retirement timelines as fixtures/golden corpus |
| IDR-SRV-056 | Build standards, Explorer, generated-client, Glaux-component, base-path, proxy, cache, and profile interoperability matrix |
| IDR-SRV-057 | Refresh upstream deltas and decide whether each new standards revision is compatible adoption, adapter, major transition, monitoring, or deferral |

No handoff grants permission to begin those topics or makes their unresolved choices here.

---

## 7. Research Conclusions

### 7.1 Recommended Baseline

Glaux should expose **one stable, discoverable, standards-aligned CSAPI 1.0 API root whose canonical resource URLs do not contain a Glaux release number**. Routine Glaux releases must not insert `/v1`, require an `api-version` query parameter, or require a proprietary versioned `Accept` value. Those mechanisms would alter every client request, canonical URL, stored link, cache identity, and generated client without being required by CSAPI.

The server software should use Semantic Versioning, but its software release number must remain distinct from the version of the OGC standards, the Glaux public API contract, the OpenAPI syntax, the OpenAPI document, JSON Schema dialects, individual schema/profile packages, and the conformance classes actually implemented by a deployment. A single capability/contract registry should generate the router, landing page, conformance declaration, OpenAPI description, compatibility manifest, and release tests so those views cannot drift.

After Glaux reaches `1.0.0`, patch releases should preserve the stable contract, minor releases may add compatible behavior and announce deprecations, and major releases are required for accepted breaking changes. Correcting a standards defect can still break real clients; “more conformant” is a reason to make a correction, not a reason to skip migration and compatibility analysis. A second API root should exist only when an unavoidable incompatible contract must coexist during migration, and it must be a complete API in its own right—with its own landing page, conformance declaration, collections graph, OpenAPI description, schemas, and canonical links.

### 7.2 Principal Conclusions

1. **CSAPI versions its standards, not an implementation's runtime path.** Parts 1 and 2 identify their normative provisions and conformance classes under versioned `/1.0` OGC URIs. They define canonical suffixes below an arbitrary `{api_root}` but define no server `apiVersion` field, mandatory `/v1` path, compatibility taxonomy, deprecation protocol, or retirement window.
2. **The stable default API should therefore remain unversioned at the resource-path level.** Standards versions belong in exact conformance URIs and source metadata; Glaux software releases belong in SemVer metadata and release artifacts.
3. **Canonical identity is part of compatibility.** Changing the API root, `/systems/{id}`, `/datastreams/{id}`, or another canonical route is breaking even if the replacement endpoint is otherwise conformant.
4. **Compatibility has source, wire, semantic, generated-client, conformance, and operational dimensions.** A syntactically additive OpenAPI or JSON change can still break closed enums, strict decoders, generated method names, pagination assumptions, command state machines, or external clients.
5. **Additive changes are conditional, not automatically safe.** New optional response fields, links, conformance classes, collections, media types, and enum values require old-client tests and unchanged defaults. New required inputs, narrower validation, changed defaults, removed or renamed elements, new mandatory selectors, or changed lifecycle semantics are breaking.
6. **A standards correction can be breaking.** OGC policy expressly permits a corrigendum to be incompatible when correcting an error. Every new OGC revision, finalization of the draft Features Part 4 dependency, and post-release upstream correction requires a delta and client-impact review.
7. **The official CSAPI OAS does not define an authoritative Glaux runtime-version selector or lifecycle policy.** Both tagged root files say `openapi: 3.1.0` and `info.version: 0.0.1`; that `0.0.1` is neither CSAPI Version 1.0 nor server version 0.0.1. The artifacts are incomplete support evidence, so Glaux must generate an instance-accurate description that does become a tested contract for its running release.
8. **OpenAPI version fields must not be conflated.** The root `openapi` field identifies the OAS feature set. `info.version` versions the OpenAPI document. Neither alone selects a runtime API contract or identifies the server binary.
9. **Deprecation and retirement are different states.** RFC 9745 deprecation warns consumers while preserving behavior. RFC 8594 Sunset signals an expected future loss of availability. A deprecation date never by itself authorizes removal.
10. **The correct deprecation wire form is exact.** `Deprecation` is a Structured Field Date such as `Deprecation: @1788134400`. `Deprecation: true` and an HTTP-date value are invalid under RFC 9745. `Sunset`, by contrast, uses an HTTP-date.
11. **Element-level and resource-level scope must not be confused.** OpenAPI/schema annotations are primary for a deprecated field or parameter. A `Deprecation` response header refers by default to the responding resource; Glaux should emit it for a narrower element only when the broader scope is explicitly and consistently documented.
12. **Glaux should adopt a default stable retirement floor of two stable minor releases and 12 months, whichever is longer.** Retirement should normally occur only at the next contract major. Longer support is appropriate for compatibility adapters created because published CSAPI artifacts conflict. An emergency security, safety, legal, or standards-truth exception must be recorded and tested.
13. **Known CSAPI route defects need controlled adapters, not alternate identities.** Canonical plural routes remain primary. Safe GET/HEAD aliases may directly invoke the canonical handler, share state, authorization, validators, and canonical links, and remain absent from ordinary navigation. Unsafe command/tasking write aliases and automatic redirects remain disabled until lifecycle and idempotency research proves them safe.
14. **Experimental behavior must be opt-in and visibly outside core conformance.** Profiles may signal compatible extra constraints or conventions; they must not disguise incompatible resource or command semantics. Planned or experimental features must never appear in `/conformance` as achieved classes.
15. **Release gates must run old clients against the new server.** Contract diffing, prior-release request replay, prior generated-client compilation, strict standards tests, alias tests, schema/dialect closure, negotiation/cache tests, dynamic/tasking state traces, and pinned external-client tests are all required to make the compatibility promise real.

### 7.3 Plain-Language Policy

The practical rule is simple: **keep the public CSAPI URLs and meanings stable; label Glaux software releases separately; add capabilities carefully; warn before removing anything; and prove old clients still work.** Only create another API surface when the old and new meanings truly cannot coexist.

Acceptance of this report would establish that planning baseline. It would not choose the final OAS dialect, exact build-metadata extension names, production base URL, error body after retirement, write-alias policy, or future-major URI syntax; those decisions remain with the downstream topics identified in this report.

---

## 8. Scope and Plan Alignment

### 8.1 Included

This report covers the externally observable compatibility contract for:

- landing-page and API-definition discovery;
- conformance declarations;
- collections, canonical resources, nested routes, aliases, and links;
- request and response schemas, fields, enums, defaults, and identifiers;
- query parameters, result ordering, pagination, media types, and content negotiation;
- OpenAPI documents and generated clients;
- DataStreams, Observations, ControlStreams, Commands, Feasibility, status, events, and errors;
- standards, software, contract, schema, representation, profile, and document version identities;
- stable, deprecated, sunset-scheduled, retired, and experimental lifecycle states; and
- release documentation, compatibility tests, external clients, and downstream design handoffs.

### 8.2 Excluded or Deferred

This report does not design:

- a complete product release-management or organizational approval system;
- the final query grammar, content-negotiation precedence, or error representation;
- internal Rust domain types, framework selection, database migrations, or deployment tooling;
- exact future-major URLs or multi-tenant base-path rules;
- final authorization, privacy, telemetry, or audit policy;
- final command idempotency, cancellation, or authority-transfer semantics; or
- the implementation of a future CSAPI part, profile, or unpublished OGC revision.

It provides constraints and handoffs for those topics rather than preempting them.

### 8.3 Dependency and Execution Check

The Glaux Project Lead's August 1, 2026 combined instruction accepted IDR-SRV-010 and authorized exactly IDR-SRV-010A. IDR-SRV-006 through IDR-SRV-010 are complete and accepted; the overall plan, project goal, governance sources, report template, approved OGC standards, official artifacts, and required HTTP/API guidance were available. No prerequisite exception was used and no later topic was begun.

### 8.4 Question and Deliverable Coverage

The plan contains five core questions and 36 top-level detailed questions. One detailed question expands into 11 named compatibility surface areas; all 11 are addressed. Sections 1–15 answer the questions directly, Section 16 validates all six phases and eleven success criteria, and Appendix A gives a question-by-question map. All fourteen required report areas and all eleven required matrix fields are present.

---

## 9. Evidence Base, Method, and Authority

### 9.1 Evidence Labels

This report keeps obligations, observations, analysis, and recommendations visibly separate:

| Label | Meaning | Permitted use |
|---|---|---|
| **N** | Approved normative standard or applicable standards policy | Establishes an obligation or authoritative interpretation within its scope |
| **OA** | Official published/tagged support artifact | Documents artifact content; does not override the approved standard |
| **PB** | Official repository history whose result is present in the published baseline | Explains provenance or intent; the published result remains the rule |
| **PCD** | Post-publication official direction, merge, or approved change not in a new release | Compatibility/monitoring evidence only |
| **UP** | Unresolved proposal, issue, contradiction, or unmerged work | Identifies uncertainty; creates no Glaux obligation |
| **G** | External API guideline or implementation/client precedent | Comparative guidance only |
| **A** | Analysis derived by reconciling evidence | Explains consequences; not itself a source obligation |
| **P** | Proposed Glaux planning baseline | Becomes project policy only after plan-owner acceptance and downstream implementation |

### 9.2 Pinned Source Inventory

| Evidence set | Reproducible baseline | Authority/use |
|---|---|---|
| CSAPI Part 1 | [OGC 23-001, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html) | N; Part 1 obligations and identifiers |
| CSAPI Part 2 | [OGC 23-002, Version 1.0](https://docs.ogc.org/is/23-002/23-002.html) | N; Part 2 obligations and identifiers |
| OGC API - Features | [OGC 17-069r4, Version 1.0.1](https://docs.ogc.org/is/17-069r4/17-069r4.html) | N; inherited discovery, API-definition, conformance, URI, and extension behavior |
| OGC version policy | [OGC Policy Directive 18](https://portal.ogc.org/public_ogc/directives/directives.php) | N policy; major/minor/corrigendum compatibility expectations |
| OGC API naming policy | [OGC 20-059r4](https://docs.ogc.org/pol/20-059r4.html) | N policy; versioned requirement/conformance namespaces |
| Published CSAPI source | [`v1.0.0`, commit `8e03b236`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api) | OA/PB; reproducible OAS/schema/examples baseline |
| Current official source | [`master`, commit `3fd86c73`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f) checked August 1, 2026 | PCD; current maintenance comparison |
| Upstream-history register | [Glaux register Version 1.4](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md) | Project evidence routing and disposition |
| HTTP semantics/caching/linking | [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html), [RFC 9111](https://www.rfc-editor.org/rfc/rfc9111.html), [RFC 8288](https://www.rfc-editor.org/rfc/rfc8288.html) | N; representation selection, caching, links, redirects, and status semantics |
| HTTP lifecycle | [RFC 9745](https://www.rfc-editor.org/rfc/rfc9745.html), [RFC 8594](https://www.rfc-editor.org/rfc/rfc8594.html) | RFC 9745 is Standards Track; RFC 8594 is Informational; exact deprecation and Sunset semantics |
| HTTP registries | [IANA HTTP Fields](https://www.iana.org/assignments/http-fields/http-fields.xhtml), [IANA Link Relations](https://www.iana.org/assignments/link-relations/link-relations.xhtml), checked August 1, 2026 | Authoritative current registration state |
| Profiles/version links | [RFC 6906](https://www.rfc-editor.org/rfc/rfc6906.html), [RFC 5829](https://www.rfc-editor.org/rfc/rfc5829.html) | Supporting RFC guidance; correct scope of profiles and successor-version |
| OpenAPI | [OAS 3.2.0](https://spec.openapis.org/oas/v3.2.0.html), released September 19, 2025; [OAS 3.1.2](https://spec.openapis.org/oas/v3.1.2.html) | Current official OAS semantics; tagged CSAPI artifact remains OAS 3.1.0 |
| JSON Schema | [Draft 2020-12 Core](https://json-schema.org/draft/2020-12/json-schema-core) and [Validation](https://json-schema.org/draft/2020-12/json-schema-validation) | Primary dialect, identification, and `deprecated` annotation semantics |
| Semantic Versioning | [SemVer 2.0.0](https://semver.org/spec/v2.0.0.html) | Public software/API release-label rules |
| Microsoft guidance | [`api-guidelines` commit `577874dc`](https://github.com/microsoft/api-guidelines/tree/577874dc49093ae8f6c4a5996a8f030d3ffbda7b) | G; Azure/Graph comparison, not OGC authority |
| Google guidance | [`google.aip.dev` commit `b38f2461`](https://github.com/aip-dev/google.aip.dev/tree/b38f24615f8f3b9e90d72f06df9d712a4f893ed7) | G; source/wire/semantic compatibility and stability levels |
| Zalando guidance | [`restful-api-guidelines` commit `41b74fa6`](https://github.com/zalando/restful-api-guidelines/tree/41b74fa65d9fbaaf5ad66b471563e92bd15477bb) | G; compatible evolution/deprecation comparison |
| Heroku guide/policy | [`http-api-design` commit `5fd8c92f`](https://github.com/interagent/http-api-design/tree/5fd8c92fb18c52d9234eda01e4d7959225ab610d); [policy checked August 1, 2026](https://devcenter.heroku.com/articles/api-compatibility-policy) | G; vendor-media/stability/window precedent |
| [CSAPI Explorer](https://github.com/OS4CSAPI/ogc-csapi-explorer/tree/00f1c188e05738ee03390fd95f09d351e073a9c3) | commit `00f1c188e05738ee03390fd95f09d351e073a9c3`; inspected pinned [`model.ts`](https://github.com/OS4CSAPI/ogc-csapi-explorer/blob/00f1c188e05738ee03390fd95f09d351e073a9c3/src/ogc-api/csapi/model.ts), [`helpers.ts`](https://github.com/OS4CSAPI/ogc-csapi-explorer/blob/00f1c188e05738ee03390fd95f09d351e073a9c3/src/ogc-api/csapi/helpers.ts), [`url_builder.ts`](https://github.com/OS4CSAPI/ogc-csapi-explorer/blob/00f1c188e05738ee03390fd95f09d351e073a9c3/src/ogc-api/csapi/url_builder.ts), and [`command-routing.ts`](https://github.com/OS4CSAPI/ogc-csapi-explorer/blob/00f1c188e05738ee03390fd95f09d351e073a9c3/src/ogc-api/csapi/command-routing.ts) | Supporting client evidence; never standards authority |

RFC 9745 is a material current source that the plan's static list did not name. It became a Standards Track RFC in March 2025 and is now the controlling definition of the permanent `Deprecation` field. Adding it is necessary to fulfill the plan's requirement for authoritative and current evidence; it does not expand the research topic.

### 9.3 Methodology Performed

1. Reconciled accepted IDR-SRV-006 through IDR-SRV-010 conclusions and the project goal.
2. Re-read approved CSAPI and inherited Features clauses for identifiers, API roots, canonical URLs, extension behavior, API definitions, and conformance declarations.
3. Mechanically inspected tagged Part 1/Part 2 OAS and JSON Schema fields and compared the tag with current official `master`.
4. Date-checked all versioning/compatibility-relevant upstream register entries and their linked issues, PRs, commits, tags, and release inclusion.
5. Compared exact HTTP lifecycle syntax, cache/negotiation behavior, link relations, current OpenAPI semantics, SemVer, and four guideline families.
6. Classified Glaux contract changes across every surface named by the plan and against known generated/external client failure modes.
7. Synthesized a lifecycle, alias, extension, OpenAPI/documentation, release, and compatibility-test baseline.
8. Performed three independent read-only audits, reconciled corrections, and validated the report against the plan.

### 9.4 Evidence Limitations

- CSAPI 1.0 contains no normative server deprecation or API-versioning policy; the Glaux lifecycle is necessarily project policy built on HTTP and API guidance.
- The GitHub release remains marked “prerelease,” although Parts 1 and 2 are approved OGC standards. Approved standards control; the flag is mutable repository metadata.
- No later CSAPI Part 1/2 release exists. Approved and merged post-tag work remains maintenance evidence until included in a published revision.
- Official OAS and schema artifacts contain known omissions and contradictions; they were analyzed as support evidence, never substituted for normative clauses.
- Open-source server operators cannot be assumed to report client usage. A retirement policy therefore cannot depend on complete usage telemetry or individual client consent.
- Vendor guidelines encode their own product constraints. Their selectors, custom headers, windows, and “compatible” definitions were not adopted wholesale.
- Final behavior for unsafe tasking aliases, retirement errors/redirects, OAS dialect publication, and exact metadata names depends on later topic research.

---

## 10. Versioning and Compatibility Vocabulary

### 10.1 Separate Version Domains

| Domain | What it identifies | Current/potential Glaux value | Where it belongs | What it must not mean |
|---|---|---|---|---|
| OGC standards version | Approved CSAPI/Features edition and normative identifier family | CSAPI Parts 1/2 `1.0`; Features document `1.0.1`, requirement base `1.0` | Exact conformance URIs, standards manifest, documentation | Glaux binary or runtime API selector |
| Server software version | Released Rust application/package/build | SemVer such as `0.y.z` then `1.y.z` | Git tag, package/container, build metadata, diagnostics | OGC standard or OAS syntax version |
| Public API contract version | Coherent externally promised behavior for a release/configuration | Project SemVer/fingerprint aligned with software-release policy | Immutable compatibility manifest and OAD snapshot | Automatically a URL segment or request parameter |
| Deployment profile/configuration identity | Enabled classes, extensions, media, aliases, security posture | Reproducible profile/fingerprint | Operator config manifest and generated outputs | Permission to advertise unsupported classes |
| OpenAPI Specification version | OAS feature set used to parse the document | Tagged upstream `3.1.0`; current OAS `3.2.0` | Root `openapi` field | API or implementation version |
| OpenAPI document version | Revision of one OpenAPI document | Root `info.version` | Each immutable OAD | Runtime version selector or proof of compatibility |
| JSON Schema dialect | Vocabulary used to interpret schema keywords | `https://json-schema.org/draft/2020-12/schema` | `$schema` | Schema package release/version identity |
| Schema/package version | Immutable schema set and resolution graph | OGC `/part1/1.0/...`, Glaux overlay revision/digest | Versioned URI/path, source pin, digest manifest | JSON Schema dialect |
| Representation/media version | Incompatible representation format generation, if one exists | None required for the default CSAPI surface | Registered media type/parameter only when semantically defined | General API method/URI semantics |
| Profile version | Compatible additional constraints/conventions | Future profile URI, if defined | Profile relation/parameter and profile documentation | A disguise for incompatible base semantics |
| Conformance state | Classes actually satisfied by this running deployment | Exact `/conf/...` URIs | `/conformance`, generated OAD/evidence registry | Roadmap, partial support, or marketing claim |
| Resource data revision | Evolution of an individual System, Observation, Command, etc. | Entity-specific revision/ETag/valid time | Resource lifecycle and conditional requests | API contract version |
| Experimental maturity | Stability promise for an opt-in extension | experimental/preview/stable/deprecated | Separate extension registry/docs/OAD markings | OGC conformance class unless defined and proved |

### 10.2 Compatibility Dimensions

| Dimension | Test question |
|---|---|
| Source compatibility | Does code generated or written against the old contract still compile? |
| Wire compatibility | Can old requests and representations still be serialized, transmitted, parsed, and validated? |
| Semantic compatibility | Does the same valid interaction retain the meaning and outcome a reasonable old client relied on? |
| Hypermedia/identity compatibility | Do stored canonical URLs, relation types, link directions, and traversal paths retain their identity and meaning? |
| Generated-client compatibility | Do old operation IDs, types, enum handling, auth, errors, and method signatures still work? |
| Conformance compatibility | Does the deployment still truthfully satisfy every class it advertised, including prerequisites and operation/media details? |
| Operational compatibility | Do retry, idempotency, caching, pagination, timing, stream decoding, and command lifecycle expectations remain safe? |
| Configuration compatibility | Does the same supported deployment profile expose the same promised capabilities and behavior? |

An API change is “compatible” only when the relevant dimensions for its consumers remain compatible. Passing a schema diff alone is insufficient.

### 10.3 Glaux Change Classes

| Code | Class | Meaning and release handling |
|---|---|---|
| **NB** | Non-breaking | No previously valid stable request becomes invalid and no promised response, identity, or semantic behavior changes; patch or minor depending on scope |
| **A** | Additive | New optional behavior preserves prior defaults and semantics; minor release after compatibility tests |
| **CB** | Compatibility-sensitive additive | Additive by design but may break strict schemas, enums, code generators, or known clients; minor only after explicit evidence and mitigation |
| **B** | Breaking | A valid stable request, response, URL, link, identifier, lifecycle, generated client, or conformance fact no longer works or changes meaning; new contract major/parallel transition |
| **D** | Deprecated | Still operational and semantically unchanged, with an applicable replacement or explicit no-replacement rationale/mitigation plus future-risk communication; minor release under SemVer |
| **E** | Experimental | Opt-in and outside the stable compatibility promise and standards conformance declaration; changes follow its published maturity policy |
| **I** | Implementation-only | Internal change with no externally observable contract effect; patch release is normally sufficient |
| **U** | Unresolved/conditional | Classification depends on a downstream semantic or safety decision; cannot enter a stable release until resolved |
| **X** | Invalid state | Misleading conformance claim, undocumented incompatible change, or lifecycle communication that violates controlling syntax/scope; must be corrected, with compatibility handling if clients were exposed |

### 10.4 Directional Rules

- A new optional **request** field is normally additive only when omission preserves old behavior and servers continue accepting old payloads.
- A new **response** field is compatibility-sensitive because strict decoders and generated clients may reject it; the old schema must permit it or the client lane must prove tolerance.
- Widening accepted input can be additive; widening output value sets, string formats, or enums can break clients that validate or exhaustively switch.
- Narrowing accepted input, increasing requiredness, removing fields or values, changing defaults, or changing presence/serialization behavior is breaking.
- A new endpoint, collection, link, or conformance class can be additive; removing, renaming, recasing, reparenting, or changing its meaning is breaking.
- Correcting a nonconforming old behavior is classified by client impact first, then migrated in a standards-truthful way.

---

## 11. OGC/CSAPI Versioning and Evolution Constraints

### 11.1 What the Standards Actually Version

**N:** CSAPI Part 1 is approved OGC 23-001 Version 1.0 and uses normative base `http://www.opengis.net/spec/ogcapi-connectedsystems-1/1.0`. Part 2 is approved OGC 23-002 Version 1.0 and uses `http://www.opengis.net/spec/ogcapi-connectedsystems-2/1.0`. Their requirement and conformance identifiers are relative to those bases. OGC API naming policy likewise requires a part and standards version in requirement/conformance namespaces.

**N:** Features is a useful counterexample to conflation. The corrigendum document is Version 1.0.1, but its requirement and conformance base remains `/ogcapi-features-1/1.0`. A document correction did not create a mandatory `/1.0.1` runtime API root.

**A:** Exact conformance URIs are the authoritative runtime signal for the implemented conformance class and its major/minor normative namespace. They do not distinguish a corrigendum or document edition such as Features 1.0.0 versus 1.0.1; Glaux must pin the exact adopted document edition in its standards manifest. Glaux must not rewrite conformance URIs to embed a server release or treat their `/1.0` segment as a template for its own resource paths.

### 11.2 API Root and Canonical Identity

**N:** CSAPI Part 1 §7.6 and Part 2 §7.4 define endpoint suffixes below `{api_root}`. Family requirements then require canonical resource URLs such as `{api_root}/systems/{id}` and canonical links from alternate access locations. Features treats paths as relative to the API base/landing page and leaves the containing base deployment path to the implementation.

**A:** The standards neither require nor categorically prohibit a version segment above those suffixes. However, once Glaux publishes an API root, moving it or adding/removing `/v1` changes every canonical identity, landing link, collection link, nested link, stored bookmark, cache key, and generated operation target. That is a breaking contract change even if the new root is conformant.

**P:** Glaux's default CSAPI 1.0 root should be stable and unversioned by Glaux software release. An operator's fixed deployment prefix is part of that deployed API's identity but is not automatically an API-versioning scheme.

### 11.3 Discovery, API Definitions, and Conformance

**N:** Features requires landing-page discovery, at least one linked machine-readable or human-readable API definition, fixed `/conformance`, and exact declarations of implemented classes. It explicitly says the API definition is metadata about the API and is strictly not part of the API itself; it may be hosted separately and multiple formats may exist. The accepted Glaux project baseline chooses to publish both API-definition forms.

**A/P:** This separation permits immutable versioned OAD/documentation snapshots without changing resource URLs. The landing page can continue linking the description of the running contract, while release archives preserve older OADs. `/conformance` remains a truth statement about the running deployment, not a version negotiation resource.

**N:** Features 1.0.1 provides an explicit compatibility example for collection `self`/`alternate` links: they were added as recommendations to avoid a breaking change to 1.0.0 implementations. The corrigendum also added previously omitted landing-page `self`/`alternate` links only as recommendations, although that landing-page note does not repeat the explicit breaking-change rationale. Together, these changes show that standards evolution itself treats apparently useful additions with compatibility care.

### 11.4 Extensions and Profiles

**N:** CSAPI permits additional encodings through extensions, and Features permits added links, operations, and declared vendor parameters. Those permissions do not turn project behavior into a CSAPI class or define its stability policy.

**Informational RFC / IANA relation semantics:** RFC 6906 profiles add constraints, conventions, or additional semantics without changing the representation's base semantics for clients that do not know the profile.

**P:** A Glaux extension must be namespaced, documented, opt-in where it can affect behavior, represented separately from core conformance, and collision-reviewed against future CSAPI/SensorML/SWE/AEP work. A profile is appropriate only if an unaware client can still safely process the base representation. Incompatible method, URI, command, or lifecycle behavior needs a separate contract/surface, not a profile label.

### 11.5 OGC Standards Revision Policy

**N:** OGC Policy Directive 18 uses major, minor, and corrigendum versions. A minor revision is expected to be backward compatible within a major. A corrigendum corrects errors but may itself be incompatible because erroneous behavior or artifacts are being repaired; it must not be used to smuggle in new features.

**P:** Glaux must pin each adopted standards package, diff every later revision/corrigendum, classify its effects against Glaux's contract and real clients, and record any compatibility adapter. It must never auto-track mutable OGC files. Final publication of the currently draft Features Part 4 dependency requires a full delta review before transaction classes or behavior change.

### 11.6 Tagged OpenAPI and JSON Schema Findings

| Finding | Evidence class | Compatibility consequence |
|---|---|---|
| Both Part 1 and Part 2 root OAS files say `openapi: 3.1.0` | OA, tag `8e03b236` | OAS syntax/tooling identity only |
| Both say `info.version: 0.0.1` | OA | Versions those support documents; not CSAPI 1.0 and not server software |
| Both use identical unversioned demo server URLs | OA | No official precedent for mandatory runtime path versioning; demo URLs are not Glaux requirements |
| Neither tree has an API-version/schema-version field or deprecation annotation; `x-logo`, inside each Info Object, is the only specification extension observed in the two root files | OA | The support artifacts do not supply a lifecycle policy |
| All 69 tagged Part 1/2 JSON schema files use Draft 2020-12 `$schema`; none has `$id` | OA | `$schema` is dialect, not package identity; retrieval/base URI matters |
| Official hosted schema paths contain `/part1/1.0/` or `/part2/1.0/` | OA | Package version identity comes from the versioned distribution path/source pin |
| Issue #87/PR #124 migrated schemas from Draft-07 to 2020-12 before publication | PB | Dialect changes are real validator/tooling compatibility events |
| Released bundles retain 32 Part 1 and 51 Part 2 relative references | OA | Glaux cannot publish them unchanged as a closed instance contract |

OAS 3.2.0 now states particularly clearly that root `openapi` is unrelated to `info.version`, and that `info.version` is distinct from the OAS version, API version, and implementation version. Glaux should reflect that distinction even if IDR-SRV-014 selects OAS 3.1.x for near-term tool compatibility.

### 11.7 Official Release and Repository History

The live August 1, 2026 refresh found no material change from register Version 1.3:

- tag `v1.0.0` remains commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`;
- current `master` remains `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f`;
- there are still 141 issues (60 open, 81 closed) and 58 PRs (4 open, 51 merged, 3 closed unmerged);
- no Part 1/2 1.0.x or 1.1 release exists; and
- the only Part 1/2 API-tree commit after the tag remains PR #176's three example relation corrections in one file.

| Evidence | State/class | Present meaning for Glaux |
|---|---|---|
| `v1.0.0` release/tag | OA/PB | Published support baseline; GitHub prerelease flag does not make approved standards drafts |
| #23 | Open, UP | Conformance classes do not express every encoding×method detail; OAD/runtime registry must remain exact |
| #48 and #77 | Open, Mixed/UP | OAS 3.1 was published; discussed 3.0 compatibility remains unrealized; do not claim OAS 3.0 class from a 3.1 file |
| #87/PR #124 | Closed/merged in tag, PB | Published schema-dialect migration; dialect evolution needs tooling tests |
| #141 | Open, UP | Features Part 4 dependency remains draft; publication requires a delta gate |
| #149 | Open, UP/PCD | System history/version-resource direction remains unresolved; stale OAS paths do not create a CSAPI 1.0 feature |
| #169/PR #196 | Open/approved/unmerged, PCD | Required `recursive` OAS correction is not in a release; Glaux implements the normative rule through a documented overlay |
| #170 | Open, UP | PATCH formats/semantics remain unresolved; adding a method now can create later compatibility risk |
| #173/PR #176 | Closed/merged after tag, PCD | Partial example relation correction only; no silent new Version 1.0 baseline |
| #174/PR #199 | Open/approved/unmerged, PCD | Optional Procedure `validTime` schema correction remains unreleased |
| #177 | Open, UP | Required deployment stream routes remain absent from OAS; the standard plus overlay controls |
| #186 | Open, UP | Versioned OGC-hosted standalone bundle is a proposal, contradicted by current residual references |

**A/P:** Glaux's compatibility manifest should pin the published tag, schema/OAS hashes, normative overlays, upstream state, extension state, and aliases. Adopting an approved-standard rule that a defective OAS omitted is not a new standards version, but the divergence must be documented and tested. An open/approved upstream change that goes beyond the published obligation remains monitoring evidence.

### 11.8 AEP-4789 Context

Accepted IDR-SRV-001 through IDR-SRV-003 establish that AEP-4789 adopts the CSAPI/SensorML/SWE package as the implementation basis; it does not authorize a separate incompatible Glaux wire API. Compatibility should therefore preserve that package's exact normative identities and interoperability while allowing implementation releases to evolve independently. A Glaux extension may support AEP operational needs, but it must remain distinguishable from the adopted standards baseline and cannot weaken direct server obligations.

---

## 12. Versioning and Deprecation Approach Comparison

### 12.1 Runtime Versioning Options

| Approach | Benefit | Cost/risk for Glaux | OGC/CSAPI fit | Recommendation |
|---|---|---|---|---|
| One stable unversioned resource surface | Preserves canonical identity, discovery, ordinary clients, cache simplicity | Requires disciplined compatible evolution | Strong; CSAPI already identifies standard classes separately | **Default** |
| `/v1` URI path | Visible selector, separate cache identity | Rewrites every canonical URL/link; propagates through hypermedia and generated clients; encourages parallel trees | Allowed above `{api_root}` but not required | Reject for routine releases; consider only for an unavoidable major root |
| `api-version` query | Explicit per request | Pollutes every link and pagination URL; missing selector breaks normal CSAPI clients; multiplies cache/validation cases | Azure-specific, not CSAPI convention | Reject as default |
| Custom version request header | Keeps URI stable | Hidden in ordinary links; tooling/caches need extra handling; custom client burden | No CSAPI support | Reject as default |
| Vendor media-type version | Can separate incompatible representations at same URI | Required proprietary `Accept` can break generic/OGC clients; versions payload only, not arbitrary method/URI semantics; requires `Vary: Accept` | Use only for a genuinely defined representation generation | Not general API versioning |
| Profile relation/parameter | Identifies compatible constraints/conventions | Cannot safely carry incompatible base semantics | Useful for compatible Glaux/AEP profiles | Conditional, not a breaking-change escape hatch |
| OpenAPI `info.version` | Tracks immutable OAD revisions | Not a runtime selector; consumers may conflate it | Fully compatible as document metadata | Required for OAD revisioning, not HTTP routing |
| SemVer server/contract label | Clear release impact when public API is defined | Does not select resources; requires disciplined classification | Independent of OGC paths | Adopt for Glaux releases/contract policy |
| Separate complete API root | Allows incompatible majors to coexist | Doubles routing, data consistency, conformance, docs, tests, security, caches, and operations | Permissible if each root is complete and truthful | Last resort for a genuine incompatible major |

### 12.2 External Guidance Comparison

| Source | Mechanism | Compatibility rule | Deprecation/retirement | Glaux applicability |
|---|---|---|---|---|
| RFC 9745 + RFC 8594 | No API-version scheme; standard lifecycle headers/links | Deprecation does not change behavior | `Deprecation: @epoch`; deprecation link; distinct `Sunset: HTTP-date` and sunset policy link | Controlling lifecycle behavior; adopt exact syntax and scope |
| SemVer 2.0.0 | `X.Y.Z` release label after declaring a public API | Patch compatible fixes; minor compatible functionality/deprecation; major incompatible | Deprecation increments minor | Use for Rust/software and project contract policy; never a selector by itself |
| Microsoft Azure, pinned `577874dc` | Required date `api-version` query; forbids path version | Existing workloads should not break | Proprietary `azure-deprecating`; preview policy | Compatibility principle useful; selector/header rejected as Azure-specific |
| Microsoft Graph, same pin | GA/beta model and element versioning | Additive default; code/behavior/contract changes that require clients to change are breaking | 24/36-month GA evidence; stale draft header examples | Taxonomy/window evidence useful; OData/header syntax not reusable |
| Google AIP-180/181, pinned `b38f2461` | Major versions plus alpha/beta/stable levels | Source, wire, semantic compatibility; additions conditional | Beta period defined up front; stable break means next major; emergency exception | Strong generic classification baseline; protobuf-specific details adapted |
| Zalando, pinned `41b74fa6` | Avoid versions; media type if unavoidable; no URL version | Directional input/output evolution and tolerant readers | OAD flags, docs, headers, consent/monitoring | Useful taxonomy; several current examples conflict with RFC/HTTP and are rejected |
| Heroku, pinned guide/current policy | Required versioned vendor `Accept` | Stability-tier policy | 1/6/12-month minima; 410 after retirement | Window/stability evidence useful; proprietary selector and undocumented-behavior rule rejected |

### 12.3 Guidance That Must Not Be Copied

1. Zalando's current `Deprecation: true` example violates RFC 9745's MUST-be-Date syntax. Glaux emits only a Structured Fields Date.
2. Zalando's `Vary: Content-Type` example is wrong when request `Accept` selected the response. Glaux enforces `Vary: Accept` as a project invariant for every cacheable Accept-selected representation.
3. Zalando's preference against deprecation/sunset Link relations is organizational, not a Web standard. Glaux benefits from discoverable HTTPS lifecycle documentation.
4. Microsoft's proprietary `azure-deprecating` header and stale draft `Deprecation` examples are not used; standardized RFC 9745/8594 fields control.
5. Azure's required query selector and Heroku's required vendor `Accept` would change ordinary CSAPI clients and conformance tests; they are not default Glaux mechanisms.
6. Heroku treats some undocumented behavior changes as compatible. Glaux instead follows semantic compatibility: reasonable visible client reliance still matters.
7. No profile, OpenAPI document number, Git tag, or OGC release flag may be used to imply a runtime contract different from what the router and `/conformance` actually expose.

---

## 13. Proposed Glaux Versioning and Release Policy

### 13.1 One Primary Contract

**P-010A-01:** Glaux publishes one primary CSAPI 1.0 API root. Its canonical routes, relation meanings, defaults, content-negotiation behavior, and conformance declaration comprise the stable HTTP contract. The root is not rewritten for each Glaux software release.

**P-010A-02:** The capability/contract registry is the source of truth for enabled classes, routes, methods, media types, schemas, aliases, deprecation state, extension maturity, and release profile. Router composition, landing links, `/conformance`, OAD, compatibility manifest, and tests are generated from or validated against that registry.

### 13.2 SemVer and Contract Stability

| Release stage | Public promise |
|---|---|
| `0.y.z` experimental development | The whole Glaux contract is pre-stable, but every release remains standards-truthful, source-pinned, semantically diffed, and regression-tested. Breaks are announced and occur at a new minor, not silently in a patch. Public preview features state their own window. |
| `1.0.0` | Defines the first stable Glaux public contract and freezes canonical identity/semantics for the major line. |
| `x.y.Z` patch, `x > 0` | Internal changes and backward-compatible fixes only. A “bug fix” with material client impact is not automatically patch-safe. |
| `x.Y.0` minor, `x > 0` | Compatible additions and deprecation announcements. Existing requests, defaults, semantics, and conformance claims remain valid. |
| `X.0.0` major | May introduce an approved breaking contract after migration planning. Old/new coexistence is an explicit project and operational decision, not automatic. |

SemVer applies to the declared Glaux public contract and software release. It does not turn a release number into a request selector or OGC standards version. `info.version` may be deliberately aligned with a contract snapshot by IDR-SRV-014, but its OpenAPI meaning remains “document version.”

### 13.3 Contract Fingerprint

Every released deployment profile should have a reproducible fingerprint covering at least:

- Glaux software version and source commit;
- Glaux public contract/OAD revision;
- approved OGC documents and exact conformance URIs;
- official CSAPI tag/schema hashes and Glaux normative overlays;
- OAS and JSON Schema dialects;
- enabled routes, methods, media, query defaults, and error contract;
- enabled compatibility aliases and lifecycle dates;
- experimental/profile features and their maturity; and
- enabled security/configuration profile that changes the public surface.

A binary version alone cannot describe a deployment whose operator disables a class, alias, media type, or route. Such a configuration change must produce a different contract/profile identity and truthful generated outputs.

### 13.4 When a New API Root Is Justified

A second root is justified only when:

1. the new behavior is genuinely incompatible and cannot be represented as an additive new resource, link, field, media representation, or extension;
2. the benefit and standards alignment outweigh the permanent maintenance cost;
3. the project lead approves a contract major and migration policy;
4. old and new roots can be served and tested coherently for the support window; and
5. each root has a complete landing page, conformance declaration, collections graph, OAD, schemas, canonical links, security behavior, and lifecycle documentation.

The exact future-major root syntax is deferred to IDR-SRV-016 and deployment research. This report deliberately does not preselect `/v2`, a host name, or a query/header selector.

### 13.5 Standards Update Intake

For every OGC revision, corrigendum, newly final dependency, or post-tag candidate change:

1. pin the authoritative artifact and source state;
2. diff normative text, ATS, schemas, OAS, examples, and identifiers separately;
3. classify each change against the current Glaux contract and client corpus;
4. implement existing approved obligations through a documented overlay where support artifacts are defective;
5. never call an unmerged/merged-but-unreleased change part of the published standard;
6. choose compatible adoption, an adapter, a major transition, monitoring, or explicit deferral; and
7. update conformance claims only when implementation and evidence are complete.

---

## 14. Recommendations

Acceptance of this report would adopt the following planning baseline for downstream design. These are project recommendations, not statements that CSAPI itself mandates Glaux's release policy.

1. **Keep one primary, stable CSAPI 1.0 API root.** Do not add `/v1`, a required `api-version`, or a proprietary versioned `Accept` value for routine Glaux releases.
2. **Version the Rust server with SemVer after declaring the public contract.** Patch means compatible fix, minor means compatible addition or deprecation, and major means accepted incompatible contract change.
3. **Keep all version identities separate.** Standards version, software release, contract/profile fingerprint, OAS syntax, OAD revision, JSON Schema dialect, schema package, representation/profile, conformance state, and resource revision must never share one ambiguous field.
4. **Build one versioned capability/contract registry.** Router, discovery, conformance, OAD, schemas, aliases, lifecycle metadata, compatibility manifest, and tests must agree with it for the exact deployment profile.
5. **Preserve canonical URLs and semantics throughout a stable major.** Treat route/root/relation/identifier/default/lifecycle changes as breaking even when a replacement remains standards-conformant.
6. **Classify additions directionally and test real old clients.** Optional response fields, enum values, links, collections, classes, media, and OAD operations are compatibility-sensitive until old decoders/generators prove tolerance.
7. **Generate immutable, instance-accurate OAD and schema artifacts.** Pin standards/tag/schema sources and digests; maintain a machine-readable normative overlay; never publish the defective official bundles unchanged as the Glaux contract.
8. **Use exact standardized lifecycle signals.** Emit only RFC 9745 Structured Date syntax for `Deprecation`; use RFC 8594 HTTP-date for a real Sunset; link HTTPS migration/retirement documentation; never use proprietary or obsolete draft forms.
9. **Adopt the stable retirement floor.** Keep deprecated stable behavior for at least two stable minor releases and 12 months, whichever is longer, normally until the next major; record any emergency exception.
10. **Provide bounded read adapters for known published route defects.** Canonical plural paths remain primary; safe aliases share state/auth/validators/canonical links and remain unadvertised. Keep published-defect read adapters through the first stable major unless later evidence justifies more.
11. **Prohibit unsafe automatic tasking/write redirects.** Mutation aliases remain disabled until downstream lifecycle/idempotency research proves that direct handling cannot duplicate or misdirect physical/control effects.
12. **Isolate experimental extensions and profiles.** Use collision-resistant naming, explicit maturity, separate documentation/OAD treatment, negative conformance tests, and promotion gates. Never list planned/preview behavior as achieved CSAPI conformance.
13. **Run compatibility as a release gate.** Semantic registry diff, prior request replay, old generated clients, strict conformance, adapters, schemas/dialects, negotiation/cache, dynamic/tasking, lifecycle, external clients, and configuration profiles all gate releases.
14. **Treat every standards update as a compatibility event.** Diff approved text, ATS, schemas, OAS, examples, and identifiers; distinguish published obligation from artifact defect and post-release direction; never auto-adopt mutable upstream state.
15. **Create a second complete root only for a genuine major incompatibility.** Require project approval, a migration plan, old/new coexistence proof, independent conformance truth, and the full support window.

### 14.1 Acceptance Decisions Embedded in the Baseline

Plan-owner acceptance would specifically approve these decisions for later implementation planning:

- stable unversioned-by-release CSAPI resource URLs as the default;
- SemVer-governed software/public-contract releases;
- additive-only stable minors and breaking-change major boundaries;
- the two-minor/12-month stable retirement floor;
- direct same-handler GET/HEAD compatibility adapters for the bounded published route conflicts, subject to downstream route/security verification;
- no unsafe mutation/tasking redirects without later proof;
- exact RFC 9745/RFC 8594 lifecycle semantics;
- experimental behavior outside core conformance; and
- prior-client/contract replay as a mandatory release-quality gate.

---

## 15. Risks, Constraints, and Open Questions

### 15.1 Principal Risks and Controls

| Risk | Failure mode | Control established here |
|---|---|---|
| Over-versioning | Fragmented roots, canonical identities, caches, clients, tests, and operations | One primary root; new root only for a genuine major |
| Under-versioning | Silent breaking changes under the same release/contract identity | SemVer, registry diff, old-client gates, major classification |
| Version conflation | `info.version`, `/1.0` conformance URI, server release, and schema dialect mislead users | Separate version-domain model and fingerprint |
| False conformance | Planned/partial/deprecated-removed capability remains in `/conformance` | Evidence-gated generated declaration; truth overrides compatibility theater |
| Standards auto-update | Mutable upstream or corrigendum silently changes behavior | Pinned package, authority classes, delta/adoption gate |
| Artifact-driven implementation | Defective OAS/example becomes accidental API contract | Normative registry/overlay; generated instance OAD |
| “Additive” client break | Closed enums, strict schemas, generated clients reject new material | CB classification and old-client lanes |
| Unsafe tasking migration | Redirect/retry duplicates Commands or physical actions | No automatic unsafe redirects; lifecycle/idempotency proof first |
| Weak deprecation | Warning has neither an applicable replacement nor an explicit no-replacement rationale/mitigation, or lacks date, scope, or consistent docs | Lifecycle bundle and automated parity tests |
| Premature retirement | Unknown open-source clients lose access | Two-minor/12-month floor and major boundary |
| Stale lifecycle metadata | Caches continue serving old dates or wrong variants | Deliberate freshness/validators and `Vary: Accept` invariant |
| Extension collision | Glaux preview conflicts with future OGC/AEP identifiers or semantics | Namespacing, profile limits, promotion/collision review |
| Configuration drift | Same binary exposes different untracked contracts | Deployment profile/fingerprint and profile-specific gates |
| AI-assisted implementation drift | Router, OAD, docs, tests, and claims evolve separately | One contract registry plus generated-output parity gate |

### 15.2 Resolved Plan Open Questions

| Plan question | Outcome |
|---|---|
| Does Glaux need explicit runtime API versioning beyond standards/software/OAD/schema/conformance identities? | **No, not for the default CSAPI 1.0 surface.** A new complete root is reserved for an unavoidable future contract major. |
| Should Glaux avoid URI path versioning unless a future requirement demands it? | **Yes.** Path versioning is not used for routine releases; an incompatible future major triggers a separate-root design decision. |
| How should experimental extensions avoid looking core? | Opt-in, namespaced/profiled where semantically valid, separately documented/tested, and absent from core `conformsTo`. |
| What compatibility promise applies before `1.0.0`? | The overall contract is experimental, but releases remain standards-truthful, pinned, semantically diffed, and regression-tested; breaks are announced at a new minor, not hidden in patches. |

### 15.3 Downstream Decisions Intentionally Left Open

| Open decision | Why not finalized here | Owner/handoff |
|---|---|---|
| Exact public metadata endpoint/header/OAD extension names | Requires OAD, security, observability, and deployment design | 014, 044–049 |
| Final OAS dialect(s), alternate dialect coexistence, and OAD URLs | Tooling study and complete OAD strategy | 014, 023, 056 |
| Exact future-major root syntax and cross-root identity links | Canonical URI/deployment lifecycle problem | 016, 046 |
| Exact retired response: 308, 404, 410, other status, Problem Details, tombstone duration | Error/security/resource-lifecycle semantics | 013, 016, 039 |
| Final enabled list and direct-response mechanics for safe aliases | Needs route composition, auth, method, and external-client validation | 013, 016, 050, 056 |
| Unsafe mutation/Command/Feasibility alias behavior | Idempotency and safety semantics are not yet established | 029, 031, 036–038, 055 |
| Exact experimental namespace/profile URI and OAD overlay format | Identifier, relation, profile, and OAD decisions | 014, 016, 017, 035 |
| Alias-usage telemetry and privacy controls | Security/privacy/operations scope | 039–041, 046–048 |
| Open versus closed status/event/semantic vocabularies | Domain model and lifecycle semantics | 020–024, 036 |
| Final Features Part 4 adoption effect | The dependency is still draft; no final delta exists | 013, 029, 050, 057 |
| Legacy bare relation input on writes | Writable-link meaning remains unsettled | 017, 031 |
| Placeholder SystemEvent identifier migration | Future authoritative vocabulary remains unknown | 020, 024, 057 |

These open decisions do not weaken the baseline. They identify where exact mechanics must be chosen without undoing stable identity, truthful conformance, compatibility classification, lifecycle syntax, and release gates.

---

## 16. Validation Against the Research Plan

### 16.1 Methodology Phase Validation

| Phase | Status | Evidence/output in this report |
|---|---|---|
| 1. Source collection and vocabulary | Complete | §§9–10; pinned inventory, authority labels, version domains, compatibility classes |
| 2. OGC/CSAPI constraint review | Complete | §11; approved identifiers, roots, canonical identity, OAS/schema facts, upstream history |
| 3. General best-practice review | Complete | §§3, 9, 12; RFC/OAS/SemVer and four guideline families reconciled |
| 4. Glaux compatibility analysis | Complete | §§2, 7, 10, 13; every named API surface classified |
| 5. Lifecycle, documentation, and tests | Complete | §§3–5; deprecation/Sunset, OAD/docs/conformance, clients, release gates |
| 6. Synthesis | Complete | §§1, 6, 14–15; decision baseline, handoffs, recommendations, risks, decisions |

### 16.2 Success Criteria

- [x] CSAPI, OGC API - Features, OpenAPI, HTTP, and relevant API-guideline sources were reviewed directly.
- [x] Standards, server, contract, schema, media/profile, OAD, deprecation, and Sunset concepts are distinguished.
- [x] OGC/CSAPI constraints on versioning and evolution have explicit anchors.
- [x] Material release/tag/issue/PR/commit history is reconciled and authority-classified.
- [x] Runtime versioning approaches are compared for OGC/CSAPI fit.
- [x] Breaking, non-breaking, additive, compatibility-sensitive, deprecated, experimental, implementation-only, invalid, and unresolved classes are defined.
- [x] Deprecation, replacement, Sunset, and retirement communication mechanisms are specified with exact scope/syntax.
- [x] Documentation, OAD, conformance, validation, testing, caching, and interoperability implications are documented.
- [x] Recommendations are decision-usable and bounded to Glaux Server.
- [x] Over-versioning, under-versioning, overclaiming, standards drift, unsafe aliases, and client-break risks are identified with controls.
- [x] References and mutable source pins are explicit and reproducible.

### 16.3 Required Deliverable Content

| Required content | Location | Status |
|---|---|---|
| 1. Executive summary | §§1, 7 | Complete |
| 2. Scope and plan alignment | §8 | Complete |
| 3. Evidence base and authority classification | §9 | Complete |
| 4. Versioning terminology and distinctions | §10 | Complete |
| 5. OGC/CSAPI constraints | §11 | Complete |
| 6. General best-practice comparison | §12 | Complete |
| 7. Breaking-change/compatibility matrix | §2 | Complete |
| 8. Deprecation/Sunset/replacement/experimental findings | §3 | Complete |
| 9. Documentation/OAD/conformance/validation/test implications | §§4–5 | Complete |
| 10. Downstream handoff matrix | §6 | Complete |
| 11. Recommendations | §14 | Complete |
| 12. Risks, constraints, open questions | §15 | Complete |
| 13. Success-criteria validation | §16 | Complete |
| 14. References | §18 | Complete |

### 16.4 Compatibility-Matrix Field Validation

The §2 matrix contains every required field: API surface area; example change; classification; source guidance/anchor; client impact; conformance impact; documentation impact; test implication; recommended handling; downstream handoff; and notes/unresolved issues.

### 16.5 Question Validation

Appendix A maps every one of the 36 top-level detailed questions and all 11 compatibility subareas to findings and sections. Appendix B answers all five core questions directly. No question was marked unavailable or answered by unsupported inference.

### 16.6 Review and Scope Gate

The report was reconciled against accepted prerequisite findings and three independent read-only audits. It includes the current Standards Track RFC that supersedes draft deprecation syntax, a live OGC repository/OAS/history refresh, and a prior-client/test audit. No IDR-SRV-011 work or other later research topic was begun.

---

## 17. Status and Controlled Transition

### 17.1 Current Status

- Research, source pinning, and three independent evidence audits: complete.
- All five core and 36 top-level detailed questions, including all 11 compatibility subareas: answered.
- All six methodology phases and eleven success criteria: complete.
- All fourteen required content areas and eleven matrix fields: complete.
- Deliverable: complete and internally reviewed.
- Plan-owner acceptance: **pending**.

The report remains **In Review**. No downstream topic was executed.

### 17.2 Next Two Actions

The next two controlled actions are:

1. the Glaux Project Lead accepts or returns IDR-SRV-010A; then
2. after acceptance, authorize execution of exactly one next eligible plan: **IDR-SRV-011, Query, Filtering, Sorting, Pagination, and Selection Semantics**.

The combined response pattern is:

> Approve IDR-SRV-010A and execute exactly one Glaux Server research plan: IDR-SRV-011.

That wording alone records nothing until the project lead sends it. Until then, IDR-SRV-010A remains in review and IDR-SRV-011 remains unstarted.

---

## 18. References

### 18.1 Project and Governance Sources

- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)
- [IDR-SRV-010A Research Plan](../IDR%20Plans/idr-srv-010a-api-versioning-backward-compatibility-and-deprecation-strategy.md)
- [Glaux Server Goal and Definition](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- [Research Report Template](../../../../../Governance/research-report-template.md)
- [Research Planning Approach](../../../../../Governance/research-planning-approach.md)
- [Official CSAPI Upstream-History Register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md)
- [IDR-SRV-006 Part 1 Baseline](idr-srv-006-csapi-part-1-requirement-baseline-report.md)
- [IDR-SRV-007 Part 2 Baseline](idr-srv-007-csapi-part-2-requirement-baseline-report.md)
- [IDR-SRV-008 Conformance Mapping](idr-srv-008-conformance-class-and-requirement-mapping-report.md)
- [IDR-SRV-009 Entry-Point Behavior](idr-srv-009-landing-page-api-definition-and-conformance-declaration-behavior-report.md)
- [IDR-SRV-010 Navigation Behavior](idr-srv-010-collections-resources-links-and-navigation-behavior-report.md)

### 18.2 Controlling OGC Standards and Policies

- [OGC 23-001, OGC API - Connected Systems - Part 1: Feature Resources](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC 23-002, OGC API - Connected Systems - Part 2: Dynamic Data](https://docs.ogc.org/is/23-002/23-002.html)
- [OGC 17-069r4, OGC API - Features - Part 1: Core corrigendum](https://docs.ogc.org/is/17-069r4/17-069r4.html)
- [OGC Policy Directives, Directive 18: standard versioning](https://portal.ogc.org/public_ogc/directives/directives.php)
- [OGC 20-059r4, Naming of OGC API Standards, Repositories & Specification Elements](https://docs.ogc.org/pol/20-059r4.html)
- [Versioned OGC CSAPI schemas](https://schemas.opengis.net/ogcapi/connected-systems/)

### 18.3 Official CSAPI Artifacts and Repository History

- [Official `v1.0.0` release](https://github.com/opengeospatial/ogcapi-connected-systems/releases/tag/v1.0.0)
- [Tagged source, commit `8e03b236`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api)
- [Part 1 tagged root OAS](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/openapi-connectedsystems-1.yaml)
- [Part 2 tagged root OAS](https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/openapi-connectedsystems-2.yaml)
- [Current official `master`, commit `3fd86c73`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f)
- [Issue #23, conformance/encoding combinations](https://github.com/opengeospatial/ogcapi-connected-systems/issues/23)
- [Issue #48, OAS version direction](https://github.com/opengeospatial/ogcapi-connected-systems/issues/48)
- [Issue #77, OAS 3.0/3.1 artifacts](https://github.com/opengeospatial/ogcapi-connected-systems/issues/77)
- [Issue #87 / PR #124, JSON Schema 2020-12 migration](https://github.com/opengeospatial/ogcapi-connected-systems/issues/87)
- [Issue #141, Features Part 4 dependency](https://github.com/opengeospatial/ogcapi-connected-systems/issues/141)
- [Issue #149, System history/versioning](https://github.com/opengeospatial/ogcapi-connected-systems/issues/149)
- [PR #196, `recursive` OAS correction](https://github.com/opengeospatial/ogcapi-connected-systems/pull/196)
- [Issue #170, PATCH behavior](https://github.com/opengeospatial/ogcapi-connected-systems/issues/170)
- [PR #176, post-tag relation example correction](https://github.com/opengeospatial/ogcapi-connected-systems/pull/176)
- [PR #199, Procedure `validTime` correction](https://github.com/opengeospatial/ogcapi-connected-systems/pull/199)
- [Issue #177, deployment-scoped stream OAS omissions](https://github.com/opengeospatial/ogcapi-connected-systems/issues/177)
- [Issue #186, versioned bundled OAS proposal](https://github.com/opengeospatial/ogcapi-connected-systems/issues/186)

### 18.4 HTTP, Linking, OpenAPI, and Versioning Authorities

- [RFC 9110, HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)
- [RFC 9111, HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111.html)
- [RFC 8288, Web Linking](https://www.rfc-editor.org/rfc/rfc8288.html)
- [RFC 9745, The Deprecation HTTP Response Header Field](https://www.rfc-editor.org/rfc/rfc9745.html)
- [RFC 8594, The Sunset HTTP Header Field](https://www.rfc-editor.org/rfc/rfc8594.html)
- [RFC 6906, The `profile` Link Relation Type](https://www.rfc-editor.org/rfc/rfc6906.html)
- [RFC 5829, Link Relation Types for Simple Version Navigation](https://www.rfc-editor.org/rfc/rfc5829.html)
- [IANA HTTP Field Name Registry](https://www.iana.org/assignments/http-fields/http-fields.xhtml)
- [IANA Link Relation Types](https://www.iana.org/assignments/link-relations/link-relations.xhtml)
- [OpenAPI Specification 3.2.0](https://spec.openapis.org/oas/v3.2.0.html)
- [OpenAPI Specification 3.1.2](https://spec.openapis.org/oas/v3.1.2.html)
- [JSON Schema Draft 2020-12 Core](https://json-schema.org/draft/2020-12/json-schema-core)
- [JSON Schema Draft 2020-12 Validation](https://json-schema.org/draft/2020-12/json-schema-validation)
- [Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html)

### 18.5 Comparative API Guidance and Client Evidence

- [Microsoft API Guidelines, pinned commit `577874dc`](https://github.com/microsoft/api-guidelines/tree/577874dc49093ae8f6c4a5996a8f030d3ffbda7b)
- [Google AIP-180, Backwards Compatibility, pinned commit](https://github.com/aip-dev/google.aip.dev/blob/b38f24615f8f3b9e90d72f06df9d712a4f893ed7/aip/general/0180.md)
- [Google AIP-181, Stability Levels, pinned commit](https://github.com/aip-dev/google.aip.dev/blob/b38f24615f8f3b9e90d72f06df9d712a4f893ed7/aip/general/0181.md)
- [Zalando compatibility guidance, pinned commit](https://github.com/zalando/restful-api-guidelines/blob/41b74fa65d9fbaaf5ad66b471563e92bd15477bb/chapters/compatibility.adoc)
- [Zalando deprecation guidance, pinned commit](https://github.com/zalando/restful-api-guidelines/blob/41b74fa65d9fbaaf5ad66b471563e92bd15477bb/chapters/deprecation.adoc)
- [Heroku HTTP API Design Guide, pinned commit](https://github.com/interagent/http-api-design/tree/5fd8c92fb18c52d9234eda01e4d7959225ab610d)
- [Heroku API Compatibility Policy](https://devcenter.heroku.com/articles/api-compatibility-policy)
- [OS4CSAPI organization](https://github.com/OS4CSAPI)
- [CSAPI Explorer live application](https://ogc-csapi-explorer.pages.dev/) and [pinned source snapshot](https://github.com/OS4CSAPI/ogc-csapi-explorer/tree/00f1c188e05738ee03390fd95f09d351e073a9c3)

---

## Appendix A. Detailed Research-Question Coverage

### A.1 Standards and Versioning Context

| ID | Detailed question | Answer/result | Location |
|---|---|---|---|
| S1 | How do Parts 1/2 identify version, requirements, conformance, schemas, and OAS? | Approved Version 1.0 documents and `/1.0` normative identifiers; versioned hosted schema packages; tagged OAS 3.1.0 with document `0.0.1` | §§10–11 |
| S2 | Do they define server standards/API/schema version exposure? | Exact conformance URIs expose the class and major/minor normative namespace, while the exact document/corrigendum is pinned separately; no server API-version field, selector, or lifecycle policy exists; schema dialect/package identity remains separate | §§10.1, 11.1, 11.6 |
| S3 | How do Features/OGC practice handle definitions, declarations, and evolution? | Linked OAD metadata, fixed `/conformance`, relative API paths, exact class URIs, optional extensions, and explicit breaking-change awareness in 1.0.1 | §§11.2–11.5 |
| S4 | How should implementation and standards versions differ? | Separate software SemVer/fingerprint from exact OGC sources and conformance URIs | §§10.1, 13.2–13.3 |
| S5 | How does AEP-4789 affect expectations? | Preserve the adopted standards package/interoperability; distinguish Glaux extensions and never weaken direct obligations | §11.8 |
| S6 | What does official release/tag/issue/PR/commit history establish? | `v1.0.0` is the baseline; no later release; one example-only post-tag change; unresolved/approved maintenance does not rewrite the standard | §§9.2, 11.7 |

### A.2 API Versioning Strategy

| ID | Detailed question | Answer/result | Location |
|---|---|---|---|
| V1 | Path, media/profile, OAD, SemVer, or other? | One stable resource root + software/contract SemVer + immutable OAD; media/profile only for their real representation/constraint semantics | §§1, 12.1, 13 |
| V2 | Which approaches fit OGC/CSAPI? | Stable root and linked definitions fit strongly; mandatory query/header/vendor media selectors do not; separate root is last-resort major | §12.1 |
| V3 | Interaction with landing, collections, items, conformance, OAD, schemas? | Preserve resource graph/canonical IDs; `/conformance` stays truth; descriptions/schemas can have immutable versioned artifacts | §§2, 4, 11.3, 13.3 |
| V4 | One surface or concurrent versions? | One by default; concurrent complete roots only for an approved unavoidable major migration | §§13.1, 13.4 |
| V5 | What evidence supports it? | CSAPI identifier/root model, Features metadata separation, OGC policy, HTTP caching, SemVer, guidelines, client/operational costs | §§9, 11–12 |

### A.3 Backward Compatibility and Breaking Changes

| ID | Detailed question | Answer/result | Location |
|---|---|---|---|
| B1 | What constitutes a breaking change? | Loss/change of any valid stable request, response, identity, link, default, semantic, generated-client, conformance, operational, or profile promise | §§2, 10.2–10.4 |
| B2 | Which additions are likely non-breaking? | Optional additions with unchanged defaults are A/CB, never presumed safe until old-client/schema/tool evidence | §§2, 10.4 |
| B3 | Which changes are likely breaking? | Removal/rename/reparent, requiredness/narrowing, changed identifiers/defaults/media/errors/lifecycle/conformance, mandatory selectors | §2 |
| B4 | How does compatibility apply across the named surfaces? | Surface-specific rules and tests are enumerated below | §2 and A.4 |
| B5 | Which findings transfer downstream? | Every owning topic receives explicit change-classification, identity, lifecycle, OAD/schema, and test constraints | §6 |

### A.4 The Eleven Required Compatibility Subareas

| Subarea | Baseline |
|---|---|
| Landing page | Preserve required discovery links/targets/types; optional links are CB until old bootstrap tests pass |
| Conformance declarations | Add only proved classes; removal is breaking; planned/experimental claims are invalid |
| Collections/resource links | Preserve collection IDs/types/membership meaning and canonical link semantics; additions are CB |
| Schemas/representations | Preserve old valid input, response decoding, identifiers/dialects/defaults; do not mutate stable schema targets |
| Query parameters | Add only with identical omission behavior; changing grammar/default/order/page/result semantics breaks |
| Media/content negotiation | Preserve old Accept/default/precedence/media; new variants require accurate OAD and `Vary: Accept` |
| OpenAPI descriptions | Treat paths/operation IDs/types/media/security/errors as generated-client contract; immutable snapshots/diffs |
| DataStreams/Observations | Ordinary data is not API evolution; schema/encoding/time/order/history changes are breaking |
| ControlStreams/Commands | Parameter additions are CB; lifecycle/status/idempotency/result/retry changes are high-risk breaking |
| Status/events | New instances are data; new types/status/semantic IDs are CB or breaking depending on declared openness |
| Error responses | Optional extension may be CB; changed status/media/code/retryability/meaning is breaking |

### A.5 Deprecation and Retirement

| ID | Detailed question | Answer/result | Location |
|---|---|---|---|
| D1 | How are endpoints/fields/links/params/schemas/classes/extensions deprecated? | Element-appropriate OAD/schema markers plus scoped RFC metadata, registry, docs, changelog, and unchanged old behavior | §3.3 |
| D2 | Roles of docs, headers, links, OAD, changelog, conformance? | A required mutually consistent bundle; conformance remains implementation truth rather than maturity metadata | §§3–4 |
| D3 | What retirement window? | Two stable minors and 12 months, whichever longer, normally next major; documented emergency exception | §3.4 |
| D4 | How are replacements discovered? | When a replacement exists, HTTPS `deprecation` documentation and semantically correct canonical/alternate/version relations identify it; otherwise documentation states the no-replacement rationale/mitigation; no generic misuse of successor-version | §§3.2–3.3 |
| D5 | How avoid misleading planned/experimental/staged behavior? | Separate maturity/profile/OAD metadata, opt-in isolation, and negative leakage tests; absent from core `conformsTo` | §§3.7, 4.3 |

### A.6 Experimental, Extension, and Profile Behavior

| ID | Detailed question | Answer/result | Location |
|---|---|---|---|
| E1 | How avoid polluting standards surface? | Opt-in, namespaced, separate OAD/docs, absent from core claims, promotion gate | §3.7 |
| E2 | How name/link/document/test project resources/extensions/profiles? | Collision-resistant URI namespace/profile selected downstream, precise links, maturity metadata, strict negative/extension lanes | §§3.7, 5, 6 |
| E3 | How avoid future CSAPI/SensorML/SWE/AEP conflict? | Pin and collision-review upstream; never infer ownership; adopt/rename through delta and compatibility process | §§11.5, 13.5 |
| E4 | How separate profile behavior from core? | Profile adds compatible semantics only and remains safe to unaware clients; incompatible behavior receives separate contract | §§11.4, 12.1 |
| E5 | What handoff to OAD/schema/interoperability? | Exact extension metadata, overlay, namespace, validation, promotion, and client lanes assigned | §6 |

### A.7 Documentation, OpenAPI, and Client Implications

| ID | Detailed question | Answer/result | Location |
|---|---|---|---|
| O1 | How do versions/claims/deprecations appear in OAD? | Separate `openapi`, `info.version`, optional project metadata, exact classes/source pins, schema dialect/package, `deprecated` markers | §§4.1–4.2 |
| O2 | How distinguish stable/experimental/deprecated/removed? | Lifecycle registry and consistent OAD/docs/changelog/runtime bundle | §§3.1, 4.5 |
| O3 | How should generated clients handle evolution? | Preserve stable generator contract; old-client generation/compile/runtime is a release gate; test unknown additions explicitly | §§4.4, 5 |
| O4 | What transfers to IDR-SRV-014? | Dialects, immutable snapshots, metadata names, operation IDs, deprecation, experimental overlays, semantic diffs | §6 |
| O5 | What external-client concerns must be tested? | Relation/type/status closure, pagination, Command fallback, nonstandard routes, old generated clients, proxies/config profiles | §§4.4, 5.3 |

### A.8 Testing and Verification

| ID | Detailed question | Answer/result | Location |
|---|---|---|---|
| T1 | What tests protect compatibility? | Registry diff, old request replay, old generated clients, strict/adapters/schema/query/media/tasking/profile lanes | §5.1 |
| T2 | What tests cover lifecycle metadata/replacement/docs? | Exact grammar/scope/dates, fake clock, OAD/docs/link parity, applicable replacement readiness or no-replacement rationale, and pre/post-retirement behavior | §5.2 |
| T3 | What detects accidental OAD/schema/shape/link/error breaks? | Semantic OAD/registry diff, schema closure, golden/prior fixtures, old decoder/client execution, graph/error replay | §§5.1–5.2 |
| T4 | What golden/contract/snapshot/external strategies follow? | Immutable prior contracts and clients; request/response/relation/schema/lifecycle timeline corpus; pinned external matrix | §§5.1–5.3 |
| T5 | What transfers to 050/051/053/056? | Executable gates, change-requirement-test graph, immutable fixture corpus, and external/generated/component interoperability matrix | §6 |

---

## Appendix B. Core Question Answers

| Core question | Direct answer |
|---|---|
| 1. What versioning strategy should Glaux use? | One stable CSAPI 1.0 resource root, separate SemVer software/public-contract releases, immutable OAD/schema/profile identities, exact conformance URIs, and a second complete root only for an unavoidable future major. |
| 2. What changes are breaking, non-breaking, additive, deprecated, experimental, or implementation-specific? | The §10.3 taxonomy and §2 matrix classify changes by source, wire, semantic, identity, generated-client, conformance, operational, and profile impact; additions are conditional. |
| 3. How should backward compatibility be preserved? | Freeze canonical identity/defaults/semantics per stable major; generate from one registry; pin sources; maintain safe adapters; run prior requests and clients at every release. |
| 4. How should deprecation, retirement, staged conformance, and experiments be communicated? | Exact scoped RFC 9745/8594 metadata, OAD/schema annotations, HTTPS migration docs, changelog/manifest, support window, truthful `/conformance`, and separate experimental maturity. |
| 5. What implementation/test/documentation implications follow? | A contract registry/fingerprint, immutable generated artifacts, semantic diffs, old-client lanes, strict and adapter tests, lifecycle/cache tests, external-client matrix, and the §6 downstream handoffs. |

---

## Appendix C. Decision and Evidence Register

| Decision ID | Proposed baseline | Primary evidence | Acceptance state |
|---|---|---|---|
| P-010A-01 | One stable, unversioned-by-release CSAPI 1.0 root | CSAPI `{api_root}`/canonical requirements; Features relative paths; OGC/HTTP/client analysis | Pending plan-owner acceptance |
| P-010A-02 | One capability/contract registry drives every public projection | Accepted IDR-008–010 drift controls; compatibility analysis | Pending plan-owner acceptance |
| P-010A-03 | Stable retirement floor = two stable minor releases and 12 months, whichever longer | RFC lifecycle separation; Heroku 12-month and Microsoft longer-window evidence | Pending plan-owner acceptance |
| P-010A-04 | SemVer governs software/public-contract impact, not runtime selection | SemVer 2.0.0; OAS/version-domain distinctions | Pending plan-owner acceptance |
| P-010A-05 | Exact RFC 9745/8594 lifecycle bundle | RFC 9745, RFC 8594, IANA registries | Pending plan-owner acceptance |
| P-010A-06 | Safe published-defect GET/HEAD adapters; no unsafe redirects | IDR-010 contradictions; HTTP redirect/cache and tasking safety analysis | Pending plan-owner acceptance |
| P-010A-07 | Experimental behavior stays separate from core conformance | Features/CSAPI extension boundaries; RFC 6906; accepted conformance policy | Pending plan-owner acceptance |
| P-010A-08 | Prior contract/client replay is a release gate | AIP-180 dimensions; OAD/client evidence; Glaux risk model | Pending plan-owner acceptance |
| P-010A-09 | Every upstream standards change receives a pinned delta/adoption decision | OGC Directive 18; repository release/history findings | Pending plan-owner acceptance |

---

## Appendix D. Final Completion Check

- [x] Exactly one topic, IDR-SRV-010A, was researched.
- [x] IDR-SRV-010 acceptance was recorded from the authorizing combined instruction.
- [x] Approved standards controlled over OAS, examples, and repository discussions.
- [x] Current RFC 9745 evidence replaced obsolete draft syntax.
- [x] Official OAS files and all relevant repository history were reviewed and pinned.
- [x] Standards obligation, artifact/source finding, analysis, and project recommendation are distinguished.
- [x] All plan questions, phases, success criteria, required content areas, and matrix fields are validated.
- [x] Evidence limitations and downstream uncertainties are explicit.
- [x] No later research topic was begun.
- [ ] Plan-owner acceptance and date remain pending review.

---

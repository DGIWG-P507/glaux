# 02 — Step 2: test-source audit

**Question:** For a sample of CSAPI-facing tests, where does each expected answer come from, and does the test check that answer without deriving it from the server's own output?

**Date:** 3 October 2026<br>
**Authorised by:** the project lead's `proceed` after merging [PR #111](https://github.com/DGIWG-P507/glaux/pull/111) and explicitly confirming that the merge accepts IDR-SRV-066. This authorizes step 2 only.<br>
**Performed by:** Codex (OpenAI), with separate source-analysis agents for System creation/retrieval and schema validation. Exact model version is not exposed in this session. Context was not fresh: these agents previously worked on the client studies. A separate, non-author reviewer checks the actual documentation diff before publication; its exact reviewed commit and outcome are recorded in the delivery PR. Neither this nor the source analysis is independent human expert review.<br>
**Examined:** server [`27955c1b9260cd811ad6bc08f85feab43ad65028`][Server]; planning [`f74eb659bab70b394b1027d1d43b3066561762de`][Planning], including Guide v1.21 and Roadmap v1.40. Public primary-source checks were made on 3 October 2026. Server code was inspected in a new read-only working clone; unrelated local work was preserved.

## Answer in brief

**The sampled expected answers are supported. No new defect finding is warranted from this sample.** They are not merely tests that ask whether the server agrees with itself: important values, identities, schema outcomes and HTTP properties have independently written expectations. The side-by-side comparisons below make their basis checkable.

There are three different sources of those answers:

- **Published standards and original schemas:** for example, System type names, required JSON fields and HEAD response behavior.
- **The selected transaction draft:** for example, the create response. This is a pinned project dependency, not a claim that the draft is a published CSAPI rule.
- **Glaux's explicit choices and current implementation stage:** for example, concealing forbidden Systems with 404, minting UUIDv7 identifiers and advertising no completed conformance classes yet.

Some exact expectations intentionally describe only Phase 1. A read response currently has one self link; SensorML and richer creation inputs are not implemented yet. Those tests must evolve with their owning capabilities. Their current success would not prove the complete link, representation or conformance requirements.

**Nothing was executed:** no server, database, test suite, mutation campaign, security scanner or external client. This step answers where the sampled expectations come from, not whether all tests pass or the whole implementation is correct.

## 1. Sample and method

The sample follows the small implemented client path: discover the API, create a minimal System, retrieve the same System and distinguish errors. It also samples the offline schema validator that supplies structural answers. The tables list expectation families, not a statistical sample, test count or coverage percentage.

For each row, the audit read the assertion and relevant fixture, its existing contract documentation, and the controlling primary clause/schema or approved project choice. Python wrappers were followed into the Rust proof programs where most API assertions actually live. Only enough implementation code was read to identify the tested boundary; this was not the architecture review.

The server [corpus documentation][CorpusDoc] identifies original artifacts separately from Glaux-authored fixtures. Its four standard-header snapshots are **not complete standard prose or Annex A**. Consequently, this audit also consulted the named published documents and pinned draft requirements. It did not claim that every normative source was available offline.

**Existing attribution versus this audit:** the contract documents already identify source/Guide sections and expected behaviors; corpus cases additionally name schemas and reasons. The fine-grained assertion-to-clause comparison here was reconstructed during this audit. It was not already an inline citation beside every assertion. A search for `/req/` strings alone cannot measure that chain.

## 2. Discovery and shared HTTP behavior

All test links below are fixed to the examined server commit. “Source supported” means the expected answer is justified, not that the check ran here.

| What a client should see / what the test expects | Assertion inspected | Where the answer comes from; assessment |
|---|---|---|
| A successful landing page links to API documentation and conformance information using the proper relation names. | [Discovery proof][DiscoveryProof], `landing_matches`, `exact_links`, `discover` (lines 385–441, 627–664); literal-link controls at 448–473. | [Common Part 1][Common] requirements 12–15 supply root GET/200 and documentation/conformance relations. The full conformance URI is prescribed. Glaux's exact title, extra download links, configured root and choice to offer **both** documentation relations are [documented project choices][DiscoveryDoc], not additional universal requirements. Source supported. |
| The initial conformance response is exactly an empty class list, not a missing field or a claim to an unfinished class. | [Discovery proof][DiscoveryProof], `no_classes`, controls and retrieval (444–446, 474–483, 665–682). | Common requirement 17 supplies `conformsTo` and truthful declarations; [Guide §7.3][Guide] supplies the stage rule. **Empty at this stage** is Glaux's choice, not a standard requirement for completed servers. Source supported. |
| The OpenAPI document names the enabled routes and configured public server, without phantom capabilities. | [Discovery proof][DiscoveryProof], independent `INVENTORY` (30 onward), `discover` (684–703) and `routes` (707 onward). | Common requirements 14–15 justify accessible API definition/200. Exact OpenAPI 3.1.0, dialect, inventory and deployment URL are [Glaux's discovery contract][DiscoveryDoc] and Guide §§4.1/7.3. The expected route list is literal, not generated from production route metadata. Source supported; no complete OAS/CSAPI conformance claim. |
| Errors have a numeric status matching HTTP, safe fields and tolerated extension content; malformed versions must fail the checker. | [HTTP proof][HttpProof], `contract`, `matches_problem`, `oracle_controls` (208–348). | [RFC 9457 §3.1.2 / §3.2][Problems] supplies numeric/matching status **if present** and extension tolerance. Requiring its presence, exact safe text/URN, correlation and no-store is the [Glaux catalog][HttpDoc], not mandatory presence of every RFC member. Literal correct/missing/string/wrong-value/extension fixtures inspect generic JSON before production-type conversion. Source supported. |
| HEAD returns no body; a sent Content-Length describes the GET bytes, including on errors. | [HTTP proof][HttpProof], `problems` (592–609). | [RFC 9110 §§9.3.2/8.6][Http] supplies the no-body and sent-length rules. Always supplying the known length is the documented Glaux choice. Comparing GET byte length with HEAD is a valid relational check of that HTTP property, not proof that the GET representation itself is correct. Source supported. |
| An explicit `q=0` exclusion is not overridden by a wildcard; when no offered format is acceptable, this server returns 406. | [HTTP proof][HttpProof], `media` (362–412). | RFC 9110 §§12.4.2/12.5.1 supplies quality and specificity; §12.4.1 permits rejection or ignoring the preference. The [Glaux contract][HttpDoc] chooses rejection. Expected media/status values are literal, separate from production selection. Source supported; synthetic fixture routes do not prove resource-specific codecs. |

## 3. System creation and retrieval

The creation and retrieval [contracts][CreateDoc] / [contracts][ReadDoc] explicitly limit this increment. The selected transaction source is OGC API Features Part 4 draft commit `9ca25f56a58ed822ea8a685a7a41afa7181aaa8b`. Original CSAPI schemas are pinned through the server corpus to `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`.

| What a client should see / what the test expects | Assertion inspected | Where the answer comes from; assessment |
|---|---|---|
| Creating a System returns 201 and its canonical address in Location. | [Create proof][CreateProof], `creation_id` / `exact_commit` (294–307, 403–412); good/bad receipt controls at 308–341. | Selected draft [`/req/create-replace-delete/post-response`][PostResponse] A/B; [CSAPI Part 1][Part1] requirement 5 supplies the canonical System path. UUIDv7, configured origin, empty receipt and no ETag are Guide/increment choices, not all part of that source rule. Source supported. |
| A supplied local ID is ignored while the submitted UID remains the identity of the described thing. | [Create proof][CreateProof] (440–442, 786–807); [read proof][ReadProof] (698–712). | The draft [ID permission][RidPermission] permits server assignment; [original feature schema][FeatureSchema] describes ignoring local input IDs. Part 1 requirements 1–2 distinguish IDs/UIDs; Guide §4.2 chooses minting and UID preservation. The schema description is an annotation, not an executable rejection constraint. Known submitted literals are checked separately. Source supported. |
| Minimal System JSON has Feature, null geometry and the required uid/name/featureType properties. | [Create proof][CreateProof] fixture and invalid matrix (343–345, 603–645); [read proof][ReadProof] `expected` (319–326). | Part 1 requirements 80–81, Table 39 and A.87–A.88; [feature schema][FeatureSchema] and its GeoJSON reference. **Accepting only null geometry/richer-content rejection now** is the documented Phase 1 restriction, not a universal validity rule. Expected JSON comes from submitted literals. Source supported within that slice. |
| All ten permitted System type spellings are accepted and retained; an unlisted `PhysicalSystem` spelling fails. | [Create proof][CreateProof] (462–465, 603–645, 856–893); [read proof][ReadProof] (696–712). | Part 1 requirement 82 / Tables 6 and 40; the [System schema][SystemSchema] references the [ten-value enumeration][SystemUris]. Retaining the submitted permitted spelling is the documented project contract. The proof enumerates strings independently and compares retained data with input. Source supported. |
| GET returns the created ID and submitted identity/meaning, with a canonical self link. | [Read proof][ReadProof], `expected` / `representation_error` and fixtures (319–352, 651–675, 698–712); negative controls at 414–481. | Part 1 requirement 5 / A.88 and [Features Part 1][Features] `/req/geojson/content`, `/req/core/f-links`. Exact one-link shape, title and no extra fields/validators are [Phase 1 assertions][ReadDoc], not complete standard link requirements. Only the generated ID is obtained from the receipt; UID/name/type are independently supplied. Source supported. |
| Unsupported write media returns 415; an unoffered read representation returns 406. | [Create proof][CreateProof] (646–652); [read proof][ReadProof] (718–725). | RFC 9110 §§15.5.16/15.5.7 and Guide §6.4; the currently offered format set is the documented increment choice. SensorML rejection is temporary, not a claim that CSAPI forbids it. Source supported. |
| Forbidden and missing Systems have the same permitted error view. | [Read proof][ReadProof], `safe_problem` / `concealed` and caller cases (363–412, 839–858, 904–915). | **Project policy:** Guide §§4.10/6.4 and [read contract][ReadDoc], not a universal OGC requirement for every denial to use 404. Independent problem/header checks qualify the differential comparison; it covers normalized body, selected headers and lengths, **not every observable value or timing**. Source supported within those limits. |
| After an ordinary server restart, the same System identity and meaning remain retrievable. | [Read proof][ReadProof] (816–837), reusing independently authored values at 659–664 and 705–710. | **Project requirement:** [Roadmap task 1.5.2][Roadmap] and Guide §8.2; [read-test contract][ReadTests] adds byte stability and unchanged stored state. Post-restart responses are checked against independent JSON **as well as** earlier bytes, not only against themselves. Source supported; not backup/restore or arbitrary crash-boundary evidence. |

The complete Features link rule includes collection and applicable alternate links. The current exact-one-link assertion must change when those capabilities are implemented; the read documentation already excludes them and full conformance. Likewise, negative tests for currently unsupported formats/richer input must not be reused later as universal standard-invalid fixtures.

## 4. Original schemas and explicit validation adaptations

These fixtures are **Glaux-authored inputs checked against official schemas**, not official OGC example fixtures. The [case manifest][Cases] stores expected booleans, schema targets and reasons independently of validator output. Original bytes, revision/digests and the explicitly adapted request catalog remain distinct.

| Example / expected answer | Assertion inspected | Source chain; assessment |
|---|---|---|
| A labelled Quantity with definition and unit is structurally valid. | `quantity-labelled.json`; [cases][Cases] lines 71–77; [validator tests][ValidationTests] lines 19–44 compare with the stored expectation. | [Quantity][Quantity] lines 28–33 requires type/definition/label/uom; the inherited label constraint is satisfied. Supported positive structural example, not proof that its synthetic definition URI resolves or that all SWE semantics hold. |
| Missing and empty Quantity labels are invalid for different reasons. | `quantity-missing-label.json`, `quantity-empty-label.json`; [cases][Cases] lines 88–111. | Missing violates Quantity's required list. Empty violates the inherited [identifiable label][Label] string/minLength rule, reached through AbstractSimpleComponent and AbstractDataComponent. [SWE Common requirement 55 / A.54][Swe] directs JSON-schema validation. Supported; conceptual optionality does not remove the JSON rule. This is not a schema defect or an invented Glaux restriction. |
| Relaxing generated request fields must not relax a nested Quantity's label; neither every component nor every nonempty label has a stricter rule. | [Projection tests][ProjectionTests] lines 299–387: unlabelled DataRecord with labelled Quantity succeeds; missing/empty/null/numeric Quantity label fails; one-space label succeeds. | [DataStream][DataStream] schema → observationSchema → observationSchemaJson → common/sweCommonDefs → sweCommon → DataRecord fields/AnyComponent → Quantity. DataRecord requires type/fields, not label; the Quantity rule remains. Guide §4.3 preserves nested constraints. Supported discriminating input-projection examples, not semantic stream/codec execution. |
| The same Binary descriptor fails the original aggregate root but passes its named Binary definition and valid full wrapper. | [Cases][Cases] lines 4–28; [validator tests][ValidationTests] lines 48–107 also vary byte order, members, wrapper and unresolved component reference. | Original [encodings.json][Encodings] root lists Text/XML/JSON only (257–261), while BinaryEncoding exists at 107–162. The [observation wrapper][SweWrapper] selects that named definition. Guide §4.3 documents the product entry point. Supported **original-source diagnostic plus project selection**, not a silently repaired original schema or proof of binary decoding/component resolution. |
| A minimal Observation request may omit generated IDs; its response must contain them. Result/time obligations remain. | [Projection tests][ProjectionTests] lines 119–126, 160–254 and 561–594 distinguish original/projected outcomes and remove required members. | [Original Observation][Observation] marks id/datastream@id readOnly but also requires them. [JSON Schema §9.4][DirectionAnnotations] does not rewrite `required` from annotations. Guide §4.3/§13 and the [direction contract][DirectionDoc] explicitly adapt the request required array, retaining response IDs and result/time. Supported **Glaux-adapted outcome**, not unchanged-schema acceptance. |
| A write-only stream schema must be absent from a response, including not appearing as null. | [Projection tests][ProjectionTests] lines 273–297 compare original acceptance with projected rejection; omission succeeds. | [DataStream][DataStream] lines 132–135 / [ControlStream][ControlStream] lines 102–105 mark schema writeOnly. The explicit direction rule in Guide §4.3/§13 and [contract][DirectionDoc] supplies the application-level check beyond original structural evaluation. Supported paired example of the adaptation boundary. |

Projection tests use fixed required-member lists rather than deriving them from production ownership metadata (lines 183–186). These are tests of a validation primitive, **not implemented Observation/stream endpoints or actual HTTP-response interpretation**. Their presence does not pull Phase 2/3 behavior into Phase 1's conformance claims.

## 5. Independence and the accepted client studies

The HTTP/discovery/create/read examples inspect wire headers, bytes and ordinary `serde_json::Value` values. Their examined expected answers are not constructed by deserializing through the server's production resource types. The error and discovery examples contain deliberately bad response objects and permitted-extension controls. The creation/read proofs similarly mutate required receipt/representation facts. **These controls were inspected, not run.** Their presence alone does not establish detection strength; that remains step 4's question.

Relational checks are not automatically circular. HEAD length is properly compared with the corresponding GET; concealment compares two views after independently checking their required shape; restart checks both independently expected content and stability. Those checks support their stated property, not every property of either response.

The four accepted studies are reused through [IDR-066's cross-study reconciliation][Clients], not reopened:

- Older OSH/OSCAR client conventions do not override published clauses.
- cs-client-ts fixture origins and shared authorship do not make every client expectation independent.
- Aleph's SensorML/list workflows are beyond the current minimal slice. A useful programmable library path does not establish whole-application compatibility.
- No client was run and no peer code/fixture was copied. Later representation, browser, authentication and event checks retain their existing owners; this audit adds no requirement to make an unchanged application pass now.

## 6. Result, limits and handoff

The [original scoping observation §6](00-scoping-observations.md#6-test-expectations-traced-to-the-standard) correctly warned that a search for requirement identifiers misses prose citations. The actual sample supplies the missing comparison: source-linked contracts and independently authored answers are present. No finding is opened merely because each assertion lacks an inline normative URI, and no new traceability platform or research plan is proposed.

Limits:

- This is a named sample, not a full review of every assertion, Phase 1 task or conformance class. Unexamined tests receive no verdict.
- Published clauses, original-schema diagnostics, direction projections and partial-stage contracts are different evidence. None is silently promoted into another.
- No CI history was revalidated, tests executed, coverage measured or defects injected during this step. Source-visible test machinery is not a new passing result.
- No human expert reviewed the sample. OpenAI's review of selected Claude-authored #25 behavior is cross-provider evidence for **that sample only**, not the charter's required full-phase provider accounting. Earlier OpenAI-authored work and #26 still need the full gate review's coverage/limitations record.
- No server code, issue, gate, repository setting, action-list history, Goal, Guide or Roadmap changed. All fifteen review gates were checked open before starting.

**Step 2 is complete; no new finding or adopted remedy.** Steps 3–5 remain. The next `proceed` authorizes **step 3, Phase 1 architecture**, to examine whether the foundations can accommodate the next resource families without excessive duplication. Step 4 begins with its own proposal before tools are run; step 5 needs its approved disposable comparison environment. Neither is started here. [Gate 1 (#339)](https://github.com/DGIWG-P507/glaux-server/issues/339) stays open and blocks Phase 2 until the project lead closes it.

[Server]: https://github.com/DGIWG-P507/glaux-server/tree/27955c1b9260cd811ad6bc08f85feab43ad65028
[Planning]: https://github.com/DGIWG-P507/glaux/tree/f74eb659bab70b394b1027d1d43b3066561762de
[Guide]: https://github.com/DGIWG-P507/glaux/blob/f74eb659bab70b394b1027d1d43b3066561762de/Docs/Plans/glaux-server/glaux-server-implementation-guide.md
[Roadmap]: https://github.com/DGIWG-P507/glaux/blob/f74eb659bab70b394b1027d1d43b3066561762de/Docs/Plans/glaux-server/glaux-server-roadmap.md
[CorpusDoc]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/standards-corpus.md
[DiscoveryDoc]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/discovery.md
[DiscoveryProof]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/examples/discovery-proof.rs
[HttpDoc]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/http-boundary.md
[HttpProof]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/examples/http-boundary-proof.rs
[CreateDoc]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/system-create.md
[CreateProof]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/examples/system-create-proof.rs
[ReadDoc]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/system-read.md
[ReadTests]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/system-read-tests.md
[ReadProof]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/examples/system-read-proof.rs
[Common]: https://docs.ogc.org/is/19-072/19-072.html
[Part1]: https://docs.ogc.org/is/23-001/23-001.html
[Features]: https://docs.ogc.org/is/17-069r4/17-069r4.html
[Http]: https://www.rfc-editor.org/rfc/rfc9110.html
[Problems]: https://www.rfc-editor.org/rfc/rfc9457.html
[PostResponse]: https://github.com/opengeospatial/ogcapi-features/blob/9ca25f56a58ed822ea8a685a7a41afa7181aaa8b/extensions/transactions/create-replace-update-delete/standard/requirements/create-replace-delete/create/REQ_response.adoc
[RidPermission]: https://github.com/opengeospatial/ogcapi-features/blob/9ca25f56a58ed822ea8a685a7a41afa7181aaa8b/extensions/transactions/create-replace-update-delete/standard/recommendations/create-replace-delete/create/PER_rid.adoc
[FeatureSchema]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/schemas/geojson/feature.json
[SystemSchema]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/schemas/geojson/system.json
[SystemUris]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/schemas/common/uris.json
[Cases]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-standards/corpus/fixtures/cases.json
[ValidationTests]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-standards/src/validation/tests.rs
[ProjectionTests]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-standards/src/projection/tests.rs
[DirectionDoc]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/direction-validation.md
[Quantity]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/swecommon/schemas/json/Quantity.json
[Label]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/swecommon/schemas/json/AbstractSweIdentifiable.json
[Encodings]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/swecommon/schemas/json/encodings.json
[SweWrapper]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/schemas/json/observationSchemaSwe.json
[Observation]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/schemas/json/observation.json
[DataStream]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/schemas/json/dataStream.json
[ControlStream]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part2/openapi/schemas/json/controlStream.json
[Swe]: https://docs.ogc.org/is/24-014/24-014.html
[DirectionAnnotations]: https://json-schema.org/draft/2020-12/json-schema-validation#section-9.4
[Clients]: ../../../../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-066-aleph-connected-systems-ui-client-study-report.md#short-cross-study-note--no-earlier-report-reopened

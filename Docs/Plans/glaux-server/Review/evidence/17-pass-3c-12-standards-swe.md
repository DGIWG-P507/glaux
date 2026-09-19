# Pass 3c, iteration 12 — standards batch: SWE Common 3.0 quality and array flags

**Date:** 2026-09-19
**Provider/model:** GitHub Copilot; model self-reported as Claude Fable 5.1
**Batch:** `standards-quality`, `standards-array-flags` (from `current_work.next_batch` at planning commit 48f0d53068cf86daaa132168c0f287a4c9115637)
**Mode:** read-only review of planning documents, pinned sources and issue bodies; review artifacts updated and published; no implementation, Goal/Guide/Roadmap, issue, settings or upstream changes.

## 1. Sources consulted this batch

| Source | Access | Used for |
|---|---|---|
| OGC 24-014 SWE Common Data Model 3.0 (published HTML, https://docs.ogc.org/is/24-014/24-014.html, Publication Date 2025-07-16) | Full document retrieved; clauses 7.4.1, 8.2.2, 8.2.3, 8.2.15, 8.7.1, 9.1, 9.6.1, 10.2.3, Annex A.3.1, A.4.2 (A.85, A.86), Annex B.2.4 read | Both questions |
| Published JSON schemas at https://schemas.opengis.net/sweCommon/3.0/json/ — `AbstractSimpleComponent.json`, `AbstractDataComponent.json`, `Quantity.json`, `sweCommon.json`, `encodings.json`, `DataArray.json` | Retrieved 2026-09-19 (live URL, no commit pin; the research also used the live registry, IDR-022 Section 3 access 2026-09-14) | Both questions |
| Guide v1.3: Section 4.3 lines 408–414 (quality interpretation, dynamic quality), Section 13 rows at lines 1150 and 1152, Section 7 table row line 889 (`swecommon-json`), Section 8.4 workflow 9 line 976 | Read | Interpretation rows under review |
| IDR-022 (SWE Common data component strategy) lines 81, 144, 198, 200, 247, 299, 397, 542, 612, 646, 710, 778 | Targeted read | Research position and pins |
| Upstream-history register v1.16 row #71 (line 151) and IDR-022/023 handoff lines 301, 337 | Read | Research claim under verification |
| Roadmap v1.18 leaves 2.1.10 (#36), 2.1.11 (#37), 4.1.3 (#130), 4.1.5 (#132); issue #130 body read in full | Read | Owning leaves and their acceptance text |

## 2. Question 1 — `standards-quality`: "published simple-component JSON schema omits conceptual `quality`" (Guide Section 4.3 line 410; Section 13 line 1152)

### Result: **Supported.** No correction to the Guide interpretation is required.

### Evidence

1. **Conceptual model declares `quality`.** 24-014 Clause 8.2.3 ("Attributes shared by all simple data components"): "The optional 'quality' attribute is used to provide simple quality information as discussed in Clause 7.4.1. It is of type 'Quality' which is a union of several classes as defined in Clause 8.2.15. Its multiplicity is more than one which means that several quality measures can be given on for a single data component." Clause 8.2.15 (Quality Union) names the members: Quantity, QuantityRange, Category and Text, and its NOTE states that reusing data components for quality "allows the inclusion of quality values in the data stream itself". Clause 7.4.1 distinguishes static quality from dynamic quality "carried by another component of the dataset".
2. **Published JSON schema does not declare it.** `AbstractSimpleComponent.json` declares `referenceFrame`, `axisID`, `nilValues`, `constraint` and `value` only; `AbstractDataComponent.json` adds `type`, `updatable`, `optional`, `definition`; `Quantity.json` adds `uom` and constrains `constraint`, `nilValues`, `value`. No `quality` member appears in any of these files.
3. **An extra `quality` member is schema-permitted but unvalidated.** None of `sweCommon.json` (entry point; `oneOf` over component types discriminated by `type`), `AbstractSimpleComponent.json`, `AbstractDataComponent.json` or `Quantity.json` sets `additionalProperties: false` or `unevaluatedProperties: false`. Therefore Requirement 55 `/req/json-simple-components/schema-valid` ("valid with respect to the JSON schema 'sweCommon.json'") is not violated by a `quality` member, and Abstract Test A.54 cannot detect a malformed one. This is exactly the Guide's statement: "permissive acceptance of an unknown member is not validation".
4. **Guide interpretation versus standard.** The Guide's array-of-components reading follows the multiplicity statement in 8.2.3 and the union in 8.2.15. The inline-value rule (a component with a value supplies static quality; a description without a value does not imply zero uncertainty) follows 8.2.3's descriptor/container distinction. The local `href: "#id"` reference for dynamic quality is a Glaux choice: 24-014 says dynamic quality is carried by another component but defines no JSON syntax for that association. The Guide labels it correctly as "a Glaux interpretation … not a claim that the published JSON schema defines this form". The reference-object shape is consistent with the schema's own association pattern (`basicTypes.json#/$defs/AssociationAttributeGroup`, used for `elementCount` references in `DataArray.json`), which is supporting evidence, not a normative endorsement.

### Planning and test implications

- Requirement 58 `/req/json-simple-components/inline-value-constraint-valid` applies to any inline quality component that carries a value; leaf 2.1.10 (#36) already requires constraint/unit validation of quality entries, so no new acceptance text is needed.
- Because the vendored schema cannot reject a malformed `quality`, the Glaux semantic check is the only guard; 2.1.10's "malformed quality strings, wrong types and invalid units fail" and Guide Section 8.4 workflow 9 ("malformed `quality`") cover this. Written tests remain planned, not executed, evidence.
- The legacy OSH `SWEJsonBindings` pin cited as `[QualityPeer]` is peer evidence only; the conclusion above does not depend on it.

## 3. Question 2 — `standards-array-flags`: `recordsAsArrays` / `vectorsAsArrays` "true means arrays" (Guide Section 13 line 1150; register #71)

### Result: **Supported**, and the register's description of the source defect is **confirmed verbatim**. One consequence for conformance evidence is added (Section 3.3).

### 3.1 Published text, as retrieved

- Clause 8.7.1 (JSONEncoding class): "The 'recordsAsArrays' attribute specifies whether 'DataRecord' values are encoded as JSON objects or JSON arrays. … Both attributes are optional and default to false, meaning 'DataRecord' and 'Vector' values are per default encoded as JSON objects."
- `encodings.json#/$defs/JSONEncoding`: `recordsAsArrays` — `"type": "boolean", "default": false, "description": "If true, DataRecord values are encoded as JSON arrays instead of JSON objects"`; `vectorsAsArrays` identical for Vector.
- Clause 10.2.3 introductory sentence: "'DataRecord' and 'Vector' components are encoded using a JSON Object whose members are named like the record fields per default. The attributes vectorAsArrays and recordsAsArrays of the corresponding JSON Encoding can be used to switch to a more compact encoding using JSON arrays." (singular `vectorAsArrays` typo confirmed; the schema property is `vectorsAsArrays`.)
- Requirement 85 `/req/json-encoding-rules/record-object-valid` part A: "If the attribute 'recordsAsArrays' of the corresponding 'JSONEncoding' is true, all 'DataRecord' values shall be encoded as JSON objects, else as JSON arrays. If the 'recordsAsArrays' or the corresponding 'JSONEncoding' is omitted, 'DataRecord' values shall be encoded as JSON objects by default." Requirement 86 part A repeats the pattern for `vectorsAsArrays`.
- Requirement 85 part C: "If 'DataRecord' values are encoded as JSON arrays, the order of JSON array items shall be the same as the 'DataRecord' fields. … If a record field is marked as 'optional', the corresponding JSON array item can be set null, but cannot be omitted." (Part B: object form, optional member may be omitted or null.)
- Annex B.2.4 (informative), "when vectorsAsArrays is true": `"location": [ 45.3, -90.5, 311 ]`.
- Clause 9.6.1 example: `{"type": "JSONEncoding", "recordsAsArrays": false, "vectorsAsArrays": false}`.

### 3.2 Assessment

The literal sentence in Requirements 85A/86A (true → objects, else → arrays) contradicts the attribute names, the UML prose in 8.7.1, the 10.2.3 introduction, the published schema descriptions and defaults, and the informative example; it also contradicts itself, because "omitted" is the default `false` yet is assigned objects while explicit `false` is assigned arrays. The register #71 row states this accurately. The only reading consistent with every other source is the Guide's: `true` means arrays, default/omitted means objects. Confirmed as **supported**. The Guide correctly requires that the contradictory requirement text and both fixture shapes be recorded rather than presenting the interpretation as an upstream correction; issue #130 (4.1.3) carries the same instruction ("Retain the contradictory source wording and interpretation qualification in the test explanation"; "do not … present the array-flag interpretation as an upstream correction") and tests omitted/false/true independently and mixed, with a reversal required to fail.

### 3.3 Added consequence — Annex A tests cover the object form only

Abstract Tests A.85 `/conf/json-encoding-rules/record-object-valid` and A.86 `/conf/json-encoding-rules/vector-object-valid` prescribe test methods that check only that "Each JSON value corresponding to a record component is a JSON Object" (respectively "vector component is a JSON Object") with member names equal to field/coordinate names. They contain no step for the array form. Consequences:

- An array-form value block (`recordsAsArrays: true` or `vectorsAsArrays: true`) cannot pass the literal A.85/A.86 test method even though Requirement 85C/86C defines its rules. Any Glaux conformance evidence for array-form values is therefore an **adapted** check (against 85C/86C: descriptor order; optional field → `null`, never omitted) and must be labelled as adapted, not as an unmodified official pass. This is the same discipline already recorded for F-08 and F-09; it is recorded there as a related instance rather than as a new finding.
- The Guide Section 13 row already says "needs explicit conformance interpretation"; a one-clause addition naming A.85/A.86's object-only scope would make the required labelling concrete. Owner: the conformance-claim leaves that assemble the `swecommon-json` class evidence (Guide Section 7 row line 889; Phase 9 reverification), not leaf 4.1.3, whose codec tests are already adequate.
- Editorial note in the standard: A.85's test method says "same names as the fields in the vector description" for the record test (copy error). No action for Glaux beyond recording it with the other artifact seams (IDR-022 line 542 register).

### 3.4 Planning and test implications

- No change to the Guide's selected interpretation, to leaf 4.1.3 (#130), or to Roadmap sequencing.
- Optional documentation clarification (Section 13 row 1150): cite A.85/A.86 object-only test methods and the adapted-check labelling for array-form fixtures.
- No upstream filing is authorized by this review; the register lists #71 as closed and IDR-022 line 646 hands rechecking to IDR-057. The Goal (line 58) already requires documenting the selected interpretation and revisiting it when authoritative corrections appear.

## 4. Material follow-up questions recorded, not scheduled

These arose from the same schema reads. They are recorded so a later batch can decide whether they matter; recording them does not make them review obligations.

1. `encodings.json` root `oneOf` lists `TextEncoding`, `XMLEncoding` and `JSONEncoding` but not `BinaryEncoding`. `DataArray.json` references each encoding via `encodings.json#/$defs/...` directly (including `BinaryEncoding`), so component validation is unaffected; the omission matters only if anything validates against the `encodings.json` root. Relevant to the `swecommon-binary` class (Guide line 891; Phase 4.3 leaves) only if a root-level validation is ever used.
2. `Quantity.json` requires `type`, `definition`, `label` and `uom`, whereas UML Clause 8.2.2 describes `label` as optional. Under Requirement 55 a `Quantity` without `label` is schema-invalid. A single grep of IDR-022/023/024 found no explicit record of the required `label` (IDR-022 line 536 mentions "required … labels" only as a profile item); this absence check was one search, not a full read. Affects minimal fixtures and publisher-supplied schemas in Phase 2.1/4.1 leaves if unrecorded.
3. `XMLEncoding` remains defined in `encodings.json` and `DataArray.json#encoding.oneOf` although XML encodings were removed from 3.0 conformance — already recorded by IDR-022 line 198; no new action.

## 5. Disposition summary for the checkpoint

| Check id | Disposition | Guide change needed | Owning work |
|---|---|---|---|
| `standards-quality` | Supported (schema omission and semantic interpretation verified against 24-014 8.2.3/8.2.15/7.4.1 and published schemas) | None | 2.1.10 (#36), 2.1.11 (#37), 4.1.5 (#132) unchanged |
| `standards-array-flags` | Supported; register #71 confirmed; A.85/A.86 object-only test methods added as an adapted-test labelling consequence | Optional one-clause addition to Section 13 row 1150 | 4.1.3 (#130) unchanged; conformance-claim/Phase 9 leaves for labelling |

Remaining standards checks after this batch: `standards-features-cql2`, `standards-sensorml-classes`, `standards-ats-rows`, `standards-part2-inheritance`, plus `observation-extension-policy` (F-13).

## 6. Statement of limits

- Schema files were read from the live `schemas.opengis.net` registry on 2026-09-19; no commit pin exists for that registry, so a later silent change cannot be excluded. The research used the same registry (IDR-022, accessed 2026-09-14).
- Conclusions concern published text and planning documents. No Glaux code exists yet; nothing here is executed verification.
- The two "material follow-up questions" were not investigated beyond the reads listed in Section 1.

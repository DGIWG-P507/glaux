# Glaux Server review findings and assessment

**Checkpoint:** September 19, 2026, through Copilot Pass 3c iteration 14 (standards batch: the nine remaining IDR-011 Section 14.3 abstract-test rows, all confirmed against the published Annex A text, and the Part 2 Annex A.1 inheritance question, supported).
**Status:** Partial independent review; no implementation changes approved or performed by this checkpoint.
**Purpose:** Preserve paid-for work and let another reviewer continue without chat memory or a restart.

Read this file and [the continuation instructions](instructions.md) first. Consult archived reports only for a specific finding or evidence question. Do not load the entire archive just to resume.

## What is saved, and what is not

- All **15 supplied Copilot responses** are preserved in `evidence/`. Public copies normalize local workstation paths where needed; the remaining responses are byte-identical to the supplied files.
- Original and published byte counts and SHA-256 hashes, plus the exact privacy transformations, are in [the source manifest](evidence/source-manifest.json). Untouched originals were also retained locally. Hashes establish copy integrity, not correctness of the review.
- The superseded long continuation brief is retained as `evidence/16-superseded-resumption-brief.md` for history only. It is **not** active instruction.
- The **22 numbered findings, including two withdrawals**, are consolidated below. Reported severities are preserved; they are not newly validated scores or approvals to implement.
- This file is the authoritative findings record. [The machine-readable checkpoint](review-state.json) tracks all **71 topic-report filenames**, all **286 implementation-issue IDs**, review coverage and the active handoff. It does not duplicate the findings.
- Source-report facts and Codex's subsequent qualifications are distinguished here. Some latest qualifications were not yet incorporated by Copilot when the last supplied response was written.
- These copies are the review reports, not a complete archive of every standard, repository page or issue body the reviewer consulted. Pinned project sources and targeted original evidence may still need to be accessed.
- No private Copilot memory file has been located, read or verified. This handoff does not depend on it.
- The shared home is `Docs/Plans/glaux-server/Review` in the planning repository. Git history records published updates; reviewers must distinguish saved local work from a verified remote commit. Provider memory is not the record.

## Interim assessment from the existing work

The available review has produced specific licensing, repository-control, interpretation, test and accountability concerns. It has **not established a need to discard the research, replace the server architecture or rewrite the entire Roadmap**. It also has **not completed the promised comprehensive validation**.

The earliest recorded action candidates are project-license selection around #4 and required-check enforcement after #6/before subsequent code merges. Later candidates have their own affected task or phase. They should not all become blanket prerequisites to the first foundation task. This is a summary of existing recommendations, not authorization to start coding or a guarantee that unreviewed areas contain no defects.

No defensible overall effort percentage, remaining dollar cost or remaining turn count has been established. "15 of 286 issue bodies read" is a coverage count, not proof that 95% of the effort remains. The earlier broad characterization of being less than halfway through the work was not a measured effort estimate.

## Coverage at the checkpoint

| Area | Documented work | Not established complete |
|---|---|---|
| Baseline | Goal v1.8, Guide v1.3, Roadmap v1.18, README, CONTRIBUTING and issue template read | Only later changes would need targeted checking |
| Standards | Substantial pinned-source checks; Part 1 normative body/Annex A read; targeted Part 2 and other checks; SWE Common 3.0 quality and array-flag interpretations verified against 24-014 and the published schemas (iteration 12); Features Part 3/CQL2 class identifiers, dependencies, `cql2-json`-only default and GeometryCollection `minItems` verified against 19-079r2, 21-065r2 and `cql2.json`, and the four SensorML 3.0 JSON class identifiers verified against 23-000 (iteration 13); all thirteen IDR-011 Section 14.3 abstract-test rows confirmed against the published 23-001/23-002 Annex A text and the Part 2 Annex A.1 inheritance chain verified (iteration 14) | F-13 extension-rule check |
| Research | Synthesis/register read; full reads of 011, 029, 031, 036, 041, 050, 052; other partial reads | Corpus-wide key-section sweep and several previously promised deep reads |
| Scenarios | Selected transaction, command, security and disconnected-operation analysis | Systematic end-to-end pass |
| Test quality | Relevant research and selected issue assertions examined | Systematic verification-credibility pass |
| Issues | #3, #15, #19, #21, #22, #56, #71, #72, #104, #156, #163, #164, #168, #174, #240 reported fully read | 271 other bodies; complete dependency/coverage/sizing review |
| Final report | This preserved partial assessment | Completed comprehensive assessment |

The original six-pass review is the completion objective, as summarized in [the start page](README.md). Work is performed in bounded, user-authorized iterations. A subscription limit creates a handoff, not a restart or a completed-review claim. Scope changes must be explicit; no reviewer may quietly convert the remaining inventory into unlimited work.

### Remaining checks already named in the review

Standards: F-13 extension rules only. (SWE quality and array flags were closed as supported in iteration 12; Features Part 3/CQL2 identifiers with GeometryCollection `minItems` and the SensorML class identifiers were closed as supported in iteration 13; the nine remaining IDR-011 Section 14.3 abstract-test rows were confirmed and the Part 2 Annex A.1 inheritance question was closed as supported in iteration 14; see "Resolved standards checks" below.)

Research portions: IDR-008 sections 16-17; 037/038 appendices; 039 section 7 and 21-22; 040 sections 9-21; remaining 042/043 sections. Additional promised deep reads were 030, 034, 039A and 055. Some CS-GO/OSH pinned-source spot checks remain unaccounted for.

These entries preserve unfinished commitments. The broad research obligation is a key-section sweep plus consequential deep reads, not cover-to-cover rereading of every historical report or plan. Reuse documented work and keep this assessment usable during completion.

## Findings register

**Reading the status:** "Reported" describes the saved review, not a new audit. "Candidate" or "proposed" requires judgment before adoption. "Withdrawn" stays withdrawn absent specific new contrary evidence. No finding below records an implemented fix or project-lead approval of its proposed change.

### F-01 - Project license

**Status:** Reported omission. **Reported severity:** Medium.
**Affected work:** #4; later release checks.

The inspected baseline had no project license; dependency and bundled-artifact notices do not select one. License selection is an owner/organizational decision, not delegated by the review.

**Next consideration:** Confirm/select the project license before the affected dependency/code work; do not select one automatically.

**Saved evidence:** [01-pass-1.txt](evidence/01-pass-1.txt), starting at line 205. Consult later qualifications below where applicable.

### F-02 - Enforced repository checks

**Status:** Reported risk; baseline-specific. **Reported severity:** High.
**Affected work:** #6 / before subsequent code merges such as #7.

The review reported no enforced required-check gate at its inspected server baseline. Written assistant review/merge rules do not themselves configure GitHub enforcement. This checkpoint does not assert that live settings are unchanged.

**Next consideration:** Confirm actual controls when action is authorized; decide required CI enforcement. Do not add a mandatory human-review gate or change settings automatically.

**Saved evidence:** [01-pass-1.txt](evidence/01-pass-1.txt), starting at line 214. Consult later qualifications below where applicable.

### F-03 - Minimal authorization semantics

**Status:** Narrowed clarification candidate. **Reported severity:** Low-Medium.
**Affected work:** #22.

Issue #22 already requires an independent action/source/resource matrix, allowed/denied/unavailable cases and fault checks. The residual question is selected action/scope/inheritance/source-binding/concealment semantics, not absence of an authorization model or tests.

**Next consideration:** Make the selected semantics explicit enough for independent expectations. A universal policy engine and research-specific approval/lease machinery are not automatically needed.

**Saved evidence:** [03-pass-3a.txt](evidence/03-pass-3a.txt), starting at line 101. Consult later qualifications below where applicable.

### F-04 - Issue size

**Status:** Acknowledged planning risk; nonblocking. **Reported severity:** Low.
**Affected work:** Relevant implementation leaves / phase recalibration.

Some leaves may exceed one iteration. The Roadmap already provides splitting and recalibration rules. No demonstrated need to rewrite the entire backlog follows.

**Next consideration:** Recalibrate against implementation evidence; no automatic splitting in this review.

**Saved evidence:** [01-pass-1.txt](evidence/01-pass-1.txt), starting at line 232. Consult later qualifications below where applicable.

### F-05 - Tasking sequence

**Status:** Optional scheduling improvement. **Reported severity:** Low.
**Affected work:** #156 / group 5.1.

Current dependencies delay tasking until the observation encodings are integrated. Earlier JSON-only tasking was proposed as a scheduling alternative, not a missing capability or standards defect.

**Next consideration:** Leave as optional unless the owner changes sequencing; do not rewrite dependencies automatically.

**Saved evidence:** [01-pass-1.txt](evidence/01-pass-1.txt), starting at line 238. Consult later qualifications below where applicable.

### F-06 - Exchange subsystem lacks a consumer

**Status:** Withdrawn. **Reported severity:** Withdrawn.
**Affected work:** None.

The reviewer withdrew the premise: configured peers/files, SSE bootstrap and MQTT recovery are consumers, and Phase 8 also contains ordinary recovery work.

**Next consideration:** Preserve withdrawal. Do not resurrect the original finding after compaction.

**Saved evidence:** [03-pass-3a.txt](evidence/03-pass-3a.txt), starting at line 104. Consult later qualifications below where applicable.

### F-07 - resultTime=latest interpretation

**Status:** Interpretation/documentation concern. **Reported severity:** Medium.
**Affected work:** #112 / observation selection.

The Guide selects the maximum resultTime within the authorized, filtered endpoint scope, retaining ties. Research already chose that interpretation. Peer per-series behavior is informative, not normative; nested datastream endpoints can still contain multiple sampling-feature series.

**Next consideration:** Record the interpretation and relevant interoperability limits clearly; use discriminating fixtures. A semantic change or upstream filing is not authorized.

**Saved evidence:** [03-pass-3a.txt](evidence/03-pass-3a.txt), starting at line 107. Consult later qualifications below where applicable.

### F-08 - Abstract-test prerequisite conflicts

**Status:** Narrowed interpretation/documentation concern. **Reported severity:** Low-Medium.
**Affected work:** #80 and Part 2 inheritance follow-up.

The prerequisite conflict was already recorded in research; the original claim that it was unrecorded everywhere was withdrawn. Annex A is normative, so an adapted procedure needs an explicit qualification.

**Next consideration:** Carry the selected treatment into test evidence; distinguish original from adapted abstract-test outcomes. The Part 2 inheritance check was closed as supported in iteration 14 (related instance below); the requested Section 13 row should be worded for both Parts.

**Related instance (iteration 12, SWE Common 3.0):** Abstract Tests A.85 `/conf/json-encoding-rules/record-object-valid` and A.86 `/conf/json-encoding-rules/vector-object-valid` in OGC 24-014 prescribe test methods that check only the JSON-object form of DataRecord/Vector values. Array-form values (`recordsAsArrays`/`vectorsAsArrays` true) therefore cannot pass the literal test method even though Requirement 85C/86C defines their rules; any Glaux evidence for array-form values is an adapted check and must be labelled as such. Owning work: the conformance-claim leaves assembling `swecommon-json` evidence (Guide Section 7 row for `swecommon-json`; Phase 9 reverification), not the codec leaf 4.1.3 (#130), whose tests are adequate. Evidence: [17-pass-3c-12-standards-swe.md](evidence/17-pass-3c-12-standards-swe.md), Section 3.3.

**Related instance (iteration 13, CQL2 and Features Part 3):** In OGC 21-065r2 Annex A the tests "against the test dataset" (A.3.5, A.3.6, A.7.2, A.8.2) are conditional on hosting the Natural Earth feature collections, which an observation-only server cannot host as Features collections; they must be recorded as not applicable (conditional), never as passed. No abstract test exercises a singleton GeometryCollection, so the Guide Section 13 row 1162 acceptance is a Glaux adaptation that only Glaux fixtures (leaf 7.3.3, #231) can evidence. Conversely, A.3.2 `/conf/basic-cql2/comparison` evaluates all six comparison operators against every String and Boolean queryable and expects success, and A.7.1/A.8.1 expect `POINT(90 180)` to fail: these are official expectations the runner must implement literally, not adaptations. Owning work: 7.3.1-7.3.4 (#229-#232), 7.2.6 (#228) and the Phase 9 claim leaves. Evidence: [18-pass-3c-13-standards-cql2-sensorml.md](evidence/18-pass-3c-13-standards-cql2-sensorml.md), Sections 2.5-2.6.

**Related instance (iteration 14, Part 2 inheritance and the IDR-011 abstract-test overlay):** OGC 23-002 Clause 8 `/req/api-common` and Annex A.1 `/conf/api-common` both name the Part 1 `api-common` class as prerequisite, and every Part 2 resource conformance class (A.2-A.6) lists Part 2 `/conf/api-common`; so every Part 2 declaration inherits the Part 1 Annex A prerequisite conflict already covered by this finding, and Part 2 adds no further inherited class beyond the Guide Section 1.2 table. Part 2 A.1 has no test of its own; A.2 runs eighteen Features Part 1 core tests only for exposed collections whose `itemType` is not `feature`, and Part 2 collections are optional, so Part 2 `api-common` evidence must be labelled as the inherited Part 1 result plus conditional A.2 tests, never as a vacuous pass. Separately, all nine remaining IDR-011 Section 14.3 rows were confirmed verbatim in the published Annex A text (23-001 A.65; 23-002 A.31/A.33, A.35, A.36, A.40/A.42, A.43, A.50, A.61, A.62): the corrected procedures Glaux runs for those tests are adapted checks against the normative requirements and must be labelled as such, and where the official test is vacuous (Part 2 canonical-URL tests without exposed collections) or silent (`latest`, `limit`/`datetime` on status and result endpoints) the supplement is a Glaux check, not an official pass. Owning work: 2.6.1-2.6.3 (#80-#82), 3.5.1-3.5.2 (#121-#122), 5.5.16 and the Phase 9 claim leaves. Evidence: [19-pass-3c-14-standards-ats-rows-part2-inheritance.md](evidence/19-pass-3c-14-standards-ats-rows-part2-inheritance.md), Sections 2-3.

**Saved evidence:** [03-pass-3a.txt](evidence/03-pass-3a.txt), starting at line 113. Consult later qualifications below where applicable.

### F-09 - deployedSystems abstract-test assumptions

**Status:** Source-conflict / test-adaptation concern. **Reported severity:** Low.
**Affected work:** #81.

Some abstract-test steps presume collection dereferencing, while the selected mappings use inline associations. The claimed normative /deployments/{id}/systems route was withdrawn.

**Next consideration:** Document the specific adaptation and independent association expectation. Do not add an invented route or report an adapted test as an unmodified official pass.

**Saved evidence:** [03-pass-3a.txt](evidence/03-pass-3a.txt), starting at line 116. Consult later qualifications below where applicable.

### F-10 - Malformed research source links

**Status:** Reported reproducibility defect. **Reported severity:** Low.
**Affected work:** IDR-011 source links / affected conformance tasks.

Nine IDR-011 links reportedly use a malformed shortened commit hash. The checked Guide/Roadmap pins were not the affected links.

**Next consideration:** Correct the specific links when authorized; retain the pinned source rather than substituting a moving branch.

**Saved evidence:** [02-pass-2.txt](evidence/02-pass-2.txt), starting at line 143. Consult later qualifications below where applicable.

### F-11 - Relation wire spelling

**Status:** Interpretation/documentation concern. **Reported severity:** Medium.
**Affected work:** #56 / relationship and conformance fixtures.

The standard's tables, mappings and examples have conflicting relation spellings. Issue #56 already checks independent expected edges, relation meaning and canonical URLs: missing navigation tests is not the finding.

**Next consideration:** Document the selected spelling/compatibility rule in existing fixtures. Do not prefix unrelated IANA relations or JSON properties or case-fold target URLs.

**Saved evidence:** [03-pass-3a.txt](evidence/03-pass-3a.txt), starting at line 137. Consult later qualifications below where applicable.

### F-12 - Independent response interpretation

**Status:** Narrowed optional hardening. **Reported severity:** Low.
**Affected work:** #19 and runner extensions such as #80.

The Guide already specifies independent expectations and prohibits implementation logic as the sole oracle. The residual concern is using production wire types to interpret supposedly independent responses.

**Next consideration:** If useful, clarify the response-interpretation boundary and a discriminating negative fixture. A new crate, particular runner tool or dependency graph alone is neither required nor sufficient.

**Saved evidence:** [05-pass-3c-01.txt](evidence/05-pass-3c-01.txt), starting at line 117. Consult later qualifications below where applicable.

### F-13 - foi@id example versus samplingFeature@id

**Status:** Source-example hazard; input policy unresolved. **Reported severity:** Low.
**Affected work:** Observation representation tasks; exact leaf not established here.

An extra foi@id member can be schema-permitted without establishing the standard samplingFeature@id association. The example's data URL is a separate issue. Rejecting every unfamiliar @id/@link member was not approved.

**Next consideration:** Use canonical association fixtures and finish the controlling extension-rule check before prescribing unknown-member rejection.

**Saved evidence:** [06-pass-3c-02.txt](evidence/06-pass-3c-02.txt), starting at line 47. Consult later qualifications below where applicable.

### F-14 - Three interpretation details

**Status:** Qualified documentation/mapping candidates. **Reported severity:** Low.
**Affected work:** #71/#72; Deployment writes; observation listing.

(a) Root-parent semantics and known-hit tests already exist in #71/#72. (b) The reported local Deployment write mapping requires a System UID URI; canonical-URL compatibility and an additional returned uid are separate choices. (c) Research selected ascending resultTime/ID order; Guide wording should not obscure direction.

**Next consideration:** Do not rewrite existing parent tests as if absent. Preserve required UID handling separately from optional conveniences; state ordering and its client consequences.

**Saved evidence:** [06-pass-3c-02.txt](evidence/06-pass-3c-02.txt), starting at line 35. Consult later qualifications below where applicable.

### F-15 - Unkeyed command submissions

**Status:** Explicit tradeoff / documentation comparison. **Reported severity:** Low-Medium.
**Affected work:** #168 / #174; Guide command semantics.

The Guide explicitly permits unkeyed POSTs, explains ambiguous recovery and forbids assuming safe automatic resubmission. Existing issues test keyed replay and potentially distinct unkeyed actions. Research had preferred mandatory keys.

**Next consideration:** Retain the stated operational limitation. A mandatory-key deployment option is a new choice, not an already approved correction.

**Saved evidence:** [08-pass-3c-04.txt](evidence/08-pass-3c-04.txt), starting at line 27. Consult later qualifications below where applicable.

### F-16 - Batch atomicity reversal

**Status:** Withdrawn. **Reported severity:** Withdrawn.
**Affected work:** None; #104 already tests the selected policy.

The research allowed multiple transaction models; the Guide explicitly chooses bounded request-level all-or-nothing behavior, and #104 specifies rollback tests. The original claim of an unrecorded inconsistent choice was withdrawn.

**Next consideration:** Preserve withdrawal. Do not prescribe per-item success or another batch protocol.

**Saved evidence:** [08-pass-3c-04.txt](evidence/08-pass-3c-04.txt), starting at line 44. Consult later qualifications below where applicable.

### F-17 - Stale acceptance wording

**Status:** Optional editorial cleanup. **Reported severity:** Low.
**Affected work:** Affected historical reports/plans.

Dated acceptance records exist for checked reports; some closing paragraphs or headers retain earlier In Review wording. The invented rule that a header automatically controls was corrected.

**Next consideration:** If cleaning up, use the actual dated acceptance record. Do not reinterpret accepted research as unaccepted or introduce new governance precedence.

**Saved evidence:** [08-pass-3c-04.txt](evidence/08-pass-3c-04.txt), starting at line 50. Consult later qualifications below where applicable.

### F-18 - ControlStream live/admission semantics

**Status:** Narrowed behavior clarification candidate. **Reported severity:** Low.
**Affected work:** #156 / #163 / #164; relevant Feasibility tasks.

Schema annotations constrain representations but do not supply a live-derivation algorithm or every admission rule. false and null, and Command and Feasibility, need distinct consideration. The research's 409 recommendation is not automatically the only standard-compliant outcome.

**Next consideration:** State and test the selected behavior within existing owners. Do not invent a stream lifecycle, conflate Feasibility with actuation, or infer unrestricted ownership from a missing annotation.

**Saved evidence:** [10-pass-3c-06.txt](evidence/10-pass-3c-06.txt), starting at line 21. Consult later qualifications below where applicable.

### F-19 - Audit modification and retention boundary

**Status:** Clarification candidate. **Reported severity:** Low-Medium.
**Affected work:** #15; retention/configuration owners.

The explicit Guide floor covers durable transactional accountability, not a complete tamper-evidence architecture. The reviewer withdrew its INSERT-implies-UPDATE/DELETE premise. The remaining question is intended audit protection and retention behavior.

**Next consideration:** Separate serving access from owner/migration and authorized retention roles. Append-only recommendations must account for permitted disposition; hash chains are not automatically required.

**Saved evidence:** [15-pass-3c-11.txt](evidence/15-pass-3c-11.txt), starting at line 55. Consult later qualifications below where applicable.

### F-20 - Denial-audit deliverable

**Status:** Clarification/test candidate. **Reported severity:** Low.
**Affected work:** #15 / #22.

The Guide requires relevant denials to be recorded safely; the reviewed issue text allows safe denial auditing but does not explicitly test selected record creation/failure behavior. Existing owners are present. Proposed event categories are suggestions.

**Next consideration:** Define selected categories, bounds and failure behavior if adopted. Audit failure must never authorize the denied action; no independent spool follows automatically.

**Saved evidence:** [15-pass-3c-11.txt](evidence/15-pass-3c-11.txt), starting at line 56. Consult later qualifications below where applicable.

### F-21 - Node-local versus exchanged audit history

**Status:** Documentation clarification candidate. **Reported severity:** Low.
**Affected work:** #240 / exchange documentation.

The described exchange format is not a replication protocol for the complete authenticated audit trail. Supplied production context may still travel with resources. Full cross-node audit durability was not promised.

**Next consideration:** Clarify the boundary if useful. Backup protects its captured recovery point, not every later event. Do not automatically add audit synchronization.

**Saved evidence:** [15-pass-3c-11.txt](evidence/15-pass-3c-11.txt), starting at line 74. Consult later qualifications below where applicable.

### F-22 - Exporter-side accountability

**Status:** Proposed strengthening / credible risk. **Reported severity:** Medium.
**Affected work:** #240 / Guide audit floor.

No explicit exporter-side durable audit requirement was found in the checked Guide and export issue. Import receipts are not proof of an exporter-side record. Implementation has not begun, so this is not proof of an observed missing runtime record.

**Next consideration:** Consider scoped export auditing. First distinguish generation, release/handoff and confirmed receipt; protect actor/recipient/scope metadata and define failure/partial-output behavior. No delivery-receipt platform is implied.

**Saved evidence:** [15-pass-3c-11.txt](evidence/15-pass-3c-11.txt), starting at line 81. Consult later qualifications below where applicable.

## Resolved standards checks (no finding)

These carried standards questions were closed with evidence and produced no new finding. They are recorded here so a successor does not redo them; the structured status lives in `remaining_checks` of [review-state.json](review-state.json).

| Check | Disposition | Where verified |
|---|---|---|
| SWE Common 3.0 `quality` schema/semantic claim (Guide Section 4.3 and Section 13 row) | **Supported.** 24-014 Clause 8.2.3 declares an optional multi-valued `quality` attribute of the Clause 8.2.15 union; the published `AbstractSimpleComponent.json` declares no `quality` member and no schema sets `additionalProperties`/`unevaluatedProperties: false`, so the member is schema-permitted but unvalidated. The Guide's array-of-components reading follows the model; its local `href` reference for dynamic quality is a labelled Glaux choice. No Guide correction. | [17-pass-3c-12-standards-swe.md](evidence/17-pass-3c-12-standards-swe.md), Section 2 |
| `recordsAsArrays`/`vectorsAsArrays` "true means arrays" (Guide Section 13 row; register #71) | **Supported; register #71 confirmed verbatim.** Requirement 85A/86A literally reverse the booleans and contradict their own default sentence; Clause 8.7.1, the 10.2.3 introduction, `encodings.json` and Annex B.2.4 all make `true` mean arrays. Consequence added under F-08: A.85/A.86 test only the object form. Optional one-clause clarification to the Guide row; no upstream filing authorized. | [17-pass-3c-12-standards-swe.md](evidence/17-pass-3c-12-standards-swe.md), Section 3 |
| Features Part 3/CQL2 class identifiers, dependencies, `cql2-json`-only default and GeometryCollection `minItems` (Guide Section 7.4, Section 6.3.1 line 799, Section 13 row 1162) | **Supported.** All six `/conf/` URIs exist verbatim in 19-079r2 and 21-065r2; the dependency prose matches Clauses 6.1/7.5/7.6/8.3 and 19-079r2 Clause 8.1/Annex A.1. Listing only `cql2-json` and making it the default is conformant under 19-079r2 Requirement 6 B/C (no language is mandatory; the text/JSON SHOULDs belong to the unselected Features Filter class). Annex C.1 and the published `cql2.json` set `geometrycollection.geometries` `minItems: 2` while Clause 7.6.1 prose and the Annex B BNF allow one or more, exactly as the Guide row states; singleton acceptance exceeds the Requirement 33/38 schema-validity floor and is untested by the official suite, so it remains a labelled Glaux adaptation. Test-evidence notes recorded under F-08. No Guide correction. | [18-pass-3c-13-standards-cql2-sensorml.md](evidence/18-pass-3c-13-standards-cql2-sensorml.md), Section 2 |
| SensorML 3.0 class identifiers `json-simple-process`, `json-physical-system`, `json-deployment`, `json-derived-property` (Guide Section 1.2 line 255) | **Supported.** OGC 23-000 (published 2025-07-16) defines all four under `http://www.opengis.net/spec/sensorML/3.0` with conformance classes A.12, A.15, A.16, A.17 (Requirements 47, 50, 51, 52, each schema validation against the published JSON schema). Published prerequisites: `json-physical-system` requires `json-aggregate-process` (A.13) and `json-physical-component` (A.14); all four rest on `json-core` (A.11; Requirement 46 media type `application/sml+json`, as the Guide uses). Optional clarification: name A.11/A.13/A.14 explicitly beside the four selected classes so the claim manifest cannot omit them. No Guide correction. | [18-pass-3c-13-standards-cql2-sensorml.md](evidence/18-pass-3c-13-standards-cql2-sensorml.md), Section 3 |
| Remaining IDR-011 Section 14.3 abstract-test discrepancy rows (nine of thirteen; Guide Section 7.2 adapted-test rule, Section 13 singular-path and System Event rows) | **Confirmed; all thirteen rows now verified against the published text.** 23-001 A.65 `indirect-prop` repeats `/systems` in its sampling-feature step (recommendation test, warning-only); 23-002 A.61 iterates Commands and reads `currentStatus` where Requirement 61 B filters CommandStatus by `statusCode`; A.35 iterates `itemType` Command for Feasibility; A.36 tests `/controlstreams/{dsId}/commands` for `/controlstreams/{csId}/feasibility`; A.40/A.42 use ControlStream targets for System Events; A.43 uses `/systems/{sysId}/systemEvents` against normative `/systems/{sysId}/events`; A.62 lowercases `/systemevents` and reads conceptual `type` (JSON `definition`); A.50 omits Requirement 50 D `latest`; A.31/A.33 never exercise Requirement 31 B/33 B `limit`/`datetime`. IDR-011's corrections stand; four further published seams (Requirement 61 A description, Requirement 31 B clause cite, A.66 label, vacuous Part 2 canonical-URL tests) are recorded as overlay candidates. No Guide correction. | [19-pass-3c-14-standards-ats-rows-part2-inheritance.md](evidence/19-pass-3c-14-standards-ats-rows-part2-inheritance.md), Sections 2 and 4 |
| Part 2 Annex A.1 `/conf/api-common` inheritance (Guide Section 1.2 line 262 bases; Section 7.1 `api-common` rows and closing paragraph) | **Supported.** 23-002 Clause 8 and Annex A.1 both inherit Part 1 `api-common`; every Part 2 resource class lists Part 2 `/conf/api-common`; the Part 1 Annex prerequisite conflict (F-08) therefore propagates to every Part 2 declaration, and no additional inherited class is introduced. Research already records the chain (IDR-008 Sections 7.2-7.3, IDR-007 P2-INT-008, IDR-006 INT-CS1-001). Part 2 A.2 is conditional on exposed non-feature collections. Related instance recorded under F-08; no Guide correction beyond wording the requested F-08 row for both Parts. | [19-pass-3c-14-standards-ats-rows-part2-inheritance.md](evidence/19-pass-3c-14-standards-ats-rows-part2-inheritance.md), Section 3 |

Material follow-up questions raised by the same reads (encodings.json root `oneOf` omitting `BinaryEncoding`; `Quantity.json` requiring `label`; the CQL2 Clause 7.6.1 GeometryCollection member list omitting LineString; the 23-000 Clause 9.1.2.1 draft media-type note; whether the Glaux ATS overlay absorbs the four additional published seams; whether Glaux exposes any Part 2 resource collections; the wording of the F-08 Section 13 row for both Parts) are recorded in `follow_up_questions` of the state file. They are not review obligations unless a later batch selects them.

## Important corrections preserved outside Copilot's last report

These are Codex review qualifications already communicated in the conversation. They do not silently overwrite the archived reviewer reports or count as the project lead accepting design changes.

- **F-12:** Existing independent-oracle provisions are substantive. A separate package or its dependency graph is not sufficient proof of oracle independence.
- **F-14:** Existing root-parent tests are not missing. Required UID-form write mapping, optional canonical-URL compatibility and optional returned UID metadata are separate choices.
- **F-18:** Do not turn a schema annotation into a complete ownership/derivation algorithm. Distinguish false/null and Command/Feasibility. No new lifecycle follows.
- **F-19:** Durable atomic records do not by themselves claim append-only protection or tamper evidence. Role separation and authorized retention need an explicit boundary if selected. The research's local hash/checkpoint recommendation was not conditional in every research profile, but that does not automatically make it a Guide requirement.
- **F-20:** Audit failure cannot authorize the denied action. This does not independently determine every HTTP failure response or require a separate failure spool. The proposed denial categories are not approved policy.
- **F-21:** Resource provenance can carry creator/context assertions without reproducing the source server's full authenticated audit trail. A backup preserves the recovery point, not all later history.
- **F-22:** Generation, release to an authorized file/sink, transport handoff and recipient receipt are different events. Actor/recipient/scope metadata can itself be protected. An export-audit requirement is a proposed strengthening, not proof of an already required implemented safeguard.
- **Offline authentication:** Existing JWTs can remain usable only within token, cached-trust and policy validity. Do not universally require a locally reachable identity provider for every such request; renewal and refresh are separate dependencies.
- **Offline command audit:** Transactional pre-dispatch records do not guarantee outcome recording after storage failure or rollback of physical effects. A local store does not make all network dependencies irrelevant.
- **Exchange integrity:** Retained exchange/revision/fingerprint evidence can reveal inconsistencies without a later resource change. Such checks are not source authentication and do not guarantee detection of consistently substituted unknown data.
- **Assurance:** Planned tests are not executed verification; one admitted dispatch is not exactly-once physical execution; refusing a TLS-1.3-only profile is not permission to use obsolete security settings.

## Continuing and using these findings

The current work cursor and the next bounded batch are maintained in [review-state.json](review-state.json). Follow [instructions.md](instructions.md); do not reconstruct the next task from historical reports or private memory.

Models may disagree with a finding. Record the evidence and the disagreement under that finding, continue the current batch, and settle its disposition during the relevant comparison or final consolidation. The Codex qualifications above are evidence-informed comments, not authority over an independent reviewer.

Keep review disposition separate from implementation: supported findings identify the affected issue and when a decision or fix is needed. Optional improvements remain optional; withdrawals remain visible. A completed review does not require implementing its recommendations.

## Source order

The archive filenames 01-15 are chronological: Pass 1, Pass 2, Pass 3a, Pass 3b, then Pass 3c iterations 1-11. Later reports can narrow or withdraw earlier claims. Original files are retained unchanged so a handoff does not erase those corrections or recreate rejected findings. File 16 is the superseded resumption brief (history only). From 17 onward, evidence files are reviewer-authored iteration reports saved directly to the repository during the iteration they describe; their manifest entries record the authored hash as both original and published.

Snapshot baselines reported by the review:
- Planning: [a310eaee2e80bb861197822a3c5bb12164ca9ac3](https://github.com/DGIWG-P507/glaux/tree/a310eaee2e80bb861197822a3c5bb12164ca9ac3).
- Server: [b1a80298305fd62164160058cde6d1794f533174](https://github.com/DGIWG-P507/glaux-server/tree/b1a80298305fd62164160058cde6d1794f533174).

Those are snapshot references, not claims that remote repositories can never change.

# Glaux Server review findings and assessment

**Checkpoint:** September 19, 2026, through Pass 3c iteration 23 (queue batch 5, the committed deep read of IDR-055). **All four committed deep reads are now complete and the research deep-read check is closed.** The batch established a material status fact: unlike the other three, IDR-055 carries no acceptance record and states that it awaits project-lead review, so its recommendations are proposals and the Guide correctly never cites it. That caveat is carried forward to the verification-quality pass. No new finding; the F-03 response-cache instance gained a fifth corroborating source, the first to express the rule as a testable leakage surface naming ETags. The queue stands at **27 batches, 5 done and 22 remaining**, recorded in `batch_queue` of [review-state.json](review-state.json).
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

No defensible overall effort percentage, remaining dollar cost or remaining turn count has been established. An issue-body read count, now 18 of 286 after the iteration 16 bookkeeping correction, is a coverage count and not proof that a matching share of the effort remains. The earlier broad characterization of being less than halfway through the work was not a measured effort estimate.

## Coverage at the checkpoint

| Area | Documented work | Not established complete |
|---|---|---|
| Baseline | Goal v1.8, Guide v1.3, Roadmap v1.18, README, CONTRIBUTING and issue template read | Only later changes would need targeted checking |
| Standards | Substantial pinned-source checks; Part 1 normative body/Annex A read; targeted Part 2 and other checks; SWE Common 3.0 quality and array-flag interpretations verified against 24-014 and the published schemas (iteration 12); Features Part 3/CQL2 class identifiers, dependencies, `cql2-json`-only default and GeometryCollection `minItems` verified against 19-079r2, 21-065r2 and `cql2.json`, and the four SensorML 3.0 JSON class identifiers verified against 23-000 (iteration 13); all thirteen IDR-011 Section 14.3 abstract-test rows confirmed against the published 23-001/23-002 Annex A text and the Part 2 Annex A.1 inheritance chain verified (iteration 14); the F-13 extension-rule question resolved against 23-002 Clause 16.1.5, the tagged and published `observation.json`, 23-001 Clause 19, RFC 7946 Section 6.1 and 23-000 Requirements 9/10 (iteration 15) | None outstanding for the carried checks; later passes may raise targeted standards questions |
| Research | Synthesis/register read; full reads of 008, 011, 029, 030, 031, 034, 036, 037, 038, 039, 039A, 040, 041, 042, 043, 050, 052, 055; six partial reads with recorded evidence. All four committed deep reads complete | Key-section screen of the 47 reports whose read depth was never recorded plus the unread sections of the six partial reports (queue batches 6-14) |
| Scenarios | Selected transaction, command, security and disconnected-operation analysis | Systematic end-to-end pass |
| Test quality | Relevant research and selected issue assertions examined | Systematic verification-credibility pass |
| Issues | #3, #15, #19, #21, #22, #56, #71, #72, #98, #99, #104, #130, #156, #163, #164, #168, #174, #240 reported fully read | 268 other bodies; complete dependency/coverage/sizing review |
| Final report | This preserved partial assessment | Completed comprehensive assessment |

The original six-pass review is the completion objective, as summarized in [the start page](README.md). Work is performed in bounded, user-authorized iterations. A subscription limit creates a handoff, not a restart or a completed-review claim. Scope changes must be explicit; no reviewer may quietly convert the remaining inventory into unlimited work.

### Remaining checks already named in the review

Standards: none outstanding. (SWE quality and array flags were closed as supported in iteration 12; Features Part 3/CQL2 identifiers with GeometryCollection `minItems` and the SensorML class identifiers were closed as supported in iteration 13; the nine remaining IDR-011 Section 14.3 abstract-test rows were confirmed and the Part 2 Annex A.1 inheritance question was closed as supported in iteration 14; the F-13 extension-rule question was resolved in iteration 15; see "Resolved standards checks" below.)

Research portions: all four partial-report remainder slices are closed. IDR-008 sections 16-19, IDR-037 appendices A-D, IDR-038 appendices A-C and IDR-039 section 7 and sections 21-22 closed as slice 1 in iteration 16; IDR-040 sections 9-21 as slice 2 in iteration 17; IDR-042's unread sections as slice 3 in iteration 18; IDR-043's unread sections as slice 4 in iteration 19. Those seven reports are now fully read. IDR-030 was then completed as queue batch 2 in iteration 20. Remaining research work: three committed deep reads, 034, 039A and 055, none yet begun (queue batches 3-5); the key-section screen of the 47 reports whose read depth was never recorded, together with the unread sections of the six partial reports (queue batches 6-14); and the CS-GO/OSH pinned-source spot checks, merged into queue batch 8.

**The complete remaining queue.** Every remaining finish condition across all six areas is mapped to a finite numbered batch in `batch_queue` of [review-state.json](review-state.json): **27 batches, 2 done and 25 remaining.** The queue separates genuinely unreviewed work from coverage that was simply never recorded, and from bookkeeping already corrected in iteration 16. Iteration 20 corrected two accounting problems in it. Issue fidelity is now established by a complete mechanical comparison of all 286 issues against their Roadmap leaves rather than by an 18-issue sample, because a sample cannot establish complete coverage; feasibility was verified against the live public API before that change was recorded. The six partial reports (IDR-006, 007, 010, 014B, 017 and 044) were omitted from the first version and are now assigned to the cluster batches already covering their topics, reusing their recorded evidence and screening only unread key sections, with no batch added for them. Three issue batches were combined as a consequence of the complete comparison, since matching leaf content is reused rather than reviewed twice, which is what moved the count from 30 to 27. The queue is a workload estimate with six recorded uncertainties, not a quota and not a completion guarantee.

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

**Related instance (iteration 16, IDR-038 command authorization/safety/audit matrix):** IDR-SRV-038 Appendix A describes a fuller decision architecture than the Guide adopts: a separation-of-duties "operator approval" gate, an independent "override" authority bound to non-overrideable rules with compensating controls, and an exclusive/coordinated "target/control lease." Guide v1.3 and the Goal are both silent on `lease`, `operator approval`/`separation of duties` and command-safety `override` (confirmed by targeted search of both documents). This is the same category of research-recommended machinery this finding already treats as optional rather than a required-and-missing control; the Guide's continued silence is a project choice already accounted for here, not a new departure. No Guide correction; no new finding. Evidence: [21-pass-3c-16-research-deep-read-slice1.md](evidence/21-pass-3c-16-research-deep-read-slice1.md), Section 5.

**Related instance (iteration 17, response cache validators and variants) — an evidenced gap, not an unadopted option.** Unlike the instance above, this one concerns the Guide's own concealment commitment and needs no policy model to fix. IDR-SRV-040 §10.4 requires every representation cache key and validator to include the authorized-view identity and forbids a shared cache serving one requester's representation to another context; its §16.3 invariant 6 lists "a cache, cursor, subscription or queued decision cannot outlive its policy/trust bounds." The Guide adopts the cursor, subscription and queued-decision members (lines 471, 547, 647) and omits the cache member. It emits strong representation-specific ETags for concurrency control (line 503) and `Vary` plus representation-specific ETags for content negotiation (line 739), and names RFC 9110/9111 as an inherited dependency (line 258), but nowhere binds a validator or cache key to the authorized view, specifies any `Cache-Control` directive, or lists a cache channel among its disclosure-leak tests (lines 1019, 983). The inherited standard does not close this: RFC 9111 §3.5 restricts shared-cache reuse only for requests carrying an `Authorization` header, §5.2.2.7's `private` directive is never invoked by the Guide, and §4.1 keys `Vary` selection only on nominated request header fields. Two concrete consequences follow. First, if the ETag is computed over stored resource state rather than the emitted authorized representation, a caller whose view conceals a field can detect concealed changes by polling, because the visible body is unchanged while the validator changes — the same leak class the Guide already commits to closing at lines 469, 803 and 975. Second, in the trusted-reverse-proxy deployments the Guide supports (lines 364, 596), where a caller's authorization derives from a client certificate or a proxy-injected identity header rather than an `Authorization` header, an intermediary may serve one caller's authorized representation to another context. Evidence: [22-pass-3c-17-research-deep-read-slice2.md](evidence/22-pass-3c-17-research-deep-read-slice2.md), Section 5.

**Corroboration and a second consequence (iteration 18, IDR-042).** The batch that read IDR-SRV-042's remainder was directed to re-test this gap against it. The gap is reinforced, not qualified. IDR-042 §7 closes its resource-family table with the rule the Guide does not state: "'Cache-safe' means policy-partitioned, integrity/version checked and accompanied by correct validators; it does not mean publicly cacheable. Protected or principal-specific responses use appropriate private/no-store controls" (line 341, tagged **[N/A/P]**), and its §3.1 lists RFC 9111 §§4.2-4.2.4 and 5.1-5.2 as a normative source (line 136). Two accepted reports therefore state the requirement, partly on a normative-derived rather than purely project-chosen basis. IDR-042 also adds a second consequence the earlier statement did not cover: HTTP cache freshness must not be read as domain freshness (§4.2 line 197; §6.4 line 313, "`ageSeconds` is domain-evidence age under the named rule, not HTTP `Age`"), with two ready-made API fixtures at §15.1 lines 577-578 ("HTTP cache fresh/domain stale → `Age` and domain assessment remain independent"; "HTTP cache stale/domain fresh → cache revalidation behavior does not rewrite domain time"). The Guide establishes the underlying domain distinction thoroughly (lines 147, 481, 970) but states no HTTP-layer caching rules at all, so this is the same single omission showing a second consequence rather than a separate defect. Evidence: [23-pass-3c-18-research-deep-read-slice3.md](evidence/23-pass-3c-18-research-deep-read-slice3.md), Section 2.

**Third corroborating source (iteration 21, IDR-034).** IDR-SRV-034 Section 13.4 closes with "Principal-specific results use partitioned/private caches." Evidence: [26-pass-3c-21-idr030-completion-and-idr034.md](evidence/26-pass-3c-21-idr030-completion-and-idr034.md), Section 3.3.

**Fifth source, from the test side (iteration 23, IDR-055).** IDR-SRV-055 is the first source to express the rule as a *testable leakage surface* rather than an architectural obligation. Its Section 9 leakage inventory names "status, headers, response length, cache validators and timing"; its Section 8 requires that "Cursors and validators bind the authorized view, policy version, principal/client and query/filter"; its required-matrix row `SEC-POLICY-VIEW` gives the negative case as "hidden row changes count/page/latest/ETag"; and its implementation proof 4 requires twin-world tests to prove the public projection "including count, page, extent, latest, cursor and ETag" does not change with hidden facts. That is precisely the test formulation this instance's remedy already asks to add to the Guide's disclosure-channel list at line 1019. Because IDR-055 is not an accepted report, this is recorded as corroboration of an instance already resting on four accepted reports, not as independent authority. Substance and remedy unchanged. Evidence: [28-pass-3c-23-idr055.md](evidence/28-pass-3c-23-idr055.md), Section 4.2.

**Fourth source, and the first naming ETags (iteration 22, IDR-039A).** IDR-SRV-039A states the rule twice and is the first source to name the validator explicitly. Section 11.3: "Counts, extents, ordering, pagination, cursor validity, ETags, `Location`, alternate representations, response timing classes and error existence behavior describe only the authorized view. Cache keys include that view and relevant policy/evidence versions." Its Section 9.5 enforcement-point registry lists a "Cache/cursor/ETag PEP" whose required behaviour is to "bind to authorized-view and policy/data/security versions." Iteration 17 had derived the validator half of this instance from RFC 9111 mechanics rather than from any research statement, and noted that the Guide's "representation-specific" wording at lines 503 and 739 left it ambiguous whether an authorized view counts as a distinct representation. That ambiguity is now removed on the research side. The instance rests on four accepted reports: IDR-040 Section 10.4, IDR-042 Section 7 line 341, IDR-034 Section 13.4 and IDR-039A Sections 9.5 and 11.3. Its substance and recommended fix are unchanged. Evidence: [27-pass-3c-22-idr039a.md](evidence/27-pass-3c-22-idr039a.md), Section 3.3.

**Next consideration for that instance:** State in Section 4.10 or Section 6 that validators and any cache key are bound to the authorized view, that authorization-dependent representations carry `Cache-Control: private` (or `no-store`), and that HTTP cache freshness is not domain freshness. Add the cache/validator channel to the line 1019 disclosure-channel test list, and the two IDR-042 §15.1 cache-versus-domain-freshness fixtures to the Section 8 checks. All are documentation and test changes within existing owners; no new capability, policy engine or caching layer is implied, and the choice between `private` and `no-store` remains the owner's.

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

**Status:** Source-example hazard confirmed against the published Standard; inbound unknown-member policy is a project choice to record (documentation gap), not a standards obligation. **Reported severity:** Low.
**Affected work:** 3.2.1 (#98), 3.2.2 (#99), 3.2.4-3.2.5 (#101-#102); Guide Section 4.6 (one sentence) and Section 13 (one row).

An extra foi@id member can be schema-permitted without establishing the standard samplingFeature@id association. The example's data URL is a separate issue. Rejecting every unfamiliar @id/@link member was not approved.

**Resolution of the controlling extension-rule check (iteration 15):** No published text imposes an extension-member rule for Observation JSON. OGC 23-002 Requirement 97 requires validity against `observation.json`, and both the tagged and published copies of that schema carry no `additionalProperties` or `unevaluatedProperties` keyword, so extra members are permitted by schema silence; Requirement 98 and tests A.66/A.67/A.81/A.82/A.97/A.98 never examine them, and no processing model (preserve, ignore, reject) is defined. The `@id`/`@link` association names are a convention of the schema member names and the Part 1 mapping tables, reserved by no requirement or schema rule. RFC 7946 Section 6.1 (foreign members MAY be used; "no normative processing model for foreign members is defined") governs GeoJSON documents only, and 23-002 Clause 7.2 places observations outside GeoJSON; 23-000 Requirements 9/10 govern the SensorML `extension` slot only. The published Clause 16.1.5 example's `foi@id` is confirmed and sharpened: Table 7 gives the Observation only `datastream`, `samplingFeature` and `procedure` associations, so the member maps to no association at all and a client copying the example loses the sampling-feature link and every `foi` filter match. Because no published rule requires rejection, none is prescribed; the earlier suggestion to reject or warn on unmapped `@id`/`@link` members is an owner option that would be a labelled Glaux restriction. The precise gap: Guide lines 406/432 (preserve permitted extensions; validated extensible payload), 497 (PATCH rejects changes outside the writable projection) and 713 (only advertised extension fields appear publicly) leave the outcome for a member that is neither mapped nor an advertised extension unstated for POST, PUT and PATCH. Evidence: [20-pass-3c-15-observation-extension-policy.md](evidence/20-pass-3c-15-observation-extension-policy.md), Sections 2-4.

**Next consideration:** Record in Section 4.6 the outcome for members that are neither mapped nor advertised extensions, identically for POST, PUT and PATCH (preserve-as-opaque, ignore, or reject with a problem detail naming the member; any of the three is conformant), and add a Section 13 row for the Clause 16.1.5 `foi@id` example. Keep the isolation fixtures (baseline `samplingFeature@id`; `foi@id` variant; the `data:` result URL tested separately; `foi@id` never emitted) with #99's supplied sampling-feature context checks. Reporting the example upstream remains the owner's decision.

**Saved evidence:** [06-pass-3c-02.txt](evidence/06-pass-3c-02.txt), starting at line 47. Consult later qualifications below where applicable.

### F-14 - Three interpretation details

**Status:** Qualified documentation/mapping candidates. **Reported severity:** Low.
**Affected work:** #71/#72; Deployment writes; observation listing.

(a) Root-parent semantics and known-hit tests already exist in #71/#72. (b) The reported local Deployment write mapping requires a System UID URI; canonical-URL compatibility and an additional returned uid are separate choices. (c) Research selected ascending resultTime/ID order; Guide wording should not obscure direction.

**Next consideration:** Do not rewrite existing parent tests as if absent. Preserve required UID handling separately from optional conveniences; state ordering and its client consequences.

**Confirmed instance for part (c) (iteration 21, IDR-034).** The ordering concern is now evidenced on both sides. IDR-SRV-034 states the direction twice and unambiguously: "Accepted default Observation order is `(normalized resultTime, ResourceId)` ascending" (Section 10.3) and "Default order is `(resultTime, ResourceId)` ascending" (Section 12.2). Guide line 444 names the same two fields and omits the direction: "Use deterministic ordering with a unique ID tie-breaker: result time then ID for observations; stable ID order for ordinary resource lists unless a family requires a different rule." A search of the whole Guide finds no occurrence of `ascending` or `descending`. The Guide therefore fixes the sort key and the tie-breaker while leaving the direction unstated, so two clients implementing it could page the same stream in opposite orders and both claim conformance. The fix is one word at line 444. Evidence: [26-pass-3c-21-idr030-completion-and-idr034.md](evidence/26-pass-3c-21-idr030-completion-and-idr034.md), Section 3.2.

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

**Confirmed instance (iteration 19, IDR-043).** This finding previously rested on a reported pattern; IDR-SRV-043 now supplies a verified example with exact locations. Its header records **Report Status: Final**, **Accepted By: Glaux Project Lead** and **Acceptance Date: September 15, 2026** (lines 4 and 17-18). Section 21 nevertheless closes with "The deliverable should remain **In Review** until project-lead acceptance" (line 704), and the completion checklist's final item still reads "Category H and implementation remain unauthorized pending acceptance" (line 765). The dated acceptance record is the controlling fact and the two trailing statements are pre-acceptance residue the acceptance did not sweep. This is editorial cleanup for the owning report, not a technical defect and not a reason to treat IDR-043 as unaccepted. Evidence: [24-pass-3c-19-research-slice4-and-batch-queue.md](evidence/24-pass-3c-19-research-slice4-and-batch-queue.md), Section 3.

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

**Related instance (iteration 20, restore versus deletion).** IDR-SRV-030 requires that a restored older point receive the later deletion and tombstone state before it can serve data (recommendation 11, Section 13.1), and rejects restoring a backup directly into service because it "Can resurrect deleted/unauthorized state" (Section 18.2). Guide Section 4.7 treats restore carefully and enumerates several things it does **not** re-establish: a rollback or fork forces a fresh recovery epoch and pre-restore cursors fail with `410` (lines 531, 547), restored pending work stays held, command reconciliation covers commands accepted after the backup, and same-key deduplication is explicitly not promised across a lost recovery interval (line 531). Deletion is absent from that list. Restoring to a point before a deletion returns the resource to the authoritative store, and because tombstones live in that same store they roll back with it, while reads are the part the Guide re-enables first (line 529). A peer that already received the deletion is protected, because an import that would resurrect a deleted resource is retained and reported rather than applied (line 647), so the exposure is local reads and any re-export from the restored node. Evidence: [25-pass-3c-20-idr030-and-queue-corrections.md](evidence/25-pass-3c-20-idr030-and-queue-corrections.md), Section 5.3.

**Sharpened (iteration 21, after finishing the IDR-030 sections iteration 20 had only skimmed).** The earlier statement hedged that this is partly inherent to point-in-time restore because there may be nothing to reapply. IDR-030 Section 12 shows the research does not stop at naming the hazard. The archive manifest is specified to carry "tombstone/deletion-ledger bounds" so a gap is detectable (Section 12.1); the restore contract is an explicit sequence whose step 2 restores "into an isolated, non-serving namespace" and whose step 6 reads "Apply the current tombstone/deletion ledger and every later disposition event before publication. Never expose restored pre-deletion state", with a "deletion-ledger gap" named as a terminal failed or review state (Section 12.2); and Section 6.2 lists the tombstone ledger among the restore gates. The Guide adopts none of that archive machinery, so the manifest mechanism is not available to it, but the ordering principle of restoring into isolation and reconciling deletions before serving is available and cheap to state. Section 16 supplies two ready-made fixtures: "Valid restore | Restore prior backup containing later-deleted record | Current deletion ledger reapplied before serving; deleted resource does not reappear" and "PITR replay | Restore point precedes deletion | Post-restore deletion/tombstone replay removes it before access". Evidence: [26-pass-3c-21-idr030-completion-and-idr034.md](evidence/26-pass-3c-21-idr030-completion-and-idr034.md), Sections 2.1-2.2.

**Next consideration for that instance:** State in Section 4.7 what a restore does to deletions recorded after the backup point, and whether authorized reads are gated until that is reconciled or the limitation is simply documented as the other restore limitations are. Add the corresponding check to the Section 8 restore verification, which already exercises tombstones during restore testing. Documentation and test changes within existing owners; no new capability is implied.

**Related instance (iteration 16, IDR-038 audit invariants):** IDR-SRV-038 Appendix B.2 invariant 10 ("No correction, reconciliation, retention, redaction, or export rewrites original audit evidence") and Appendix A's audit-search/export/redaction row ("derived redaction never alters original") sharpen this finding's open question into a specific candidate rule. Guide v1.3 line 602 states a durable-record floor (verified actor/source, operation, target/revision, time, outcome, correlation identifier) and explicitly disclaims one specific architecture ("A mandatory tamper-proof ledger ... is not proposed") without stating whether correction, redaction or export may alter the original evidence versus only a derived view. The candidate rule is not itself adopted by that floor. No Guide correction; no new finding. Evidence: [21-pass-3c-16-research-deep-read-slice1.md](evidence/21-pass-3c-16-research-deep-read-slice1.md), Section 5.

**Saved evidence:** [15-pass-3c-11.txt](evidence/15-pass-3c-11.txt), starting at line 55. Consult later qualifications below where applicable.

### F-20 - Denial-audit deliverable

**Status:** Clarification/test candidate. **Reported severity:** Low.
**Affected work:** #15 / #22.

The Guide requires relevant denials to be recorded safely; the reviewed issue text allows safe denial auditing but does not explicitly test selected record creation/failure behavior. Existing owners are present. Proposed event categories are suggestions.

**Next consideration:** Define selected categories, bounds and failure behavior if adopted. Audit failure must never authorize the denied action; no independent spool follows automatically.

**Related instance (iteration 16, IDR-038 audit event catalog):** IDR-SRV-038 Appendix B.1 names a 37-item minimum audit-event catalog (`request_received` through `audit_integrity_incident`). Guide v1.3 line 602 states only a field-level floor (verified actor/source, operation, target/revision, time, outcome, correlation identifier), not a named-event taxonomy. The catalog is a concrete candidate for the "selected categories" this finding already asks the owner to define; it is not itself an adopted requirement. No Guide correction; no new finding. Evidence: [21-pass-3c-16-research-deep-read-slice1.md](evidence/21-pass-3c-16-research-deep-read-slice1.md), Section 5.

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

Material follow-up questions raised by the same reads (encodings.json root `oneOf` omitting `BinaryEncoding`; `Quantity.json` requiring `label`; the CQL2 Clause 7.6.1 GeometryCollection member list omitting LineString; the 23-000 Clause 9.1.2.1 draft media-type note; whether the Glaux ATS overlay absorbs the four additional published seams; whether Glaux exposes any Part 2 resource collections; the wording of the F-08 Section 13 row for both Parts; whether one unknown-member rule should cover ordinary-JSON Part 2 resources, GeoJSON `properties` foreign members and the SensorML `extension` slot together; whether Guide Section 4.12 should state metrics/traces exposure is internal/access-restricted by default; which CS-Go repository URL is the correct pinned source; whether the approved Part 2 baseline actually prohibits a future `resultTime`) are recorded in `follow_up_questions` of the state file. They are not review obligations unless a later batch selects them.

The F-13 controlling extension-rule check (`observation-extension-policy`, iteration 15) was also closed in the state file; because it re-dispositions an existing finding rather than producing none, its record is the F-13 entry above and [20-pass-3c-15-observation-extension-policy.md](evidence/20-pass-3c-15-observation-extension-policy.md). No new finding number was created.

**Research deep-read remainders, slice 1 (iteration 16):** IDR-008 Sections 16-19, IDR-037 Appendices A-D, IDR-038 Appendices A-C, and IDR-039 Section 7 and Sections 21-22 were read in full and accounted against Guide v1.3. No Guide correction and no new finding number resulted; three related instances were recorded above (F-03, F-19, F-20) and one new follow-up question was recorded (`fq-09`, metrics/traces exposure). All four reports move from partial to fully read. Evidence: [21-pass-3c-16-research-deep-read-slice1.md](evidence/21-pass-3c-16-research-deep-read-slice1.md).

**Research deep-read remainders, slice 2 (iteration 17):** IDR-040 Sections 9-21 were read in full and accounted against Guide v1.3, with RFC 9111 Sections 3.5, 4.1 and 5.2.2.7 retrieved to bound the one gap statement. The report's invariants are substantially adopted in the Guide's own vocabulary; its typed policy machinery (`PolicyAssertion`/`PolicyBinding`/`DisclosureDecision`/`DerivedViewProvenance`, `INDETERMINATE` three-state decisions, transform registry, bilateral federation mappings, per-view OpenAPI derivatives) is not adopted, recorded as a scope choice consistent with this register's F-03 disposition and with the report's own Section 18.2 deferrals. One evidenced gap was found and recorded as the F-03 related instance above: no Guide rule binds a response cache validator or cache key to the authorized view. No new finding number. One new follow-up question was recorded (`fq-10`, the CS-Go repository URL discrepancy, deliberately unverified). IDR-040 moves from partial to fully read. Evidence: [22-pass-3c-17-research-deep-read-slice2.md](evidence/22-pass-3c-17-research-deep-read-slice2.md).

**Research deep-read remainders, slice 3 (iteration 18):** IDR-042's unread sections (3-4, 6, 7, 9, 10, 15, 16, 19-20) were read in full and accounted against Guide v1.3. Its non-collapse rules, freshness and last-known vocabulary, `resultTime=latest` semantics, delayed and replayed input handling, opaque policy-bound cursors, MQTT-session-is-not-a-cursor rule and reconnect/resnapshot behavior are substantially adopted, several passages near-verbatim. `RepresentationAssessmentV1`, the Glaux-specific time fields, the DDIL mode taxonomy and numeric thresholds are not adopted, a recorded scope choice consistent with the report's own Section 3.2 and Section 19. The planned re-test of the F-03 response-cache gap against this report found it **reinforced**: a second accepted report states the missing rule and adds a cache-versus-domain-freshness consequence with two fixtures, recorded in the F-03 instance above. No new finding number and no new follow-up question. IDR-042 moves from partial to fully read. Evidence: [23-pass-3c-18-research-deep-read-slice3.md](evidence/23-pass-3c-18-research-deep-read-slice3.md).

**Research deep-read remainders, slice 4 (iteration 19):** IDR-043's unread sections (Sections 5-14, 16-18, 21-22 and the checklist) were read in full and accounted against Guide v1.3. Its non-bypass invariant, the prohibition on a generic synchronization route, identity and revision rules, tombstone and anti-resurrection handling, inbox idempotency, the replay classification table, the prohibited universal conflict resolvers, staged-conflict isolation with an explicit resolution allowlist, federation partner-profile requirements, command non-redispatch and the observability rules are substantially adopted, several near-verbatim; Guide line 633 cites the report directly. Its `SyncEnvelopeV1` field taxonomy, five orthogonal state machines, `SyncConflictV1`, quarantine subsystem, operator review workflow and versioned federation profiles are not adopted, a recorded scope choice stated at Guide line 620. Its Section 16.1 `428` row belongs to the already-recorded no-mandatory-precondition choice. One finding instance was confirmed under F-17 above. No new finding number and no new follow-up question. IDR-043 moves from partial to fully read. Evidence: [24-pass-3c-19-research-slice4-and-batch-queue.md](evidence/24-pass-3c-19-research-slice4-and-batch-queue.md).

**Committed deep read IDR-030 (iteration 20, queue batch 2):** read in full and accounted against Guide v1.3. Its central recommendations are adopted at Guide line 527, which forbids automatic retention or purge by default and permits removal only under an explicitly configured policy that preserves visible deletion behaviour and dependencies among values, schemas, source documents, pending work and synchronization tombstones. Also adopted: the cascade and `409` rules with retained internal evidence, no reuse of a deleted identifier, cancellation as a status rather than a deletion, idempotency-key retention that outlives replay horizons, rebuildable derived state, and the prohibition on fabricating history removed by retention. Its archive tier, hold overlay, policy registry, disposition receipts, purge automation, cryptographic erase, media sanitization, five orthogonal lifecycle state dimensions and aggregation rules are not adopted, a recorded scope choice the report supports by assigning those to deployment, security and profile authorities and by supplying no numeric horizon of its own. One documentation gap was found and recorded as a related instance under F-19 above. Evidence: [25-pass-3c-20-idr030-and-queue-corrections.md](evidence/25-pass-3c-20-idr030-and-queue-corrections.md).

**Committed deep read IDR-034 (iteration 21, queue batch 3):** all nineteen sections plus the checklist read in full and accounted against Guide v1.3. Substantially adopted across latest-value selection and ties, schema evolution with successor streams, `live` semantics, the optional/nil/uncertain/withheld distinctions, authorization ordering before every derivation, the PostgreSQL-first storage posture with TimescaleDB conditional, publication after commit, and System Event separation. It confirmed the F-14(c) instance above, supplied a third source for the F-03 cache instance, and its request for snapshot-stable paging is a departure the Guide states for itself at lines 444, 471 and 473, pointing to the explicit snapshot and export mechanism instead. One incidental uncertainty was recorded as `fq-11` rather than asserted: the report says a future `resultTime` is prohibited and the Guide has no such rule, but its own footnote attributes the cited requirements to parent schemas and UTC time rather than to futurity, so the obligation was not verified. Evidence: [26-pass-3c-21-idr030-completion-and-idr034.md](evidence/26-pass-3c-21-idr030-completion-and-idr034.md).

**Committed deep read IDR-039A (iteration 22, queue batch 4):** all twenty-five sections plus the evidence legend read in full. The cross-check the queue required is confirmed: IDR-040 Section 14.1 and IDR-042 Section 14.1 both cite this report for signed versioned anti-rollback policy and trust bundles, a bounded local authorization envelope, disconnection never expanding authority, and an indeterminate result on absent or stale mandatory inputs, and every one of those elements exists in the source saying what the citing reports say, so no correction is needed in any of the three. The Guide contains none of the zero-trust vocabulary and cites the report once at line 592; that absence is compliance with the report's own rule that no enterprise zero-trust, maturity or accreditation claim may be made without a competent external authority, not an omission. The enforcement substance is substantially adopted across authorization ordering, authorized-view artifacts, fail-closed behaviour, anonymous handling, proxy trust, streaming, commands, source separation and telemetry redaction. Its named logical architecture is not adopted, a recorded scope choice in this register's F-03 family that the report itself supports through its own deferrals. No new finding and no new follow-up question. Evidence: [27-pass-3c-22-idr039a.md](evidence/27-pass-3c-22-idr039a.md).

**Committed deep read IDR-055 (iteration 23, queue batch 5):** all nineteen sections plus the legend, contents, header and closing block read in full. **This report is not accepted.** Its header carries no acceptance field, unlike the other three deep reads, and its closing block states that it awaits project-lead review; "Final" here means drafting complete. That is not an instance of F-17, because the pre-acceptance wording is correct for an unaccepted report, and the Guide correctly never cites it. The handoff from IDR-039A is consistent on all four handed-over elements, with the required fourteen-column matrix delivered as a test-plan matrix in its own columns rather than a copy of the enforcement columns. The security-test substance is substantially adopted, including the paired-world non-interference method which the Guide already holds at line 975 and the Guide's own security-test family list at line 612. Its tool portfolio, execution tiers, evidence schema, twelve proofs and simulator harness are not adopted and are correctly not recorded as gaps: the report is unaccepted, the standing instruction forbids adding a tool portfolio because an optional mechanism is absent, and F-02 already owns the enforced-CI question at the right altitude. Evidence: [28-pass-3c-23-idr055.md](evidence/28-pass-3c-23-idr055.md).

**All four committed deep reads are complete.** The research deep-read check is closed. The research area continues only through the key-section sweep, queued as batches 6-14, which screens the reports whose read depth was never recorded together with the unread sections of the six partial reports.

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

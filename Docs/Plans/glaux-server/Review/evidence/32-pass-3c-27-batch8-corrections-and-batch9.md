# Pass 3c iteration 27 - two batch-8 corrections, then queue batch 9

**Date:** September 20, 2026
**Provider:** Claude Code
**Model:** Claude Opus 5 (model ID `claude-opus-5`), per the active-model identification supplied by the environment at the time of this iteration
**Authorization:** One user `proceed` naming two bounded batch-8 corrections followed by batch 9 as already queued. No new batch, no repeat source study, no implementation change.
**Comparison target:** `glaux-server-implementation-guide.md` v1.3, 1242 lines.

---

## 1. Correction A - the withdrawn `latest` corroboration claim

### 1.1 What was claimed

Iteration 26, [31-pass-3c-26-batch8-peer-studies-and-source-checks.md](31-pass-3c-26-batch8-peer-studies-and-source-checks.md) Section 2.4, wrote that the Connected Systems Go latest-observation fixture "directly corroborates, in the pinned source, two accepted research claims made from the outside": IDR-014B Section 17.2 open question 6, which asks whether that peer's "broad `latest`" is an extension or a defect, and IDR-034 Section 14.4, which records that the peer's "broad `latest`, query names, cursors, and incomplete SWE support are not standards behavior."

### 1.2 Why it is withdrawn

The fixture seeds a single newest observation. Section 2.4 states this itself: "The fixture happens to seed only one newest observation, so the assertion passes either way." A fixture whose newest record is unique never exercises tied result times. It therefore cannot show how the peer behaves when two observations share the greatest result time, which is the behavior IDR-034 Section 10.1 says `resultTime=latest` must preserve, and it cannot corroborate the broader criticism of that peer's selector, because nothing in it distinguishes a correct implementation from an incorrect one.

The assertion `require.Equal(t, 1, len(items), "expected exactly one observation for ?resultTime=latest")` records the test author's expectation. An expectation written into a test that cannot fail on the point at issue is evidence about the author, not about the running server. The distinction matters because the two research claims are claims about behavior.

This is a reviewer error of the same kind the review has corrected before: reading a source as settling a question it was not constructed to settle. The corrected statement is narrow. The fixture shows an assumption in the peer's test code. It does not show the peer's tied-timestamp behavior, and it adds nothing to the two research claims, which stand or fall on their own evidence.

### 1.3 What is retained

Two Section 2.4 results are independently supported and are kept unchanged.

1. **The time-range test observation.** `TestObservation_List_ValidResultTimeRange_Filters` at lines 506-539 of the pinned file seeds a 2025 and a 2027 observation, queries the 2025 window, and ends with `assert.Equal(t, 1, len(items), "expected only the 2025 observation to be returned")`. It asserts the count and never asserts which observation was returned, so it would pass if the server returned the 2027 observation instead. That needs no assumption about ties, is visible in the pinned source, and is exactly the practice Guide line 446 forbids by requiring seeded records that are distinguishable and assertions on exact identities.
2. **The Guide line 1129 verification** in Section 2.3, including both of its specific claims and its own qualification that it reports source analysis rather than an observed defect. That verification never rested on the withdrawn inference.

### 1.4 What does not change

Guide line 440 remains the correct rule and was never in question: `resultTime=latest` applies predicates first, takes the greatest visible result time, and retains ties. No finding was raised on the withdrawn basis, so no finding changes. The pin verifications, the `fq-10` resolution and the closure of `peer-source-spot-checks` are independent of the withdrawn paragraph and stand.

### 1.5 How the correction is recorded

Evidence report 31 is preserved. Its authored body is byte-for-byte unchanged, verified by hashing its first 15,830 bytes to the manifest's recorded `original_sha256`. The correction is added as a clearly demarcated Appendix A after the end of the authored text, so that a reader consulting report 31 alone discovers it. The manifest keeps `original_sha256` as the record of the authored bytes and updates `published_sha256` to the new full-file hash, with `publication_change` set to `correction_appended_body_unchanged`.

That choice follows the review's own F-17 reasoning. A document whose status has changed but which says nothing about it misleads a reader who has only that document, and this review was itself misled that way in iteration 23. Appending the notice to the archived report, rather than only to its successor, is the remedy F-17 asks the project to apply.

---

## 2. Correction B - the omitted batch-8 screening, now complete

### 2.1 What was omitted

Iteration 26 screened the peer and client studies at their recommendation sections and at the first subsection of each risks-and-open-questions section, and recorded the rest of those sections as not read. The omitted subsections are the Open Questions: IDR-014C Section 17.2; IDR-014D Sections 17.2 and 17.3; IDR-014E and IDR-014F Sections 15.2 and 15.3; IDR-014G Sections 14.2 and 14.3. All 102 lines have now been read. This is a screen of recorded questions, not a fresh investigation of each question.

### 2.2 The governing result

The open questions in these studies are overwhelmingly about the peer projects and about the studies' own method, not about Glaux obligations. IDR-014D Section 17.3 is the clearest case: all ten of its remaining questions ask what the SECD prototype intends, from "Where is the intended open-source server repository, and under what license?" to "What deployment/security posture is intended beyond the public prototype?" None of these can create a Glaux obligation, and none is a Guide gap.

The resolved-question subsections close against the accepted baseline rather than leaving anything open. IDR-014E Section 15.3 states "No open question blocks this report," and IDR-014F Section 15.3 states "No open question blocks the report. Later design topics must choose exact models and policies within the accepted normative baseline." IDR-014G Section 14.2 records that existing issue #186 and the upstream-history entries already cover the current OpenAPI and Part 3 questions, and that "No new escalation blocks Glaux."

Nothing in the 102 lines changes any accounting from iteration 26, and none produces a new finding or a new follow-up question. Four items are worth recording.

### 2.3 Items worth recording

**IDR-014G Section 14.3, line 474, supports F-11.** The question is: "Will Glaux provide any explicitly noncanonical compatibility adapter, and under what namespace/profile?" That is the F-11 question asked from the peer-study side. F-11 concerns the Guide's silence on relation spelling combined with the three different relation forms it emits, one of which is a bare name that the accepted interpretation confines to a compatibility adapter. This study records the same two unknowns, the adapter and the namespace, as open for Glaux. It is support for F-11, not a separate finding.

**IDR-014E Section 15.2 and IDR-014F Section 15.2 give F-12 its first peer-study support.** IDR-014E line 514 pairs "Parsed tools create false findings" with the control "Raw transport capture is primary oracle." IDR-014F line 502 pairs "Server blamed for client/harness error" with "Preserve raw wire, parser output, code expectation, and standard anchor." F-12's residual concern is using production wire types to interpret responses that are supposed to be independent. These two rows state the same boundary from two independent studies and name the artifacts that keep it: the raw wire on one side, the parser output and the code expectation on the other, with the standard as the anchor. F-12 stays a narrowed optional hardening at Low severity; this is material for whoever writes the clarification it asks for.

**Two IDR-014E Section 15.2 rows are already adopted, one near-verbatim.** Line 511 pairs "Client assumptions become server requirements" with "Normative baseline controls; ownership column mandatory." Guide line 991, which cites this study family directly at `[IDR-014E-014G][R014e]`, states "A client workaround is evidence of interoperability behavior, not permission to change the standard contract." That is the same rule in the Guide's own words. Line 516 pairs "Public-server drift destabilizes CI" with "Self-hosted pinned gates; live canaries non-blocking," and Guide line 944 requires fixtures that "do not depend on live operational information or external servers for routine tests."

**IDR-014C Section 17.2 question 5 is partly addressed and its remainder is not asserted as a gap.** The question asks "What is the intended canonical source when SensorML and GeoJSON representations disagree?" Guide line 991 requires regression tests for "different identities across formats," which covers detection. The Guide states no rule naming which representation is canonical when the two disagree. That is recorded as an observation and not raised, because the question is asked about the pygeoapi prototype rather than about Glaux, and because establishing whether the Guide needs such a rule would require the published Part 1 encoding text, which a key-section screen does not read.

### 2.4 Coverage now recorded

Each of the five reports moves from a partial subsection list to the full risks-and-open-questions section read. None becomes a full read. Their bodies, validation sections and appendices remain unread, as recorded.

---

## 3. Batch 9 - key-section screen D: resource model, temporal validity and status

### 3.1 What was screened

Six reports, all screened at their decision-usable sections. None is recorded as fully read.

| Report | Screened | Not read |
|---|---|---|
| IDR-015 canonical resource model | Section 16 Recommendations (12 items), Section 16.1, Section 17 Risks, Constraints and Open Questions | Sections 1-15, 18-21 including all appendices |
| IDR-016 identifier, URI and lifecycle | Section 16 Recommendations table (R-016-01 to R-016-14 at lines 773-786) | Sections 1-15, 17-21 |
| IDR-017 relationship and linkage model | Section 16 Recommendations table, Section 9.3 CSAPI `ogc-rel:` vocabulary (lines 460-470), relationship register rows 227 and 346. Section 9.1 was **reused** from `evidence/12-pass-3c-08.txt`, not re-read | Sections 1-8, 10-15, 17-21 |
| IDR-018 temporal, validity and freshness | Section 15 Recommendations table, Section 15.1 rejected simplifications, property tables at lines 246-248 and 590-591, lines 373 and 442 | Sections 1-14, 16-21 |
| IDR-019 provenance, lineage, quality and trust | Section 17 in full: 17.1 Recommendations (20 items), 17.2 Rejected Options (13 rows), 17.3 Open Questions Routed Downstream, 17.4 Review Triggers; Section 18 validation table | Sections 1-16, 19-21 |
| IDR-020 status, availability and system events | Section 16 Recommendations table | Sections 1-15, 17-21 |

### 3.2 Acceptance status, checked in the governance record

Following the iteration 24 correction, acceptance was checked in `final-idr-research-report.md` rather than inferred from report metadata. Lines 153 to 158 record all six as Complete and Accepted. Each report's own header also records `Accepted By: Glaux Project Lead` with an acceptance date: September 3, 2026 for IDR-015 and September 13, 2026 for the other five.

**No new F-17 instance.** All six record their acceptance correctly in the place a reader looks first. This is a useful negative result: it confirms F-17 describes a residue in particular reports rather than a systemic practice, consistent with iteration 25's finding that four reports in a different cluster also record acceptance correctly.

### 3.3 Adoption accounting

The recommendations are substantially adopted, several near-verbatim. The clearest matches:

| Recommendation | Guide | Disposition |
|---|---|---|
| R-016-09 "never reuse IDs" | 509 "Do not reuse a deleted ID." | Adopted, near-verbatim |
| R-016-11 "Treat Command cancellation as a status transition, not DELETE" | 582 "Cancellation follows the command-status behavior, not HTTP DELETE." and 509 "Deleting a Command is not a request to cancel device execution." | Adopted, near-verbatim, in two places |
| R-016-10 "Require a new stream ID for incompatible populated schema replacement; retain old schema/context for children" | 408 "use a new stream when an incompatible contract cannot be changed legally" and 434 rejecting schema-modifying PUT/PATCH with `409` while children exist | Adopted |
| R-016-04 "Build every public URL/link/Location/OAD path through one typed canonical route and public-origin registry" | 364 absolute links from a configured public API root, forwarded origin honored only from configured proxies | Adopted |
| R-016-05 "Preserve IDs/URLs through updates, encodings, memberships, deployments, and releases" | 364 "Keep canonical URLs stable across routine software releases." | Adopted |
| R-016-07 "prohibit automatic mutation/tasking redirects until safety/idempotency proof" | Section 13 row on unsafe-method aliases | Adopted |
| R-020-01 "never use a universal status/health/readiness scalar" | 483 "Do not invent a universal readiness score or a new mandatory status API." | Adopted, near-verbatim |
| R-020-07 System Events only for genuine operational occurrences, "not every CRUD, data, command, audit, service or transport event" | 485 "They are not aliases for HTTP access logs, resource-update notifications, or every observation arrival." | Adopted, near-verbatim |
| R-020-06 preserve last-known values, report staleness honestly, use `unknown` when failure cannot be established | 481 selects current status by meaningful time and preserves delayed samples as history | Adopted |
| R-020-09 resolve the System Event prose/schema mismatch in an explicit version-pinned adapter with golden tests | 485 and the Section 13 System Event JSON row | Adopted |
| R-015-12 "Keep published conflicts in an interpretation registry" | Section 13, the source-contradictions table | Adopted; Section 13 is that registry |
| R-018-05 apply the IDR-011 field mapping, inclusive intersection, deterministic ordering and `latest` operator order exactly | 440 and 444 | Adopted |
| R-018-09 keep issue, execution, report, receipt, commit and publication time distinct | 376, 442-444 and the Section 6.1 persistence table | Adopted |
| R-018-12 "Never use HTTP `Age`, `Expires`, or Cache-Control as domain freshness" | No occurrence of `Age` or `Expires` as domain metadata anywhere in the Guide | Satisfied by silence; recorded as satisfied, not as an adoption |
| R-018-13 and R-017-13 authorize before `latest`, aggregates, counts, current projection and freshness metadata; R-017-13 extends it to "link, count, page, reverse, cache, error, or timing disclosure" | 469 authorized view before predicates, 610 authorize provenance relationships independently with no leak "through links, schemas, query NULL behavior, errors, exports or event topics" | Adopted, and Guide 610 enumerates the same channels |
| R-017-14 external-link resolution asynchronously through SSRF-safe governed workers | 420 schema allowlist with no arbitrary `$ref`, link or data-URL fetch | Adopted in substance |
| R-019-11 "prohibit a canonical scalar trust score" | 602 "A mandatory tamper-proof ledger, enterprise security platform, or universal trust score is not proposed." | Adopted, near-verbatim |
| R-019-09 and R-019-10 scoped attributed quality assertions; keep measurement uncertainty separate from evaluator confidence | 608 preserves subject, metric, units, method, evaluator and coverage and forbids normalizing them "into a generic truth probability" | Adopted, near-verbatim |
| R-019-14 redaction and release as accountable derivations with inference-resistant projections | 600 prefers authorizing complete conformant resources over field redaction and requires a valid projection or denial | Adopted in substance |
| R-019-16 keep provenance and audit distinct but correlated | 107-109 store production context separately from the audit record of who uploaded the data; 602 the audit floor | Adopted |
| R-019-05 never erase raw-source evidence during normalization; R-019-06 digest plus immutable durable reference | 406 store exact original bytes with media type and digest; 517 preserve exact bounded document bytes in `bytea` because JSONB does not preserve a source document byte-for-byte | Adopted |

**One recommendation is departed from in form, and the Guide states the departure for itself.** R-018-03 requires one evaluation time and a snapshot or watermark per request, bound to cursors. Guide line 444 does the opposite for ordinary lists: "document keyset paging as a changing view rather than claiming a cross-request snapshot." Line 471 binds cursors to the normalized filter, endpoint scope and queryable-mapping version and reauthorizes every continuation, so the binding half of R-018-03 is adopted while the snapshot half is not. This is a stated design choice in the approved Guide, not an unnoticed omission, and it is recorded here as a non-adoption rather than a defect.

**R-017-09 is half adopted.** It asks for "a versioned HTTPS Glaux relation namespace for Part 2 navigation gaps; advertise and test it as a project extension, never an OGC claim." Guide line 560 adopts the second half in the same words, "never a fabricated OGC relation," but uses a URN, `urn:glaux:rel:experimental-asyncapi`, rather than a versioned HTTPS namespace. Recorded under F-11 with the R-017-08 result below, since both concern the same emitted relation vocabulary.

### 3.4 The batch's material result: IDR-017 states the rule F-11 asks for

F-11 records that the Guide states no relation-spelling rule while emitting three different relation forms, and that the controlling interpretation comes from IDR-010 Appendix C row C-12. Batch 9 found the rule stated directly, as a numbered recommendation, in the accepted relationship and linkage report.

**R-017-08**, at line 764: "Emit exact Part 1 Table 3 `ogc-rel:` spellings, compare extension relation URIs case-insensitively, and confine bare values to a named compatibility adapter." Priority High, basis N/X/IDR-010.

**IDR-017 Section 9.3**, at line 464, states the operative consequence: "Known bare forms seen in tagged examples may be accepted only by a named compatibility adapter with telemetry and fixtures; Glaux does not emit duplicate bare and prefixed links."

The Guide contains no occurrence of `ogc-rel`, states no spelling rule, carries no Section 13 row for relation spelling, and emits a bare `rel: "system"` at line 485. Under R-017-08 that bare value belongs in a named compatibility adapter rather than in the ordinary emitted output.

This is the same shape as the F-03 result from iteration 25: a specific, numbered, accepted recommendation that the Guide neither carries nor mentions, with an observable divergence at a named Guide line. It belongs under **F-11** and receives no new finding number, following the standing instruction that disagreements are recorded under the existing finding. It does not resolve F-11's open question, which is which published reading controls; it establishes that the accepted research answers the question in one specific way and that the Guide's line 485 goes the other way.

### 3.5 Provenance: an explicitly governed non-adoption

IDR-019 is the largest source of apparent divergence in this batch and none of it is a gap. Its Section 17.1 asks for a W3C PROV-compatible Entity-Activity-Agent model with optional PROV-O export (R-019-01) and a validated provenance graph (R-019-18). The Guide adopts no such thing.

The project governed that difference explicitly and twice over. Guide Section 1.5, at line 282, states: "No second observation model, public PROV API, graph database, universal confidence score or security-marking regime is adopted." Guide line 1118 repeats it in the non-adoption list. And `final-idr-research-report.md` line 1288 records the same disposition at the governance level, that IDR-019's historical "PROV-compatible evidence graph" wording "is not a mandatory graph database, public PROV service or adoption of every earlier graph/assertion-envelope proposal," and that "Guide Section 4.10's practical records remain the starting point."

This is the clearest example in the review so far of the standing distinction between accepted research and mechanisms adopted into the approved plan. The report is accepted; its central architectural mechanism is deliberately not adopted; the decision is published in the Guide and in the governance record; and the specific obligations the research derives, listed in the table above, are adopted through ordinary description, value, revision, security and test design. No finding.

R-019-19 is worth noting because the Guide follows it precisely: "Do not invent a core CSAPI provenance endpoint/member; design any public capability later as an advertised, access-controlled profile." Guide line 1006 states that provenance and quality disclosure "follows Section 4.10 and the same write/query boundaries; it is not another server subsystem."

### 3.6 Findings touched

No new finding number and no new follow-up question. Three existing findings gain material:

- **F-11** gains the R-017-08 rule and the Section 9.3 statement, plus the R-017-09 half-adoption, plus the IDR-014G Section 14.3 question from Correction B.
- **F-12** gains its first peer-study support, from Correction B.
- **F-17** gains a negative data point: six accepted reports in this batch all record acceptance correctly.

---

## 4. Follow-up questions

### 4.1 `fq-13` gained substantial evidence on both halves

**`issueTime` is resolved in the accepted research.** IDR-018 line 248 states the semantic that `fq-13` asked for: "Command/Feasibility | `issueTime` | time the command was received by the target System | required on server response, ignored if client supplies it on create/update [N]." Line 373 confirms it as "server-owned issueTime," and line 442 places it in the clock chain "client send -> Glaux receive -> target-System issueTime -> status reportTime(s) -> estimated/scheduled executionTime -> actual executionTime -> result publication/delivery." That agrees with IDR-042 Section 6.3, which `fq-13` recorded, and settles the received-versus-issued conflict IDR-004 flagged.

The Guide covers the handling generically, at line 418 through request and response projections for server-generated members and at line 495 through "Preserve server-owned identity/generated relationships," and mentions `issueTime` once as a filter parameter name at line 755. It states no `issueTime` semantic and carries no Section 13 row for it. The question of whether it should is unchanged; what is new is that the accepted research now supplies the answer to be recorded if the project decides to record one.

**`systemKind` is an accepted relationship absent from the Guide.** IDR-017 line 227 and REL-005 at line 346 record it: System to Procedure, zero-or-one, "direct `systemKind@link` / SensorML `typeOf`," with a resolvable Process-compatible target and cross-encoding equality. Line 470 adds that "Part 1 uses direct `systemKind@link`, `platform@link`, `deployedSystems@link`, and `sampledFeature@link` members alongside generic link arrays." None of `systemKind`, `systemType`, `typeOf`, `platform@link`, `deployedSystems@link` or `sampledFeature@link` appears anywhere in the Guide.

This is recorded as an observation, not raised as a finding. The Guide is a design guide and does not enumerate wire member names generally, so the absence of member names is not by itself a gap, and establishing whether published Part 1 requires handling the Guide should state would need the Part 1 text and the Guide Section 7 requirement tables, which a key-section screen does not read. The batch that accounts Part 1 requirement coverage remains the right home, as `fq-13` already records.

**`systemType` is narrowed.** It appears in none of the six batch-9 reports, including the two that model resources and relationships. It appears in IDR-004. On this evidence it is most likely IDR-004's conceptual name for the feature type rather than a separately tracked obligation, which is consistent with Guide line 386 treating `properties.featureType` only for sampling features. Recorded as a narrowing, not a resolution.

### 4.2 `fq-12` is extended, not duplicated

`fq-12` asks whether the Guide should cite IDR-055, and already notes that R051 and R057 are likewise undefined so IDR-055 is not uniquely uncited. Batch 9 adds three more: the Guide defines reference markers for R015, R018 and R020 but defines none for **IDR-016, IDR-017 or IDR-019**, and cites none of the three anywhere, although all three are accepted and all three own subject matter the Guide relies on. The Guide defines 47 report markers in total.

The provenance case has an explanation on the record: the Guide reaches IDR-019 through R061, the provenance interoperability study, which line 1296 of the final research report says reconciles the IDR-019 findings. IDR-016 and IDR-017 have no such intermediary. This extends the existing question rather than creating a new one, and its character is unchanged: reference completeness for the consolidation sweep, not a substantive gap.

---

## 5. Disposition summary

| Item | Disposition | Where |
|---|---|---|
| Correction A | Withdrawn. The unique-newest fixture cannot establish tied-timestamp behavior or corroborate the broad `latest` criticism. Time-range observation and Guide 1129 verification retained | Section 1 |
| Evidence 31 | Body preserved byte-for-byte; correction appended as Appendix A; manifest records both hashes | Section 1.5 |
| Correction B | Complete. All 102 omitted lines read. Questions are about the peer projects and study method, not Glaux obligations | Section 2 |
| F-12 | First peer-study support, from two independent studies | Section 2.3 |
| Batch 9 coverage | Six reports screened at decision-usable sections; IDR-017 Section 9.1 reused not re-read; none recorded as fully read | Section 3.1 |
| Acceptance check | All six correctly recorded. **No new F-17 instance**; a negative result that bounds F-17 | Section 3.2 |
| Adoption | Substantially adopted, many near-verbatim. One stated non-adoption (R-018-03 snapshot paging) and one half-adoption (R-017-09 namespace form) | Section 3.3 |
| **F-11** | **R-017-08 states the rule F-11 asks for; Guide line 485 diverges from it.** Recorded under F-11, no new number | Section 3.4 |
| IDR-019 provenance | Central mechanism deliberately not adopted, governed explicitly in Guide 282, Guide 1118 and the governance record. Derived obligations adopted. No finding | Section 3.5 |
| `fq-13` | Substantially narrowed on both halves; `issueTime` answered by accepted research; `systemKind` absence recorded as observation | Section 4.1 |
| `fq-12` | Extended to IDR-016, IDR-017 and IDR-019. No new question | Section 4.2 |

**Remaining: 18 batches of 27.** Next selected batch is **batch 10 of 27, key-section screen E**, covering encoding, schema and validation: IDR-021, IDR-022, IDR-023, IDR-024 and IDR-025.

---

## 6. Statement of limits

This iteration read 102 previously omitted lines in five peer studies, the decision-usable sections of six research reports, six acceptance rows in the governance record, and the Guide text needed for comparison. It did not read those reports' bodies, validation sections or appendices; did not re-read IDR-017 Section 9.1 or any section recorded as already read; did not retrieve any external source; did not open any implementation issue; and did not begin any batch after 9.

The accounting above compares accepted research against Guide v1.3 text. Where the Guide does not carry a recommendation, that is recorded as a non-adoption and is raised as a finding only where a divergence is evidenced at a named Guide line, which happened once, under F-11. An unadopted research recommendation is not a defect.

No implementation, Goal, Guide, Roadmap, issue or upstream change was made. `review_complete` remains `false`.

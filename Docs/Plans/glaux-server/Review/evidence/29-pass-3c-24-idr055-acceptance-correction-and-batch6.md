# Pass 3c, iteration 24 — IDR-055 acceptance correction, then queue batch 6

**Date:** 2026-09-20
**Provider/model:** Claude Code; model identified by the runtime environment as Claude Opus 5 (model ID `claude-opus-5`).
**Batch:** (1) a bounded correction to the iteration 23 conclusion that IDR-055 was unaccepted; (2) queue batch 6, key-section screen A. Resumed from planning commit `21db20c8dc29ba8f6c92015d6e5cef9548c13df5`.
**Mode:** read-only review of research reports, two governance documents and the Implementation Guide; review artifacts updated and published; no implementation, Goal/Guide/Roadmap, issue, settings or upstream changes. No recommendation was implemented, no batch was added and no scope was expanded.

---

# Part A — Correction: IDR-055 was accepted on September 16, 2026

## A1. What the record actually says

Two project governance documents record the acceptance, and both were read this iteration:

- **`IDR Plans/overall-idr-research-plan.md`**, decision log, dated **2026-09-16**: "IDR-SRV-055 Acceptance and IDR-SRV-056 Authorization | **Accepted** the deny-by-default trace-driven security architecture; strict local-issuer/JWKS authentication proof; route/action inventory, object/property/function, twin-world disclosure and canary tests; source/ingestion, profile, secret, DDIL and streaming coverage; independent command gates, synthetic safety rules, bound single-use dispatch tickets, non-network simulator and audit/effect reconciliation; PR/nightly/manual/RC/future tiers, SecurityTestResultV1, 14-column matrix and twelve implementation proofs; authorized the bounded Interoperability Test Matrix for External CSAPI Clients iteration | **Glaux Project Lead**".
- **`IDR Reports/final-idr-research-report.md`**, topic inventory: "IDR-SRV-055 | Security and Command-Control Tests | ... | Complete | **Accepted** | Deny-default, twin-world disclosure, canary, gate, ticket, simulator, and reconciliation tests are required."

The same log shows the preceding row, "IDR-SRV-055 Research Completion," with status "Pending Glaux Project Lead review," so the report moved from completion to acceptance on the same day.

## A2. How the error was made, and its one honest feature

Iteration 23 concluded that IDR-055 was unaccepted from the report's own metadata alone: its header omits the `Accepted By` and `Acceptance Date` fields that IDR-030, IDR-034, IDR-039A and IDR-043 all carry, and its closing block still reads "**Research Result:** Complete; report awaiting Glaux Project Lead review," with its acceptance boundary written conditionally throughout.

That inference was wrong. The one thing iteration 23 did correctly was state the limit: its Section 8 recorded that "no external acceptance record was consulted, and if one exists elsewhere in project governance it would supersede this reading." It does, and it now has. The correct lesson is that the limit should have been closed by checking the governance record before publishing a status conclusion, not merely disclosed.

**Evidence report 28 is preserved unchanged**, with its recorded hash intact. This report appends the correction; it does not rewrite history.

## A3. Conclusions that depended on the incorrect status, and their corrected form

Only conclusions that actually rested on the status are revisited. Each is listed with what it becomes.

### A3.1 "Not an F-17 instance" — reversed; this is a second confirmed F-17 instance

Iteration 23 reasoned that pre-acceptance wording is correct for an unaccepted report, so F-17 did not apply. With acceptance established, the reasoning inverts: IDR-055 is an **accepted** report whose text still says it awaits review. That is exactly F-17's subject.

It is a stronger instance than the IDR-043 one recorded in iteration 19. In IDR-043 the header carried the acceptance record correctly and only a trailing sentence and a checklist line were stale. In IDR-055 the header **omits the acceptance fields entirely**, so a reader checking the report alone cannot discover that it was accepted, and the closing block affirmatively states the opposite. IDR-001 shows the contrasting good practice: its Section 8.2 records the acceptance decision and date inside the report body.

Recorded as a confirmed instance under F-17. The remedy is the same editorial one already described there: use the actual dated acceptance record.

### A3.2 The batch 16 caveat — corrected

Iteration 23 wrote into the batch 16 queue note that the verification-quality pass "must treat this content as an unaccepted proposal." That instruction is withdrawn and replaced.

The corrected caveat, now written into the batch 16 note, is that IDR-055 is **accepted research**, on the same footing as IDR-052, IDR-053 and IDR-054, and may be used as an accepted project baseline for analysing planned verification. The distinction batch 16 must still preserve is a different one, and it is the distinction this review has applied throughout: **accepted research is not the same as a mechanism adopted into the approved implementation plan.** Acceptance establishes the research baseline; whether the Guide adopted any particular mechanism is a separate question answered by reading the Guide. An accepted recommendation that the Guide did not adopt is a recorded scope choice, assessed on its merits, and is not automatically a defect.

### A3.3 The F-03 fifth source — upgraded

Iteration 23 recorded IDR-055 as a fifth source for the response-cache instance but qualified it as "corroboration only, given the unaccepted status." That qualification is removed. IDR-055 is a fifth **accepted** source, and it remains the first to express the rule as a testable leakage surface naming ETags. The instance's substance and remedy are unchanged.

### A3.4 "The Guide never cites it" — premise gone, replaced by a narrower observation

Iteration 23 treated the absence of an `[R055]` reference marker in the Guide as "expected for an unaccepted proposal." That explanation no longer holds, so the fact was re-examined this iteration.

The Guide defines and cites reference markers for the neighbouring accepted test-strategy reports: `R050` (conformance), `R052` (TDD and layers, cited at lines 903 and 952), `R053` (fixtures, cited at 944 and 952), `R054` (performance, cited at 989) and `R056` (interoperability, cited at 991). It defines **no** `R055` marker and cites the report nowhere. `R051` and `R057` are likewise undefined, so IDR-055 is not uniquely uncited.

This is a reference-completeness observation, not a substantive gap: the Guide's security-test content at line 612 has a cited lineage through `R039` and `R039a` at line 592, and iteration 23 established that its families match IDR-055's closely. Recorded as **`fq-12`**, noted and not scheduled, for the final consolidation to sweep alongside any other reference-hygiene items.

### A3.5 The non-adoption conclusion — stands, on two legs instead of three

Iteration 23 gave three reasons why IDR-055's unadopted tool portfolio, execution tiers, evidence schema, twelve proofs and simulator harness are not Guide gaps. The first reason, that the report is unaccepted, is withdrawn. The other two stand and are sufficient: the standing review instruction not to add a tool portfolio or governance framework because an optional mechanism is absent, and F-02 already owning the enforced-CI question at the right altitude. The conclusion is unchanged; its support is narrower and is restated honestly here.

## A4. Conclusions that did not depend on the status, and are unchanged

The read-coverage statement, the IDR-039A handoff verification (consistent on all four handed-over elements), and the entire adopted-versus-not-adopted accounting against the Guide in report 28 Section 4.1 did not rest on acceptance status and are unaffected.

---

# Part B — Queue batch 6: key-section screen A, standards and obligation baselines

## B1. Coverage, stated precisely

This is a screen to establish coverage, not a full read. For each report, the sections actually read this iteration are listed; everything else in these reports remains unread.

| Report | Sections read this iteration | Reused, not re-read | Not read |
|---|---|---|---|
| IDR-001 | §6 Key Recommendations; §8 Risks, Constraints and Open Questions (including §8.2 acceptance decision and §8.3 deferred questions) | — | §§1-5, 7, 9-12 |
| IDR-002 | §6 Key Recommendations; §8 Risks, Constraints and Open Questions | — | §§1-5, 7, 9-12 |
| IDR-003 | §6 Key Recommendations; §8 Risks, Constraints and Open Questions | — | §§1-5, 7, 9-12 |
| IDR-004 | §6 Recommended Glaux Server Planning Terminology; §8 Risks, Constraints and Open Questions | — | §§1-5, 7, 9-12 |
| IDR-005 | §9 Recommendations and Decision Analysis; §11 Risks, Constraints and Open Questions | — | §§1-8, 10, 12-15 |
| IDR-006 | §12 Recommendations and Decision Analysis | **§13 risks and the 23-row interpretation register**, already read and recorded in `evidence/03-pass-3a.txt` | §§1-11, 14-17 |
| IDR-007 | §12 Recommendations and Decision Analysis | **§13 risks and the 17-row interpretation register**, already read and recorded in `evidence/03-pass-3a.txt` | §§1-11, 14-17 |

No report in this batch is recorded as fully read. Executive summaries and validation-against-plan sections were not read; on the evidence of earlier iterations those are summary and process content, and the screen targeted the decision-usable sections.

## B2. Result: the recommendations are substantially adopted, several near-verbatim

These seven reports are the project's boundary and baseline layer, and the Guide reflects them closely. Representative matches:

| Report and recommendation | Guide disposition | Reference |
|---|---|---|
| IDR-006 guardrail 3: generate a Glaux OpenAPI from the decided contract; "do not fork the broken example wholesale or let code generation make standards decisions" | **Adopted, near-verbatim.** "Do not deploy the upstream example OpenAPI bundle unchanged, and do not claim OGC's separate OAS 3.0 class merely because a 3.1 document is available." | Guide line 362 |
| IDR-006 guardrail 7: "Keep opaque links opaque... clients and tests should not synthesize undocumented paging or routing rules" | **Adopted, near-verbatim.** "Return server-generated opaque `next` links, not a promised public offset API." | Guide line 444 |
| IDR-007 guardrail 6: keep HTTP resource conformance separate from Part 3 and project streaming guarantees | **Adopted, near-verbatim.** "SSE remains a Glaux interface, not a Part 3 binding." | Guide line 547 |
| IDR-001 rec 6 and IDR-003 rec 7: pin the Features Part 4 revision; keep draft dependencies and experimental profiles isolated and labelled, never advertised as adopted conformance | **Adopted.** The transaction baseline is pinned to an exact commit with the later revision's permissions explicitly not imported; extensions are "clearly labeled" and the SSE extension is disabled unless configured | Guide lines 254, 260, 362, 543 |
| IDR-003 rec 5 and IDR-006 guardrail 9: schemas and Annex A tests are complementary, not interchangeable; published requirement and test contradictions must be logged and resolved explicitly rather than copied into the harness | **Adopted**, and already the subject of a closed standards check and an existing finding | Guide §7.2 adapted-test rule and §13 rows; findings F-08 |
| IDR-001 rec 4 and IDR-005 rec 5: external dependencies are explicit ports and contracts; native decoding and platform translation sit behind adapters | **Adopted.** The module table separates domain, standards and server concerns; production device adapters implement their own protocols behind one boundary | Guide lines 304-308, 580 |
| IDR-005 risk: server follows arbitrary links or forwards caller credentials, causing SSRF or confused-deputy access | **Adopted.** Schema references resolve from a local allowlist and the server does not fetch arbitrary `$ref`, SensorML links, data URLs or result URLs during a public request | Guide line 420 |
| IDR-002 risk: hiding stale or last-known information as current | **Adopted.** Freshness rules identify old or missing evidence without inventing a failed-device state; last-known state stays timestamped and does not become false current availability | Guide lines 147, 481, 970 |
| IDR-005 recs 1-2 and IDR-001 rec 9: keep the core conformance surface to the four-standard package; no adjacent NATO publication is a current server requirement | **Adopted.** The dependency table is the four-standard package plus the inherited Features and Common bases; no national or NATO labelling adapter is selected; organizations retain release authority and accreditation | Guide lines 165, 254, 610 |
| IDR-004 vocabulary rules: API DataStream versus SWE DataStream, Observation versus result, Command versus CommandStatus versus `currentStatus`, Feasibility versus capability, SamplingFeature versus ultimate feature of interest, description validTime versus data coverage, freshness as a policy assessment | **Adopted in practice throughout**, as verified across earlier iterations | Guide lines 434, 440, 452, 467, 479-483, 576, 731-732, 754-755 |

No recommendation in this batch contradicts the Guide, and none produced a new finding.

## B3. Two candidates recorded, not asserted

Both surfaced from IDR-004's terminology crosswalk and IDR-003's defect list. Neither is claimed as a Guide gap, because in each case the underlying obligation would have to be verified against the published standard or the Guide's requirement tables first, and this batch is a screen.

**`fq-13` (new): `Command.issueTime` semantics, and `systemType`/`systemKind`.** IDR-004 §6.1 flags `Command.issueTime` as "Source-version-qualified command time pending resolution of the published 'received' versus 'issued/request-receipt' conflict," and its §8.2 explicitly defers each published prose-versus-schema defect to IDR-006-008 and the test topics. The Guide mentions `issueTime` once, as a filter parameter name at line 755, and states no semantics; IDR-042 §6.3 separately reads it as "when receiving System received a Command." Separately, IDR-004 describes `systemType` as a required conceptual URI for a System's primary activity and `systemKind` as a distinct optional link to a Procedure; neither string appears anywhere in the Guide, whose only `featureType` treatment at line 386 concerns specialized sampling-feature types. Whether either is a genuine Guide gap depends on the published Part 1 text and on the Guide's §7 requirement tables, neither of which was checked here. Recorded for the batch that accounts Part 1 requirement coverage.

**`fq-05` extended: a candidate published test seam at A.115.** IDR-003 §8.1 names four editorial contradictions in the approved text, of which three map to material this review already handles: the Part 1 Annex A SensorML 2.1 prerequisite (F-08 family), the Part 2 media-type requirement conflict (`fq-04` and Guide line 737), and the Part 1 §19.2.2 versus Requirements 89-90 mismatch. The fourth, "P2 Abstract Test A.115 binary/text mismatch," lies outside the thirteen IDR-011 §14.3 rows the review confirmed in iteration 14, which covered Part 1 A.65 and Part 2 A.31 through A.62. The Guide lists the `swecommon-text` class for Requirements 115-122 at line 890 but carries no overlay row for an A.115 defect. `fq-05` already asks whether the Glaux ATS overlay should absorb additional published seams, so this candidate is added there rather than opening a new question. It is unverified against the published Annex A.

## B4. One incidental observation supporting Part A

IDR-001 §8.2 records its own plan-owner acceptance decision and date inside the report body, and enumerates what that acceptance confirms. That is the practice IDR-055 lacks, and it strengthens the F-17 instance recorded in Part A: the corpus contains reports that carry their acceptance internally, so the omission in IDR-055 is a deviation from an established pattern rather than an absent convention.

---

## Disposition summary

| Item | Disposition | Where |
|---|---|---|
| IDR-055 acceptance | **Corrected.** Accepted 2026-09-16 per the overall research plan decision log and the final report inventory. Report 28 preserved unchanged | Part A1-A2 |
| "Not an F-17 instance" | **Reversed.** Second confirmed F-17 instance, stronger than the IDR-043 one because the header omits the acceptance fields entirely | A3.1 |
| Batch 16 caveat | **Replaced.** Treat as accepted research; continue distinguishing accepted research from mechanisms adopted into the Guide | A3.2 |
| F-03 fifth source | **Upgraded** from corroboration-only to a fifth accepted source | A3.3 |
| "Guide never cites it" | Premise withdrawn; narrower reference-completeness observation recorded as `fq-12` | A3.4 |
| Non-adoption of the tool portfolio | Conclusion stands on two of its three original reasons | A3.5 |
| Batch 6 screen | Seven reports screened at the recommendation and risk sections; IDR-006/007 §13 reused; no report recorded as fully read | B1 |
| Batch 6 result | Recommendations substantially adopted, several near-verbatim; no new finding | B2 |
| Two candidates | `fq-13` new; `fq-05` extended with the A.115 candidate. Neither asserted as a gap | B3 |

**Remaining: 21 batches of 27.** Next selected batch is **batch 7 of 27, key-section screen B**, covering conformance, API definition and documentation: IDR-009, IDR-010A, IDR-012, IDR-013 and IDR-014, plus the unread key sections of IDR-010, whose link sections were already read and must be reused.

## Statement of limits

This iteration read two governance documents to settle the acceptance question, and the recommendation and risk sections of seven research reports. It did not read those reports' executive summaries or validation sections, re-read IDR-006 §13 or IDR-007 §13, verify the `issueTime` or `systemType` obligations against the published Part 1 text, verify A.115 against the published Annex A, fetch any issue body, or begin any batch other than batch 6. Report 28 is preserved with its hash unchanged. The F-17 instance is an editorial matter in a research report, not a defect in the Guide. `review_complete` remains `false`.

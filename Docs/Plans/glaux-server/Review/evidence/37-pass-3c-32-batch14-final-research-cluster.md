# Pass 3c iteration 32 - queue batch 14, the final research cluster

**Date:** September 20, 2026
**Provider:** Claude Code
**Model:** Claude Opus 5 (model ID `claude-opus-5`), per the active-model identification supplied by the environment at the time of this iteration
**Authorization:** One user `proceed`, executing queue batch 14 as saved.
**Comparison target:** `glaux-server-implementation-guide.md` v1.3, 1242 lines.

**This batch closes `research-key-section-sweep`, the last open research check.** Review area 3 is complete.

---

## 1. Coverage comparison

Built from the section headings before reading. These five are the addendum studies and share a common structure.

| Report | Recommendations | Risks and open questions | Decision register | Not read |
|---|---|---|---|---|
| IDR-058 draft Part 4 sampling features | §6, 5 items | §8.1, §8.2 | §4.5, §5 Decision Analysis | §§1-3, 4.1-4.4, 7, 9+; executive summary; validation |
| IDR-059 enhanced querying and spatial retrieval | §6, 6 items | §8.1, §8.2 | §5 Decision Analysis | §§1-4, 7, 9+; executive summary; validation |
| IDR-060 Part 5 Protobuf-first | §6, 4 items | §8.1, §8.2 | §4.5, §5 Decision Analysis | §§1-4, 7, 9+; executive summary; validation |
| IDR-061 provenance interoperability | §6, 5 items | §8.1, §8.2 | §5 Decision Analysis; §7.1 implications and prospective checks | §§1-4, 9+; executive summary; §9 validation |
| IDR-062 CS-GO engineering practices | §6, 5 items | §8.1, §8.2 | §5 Decision Analysis; §7.1, §7.2 | §§1-4, 9+; executive summary; validation |

Every section the scope assigns was read. None of the five is recorded as fully read.

---

## 2. Acceptance, and one false positive that matters

### 2.1 These five are accepted through the addendum workflow

They are not rows in the synthesis report's main progress table, which is why a table-row search does not find them. Their acceptance is recorded in the synthesis report's addendum material at line 20 and in the per-addendum sections: IDR-058 and IDR-059 accepted September 17, 2026; IDR-060, IDR-061 and IDR-062 accepted September 18, 2026. All five also carry correct `Acceptance Date` headers. **No F-17 instance in this batch.**

### 2.2 IDR-059 matched the search and is not an instance

IDR-059 line 340 reads: "These recommendations are research outputs awaiting the project lead's decision, not new requirements imposed by report publication."

The F-17 search matched it on "awaiting … project lead." It is a **false positive**, and the sentence is not stale. It says the report's *recommendations* await a decision about whether they become requirements. It does not say the *report* is unaccepted. That distinction remains true after acceptance, and it is the same distinction this review has applied throughout: accepting a research report does not adopt its recommendations. IDR-060 recommendation 4 states the principle in its own words, "Do not conflate research acceptance with adoption."

### 2.3 The sixteen stale-wording matches were verified individually

Because the pattern has a demonstrated false-positive mode, every Form A match was read in place rather than trusted as a match. All sixteen are genuine:

- **Eleven are direct status claims:** IDR-006 ("Plan-owner acceptance remains deliberately unchecked"), IDR-014E, IDR-028, IDR-029, IDR-030, IDR-032, IDR-033, IDR-035, IDR-036, IDR-043 and IDR-044 each state the report is in review or that acceptance is unchecked.
- **Two are header-field claims:** IDR-055 and IDR-056 both carry "**Research Result:** Complete; report awaiting Glaux Project Lead review."
- **One is a decision-register claim:** IDR-010A marks its decisions "Pending plan-owner acceptance."
- **Two are a milder forward-authorization variant:** IDR-045 ("IDR-SRV-046 and all later topics remain unauthorized pending acceptance of this report") and IDR-046 (the same shape for IDR-047). These are stale because both successor topics were in fact authorized and executed, but they assert something about downstream authorization rather than about the report's own status.

**The F-17 population therefore stands at twenty**, and it is now a line-verified count rather than a regex result. That is the relevant strengthening from this batch: the number did not change, but its basis did.

---

## 3. The Guide took the harder conditional recommendation and met its preconditions

IDR-058's decision analysis offers a compatibility-aware option as "Recommended now" and a selected experimental static spatial option as a "Credible alternative if the user wants implementation included. Not selected by this report." Its recommendation 3 is conditional: "If experimentation is selected, name the types and behaviors precisely," with recommendations 4 and 5 attaching further conditions.

**The project chose the conditional path, and the Guide satisfies all three conditions.**

| IDR-058 condition | Guide |
|---|---|
| Rec 3, name the chosen types precisely and record excluded types, mobile behavior and derivation limits | 272 names static Point, Curve and Surface, then excludes by name: "Dedicated Solid, Specimen, StatisticalSample, FeaturePart, relative/parametric types, mobile snapshot behavior and derived-volume computation are outside this experiment" |
| Rec 4, pin the source; do not mint official identifiers or silently repair names and defaults | 272 pins draft commit `05a3c62d198ee52d0cf81a734b700967b7d864a1`; 390-394 give the exact alias table and require emitting full URIs, with "Do not accept arbitrary prefixes as equivalent types"; 1158-1160 record the adapted-schema provenance in §13 |
| Rec 5, require semantic round-trip and query evidence for any claimed support | 237 requires "Typed Point/Curve/Surface round trips, shape/association validation and query results" |
| Rec 1, Parts 1 and 2 remain the unchanged completion baseline | 272 "No approved Part 4 conformance URI is introduced. Existing generic Part 1 and dynamic-data requirements remain intact." |

This is worth recording because it is the opposite of the pattern a reviewer might expect. The Guide did not take the lower-effort recommendation the report marked "Recommended now." It took the conditional one, which carries obligations, and then discharged them.

---

## 4. Adoption accounting for the rest

| Recommendation | Guide | Disposition |
|---|---|---|
| IDR-059 rec 1 and its risk row, keep direct-location and ancestor-association queries distinguishable; preserve Case 2 as a discriminator against recursive association being mistaken for direct spatial selection | 975 "Seed inside/boundary/outside direct sampling points and an intersecting ancestor whose child point is outside; assert different direct-geometry and recursive `foi` results." | Adopted, near-verbatim |
| IDR-059 rec 2, select JSON initially; do not adopt a larger CQL2 surface without a use case | 799 "advertise `cql2-json` as both the only initial language and its default. `filter-lang=cql2-text` is unsupported." | Adopted exactly |
| IDR-059 rec 3, make sampling geometry and per-stream scalar meanings explicit; start with known explicit geometry and schema-bound scalars | 450, 454 bind selected scalar queryables to immutable contract and component paths with unit and nil rules | Adopted |
| IDR-059 rec 4, do not silently fill semantic gaps for missing history or relative frames | 975 "Use explicit retained geometry at phenomenon time that differs from current geometry, plus an unknown-history case… Assert no current fallback." | Adopted, near-verbatim |
| IDR-059 risk, hidden related data influences filtering | 469 authorized view before predicates; 975 protected facts changed with the permitted view unchanged | Adopted |
| IDR-060 rec 1, keep Part 5 implementation out of the completion target | 282 "defer Part 5 implementation"; §1.5 states the whole disposition | Adopted |
| IDR-060 rec 2, preserve the logical-model and codec separation and immutable stream revisions | 406-408; 1062 records the decision to defer Part 5 "while preserving codec separation" | Adopted |
| IDR-061 rec 1, use existing representations first and make their limits explicit | 282 "Apply the provenance study and synthesis Addendum D through the existing description, value, revision, security and test design below." | Adopted, and the Guide cites this report there |
| IDR-061 rec 2, distinguish production history from ingestion and correction history and asserted roles from verified submitter identity | 107-109 production context stored separately from the audit record of who uploaded the data; 604-608 | Adopted |
| IDR-061 rec 3, preserve quality meaning and verify the supported JSON form; no universal percentage conversion | 410, the quality JSON interpretation, which cites this report; 608 forbids normalizing into a generic truth probability; 1152 the §13 row | Adopted |
| IDR-061 risk, provenance can disclose more than the result itself and transformation can invalidate binding evidence | 610 authorize provenance independently, and "Preserve supplied labels/binding evidence without claiming that a transformed representation retains the original signature's validity" | Adopted, near-verbatim |
| IDR-062 rec 2, use exact expected IDs, values and links, full invalid-write non-mutation and real commit and delivery checks | 446, 511, 955; 1068 records that Guide v1.2 was "informed by primary-source checks and specific CS-GO assertion examples" | Adopted |
| IDR-062 rec 1, retain the current Goal, Rust stack and issue-sized Roadmap; no new scope decision follows from acceptance | 946 and 661 cite this report; the 286-task baseline is unchanged | Adopted |
| IDR-062 rec 4, keep evidence labels honest; unknown TDD and source-only tests must not become assertions about runtime quality | 1129's own qualification that it reports source analysis rather than an observed defect | Adopted |

All five reports are cited in the Guide, most of them several times. `fq-12`'s bounded list of 25 uncited reports predicts this batch correctly: none of IDR-058 to IDR-062 is on it.

---

## 5. Two earlier conclusions independently confirmed

**The provenance non-adoption was governed, and the later research agrees.** Batch 9 recorded that IDR-019's PROV evidence graph is an explicitly governed non-adoption, resting on Guide 282, Guide 1118 and the governance record's line 1288. IDR-061 is the study the Guide reaches IDR-019 through, and it reaches the same conclusion on its own. Its decision analysis marks "New universal report/schema/graph/security-and-confidence subsystem" as **"Not recommended,"** citing "Large scope, uncertain semantics, unjustified policy/aggregation claims" and the risk of "replacing or obscuring the reference contract." Its recommendation 5 defers public PROV and DQV profiles unless a concrete exchange warrants them. So the non-adoption is not merely a project decision recorded over the research; the later accepted research recommends it.

**The batch-8 closure basis was correct.** Iteration 26 closed `peer-source-spot-checks` on the explicit basis that IDR-062 uses the same CS-GO repository whose pins had just been verified, so no new pin check was expected from it. IDR-062's §8.1 confirms this directly: "`go`, `docker`, `psql`, `cargo` and `rustc` were not found on this shell's PATH… no installations, test runs or external service writes occurred." It is a source study that establishes no new pinned artifact. The closure basis holds.

---

## 6. F-01 gains a resolving fact

Iteration 26 recorded licensing context under F-01: one study directs the project to review MPL-2.0 obligations before direct code reuse of OpenSensorHub, and another records that an absent licence text blocks confident reuse of Connected Systems Go material.

IDR-062 recommendation 5 resolves the practical question: "**Verify reuse terms only if direct copying is proposed.** No source/fixture reuse is currently required. Independently implement Rust behavior from authoritative standards and the accepted Glaux design." Its §7.1 adds that examples "are not requirements to copy every peer helper, tool or abstraction."

So the peer licence constraints do not bite, because no reuse is planned. F-01's own subject, selecting the project's own licence, is untouched by this and its disposition does not change. Recorded because the iteration 26 context could otherwise read as a live blocker, and it is not.

---

## 7. `research-key-section-sweep` closes

All nine clusters are complete:

| Cluster | Batch | Iteration | Reports |
|---|---|---|---|
| A | 6 | 24 | IDR-001 to IDR-007 |
| B | 7 | 25 | IDR-009, 010, 010A, 012, 013, 014 |
| C | 8 | 26, completed 27 | IDR-014A to IDR-014H |
| D | 9 | 27, completed 28 | IDR-015 to IDR-020 |
| E | 10 | 28 | IDR-021 to IDR-025 |
| F | 11 | 29 | IDR-026, 027, 028, 032, 033 |
| G | 12 | 30 | IDR-035, 044, 045, 046, 047, 048 |
| H | 13 | 31 | IDR-049, 051, 053, 054, 056 |
| I | 14 | 32 | IDR-058 to IDR-062 |

**Final research coverage: 18 reports fully read, 53 screened at their key sections, 71 total. No partial reads remain, and no research report is left with unrecorded read depth.** The four committed deep reads are complete and the pinned peer-source checks are closed.

Two limits stay on the record. A key-section screen is not a full read: executive summaries, bodies, appendices and validation-against-plan sections were deliberately not read, and each report's entry lists what was and was not covered. And the sweep accounts research against Guide v1.3; it does not re-derive the standards themselves.

Review area 3 is complete. The remaining queue is batches 15 to 27: scenarios, verification quality, the issue backlog and the final assessment.

---

## 8. Disposition summary

| Item | Disposition | Where |
|---|---|---|
| Coverage | Five reports screened; comparison built before reading; none fully read | §1 |
| Acceptance | All five accepted through the addendum workflow, with correct headers. **No F-17 instance** | §2.1 |
| **F-17 false positive** | **IDR-059's match is not an instance**; the sentence is correct and states the research-versus-adoption distinction | §2.2 |
| **F-17 verification** | **All sixteen stale-wording matches read in place and confirmed genuine.** Population stands at 20, now line-verified rather than regex-matched | §2.3 |
| **IDR-058** | **The Guide took the conditional recommendation and met all three of its preconditions**, naming excluded types, pinning the draft commit and requiring round-trip evidence | §3 |
| Adoption | Close throughout. Guide 799 and 975 carry IDR-059 near-verbatim; Guide 282, 410 and 610 carry IDR-060 and IDR-061 | §4 |
| Batch 9 confirmed | IDR-061 independently recommends the PROV deferral the Guide records | §5 |
| Batch 8 confirmed | IDR-062 establishes no new pinned artifact, so the `peer-source-spot-checks` closure basis holds | §5 |
| F-01 | Peer licence constraints do not bite: no reuse is required. Disposition unchanged | §6 |
| `fq-12` | Bounded list predicts this batch correctly; all five are cited | §4 |
| **`research-key-section-sweep`** | **CLOSED. Review area 3 complete** | §7 |
| New findings | None. No new finding number and no new follow-up question | |

**Remaining: 13 batches of 27.** Next selected batch is **batch 15 of 27**, the first non-research batch.

---

## 9. Statement of limits

This iteration read the decision-usable sections of five research reports, their acceptance records in the synthesis report's addendum material, the sixteen previously matched lines in other reports for individual verification, and the Guide text needed for comparison.

It did not read those reports' executive summaries, bodies, appendices or validation-against-plan sections; did not retrieve any external source; did not open any implementation issue; and did not begin batch 15.

The F-17 count is unchanged at twenty. What changed is its basis: §2.3 records that each Form A match was read in place after §2.2 demonstrated the search pattern can produce a false positive. The two milder forward-authorization instances are distinguished from the eleven direct status claims so that the editorial pass can judge them separately.

Evidence reports 31 through 36 are preserved unchanged.

No implementation, Goal, Guide, Roadmap, issue or upstream change was made. `review_complete` remains `false`. Closing the research sweep completes one review area of seven; it is not review completion.

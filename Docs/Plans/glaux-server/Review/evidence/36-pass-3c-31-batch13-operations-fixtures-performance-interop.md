# Pass 3c iteration 31 - queue batch 13, operations, traceability, fixtures, performance and interoperability

**Date:** September 20, 2026
**Provider:** Claude Code
**Model:** Claude Opus 5 (model ID `claude-opus-5`), per the active-model identification supplied by the environment at the time of this iteration
**Authorization:** One user `proceed`, executing queue batch 13 as saved.
**Comparison target:** `glaux-server-implementation-guide.md` v1.3, 1242 lines.

---

## 1. Coverage comparison

Built from the section headings before reading.

| Report | Recommendations | Risks, constraints, open questions | Decision register | Not read |
|---|---|---|---|---|
| IDR-049 migration, backup and recovery | §17, 12 items | §18.1, §18.2, §18.3 | §7.1 Tool and execution decision | §§1-16, 19+; executive summary; validation |
| IDR-051 requirement-to-test traceability | §17, 16 items | §18, one table plus the bounded open-question list | §11.3 Recommended Repository Layout, §16.2 Adoption Sequence | §§1-10, 12-15, 19+; executive summary; validation |
| IDR-053 fixtures and scenario corpus | §16, 15 items | §17.1, §17.2, §17.3 | §7.1 Recommended Repository Layout, §7.4 Git, LFS and External Storage Decision | §§1-6, 8-15, 18+; executive summary; validation |
| IDR-054 performance and load | §17, 15 items | §18.1, §18.2, §18.3 | §15.2 Regression Decision Workflow | §§1-14, 16, 19+; executive summary; validation |
| IDR-056 external client interoperability | §17, 14 items | §18, one combined table | Evidence and Decision Legend | §§1-16, 19+; executive summary; §19 validation |

Every section the scope assigns was read. None of the five is recorded as fully read.

---

## 2. F-17: a second sub-form, never previously counted

### 2.1 What this batch found

Four of the five reports, IDR-051, IDR-053, IDR-054 and IDR-056, have **no `Accepted By` and no `Acceptance Date` header fields at all**, although the governance record lists every one of them as Accepted.

That is not the form the last two iterations enumerated. Iterations 29 and 30 searched for stale closing **wording** that contradicts a correct header. A report with no header fields cannot be found that way, and none of those scans would ever have found one.

This is the form that misled this review. In iteration 23 the reviewer concluded IDR-055 was unaccepted, precisely because its header carried no acceptance fields. The user corrected it in iteration 24. The review then built two complete scans for a different sub-form and never went back to enumerate the one that had actually caused a wrong conclusion.

### 2.2 The complete two-form population

Scanning all 67 accepted reports for both forms:

| Form | Count | Reports |
|---|---|---|
| **A** - stale closing wording, header correct | 14 | IDR-006, 010A, 014E, 028, 029, 030, 032, 033, 035, 036, 043, 044, 045, 046 |
| **B** - no acceptance header fields | 4 | IDR-051, 052, 053, 054 |
| **Both A and B** | 2 | IDR-055, IDR-056 |
| Clean | 47 | the remainder |

**Twenty of the sixty-seven accepted reports are affected**, not sixteen. IDR-057 is the synthesis report itself, `final-idr-research-report.md`, which is why the file count and the accepted count differ by one.

### 2.3 Form B is not uniform, and its worst variant is the worst case in the finding

Reading what each Form B report actually offers a reader changes the severity ordering.

- **IDR-051 and IDR-052** carry no header fields but close with a dated checklist item, "Accepted by Glaux Project Lead on September 16, 2026." The status is discoverable, just not where every other report puts it. This is an inconsistency, and the mildest variant.
- **IDR-053 and IDR-054** carry no header fields and close with an **unchecked box**: `- [ ] Plan-owner acceptance and acceptance date recorded`. The document does not merely omit the status. It affirmatively states that acceptance has not been recorded, while the governance ledger records it. A reader consulting either report alone would conclude it is unaccepted, which is exactly the error iteration 23 made.
- **IDR-055 and IDR-056** carry no header fields and stale "In Review" prose as well.

So the ordering by consequence is:

1. **Worst:** IDR-053 and IDR-054, an unchecked acceptance box asserting the opposite of the truth.
2. **Next:** IDR-055 and IDR-056, missing header plus stale prose.
3. **Then:** the fourteen Form A reports, contradictory but with a correct header available.
4. **Mildest:** IDR-051 and IDR-052, correct information in an unusual place.

**IDR-052 also corrects a coverage record.** It is one of the eighteen reports this review recorded as fully read, and its acceptance-record defect was never noted. That entry is corrected rather than left implying a clean result, as IDR-006 and IDR-014E were in iteration 29.

### 2.4 What changes and what does not

The **disposition is still unchanged**: optional editorial cleanup at Low severity. The governance ledger is correct and controlling for all twenty. The synthesis report states this explicitly at its line 120, that every topic "is recorded as accepted in the controlling overall-plan ledger and progress table" with "no unresolved acceptance dependencies." That is true, and it is precisely why the per-report residue matters: the ledger is right and twenty reports contradict it locally.

What changes is that the finding now has a **complete population across both forms with a severity ordering**, so the editorial pass can start with the two reports that state the opposite of the truth rather than treating twenty items as equivalent. The remedy recorded in iteration 30, IDR-026's dated closing acceptance record, fixes both forms at once, because it supplies in the body exactly what the Form B reports lack in the header.

### 2.5 An honest note on method

This is the third time the F-17 population has grown: three, then eleven, then sixteen, now twenty. Each growth came from the review widening its own search, not from new evidence appearing. The first two were corrections of a flawed method. This one is different in kind: the sub-form was known from iteration 24 and simply never enumerated, because attention went to the variant found later. The population is now complete for both forms that have been observed.

---

## 3. Adoption accounting

| Recommendation | Guide | Disposition |
|---|---|---|
| IDR-049 rec 2, migrations only through an explicit same-image administrative command; `serve` never auto-migrates | 529 "Migrations are immutable SQL files packaged with the server and run by an explicit administrative command. Normal startup checks compatibility rather than applying destructive upgrades silently." | Adopted, near-verbatim |
| IDR-049 rec 8, restore only into fresh isolated targets and activate through an independently authorized fenced cutover | 529 test backup and restore into a separate isolated database; 531 "Fence an old serving instance before activating its replacement; a test clone must not publish or command as the original." | Adopted, near-verbatim |
| IDR-049 rec 9, never redispatch restored commands; reconcile nonterminal effects | 531 "Restored pending work remains held... Do not automatically resubmit them" | Adopted |
| IDR-049 rec 10, stable event IDs with at-least-once recovery and lineage-bound cursors; explicit gap and resnapshot behavior | 531 "A rollback or fork establishes a fresh recovery/source epoch before serving continuity tokens... old tokens require a fresh snapshot, even when their numeric position exceeds the restored log head" | Adopted |
| IDR-053 recs 11 and 12, exact goldens only for deliberate lexical contracts; prohibit CI snapshot acceptance; govern every normalization | 955 "Review changed goldens (checked-in expected outputs) against the source contract; do not auto-accept generated output, weaken assertions or remove contractual IDs/timestamps/links merely to pass. Normalize only documented non-contractual variation." | Adopted, near-verbatim |
| IDR-053 rec 5 and its live-endpoint risk row, pin official artifacts offline, keep live dependencies optional | 944 fixtures source-attributed and versioned, "do not depend on live operational information or external servers for routine tests" | Adopted |
| IDR-053 recs 7 and 10, partition fixtures rather than relying on a few large realistic examples; synthetic first scenario | 944 "Use a small synthetic dataset rich enough to distinguish behaviors" with the enumerated partitions | Adopted |
| IDR-054 recs 1 and 7, correctness-gated evidence on a stated environment; label quantities as project benchmark definitions, not operational predictions | 989 "Record correctness alongside latency, memory, and queue growth... on a stated dataset and machine. Do not adopt invented production service levels or claim scale from a microbenchmark." | Adopted, near-verbatim |
| IDR-056 risk row, a client assumption must not become a server contract | 991 "A client workaround is evidence of interoperability behavior, not permission to change the standard contract." | Adopted, near-verbatim |
| IDR-056 recs 2 and 11, pin exact client source and releases; never block on mutable public demos | 991 "Pin client versions and record their supported subsets" | Adopted |

### 3.1 Two stated non-adoptions, neither a defect

**IDR-051's traceability graph is not adopted.** The report recommends a normalized typed requirement-to-verification graph with constrained YAML sources, canonical JSON digests, orthogonal lifecycle states and generated coverage views. The Guide contains no traceability mechanism at all; the words do not appear. Guide line 1118 declines it on two counts in its non-adoption list, naming both a "graph/evidence database" and a "separate requirements/decision-document set." This is the same shape as IDR-019's PROV graph from batch 9: a large accepted report whose central mechanism is deliberately declined on the record, while its derived obligations are adopted. The obligations are: never convert a deviation or waived failure into a conformance pass, adopted at Guide 901-907 and 959; and keep the oracle independent of the service under test, adopted at Guide 955 and 959, which also forbids a quarantined check counting as a pass.

**IDR-054's tool selections are not pinned.** It selects k6, Criterion, Vegeta and others with exact versions. The Guide names none of them, and names a different set at line 962 for property, fuzz, mutation and coverage work while saying to "select and pin actual versions/configurations when their owning tasks establish compatibility, license and platform suitability." The Guide adopts the report's principles and defers its tool choices, which is the same practice recorded for IDR-044 in iteration 30.

### 3.2 A corroboration for iteration 30

Iteration 30 recorded that Guide line 1118 declines IDR-045's package set. Guide line 297 states it directly and by citation: use three initial production packages, "without creating every package suggested in [IDR-045][R045] before it contains useful code." The Guide names the report it is declining and gives the reason. That is a cleaner record of a non-adoption than the review had found, and it confirms the earlier reading.

---

## 4. F-12 gains two more sources

F-12 concerns using production wire types to interpret responses that are supposed to be independent. Two reports state the boundary from their own side.

- **IDR-053** risk register: "fixture output copied from server | circular tests bless defects | independent expected facts and four-role separation." Its four roles are source, scenario, oracle and generated evidence, kept separate by recommendation 1.
- **IDR-056** recommendation 6: "Treat curl/Rust transcripts as attribution oracles, not extra semantic-client votes."

The Guide already carries the rule at line 955: "Independent reasoning and source data matter, not merely a separate file or assistant." Disposition unchanged; F-12 stays a narrowed optional hardening at Low severity, now with four accepted reports stating the boundary.

---

## 5. Follow-up questions

**`fq-12` holds and predicts correctly again.** Of the five reports, R049, R053, R054 and R056 have Guide reference markers. IDR-051 has none and appears on the bounded list of 25 published in iteration 29. IDR-051 is now the second report found to be both uncited and to have its central mechanism declined, after IDR-019. That pairing is recorded as an observation, not raised: a report the Guide neither cites nor adopts the architecture of is still a report whose derived obligations the Guide carries, which is what the accounting above shows.

No other carried question gained evidence, and no new follow-up question was raised.

---

## 6. Disposition summary

| Item | Disposition | Where |
|---|---|---|
| Coverage | Five reports screened; comparison built before reading; none fully read | §1 |
| **F-17 second sub-form** | **Four reports in this batch have no acceptance header at all.** This form was known from iteration 24 and never enumerated | §2.1 |
| **F-17 complete population** | **20 of 67 accepted reports**, across both forms: 14 Form A, 4 Form B, 2 both | §2.2 |
| **F-17 severity ordering** | **IDR-053 and IDR-054 are the worst case**: an unchecked acceptance box asserting the opposite of the ledger | §2.3 |
| F-17 coverage correction | IDR-052 was recorded as fully read with no instance noted; corrected | §2.3 |
| F-17 disposition | Unchanged: optional editorial cleanup, Low. Remedy unchanged and fixes both forms | §2.4 |
| Adoption | Close. Guide 529 and 531 carry IDR-049's recovery recommendations; Guide 955, 989 and 991 carry IDR-053, IDR-054 and IDR-056 near-verbatim | §3 |
| Non-adoptions | IDR-051's traceability graph declined at Guide 1118 on two counts; IDR-054's tool pins deferred. Neither a defect | §3.1 |
| Iteration 30 corroborated | Guide line 297 names IDR-045 and states the package-set non-adoption directly | §3.2 |
| F-12 | Two more accepted sources, now four. Disposition unchanged | §4 |
| `fq-12` | Predicts this batch correctly. IDR-051 recorded as the second uncited report with a declined mechanism | §5 |
| New findings | None. No new finding number and no new follow-up question | |

**Remaining: 14 batches of 27.** Next selected batch is **batch 14 of 27**, the final key-section screen, covering IDR-058, IDR-059, IDR-060, IDR-061 and IDR-062. Completing it closes the `research-key-section-sweep` check.

---

## 7. Statement of limits

This iteration read the decision-usable sections of five research reports, their acceptance rows in the governance record, the headers and closing sections of the reports named in §2, and the Guide text needed for comparison. One mechanical scan covered both F-17 sub-forms across all 67 accepted reports; §2.2 states what it matched.

It did not read those reports' executive summaries, bodies, appendices or validation-against-plan sections; did not retrieve any external source; did not verify that any named tool version exists, since the Guide defers tool pinning to implementation; did not open any implementation issue; and did not begin batch 14.

The F-17 population published in iteration 30 is superseded here. §2.5 records that this growth differs in kind from the previous two: the sub-form was known and simply never enumerated, rather than hidden by a flawed method. Evidence reports 31 through 35 are preserved unchanged.

No implementation, Goal, Guide, Roadmap, issue or upstream change was made. `review_complete` remains `false`.

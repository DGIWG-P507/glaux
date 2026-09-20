# Pass 3c iteration 34 - queue batch 16, verification-quality pass

**Date:** September 20, 2026
**Provider:** Claude Code
**Model:** Claude Opus 5 (model ID `claude-opus-5`), per the active-model identification supplied by the environment at the time of this iteration
**Authorization:** One user `proceed`, executing queue batch 16 as saved.
**Sources read:** Guide §8.1 Test layers and fixtures including §8.1.1 (lines 934-965), §8.3 Performance and regression expectations (987-992), §10 Quality Gates and Exit Criteria (1047-1060), and §7.2 Requirement-to-test connections with the surrounding verification rules (893-931).

---

## 1. Reference verification: clean

The batch scope required verifying the saved references first, because the two preceding batches each turned up a defect. All five check out:

| Saved reference | Actual heading at that line |
|---|---|
| 934, §8.1 Test layers and fixtures | `### 8.1 Test layers and fixtures` |
| 966, §8.2 Representative end-to-end scenarios | `### 8.2 Representative end-to-end scenarios` |
| 987, §8.3 Performance and regression expectations | `### 8.3 Performance and regression expectations` |
| 993, §8.4 Whole-guide walkthrough | `### 8.4 Whole-guide walkthrough and Roadmap handoff` |
| 1047, §10 Quality Gates and Exit Criteria | `## 10. Quality Gates and Exit Criteria` |

No correction was needed. These references were written in iteration 30, after the sections had been read, which is the difference from the two defective ones written during the iteration 19 queue drafting.

---

## 2. The three assessment criteria

### 2.1 Independent expected answers

Strongly established, and in three separate places.

- **§8.1 fixtures:** "Include positive and negative SensorML/SWE documents and **independently specified expected values**."
- **§8.1.1 test strength:** "State the controlling requirement, **independently expected answer** and a plausible wrong behavior." Its review bullet adds "**Independent reasoning and source data matter, not merely a separate file or assistant**," and "Encoder/decoder agreement alone cannot exclude a shared bug."
- **§7.2, line 901, the strongest statement:** "The conformance runner **must not import server selection logic, serializers, or fixture-derived expected responses as its sole oracle**. It may use ordinary HTTP/format libraries and pinned schemas, but expected resource relationships, query answers, and value meaning must be independently authored or calculated."

Line 901 is the controlling rule, and it is more specific than anything in §8. It names the three things a runner must not lean on, and it draws the permitted line at ordinary format libraries and pinned schemas.

### 2.2 Meaningful failure detection

This is the most developed part of the strategy. §8.1.1's first bullet requires demonstrating that a test can detect the intended mistake: normally write the test first, observe failure **for that behavioral reason**, then implement, with the explicit qualifier that "a setup, compilation or unrelated failure is not that proof." For already-correct behavior it requires demonstrating sensitivity with a known-bad input or controlled fault, and disclosing any remaining limitation.

Three further bullets reinforce it. Mutation and controlled-fault sampling is required at critical logic, starting from a passing reproducible baseline, with "compilation failures are not detected behavioral defects" and a requirement to inspect meaningful survivors. False-green execution is addressed directly: empty or filtered suites, unavailable services and runner errors must not yield success, there is no retry-until-green, and "a quarantined check has a linked owner/follow-up and remains an explicit evidence gap, not a pass." And §7.2 line 902 refuses the usual substitutes: "Neither a test count, a global coverage threshold nor a mutation score establishes conformance," with source coverage used to investigate untested branches rather than as a correctness percentage.

### 2.3 Appropriate real-system checks

Established as a rule about which layer may support which claim. §8.1.1: "**Use the layer that can observe the claim.** Small pure tests and narrow fakes are useful, but **database mocks do not prove SQL/PostGIS/transaction behavior, and a recording publisher does not prove broker delivery or crash recovery**. Retain real database, listener, packaged-startup and broker checks where those boundaries are claimed." It closes two specific loopholes: barriers and readiness signals rather than sleeps as evidence of ordering, and "a paused application clock does not control database or operating-system time."

The layer table backs this with real pinned PostgreSQL and PostGIS on isolated test databases, an independent HTTP runner against a running server, and pinned external clients in two languages.

### 2.4 Coverage across the planned capabilities

Every Goal §5 capability has a layer that can observe its claims.

| Goal capability | Layer that can observe it |
|---|---|
| 5.1 Discovery and navigation | HTTP contract/conformance; external-client interoperability |
| 5.2 Registration and description | Domain/codec; database integration; HTTP contract |
| 5.3 Access and exchange | All five |
| 5.4 Streaming and dynamic data | Recovery row for interruption and replay; §8.1.1 requires real listener and broker checks where those boundaries are claimed |
| 5.5 Tasking and control | Domain for status rules; recovery row for uncertain effects; HTTP contract |
| 5.6 Status and availability | Domain for status rules; HTTP contract |
| 5.7 Security, authorization, trust | Recovery/security row, access isolation |
| 5.8 Cross-environment and DDIL | Recovery row, interruption and replay |
| 5.9 Validation and conformance | §§7.2-7.3 and §8 themselves; §10 gates |

No capability is left with only a layer that cannot see its claim.

---

## 3. Two findings materially refined

### 3.1 F-12 is narrower than the review's own summary, and line 901 is its home

F-12 records that "the Guide already specifies independent expectations and prohibits implementation logic as the sole oracle. The residual concern is using production wire types to interpret supposedly independent responses."

Line 901 is the rule that does the prohibiting, and reading it exactly sharpens the residual. It forbids importing "server selection logic, **serializers**, or fixture-derived expected responses" **as the sole oracle**, and permits "ordinary HTTP/format libraries and pinned schemas."

So the position is:

- **Settled:** a runner may not rest on the server's serializers, and expected relationships, query answers and value meaning must be independently authored.
- **Settled:** a generic format library such as a JSON parser is explicitly permitted, which is the right line, since parsing JSON is not interpreting meaning.
- **Residual, and narrow:** the text does not say whether the runner may **deserialize a response into the production crate's typed structures** while holding independently authored expected values. That is not "the sole oracle," so the prohibition does not plainly reach it, and a production DTO is not an "ordinary format library," so the permission does not plainly reach it either.

The consequence is real but small: if the runner decodes with the production types, a field the production types silently drop or coerce becomes invisible to the check, and the independently authored expectation is compared against an already-normalized value. §8.1.1's "Encoder/decoder agreement alone cannot exclude a shared bug" states exactly this logic for codecs; what is missing is its application to the HTTP runner's response interpretation.

**F-12's home is Guide line 901, not §8.** Its recorded next consideration, to clarify the response-interpretation boundary, is one clause added to a sentence that already draws the line in the right place. Disposition unchanged: narrowed optional hardening, Low severity. What changes is that the finding is better supported and more precisely located than the review's summary conveyed.

### 3.2 F-09's general rule is already adopted; its residual belongs to the owning issue

F-09's recorded next consideration is to "document the specific adaptation and independent association expectation. Do not add an invented route or report an adapted test as an unmodified official pass."

Guide line 909 adopts the general half in the Guide's own words: "Implement the published abstract tests as independent HTTP checks where possible. **Extend them with negative and semantic tests when an abstract test is weak, ambiguous, or copied incorrectly. Preserve both the source procedure and the documented correction; never silently turn a server defect into a passed standard test.** Distinguish unimplemented, not run, failed, harness error, and genuinely inapplicable cases." Line 930 applies the same pattern concretely to Part 4, keeping "the pinned draft clauses, adapted-schema checks, project interpretations and independent expected results distinct from approved Part 1 tests," and §7.3 line 919 adds that passing extension tests "does not add an approved CSAPI conformance URI."

So the rule against reporting an adapted test as an official pass exists, is general, and has a worked example. **What remains under F-09 is the specific documented adaptation for the `deployedSystems` abstract-test steps**, which is fixture and test content owned by issue #81, not a Guide gap. This is a refinement of the finding's location rather than a change to its disposition, which stays a source-conflict and test-adaptation concern at Low severity.

`fq-05`, which asks whether the Glaux abstract-test overlay should absorb four further published seams, gains the mechanism rather than an answer: line 909 is where any such seam would be recorded, and the question of which seams to absorb stays open and unscheduled.

---

## 4. F-03 gains the owning layer

Batch 15 placed F-03's remedy in the indirect-disclosure clauses closing scenarios 8 and 9. §8.1's layer table names the layer that would own the check: "Recovery/security/performance | **Access isolation**, resource limits, interruption, replay, uncertain effects, representative cost." A cache serving one authorized view to another context is an access-isolation failure, so the layer exists and is already staffed with fault injection.

As at §8.2 and §9.1, no cache or validator channel is named here either. That is now the fourth independent place in the Guide where the enumeration omits it, and it is consistent rather than contradictory: the Guide does not carry the rule anywhere, so it does not appear in any of its channel lists. Disposition unchanged.

---

## 5. What the strategy does not claim

Three limits are stated by the Guide itself and are worth recording, because they bound what this assessment can mean.

- **§8.1.1 is practice, not a framework:** "These are implementation practices within existing tasks, not a separate test framework or approval process. They apply in proportion to the behavior being changed."
- **§10 gates are release checks, not drafting gates**, and the Guide records that v1.0 meets only the document-finalization condition, with the rest remaining "future implementation/release checks." It states plainly that baseline status "does not assert that every upstream ambiguity is resolved or that a conformance class has passed."
- **§7.3 forbids overclaiming:** "Do not label local conformance tests as OGC certification or organizational accreditation."

§8.3 adds the same discipline to performance: measure on a stated dataset and machine, record correctness alongside latency, and "do not adopt invented production service levels or claim scale from a microbenchmark."

**No gap was found between what the verification strategy promises and what its layers can deliver.** The strategy is unusually explicit about its own insufficiency conditions, which is the property this batch was asked to assess.

---

## 6. Research reuse

No research report was reopened and no unread section was needed. The recorded screens were sufficient: IDR-052 and IDR-053 for the test-strength and fixture practices the Guide cites at lines 952 and 944, IDR-054 for the performance discipline at 989, IDR-056 for the independent-client regressions at 991, and the IDR-051 traceability non-adoption for why the Guide keeps requirement-to-test identifiers beside executable cases rather than in a second requirements document.

---

## 7. Disposition summary

| Item | Disposition | Where |
|---|---|---|
| Saved references | **Verified clean.** No correction needed, unlike the two preceding batches | §1 |
| Independent expected answers | Strongly established in three places; Guide line 901 is the controlling rule | §2.1 |
| Meaningful failure detection | The most developed part of the strategy; substitutes for it are refused by name | §2.2 |
| Appropriate real-system checks | Established as a rule about which layer may support which claim, with two loopholes closed | §2.3 |
| Capability coverage | Complete; no capability is left with only a layer that cannot see its claim | §2.4 |
| **F-12** | **Refined and relocated.** Its home is Guide line 901, not §8. The residual is narrower than the review's summary conveyed and is one clause on an existing sentence | §3.1 |
| **F-09** | **Refined.** The general rule against reporting an adapted test as official is already adopted at line 909, with a worked example at 930. The residual is fixture content owned by issue #81, not a Guide gap | §3.2 |
| `fq-05` | Gains the mechanism, not an answer. Still open and unscheduled | §3.2 |
| F-03 | Gains the owning layer, access isolation. Fourth independent place where the cache channel is unlisted | §4 |
| Strategy limits | Stated by the Guide itself; no gap between what it promises and what its layers deliver | §5 |
| Research reuse | No report reopened; no unread section needed | §6 |
| New findings | None. No new finding number and no new follow-up question | |

**Remaining: 11 batches of 27.** Next selected batch is **batch 17 of 27**, the complete leaf-to-issue comparison across all 286 issues.

---

## 8. Statement of limits

This iteration read Guide §8.1 including §8.1.1, §8.3, §10, and §7.2 with the surrounding verification rules, in full. It read §8.2 and §8.4 in batch 15 and did not re-read them.

This assesses a written verification strategy, not executed verification. Nothing here establishes that any test exists, runs or passes; the Guide states the same at §10, where only the document-finalization condition is met. The capability-coverage matrix in §2.4 maps each capability to a layer that *could* observe its claims, which is a design property, not evidence that the layer has been built.

Two findings are refined here and neither disposition changes. F-12 is narrower and better located than the review had recorded, which is a strengthening of the finding rather than a withdrawal. F-09's general half is adopted and its residual is reassigned to the owning issue.

No research report was reopened, no implementation issue was opened, and batch 17 was not begun.

Evidence reports 31 through 38 are preserved unchanged.

No implementation, Goal, Guide, Roadmap, issue or upstream change was made. `review_complete` remains `false`.

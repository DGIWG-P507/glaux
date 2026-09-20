# Pass 3c iteration 43 - queue batch 25, Phases 6 and 7 task assessment

**Date:** September 20, 2026
**Provider:** Claude Code
**Model:** Claude Opus 5 (model ID `claude-opus-5`), per the active-model identification supplied by the environment at the time of this iteration
**Authorization:** One user `proceed` for batch 25.
**Source:** The preserved snapshot [40-batch17-issue-snapshot.json](40-batch17-issue-snapshot.json). No issue was re-fetched and no network request was made.

**All 41 batch 25 tasks are accounted for. Batch 25 is complete.**

---

## 1. Coverage

Issues #198 to #238, Phases 6 and 7, assessed in eight groups at Roadmap capability-group boundaries.

| Group | Title | Issues | Assessed | Adequate |
|---|---|---|---|---|
| 6.1 | Implement the ordered retained publication log | #198-#201 | 4 | 4 |
| 6.2 | Implement authenticated SSE and retained replay | #202-#207 | 6 | 6 |
| 6.3 | Implement the pinned experimental MQTT binding | #208-#215 | 8 | 8 |
| 6.4 | Verify transport isolation and live examples | #216-#218 | 3 | 3 |
| 7.1 | Implement static Part 4 Point/Curve/Surface types | #219-#222 | 4 | 4 |
| 7.2 | Implement scoped observation queryables and value mappings | #223-#228 | 6 | 6 |
| 7.3 | Implement the complete selected CQL2 JSON language | #229-#234 | 6 | 6 |
| 7.4 | Verify all six filtering classes and combined workflows | #235-#238 | 4 | 4 |
| | | **41** | **41** | **41** |

**No task's scope or acceptance criteria was found inadequate, and no new finding results.**

The shared version-pinning passage, the shared boilerplate and the verification strategy are reused from iterations 37, 35 and 34 and are not reassessed. All 41 issues cite Guide v1.2 through commit-pinned links, which is the construction already assessed as sound.

---

## 2. Phase 6 promises only what it can prove

Publication is where a server is most tempted to claim delivery it cannot observe. These tasks refuse the temptation in named terms rather than by formula.

**Nothing claims exactly-once.** #199 excludes "any exactly-once delivery claim" and requires that failure after send "may repeat the same identity and content but must not invent a new event". #215 states that "A successful publisher ACK is not proof that the recipient received or processed the event, and persistent sessions/retained messages must not be presented as synchronization completeness". #205 permits a replayed duplicate and says it "is not evidence that the application processed the original". #211 sends "two distinct measurements having equal values" so that equal values can never be mistaken for a duplicate delivery.

**An empty answer is not a complete one.** #200 requires proving that unavailable history "fails explicitly rather than returning a seemingly complete empty result". #234 carries the same rule into query limits: work must stop "without ignored predicates, false empty success, truncated success or a fabricated complete count".

**A checkpoint cannot run ahead of the writes.** #205 permits a cursor or checkpoint to advance "only after every earlier eligible record has been written to that stream", and #203 requires fixed-cadence empty checkpoints whose content and cadence cannot encode hidden change counts.

**Unfinished work stays unfinished.** #203 keeps resume and replay "explicitly pending its owning tasks instead of treating a temporary live-only increment as the complete SSE contract". #207 states that administrative snapshot bootstrap "is delivered by 8.2/8.4 and is not available merely because this example connects". #236 refuses to declare a conformance class complete "because basic parsing passed". #208 requires that missing service, empty test selection and runner failures "remain failures/unrun evidence, not green broker proof", and #234 that an unexecuted expensive case "remains a gap, not a claimed safe bound".

---

## 3. Fail closed, and do not disclose by shape

The batch's second consistent theme is that a denial must neither open nor lie.

**Fail closed.** #206 requires that missing or unavailable policy verification "cannot default to allow", and that deleted resources "must be authorized from retained context, not presumed public because the live row is gone". #213 requires that where topic permissions cannot enforce one whole authorized audience, "that topic [is left] disabled rather than relying on client-side filtering".

**Do not disclose by shape.** #218 is explicit that withholding the payload is not enough: "Omitting the CloudEvents data member is not enough: subject, parentid, event kind or the message's existence must not reveal the protected change." #222 requires protected resources to be absent "from results, counts/extents and relationship links".

**Concealment is not falsification.** Phase 7 states the same rule in its own vocabulary, because a filter engine can disclose by answering NULL. #223, #224, #225, #228, #232 and #238 each require that protected facts not become "row-dependent NULL, membership, counts or error detail", and #238 names the failure mode directly as an "authorization-by-NULL shortcut". #232 carries it into the adversarial case: protected properties hidden under `OR true`, `AND false` or `isNull` must receive the same scope-level outcome.

**The paired-fixture method appears where it is needed.** #213, #218, #232 and #238 each vary only the protected facts while holding the authorized view constant, and assert that the authorized view does not change. Recorded qualitatively, not as a measurement of any finding.

---

## 4. F-11: three spellings, three tasks, no rule

This finding records that the Guide emits link relations in three different forms and states no rule selecting among them, while the accepted research does state one. The batch shows what that costs downstream, because each of the three forms now has an implementing task.

| Guide line | Form | Task | What the task requires |
|---|---|---|---|
| 485 | Bare name | #212 | expected JSON preserves "the authorized `links` entry with `rel: \"system\"` and canonical parent href" |
| 560 | Glaux URN | #214 | exercise "the extension link relation `urn:glaux:rel:experimental-asyncapi`"; excludes an "invented OGC link relation" |
| 774 | Full URI | #228 | verify "the exact `http://www.opengis.net/def/rel/ogc/1.0/queryables` relation and HTTP Link header" |

Each task carries its own Guide line faithfully, and none is wrong about it. What no task does is state the selection rule, and a corpus check puts that beyond doubt: across all 286 issues there is **no occurrence of `ogc-rel`, of "compatibility adapter", or of any relation-spelling rule**. The one "case-insensitive" match, #70, is the keyword-search rule and is unrelated. The phrase "link relation" appears in three issues, #23, #94 and #214, and in the first two it names ordinary navigation checks, which this finding already records as not being the gap.

The check was proven sensitive before its empty result was accepted: injecting `ogc-rel:system` into one issue body made it match, and each of the three form-specific controls matched exactly the one issue expected.

This is the same shape as the F-03 result in iteration 41 and the finding's disposition is unchanged. It does not resolve the open question, which is which published reading controls; that question has an upstream component no Glaux decision can close. It establishes that the Guide's internal inconsistency is now reproduced in three separate task fixtures, which is precisely what this finding's recorded remedy, documenting the selected spelling rule in existing fixtures, would prevent. Recorded, not raised.

---

## 5. F-19: consistent with the narrowed boundary

Iteration 42 established that audit is verified where it is written and never across a restore or a retention boundary. This batch is consistent with that and adds two datapoints.

**Audit appears here only as a non-publication rule.** #201 requires that "private work/audit records produce no public event"; #212 that "private HTTP audit records never masquerade as System Events" and that an audit entry must not be "converted into a purported real-world System Event"; #217 that "private diagnostics/audit entries are not System Events". These protect audit from leaking, not from being lost.

**One more instance of audit verified where written.** #227 requires corrections to "prove revision/audit linkage identifies superseded evidence", which is the same shape as #163: linkage checked at the moment it is recorded.

**The retention task in this phase excludes the question.** #200 implements replay-window expiry and recovery epochs for the publication log, and its exclusions name "general observation or command-evidence purging". So the one task in these two phases that enforces a retention boundary is scoped away from the evidence this finding is about.

Phases 6 and 7 contain no restore task, so the decisive remaining check is Phase 8, which holds exchange, interrupted operation and restore and is covered by batch 26. Disposition unchanged; recorded, not raised.

---

## 6. Phase 7 says what it cannot answer

The filtering phase is unusually careful about the difference between false and unknown, and between unavailable and absent.

#230 requires three-valued logic in which "only TRUE may select a row" and "negating a NULL comparison must not make it TRUE"; #231 requires that unavailable geometry "and its negated comparison remain NULL rather than a match"; #236 repeats it as "`not` must not turn unavailable comparisons into matches". #225 requires that missing, gapped or competing geometry evidence "produce legitimate unavailability rather than reconstructed location", and never falls back to present location. #227 requires that ambiguous accepted geometry become unavailable "rather than last-arrival-wins", and states plainly that "A stale projection is not permission to return an incomplete successful result".

Two neighbouring disciplines are worth recording. The tasks refuse to invent strictness as well as leniency: #220 and #221 forbid "new topology restrictions beyond the selected sources" and additional "orientation, topology or repair assumptions as unstated rejection requirements", and #229 requires that valid thresholds "are not rejected merely for lying outside a component's measurement range". And the oracle discipline continues in its Phase 7 form: #222 derives expected spatial results "not by comparing two outputs from the same query builder", #228 compares the queryables document "to reviewed contract fixtures and direct mapping outcomes, not another projection of generator metadata", #214 compares the generated AsyncAPI description against an independently enumerated channel table "rather than another projection of the same generator metadata", and #209 authors topic tables "without obtaining expected strings from the topic builder". Recorded qualitatively only.

---

## 7. Two follow-up questions gain evidence

**fq-03 (CQL2 GeometryCollection member list).** Its relevance pointer names leaf 7.3.3, and that task is #231. The pointer is confirmed accurate: #231 carries the singleton adaptation explicitly, requiring a "singleton GeometryCollection under the approved interpretation, preserving the original schema's conflicting result and the explicit adaptation", and #237 repeats it. That is the `minItems` seam, the one Guide §13 already records. The second seam this question asks about, a `LineString` member inside a GeometryCollection, is not named; the task requires "every required geometry family, multi-geometries, GeometryCollection", which leaves the member list to the implementer's reading of "required geometry family". The question is therefore not answered by the task as written, which is what it anticipated. Status unchanged: noted, not scheduled.

**fq-09 (metrics and traces exposure).** The diagnostics half of Guide §4.12 now has tasks: #216 checks liveness and readiness "against their documented meanings", requires that an optional transport outage be identified "in protected diagnostics without misreporting required storage/configuration health", and requires that an unauthorized caller "must not gain queue, source, credential or policy details from the degraded state", which is the inference side channel this question's supporting note describes. The open part is untouched: **no task in this batch mentions metrics, traces or telemetry at all**, so the exposure boundary of that endpoint still has no owning issue. Status unchanged: noted, not scheduled.

No other carried question gained evidence. fq-06 is not answered here; no task in this batch mentions `itemType`.

---

## 8. Two scenario assertions gain implementing tasks

Iteration 33 checked three consequential scenario assertions against the design lines that must deliver them. Two of the three now have tasks. The post-backup cursor failing in a new recovery epoch is carried by #200 and #204, the latter requiring that a rollback or fork "rejects the old context even if its position is beyond the restored head" and returns the Guide's `410` with fresh-snapshot guidance. The absence of a current-geometry fallback is carried by #225. The third, a transformed payload not inheriting an unverified signature claim, belongs to Phase 8 and is covered by batch 26. Recorded as coverage, not as a finding.

---

## 9. Disposition summary

| Item | Disposition | Where |
|---|---|---|
| Coverage | **41 of 41 assessed, batch complete** | §1 |
| Adequacy | **All 41 adequate.** No new finding number and no new follow-up question | §1 |
| Delivery honesty | No exactly-once claim; ACK is not receipt; empty is not complete; checkpoints never run ahead of writes | §2 |
| Access behavior | Fail closed, no disclosure by message shape, and concealment that does not falsify | §3 |
| **F-11** | **Three Guide spellings now have three implementing tasks and no issue in the backlog states the rule.** Corpus check proven sensitive. Disposition unchanged | §4 |
| **F-19** | **Consistent with the iteration 42 boundary.** Audit appears only as a non-publication rule, plus one more linkage-where-written instance; the phase's retention task excludes the evidence in question. Decisive check is Phase 8 | §5 |
| fq-03, fq-09 | Evidence added, both still open and unscheduled | §7 |
| Scenario assertions | Two of three from iteration 33 now have implementing tasks | §8 |

**Remaining: 2 batches of 27.** Next selected batch is **batch 26 of 27**, issues #239 to #288.

---

## 10. Statement of limits

This iteration read the task-specific content of 41 issues from the preserved snapshot and the Guide lines and findings needed to judge them. It made no network request and re-fetched nothing. The one mechanical check it ran, the corpus-wide relation-vocabulary search in §4, was proven able to detect an injected occurrence before its empty result was accepted.

Coverage figures are computed from the per-issue records, not restated here as arithmetic. After this batch: **236 assessed, 1 read for targeted checks only, 237 unique reviewer-read** of 286. None of this batch's issues carried prior evidence.

This assesses written scope and acceptance criteria against the Guide. It does not establish that any task will be executed correctly, and nothing here has been run. All 286 issues remain open with an Execution record reading "Not started". No broker, transport, device or live subscriber was involved in this review at any point.

No count of tasks stating the F-12 boundary was made, following the iteration 39 withdrawal. The oracle-discipline and paired-fixture observations in §3 and §6 are recorded qualitatively and are not offered as a measurement of any finding. No claim about any peer implementation is made, following the iteration 41 withdrawal.

Evidence reports 31 through 48 are preserved unchanged except where corrections were appended in earlier iterations; those bodies remain byte-for-byte intact.

No implementation, Goal, Guide, Roadmap, issue or upstream change was made. Nothing was written to the implementation repository. `review_complete` remains `false`.

# Pass 3c iteration 44 - queue batch 26, Phases 8 and 9 task assessment

**Date:** September 20, 2026
**Provider:** Claude Code
**Model:** Claude Opus 5 (model ID `claude-opus-5`), per the active-model identification supplied by the environment at the time of this iteration
**Authorization:** One user `proceed` for batch 26.
**Source:** The preserved snapshot [40-batch17-issue-snapshot.json](40-batch17-issue-snapshot.json). No issue was re-fetched and no network request was made.

**All 50 batch 26 tasks are accounted for. Batch 26 is complete, and with it the per-task assessment of the whole backlog: 286 of 286.**

---

## 1. Coverage

Issues #239 to #288, Phases 8 and 9, assessed in eight groups at Roadmap capability-group boundaries verified against the issue bodies before assessing.

| Group | Title | Issues | Assessed | Adequate |
|---|---|---|---|---|
| 8.1 | Implement bounded administrative export/import | #239-#244 | 6 | 6 |
| 8.2 | Implement consistent snapshots and catch-up | #245-#249 | 5 | 5 |
| 8.3 | Complete migration, restore and runtime operations | #250-#257 | 8 | 8 |
| 8.4 | Verify interrupted workflows and publish reference limits | #258-#262 | 5 | 5 |
| 9.1 | Close full conformance and interpretation coverage | #263-#271 | 9 | 9 |
| 9.2 | Complete independent-client and clean-environment workflows | #272-#278 | 7 | 7 |
| 9.3 | Complete security, failure and workload checks | #279-#285 | 7 | 7 |
| 9.4 | Prepare the reference release candidate and completion summary | #286-#288 | 3 | 3 |
| | | **50** | **50** | **50** |

**No task's scope or acceptance criteria was found inadequate, and no new finding results.** One issue, #240, previously carried targeted checks only; it is now assessed as a whole unit like the rest, so no issue in the backlog remains at targeted-only depth.

The shared version-pinning passage, the shared boilerplate and the verification strategy are reused from iterations 37, 35 and 34 and are not reassessed.

---

## 2. Exchange refuses to let a claim become an authority

Phase 8 is where records arrive from somewhere else, and the recurring question is what a received document is allowed to establish. The answer is always the same: nothing that was not independently verified.

**A manifest claim is not a grant.** #239 states plainly: "Do not turn manifest-declared recipient, scope or authority into a permission grant." #240 adds that "A manifest field must not widen access", #245 that "A claimed scope/recipient cannot grant access", and #241 that spoofed source identity, mismatched recipient, unauthorized scope and expired authority "cannot create a grant".

**Authentication does not validate what the payload asserts.** #241 requires that "verified importer/audit identity remains distinct from supplied producer/creator/delegation assertions; authentication does not validate those claims or invent missing context". #277 says the same from the client side: "unknown inputs/roles remain unknown, not invented from current links or authenticated uploader identity."

**Absence is not deletion authority.** #248 is the clearest statement of it. A complete replacement snapshot may reconcile stale members only within an unchanged agreed scope; for `complete=false`, a size- or expiry-incomplete export, a changed access scope and a policy-filtered protected member, "None supplies deletion authority", and the task forbids turning "a protected omission into a tombstone" or disclosing why a hidden member was omitted.

**A bigger number is not ancestry.** #242 requires comparing decimal-string revisions "by their defined numeric meaning without treating an order comparison as proof of ancestry", and #246 that missing or mismatched overlap evidence "prevent a complete recovery claim rather than accepting any numerically smaller revision". A new source epoch "is not comparable with the old chain".

**Equivalent meaning is not equal bytes.** #243 requires that reusing an exchange identity with altered bytes conflict, "including whitespace/member-order changes that leave JSON meaning equivalent", rather than "semantic normalization or guessed equivalence"; #239 refuses to "use semantic JSON equality as exact-byte integrity".

**A settled prefix is not a settled whole.** #247 allows only "the correctly settled prefix" to be complete: a missing dependency, rejected required record or staged conflict "blocks completeness past that gap even when later changes were observed". #282 repeats it, adding that "resolving one gap does not silently settle the others".

---

## 3. F-19: the question is now decided at task level

This finding asks whether audit records are protected across modification and retention. Iterations 37, 40 and 42 found three restore tasks that omit audit, and iteration 42 narrowed the gap by showing audit **is** verified where it is written. This batch contains the two tasks that decide it, and the whole backlog is now read.

**The complete-restore task omits audit.** #251 restores "the complete data model in isolation" and inventories "every implemented resource family, selected sampling types/query mappings, exact source artifacts/schema digests, revision/parent relationships, tombstones, command/exchange receipts and pending work". Its failure cases introduce "missing/corrupt required artifact, schema or recovery/receipt state". Audit records appear in neither list. It reaches the same boundary #194 did, through "required internal evidence persists", and stops at the same place.

**The dedicated retention task omits audit.** #254 implements "dependency-safe retention controls" and derives "an independent retain/remove matrix ... for accepted values, bound schemas/exact artifacts, pending work, unresolved/terminal receipts, tombstones and exchange/publication recovery evidence". Every category of evidence except audit is enumerated. This is the task this finding's retention half is about.

**Meanwhile the same batch verifies audit wherever it is written.** #243 lists "required audit/publication records" in the expected committed state and requires that a before-commit rollback leave no partial audit. #244 requires resolution to commit "revision/audit together" and forbids "a successful outcome without its committed audit". #248 checks "revision/audit outcomes". #257 requires a request or import to leave "its coherent committed state with required audit/receipt/outgoing work or no accepted partial state". #259 checks "atomic audit/outcome records". #264 requires "resource/revision/relationship/audit/retry/outgoing state" to commit coherently or not at all. #282 preserves "recorded reasons/audit" through conflict resolution. #256 even protects audit from the observability layer, excluding "removal of durable audit facts merely to simplify diagnostic tests" and stating that "Metrics and sampled logs do not replace committed mutation/command" evidence.

**The asymmetry is complete and it is exactly the finding's shape.** Across all 286 tasks, audit is required at the moment it is written, in many places and in named terms, and is named in no restore and no retention check. #283, the release-readiness recheck of restore safety, reaches the same "private evidence survives where required" phrasing without naming audit either.

Disposition unchanged: recorded, not raised. What has changed is that the evidence is no longer a sample. The recorded remedy, a check that audit survives a restore in the Guide §8.2 scenario 6 verification, now has two named task homes as well, #251 and #254. **The final assessment should consider whether evidence this complete warrants a stronger recommendation than a clarification candidate; this iteration does not make that change unilaterally.**

---

## 4. The other three audit-family findings gain task-level evidence

Three findings were raised from the same early reading as F-19 and name this batch's issues as their affected work. All three have now had that work read in full rather than in a targeted check.

**F-22, exporter-side accountability, is confirmed at task level.** This finding records that no explicit exporter-side durable audit requirement was found in the Guide or the export issue, and that import receipts are not proof of an exporter-side record. Its affected work is #240, which until this iteration carried targeted checks only. Read in full, **#240 contains no exporter-side audit or accountability requirement anywhere**: it exercises a denied recipient, a protected dependency and an unauthorized scope change, and requires authorization to be verified separately for payloads, inputs, provenance relationships, schemas and artifacts, but it never asks that the export itself leave a durable record. The asymmetry the finding describes is now visible in the backlog rather than inferred: the import side names audit in its committed state (#243) and in resolution (#244), and the export side names it nowhere. Disposition unchanged; the finding's recorded next consideration, distinguishing generation, release and confirmed receipt, is unaffected.

**F-21, node-local versus exchanged audit history, has its premise confirmed and gains a remedy location.** The manifest field table #239 derives from Guide §4.11 is fully enumerated in the task, and it carries no audit-trail field: format, exchange identity, source and source epoch; recipient, scope and scope version; mode, snapshot identity, cursors and completeness; and per-record kind, identity, revision, predecessor, dependencies, ancestry, operation, media type, payload and digest. The exchange format is therefore not an audit replication protocol, exactly as this finding states. Its recorded remedy is to clarify the boundary in the exchange documentation if useful, and that documentation now has an owning task: **#287** reconciles the recovery examples "with tested retention, authority, scope, conflict and deduplication limits", which is where a sentence about what does and does not travel would belong. Disposition unchanged.

**F-20, the denial-audit deliverable, is unchanged and consistent.** Its affected work is in Phase 1 and was not reopened. Worth recording from this batch: #280 reruns the whole cross-family access matrix on the release candidate with permitted and denied sources throughout, and asserts nothing about an audit record for those denials. That is consistent with the finding, which asks the owner to define selected categories and failure behaviour rather than treating denial auditing as already tested. Disposition unchanged; no new evidence either way.

---

## 5. F-11 and F-03: both carry-forwards close cleanly

**F-11.** Iteration 43 identified one implementing task per relation form and found no spelling rule in any issue. The carry-forward asked whether a Phase 8 exchange or Phase 9 conformance task states one. **None does**: across #239 to #288 there is no occurrence of `ogc-rel`, of a bare `rel:` value, of an extension relation URI or of the phrase "link relation". The conformance reverification tasks and the documentation task (#287) reconcile links against tested behaviour without stating which spelling is correct. The finding's position is unchanged and its evidence is now complete.

**F-03.** This finding records that Guide line 739 requires `Vary` and representation-specific validators without the `private, no-store` default the accepted research pairs with them, and iteration 41 identified #151 as the implementing task. A corpus check now confirms the gap reaches the whole backlog: **no issue among the 286 contains a cache directive (`Cache-Control`, `no-store`, `max-age`) or a conditional-request header (`If-None-Match`, `If-Modified-Since`), and none contains `ETag`.** Validators are present as "representation-specific validators", in #151 and in its Phase 9 rerun #264, and `Vary` appears as an HTTP header in #151 alone; the eight other matches are the verb beginning a paired-fixture instruction. So the backlog implements exactly what the Guide says and no more, which is where this finding has always placed the gap.

Both corpus checks were proven sensitive before their empty results were accepted, by injecting `Cache-Control: private, no-store` and `itemType: DataStream` into one issue body and confirming each was found, and by controls that matched the expected issues.

---

## 6. The scenario assertions are fully accounted for

Iteration 33 checked three consequential scenario assertions against the design lines that must deliver them. All three now have implementing tasks, the last of them here.

| Scenario assertion | Implementing task |
|---|---|
| Post-backup cursor fails in the new recovery epoch | #200, #204, and now #252, which presents tokens issued "at positions beyond the restored log head" and requires the specified `410` and documented exchange failure |
| No current-geometry fallback | #225, reverified by #268 and #285 |
| A transformed payload does not inherit an unverified signature claim | **#277**, which requires preserving "supplied label/binding context without claiming a transformed payload retains an unverified original signature", with #152 carrying the codec side |

Guide line 976, scenario 9, is reproduced in #277 clause for clause, including the uploader-versus-creator separation, unknown context without invented defaults, all four quality component types and the signature clause. Recorded as coverage, not as a finding.

---

## 7. Two follow-up questions gain their strongest evidence

**fq-06 (are Part 2 resource collections exposed?).** This question notes that the Guide lists the `/collections` routes without saying which families are exposed, and that the answer decides whether eighteen inherited Features tests per non-feature collection are executable or must be recorded as not applicable. The backlog does not decide it: **no issue among the 286 contains `itemType`**, although 48 issues discuss collections and eight check collection membership. The question now has weight it did not have when raised, because #271 requires every claimed class to be reconciled "in both directions" with "inherited requirements ... not dropped from the denominator", and #288 requires all 25 direct CSAPI classes "with applicable prerequisites". Whatever the answer is, those two tasks are where an undecided denominator would surface. Status unchanged: noted, not scheduled, and not raised as a finding.

**fq-09 (metrics and traces exposure).** The question was recorded with "no owning issue identified yet". It has one now: **#256** extends "readiness/liveness, structured logs and bounded-cardinality metrics to command, publication and exchange work", carries the label-cardinality discipline of Guide line 667, requires that protected content and secrets cannot escape into logs or metrics, and protects durable audit identifiers separately. It still does not state who may read the metrics endpoint, which is the question's residual. Status unchanged, with the owning issue now recorded.

---

## 8. Phase 9 refuses to declare completion it cannot evidence

The release phase is the last place a false claim could enter, and its tasks are written against that.

#271 reconciles declarations with evidence "in both directions", requires that "inherited requirements are not dropped from the denominator", excludes inferring "an OAS 3.0 claim from OpenAPI 3.1", states that "a materially unresolved conflict may prevent an unqualified claim", and ends "local test results are not certification". #263 requires confirming that "required suites actually execute for this candidate" because "Test counts, schema validity or a permissive runner cannot replace requirement-identified outcomes". #286 requires that reproducibility mean "the stated workflow and contents are reproducible, not an invented assertion of identical binary bytes", and that evidence identify "the actual built commit/configuration, not another build or a partial suite relabeled complete". #287 forbids adding "approved draft conformance URIs" or implying SSE is the Part 3 binding. #288 closes it: evidence "from a different commit/configuration cannot silently substantiate this candidate", "Deployment disablement does not substitute for implementation", a candidate with gaps "is explicitly partial", and no summary may declare "local testing to be certification/accreditation or an upstream ambiguity resolved".

The §13 interpretation discipline also survives to the end: #265, #267, #269 and #287 each require the documented source conflicts and adaptations to remain visible rather than being resolved by the implementation.

---

## 9. Disposition summary

| Item | Disposition | Where |
|---|---|---|
| Coverage | **50 of 50 assessed, batch complete.** The per-task assessment is finished: **286 of 286**, none at targeted-only depth | §1 |
| Adequacy | **All 50 adequate.** No task in the entire backlog was found inadequate. No new finding number and no new follow-up question | §1 |
| Exchange | A claim is never an authority: not a grant, not ancestry, not deletion, not completeness, not equal bytes | §2 |
| **F-19** | **Decided at task level.** The complete-restore task (#251) and the dedicated retention task (#254) both omit audit while eight tasks in this batch require it at commit time. Evidence complete; disposition unchanged; flagged for the final assessment | §3 |
| **F-22** | **Confirmed at task level.** Its affected work #240, read in full for the first time, carries no exporter-side audit requirement while the import side names audit | §4 |
| **F-21** | Premise confirmed: the manifest field table carries no audit-trail field. Remedy location added: #287 | §4 |
| **F-11** | **Carry-forward closed.** No Phase 8 or 9 task states a relation-spelling rule | §5 |
| **F-03** | **Carry-forward closed.** No cache directive, `ETag` or conditional-request header appears in any of the 286 issues | §5 |
| Scenario assertions | All three from iteration 33 now have implementing tasks; the signature clause is carried verbatim by #277 | §6 |
| fq-06, fq-09 | Strongest evidence so far; both still open, fq-09 now has an owning issue | §7 |
| Release discipline | Completion may be recorded only with evidence for this candidate; a gap makes it explicitly partial | §8 |

**Remaining: 1 batch of 27.** Next is **batch 27 of 27**, the final consolidated assessment.

---

## 10. Statement of limits

This iteration read the task-specific content of 50 issues from the preserved snapshot and the Guide lines and findings needed to judge them. It made no network request and re-fetched nothing. The two corpus-wide mechanical checks in §5 and §7 were proven able to detect injected occurrences before their empty results were accepted.

Coverage figures are computed from the per-issue records, not restated here as arithmetic. After this batch: **286 assessed, 0 read for targeted checks only, 286 unique reviewer-read** of 286. The per-task assessment is complete.

This assesses written scope and acceptance criteria against the Guide. It does not establish that any task will be executed correctly, and nothing here has been run. All 286 issues remain open with an Execution record reading "Not started". No backup, restore, broker, device or production system was involved in this review at any point.

That every task's written scope and acceptance criteria are adequate is not a statement that the implementation will be correct, that the Guide is complete, or that the recorded findings are resolved. Twenty findings remain recorded, two of the twenty-two having been withdrawn, and most of them sit against the Guide, the research record or the baseline rather than against any task. The questions listed in `follow_up_questions` remain open.

No count of tasks stating the F-12 boundary was made, following the iteration 39 withdrawal. No claim about any peer implementation is made, following the iteration 41 withdrawal.

Evidence reports 31 through 49 are preserved unchanged except where corrections were appended in earlier iterations; those bodies remain byte-for-byte intact.

No implementation, Goal, Guide, Roadmap, issue or upstream change was made. Nothing was written to the implementation repository. `review_complete` remains `false`; the final assessment is batch 27.

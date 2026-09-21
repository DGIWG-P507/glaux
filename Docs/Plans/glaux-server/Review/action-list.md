# Glaux Server review follow-up actions

**Status: first bounded technical update published and verified. Later propagation is pending; other actions remain proposed.**

**Prepared:** September 20, 2026. The review remains complete. This is the working action checklist for using its results, not another review, research plan, requirements document or replacement roadmap.

## The recommendation in plain English

Keep the Goal, architecture and 286-task backlog. Choose the licence and confirm how automated checks will be enforced, then make a small set of agreed clarifications in the existing Guide and issue instructions. Handle later technical questions before their owning tasks, not as prerequisites to starting the entire project. Leave optional cleanup and withdrawn findings out of the critical path.

Sources: [final assessment and controlling Erratum A](evidence/51-pass-3c-45-final-consolidated-assessment.md#erratum-a---two-statements-corrected), [findings](findings.md), and [saved questions](review-state.json), at [review-close commit abf2efa](https://github.com/DGIWG-P507/glaux/tree/abf2efad7761d44f63c2f7d4c68f8ff023027ab1/Docs/Plans/glaux-server/Review). Reviewed baseline: Goal v1.8, Guide v1.3, Roadmap v1.18. Current approved follow-up: Guide v1.4 / Roadmap v1.19, with Goal v1.8 unchanged. Every issue number below belongs to [glaux-server](https://github.com/DGIWG-P507/glaux-server/issues).

Except for explicitly adopted rows recorded below, these are **recommendations**, not standards obligations or changes to finding severity. Document/issue updates do not establish executed software correctness.

## First authorised update — 20 September 2026

The project lead's `proceed` after the bounded next-step proposal authorised F-12 independent response checks, early F-19 audit expectations and F-20 denial auditing in the Guide and issues #15/#19/#22/#26, plus read-only F-02 inspection. It did **not** approve every row, licence selection, enforcement-setting changes, installations or coding. The completed review and its evidence remain unchanged.

| Delivery | Current result |
|---|---|
| Guide / current references | [Published planning commit 0d06e6c](https://github.com/DGIWG-P507/glaux/commit/0d06e6cc3d71ab8c8ac510c0cbcffccc0af82b57): Guide v1.4 adopts the selected rules; Roadmap v1.19 aligns the reference without changing task scope, numbering or dependencies. |
| [#15 — transactional audit](https://github.com/DGIWG-P507/glaux-server/issues/15) | **Amendment published and read back:** exact audit fields, atomic failure rollback, serving/admin/retention separation and the denial-recording boundary. |
| [#19 — HTTP response checks](https://github.com/DGIWG-P507/glaux-server/issues/19) | **Amendment published and read back:** inspect required wire fields independently of production-type normalisation; prove a known-bad response fails without banning permitted extensions. |
| [#22 — denied actions](https://github.com/DGIWG-P507/glaux-server/issues/22) | **Amendment published and read back:** selected categories as implemented, safe fields, finite bounds and failure checks that never allow the denied operation. |
| [#26 — initial restore](https://github.com/DGIWG-P507/glaux-server/issues/26) | **Amendment published and read back:** compare captured audit against an independent pre-backup manifest and reject missing/corrupt required audit. No promise of post-backup recovery. |
| Runtime proof | None: these are planning/acceptance changes. Issues remain open and implementation has not started. |

**Publication evidence:** the four amendment bodies were read back exactly on 21 September 2026 at 00:57 UTC (20 September local). Each points to Guide commit `0d06e6cc3d71ab8c8ac510c0cbcffccc0af82b57` and retains the full original issue body and preparation pins after the amendment divider. Titles and open status are unchanged. The GitHub app connector rejected the first edit with a permission error and changed nothing; the authorised edits then used the existing Git account, which has repository write permission, without changing permissions or exposing credentials. Static link/whitespace checks passed, all 286 Roadmap leaf definitions were compared unchanged, and assistant diff review found no material contradiction. No runtime test was executed or represented as passed.

**Still to propagate under these adopted Guide rules:** #80 (runner), #251/#254/#283 (full restore/retention) and #280 (denial regression), plus applicable later resource owners as implemented. Do not mark F-12/F-19/F-20 wholly delivered from four early amendments. F-19's separate post-backup-deletion proposal remains unadopted. Other proposed changes below remain for a later authorised iteration.

### Current merge-check inspection — F-02

Observed **20 September 2026 local / 21 September 2026 00:49 UTC**, server HEAD `b1a80298305fd62164160058cde6d1794f533174`:

- [Main branch](https://api.github.com/repos/DGIWG-P507/glaux-server/branches/main): `protected: false`; required-check enforcement `off`; no required contexts/checks.
- [Repository/parent rulesets](https://api.github.com/repos/DGIWG-P507/glaux-server/rulesets?includes_parents=true) and [effective main rules](https://api.github.com/repos/DGIWG-P507/glaux-server/rules/branches/main): both empty.
- [Workflow inventory](https://api.github.com/repos/DGIWG-P507/glaux-server/actions/workflows): zero workflows. The pinned tree has no workflow files; current HEAD has no check runs or commit statuses.
- The dedicated protection endpoint returned a connector administration-permission error. That error is **not** the basis for the result; the successful branch/rules reads above establish that no checks are enforced. Public effective-rules/workflow reads required no credentials.

**Outcome:** inspection delivered; configuration remains pending and unauthorised. #6 must first establish actual working checks—there are no existing job names to select. Before dependent code merges, the project lead needs to approve applying required-check enforcement to those real checks, with no new mandatory human-review gate. No setting was changed, and no successful CI run is claimed.

## Decisions needed from the project lead

There is no need to personally resolve every technical detail. The technical recommendations below can be accepted or adjusted as a group, with the explicitly unresolved interpretations left for their named tasks.

| Finding | Recommendation and decision needed | When / existing owner |
|---|---|---|
| F-01 — project licence | **Project lead selects the organisation-approved licence.** Dependencies do not select it for us. No peer copying is planned; revisit reuse terms only if copying is proposed. | Before [#4](https://github.com/DGIWG-P507/glaux-server/issues/4). |
| F-02 — enforced checks | Inspect current controls; the review only established the old baseline. **Recommend required automated checks before merges**, retaining PR-per-issue and assistant merging after checks, without mandatory human review. Report any enforcement limitation before changing settings. | Decide early; implement/verify with [#6](https://github.com/DGIWG-P507/glaux-server/issues/6), before dependent merges such as #7. |

**Decision record:** licence remains unselected; enforcement changes remain unauthorised. Only the first bounded update recorded above is approved. This does not convert the remaining recommendations into requirements.

## Recommended bounded changes

I recommend adopting these improvements in existing Guide sections and issue acceptance criteria, not adding subsystems. Later owners need not delay the first task.

| Finding | Proposed change and why | Where it belongs / evidence of completion |
|---|---|---|
| F-03 — cached responses | Bind validators/cache reuse to the authorised representation; default protected responses to `Cache-Control: private, no-store`. Require cross-caller and hidden-change leakage tests. Keep HTTP freshness distinct from observation freshness. No new cache service. | Guide §§4.6/6.2/8, scenarios 8/9; #151/#264. Done when the rule and discriminating checks are explicit. |
| F-12 — independent response checks | **Adopted in Guide v1.4:** examine relevant raw response fields with ordinary format tools, not only production types that may hide dropped/coerced fields. Require a malformed/missing-field fixture that must fail. Production-type checks may supplement, not replace, that evidence. This was an open boundary resolved by this approved change, not permission established by the reviewed baseline. | Guide §§7.2/8.1.1; #19/#80. Early delivery above; later propagation still pending. No new framework. |
| F-14(c) — observation ordering | State **ascending** explicitly for the existing observation sort key and tie-breaker, as the accepted research recommends. Do not change the selected changing-view paging model. | Guide §4.4; #113 (observation paging), then #233. Complete when direction and ordered expected results are explicit. |
| F-19 — audit survival and cleanup | Include audit at the backup's recovery point in restore comparisons. Define authorised retention and distinguish serving, administrative and retention permissions. Do not promise unavailable post-backup records. **Durability is not tamper-proofing:** no hash-chain or audit-replication project. | Guide §§4.7/4.10/8, scenario 6; #15/#26, then #251/#254/#283. Done when restore and retain/remove checks name audit and its limits. |
| F-19 — post-backup deletions | Address restored resources deleted after the backup. Reconcile available deletion evidence before serving; otherwise state the limitation and operator decision, not a false continuity guarantee. Do not assume an external deletion ledger. | Guide §4.7/scenario 6; #26/#126/#251–#252/#283. Done when behaviour and a fixture cover local reads and re-export. Separate from audit survival. |
| F-20 — denied actions | Define bounded denial categories, safe fields and recording-failure behaviour. **Audit failure never authorises the denied operation.** No independent logging platform. | Guide §4.10; #15/#22/#280. Done when categories, bounds and failure checks are specified. |
| F-21 — what exchange does not transfer | Say plainly that exchanging resources and supplied production context does not replicate the source server's complete authenticated audit trail. Backup only protects its captured recovery point. | Guide §4.11 and exchange documentation; #239/#240 and #287. Complete with consistent explanatory wording; no audit-synchronisation feature. |
| F-22 — export accountability | Record export generation and authorised release/handoff durably, protecting actor/recipient/scope metadata. Specify failure/partial-output handling. Neither event proves recipient receipt. | Guide §§4.10/4.11; #240. Done when events and failure tests are specified; no delivery-receipt platform. |
| fq-09 — metrics and traces | Make metrics/traces internal or access-restricted by default, distinct from minimal public liveness. Use the existing deployment/access-control approach; do not create another administration service. | Guide §4.12; #256. Complete when the exposure boundary and an unauthorised-access check are explicit. This remains a proposed resolution of the recorded question. |

## Interpretations to carry into existing work

These do not justify another broad standards study. Reuse the recorded evidence; consult an exact source only where the remaining choice affects the task. An implementation choice does not resolve an upstream contradiction.

| Finding | Recommended treatment | Existing owner / stopping point |
|---|---|---|
| F-07 — latest observations | Keep the selected authorised-and-filtered maximum result time, retaining ties. Document the interpretation and interoperability limits beside the existing fixtures. Make no claim about peer behaviour. | Guide §§4.4/13; #112. Stop when the selected rule and limits are explicit; no behaviour change. |
| F-08 and F-09 — imperfect published tests | Document both-parts prerequisite conflicts and the `deployedSystems` adaptation. Preserve source/adapted procedures and qualified results, including SWE array-form and CQL2 singleton/dataset qualifications. An adapted pass is not an unmodified official pass. | Guide §§7.2/13; #80–#82/#121–#122; SWE/filtering #267–#268; declarations #271/#288. Stop at traceable procedures and evidence labels. |
| F-11 — relation spelling | Record the selected spelling **for each relevant relation**, based on the existing source-conflict evidence. Do not infer that a bare value, a private URN and a full OGC URI are automatically interchangeable or that all must share one spelling. Keep compatibility handling explicit and limited. | Guide §§4.5/4.8/6.3.1/13; #56, #212, #214 and #228. Decide before the first affected fixture (#56); stop with the rule and matching fixtures. |
| F-13 — unknown observation members | Select preserve-as-opaque, ignore or reject for unmapped members; the review does **not** choose. Keep POST/PUT/PATCH consistent without weakening known/read-only-field validation. Resolve ordinary observation JSON first, then family-specific rules—not universal rejection. | Guide §§4.6/13; #98–#102. Stop with the choice, the misleading `foi@id` example, and method-consistency tests documented. |
| F-18 — command-stream `live` | State and test what `false`, `null` and missing values mean for admission, distinguishing Command from Feasibility. Use the recorded source distinctions; do not automatically adopt the research's `409` suggestion or invent a stream lifecycle. | Guide §§4.6/4.9 and scenario 4; #156/#163/#164 and feasibility tasks #177/#180. Stop with the chosen behaviour and its negative cases. |

## Leave unchanged or clean up later

| Finding | Recommended disposition |
|---|---|
| F-04 — task size | Keep the existing roadmap. Split a task only when real implementation evidence shows it will not fit, using the existing recalibration practice. No speculative backlog rewrite. |
| F-05 — earlier tasking | Keep the current sequence. Earlier JSON-only tasking is optional, not a missing capability. |
| F-06 and F-16 | Withdrawn: no action. Do not recreate them as work items. |
| F-10 — broken research citations; fq-10 | Correct only the identified pinned links in IDR-011 and IDR-040 during an authorised documentation cleanup. fq-10's repository-identity question is already resolved. Preserve source pins rather than substituting moving branches. |
| F-14(a)(b) — parents and identifiers | Retain existing parent tests and required UID handling. Do not mistake optional canonical-URL compatibility or extra returned UID metadata for missing mandatory functionality. |
| F-15 — unkeyed submissions | Keep optional retry keys and the documented ambiguous-recovery limitation. Do not silently introduce mandatory keys or safe automatic resubmission promises. |
| F-17 — stale research acceptance text | Optional bounded cleanup of the already identified reports, using their own actual acceptance dates and authority. Reuse IDR-026's style, not its date or authorisation. Do not repeat the corpus census or rewrite archived review evidence. |
| fq-12 — missing Guide citations | Add useful research markers when editing the affected Guide sections, or record why indirect citations suffice. Not a new substantive requirement. |

## Remaining questions: route them, do not commission twelve studies

The review's questions remain recorded in [review-state.json](review-state.json). The following is a proposed handling plan, **not a claim they are answered**. Linked issues are affected owners, not newly assigned research projects.

| Question | Proposed handling and timing |
|---|---|
| fq-01 — binary encoding omitted from root schema | Check whether the actual validation path uses that root before the Phase 4.3 binary work; if it does not, record inapplicability. Do not rewrite a schema that the path never uses. |
| fq-02 — optional prose label versus required schema label | Resolve for minimal component/publisher fixtures in Phase 2.1, before the Phase 4.1 codec fixtures inherit them. Record the source conflict and selected validation behaviour. |
| fq-03 — LineString in GeometryCollection | Carry the exact prose/schema discrepancy into the spatial fixtures for #231/#237, alongside the existing singleton adaptation. No full filtering re-review. |
| fq-04 — SensorML media-type note | Confirm whether a required client/fixture needs the legacy alias while implementing negotiation. Keep the published type canonical; no speculative compatibility feature. |
| fq-05 — additional abstract-test seams | Account for the recorded candidates in #80/#121 and the relevant Part 2 encoding tests. Verify the still-unverified A.115 candidate before treating it as a defect. Use F-08's qualification rule. |
| fq-06 — Part 2 collection exposure | Decide which resource families are exposed through `/collections` when defining discovery, **before** their conformance fixtures, not only at release. Carry the decision into #271/#288 so applicable inherited tests are neither omitted nor claimed inapplicable without reason. Do not choose exposure merely to reduce testing. |
| fq-07 — both-parts wording | Fold into F-08. No separate workstream. |
| fq-08 — unknown-member policy across families | Fold into F-13. Distinguish ordinary JSON, GeoJSON foreign members and SensorML extension rules before extending a rule between families. |
| fq-09 — metrics/traces access | Proposed resolution is in the bounded-changes table above; needs adoption, not a new study. |
| fq-11 — future result time | Check the exact cited obligation before adding a rejection test in observation writes (#98–#102). Do not turn an unsupported research assertion into input rejection. |
| fq-12 — reference markers | Optional cleanup above; no separate workstream. |
| fq-13 — issue time and System kind/type | Reuse the recorded deferral and settle only the affected metadata/admission fields before their fixtures. Document the selected reading; do not infer a complete new model. |

A new research plan needs a substantial unanswered design question, a consequence for the approved implementation, and project-lead agreement on bounded effort. A question appearing here does not authorise research.

## How we carry this out

1. **Decide:** record licence/enforcement choices and acceptance or adjustment of the technical proposals here. The assistant supplies technical recommendations; the lead need not reconstruct the review.
2. **Apply approved actions:** prioritise early owners (#15/#19/#22/#26/#56), then later owners before their work. Update Guide versions/cross-references and existing issue instructions, preserving their original pinned baselines and recording approved amendments. Change Roadmap leaves only when necessary; do not create another backlog.
3. **Record and implement:** link delivered changes against these rows. Distinguish document changes from executed tests. Resume the existing one-issue/PR workflow without waiting for optional cleanup.

**Current handoff:** the first bounded Guide/early-issue update is delivered; stop here. Later amendments under the adopted rules remain #80/#251/#254/#280/#283, with applicable family-specific cases handled by their owners. Other proposal rows and licence/enforcement decisions remain outstanding, not implicitly approved. No implementation issue is complete. The closed `review-state.json` remains review coverage, not an implementation queue.

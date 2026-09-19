# Finish the existing Glaux Server review

Local handoff for the existing Copilot conversation. Prepared September 19, 2026.
This is not a new research plan, project-governance document, design decision, or authorization to implement changes.

## Instruction to the reviewer

Resume the existing independent review and finish its original scope. Do not restart it, replace it with a narrower review, or turn it into new research. Preserve completed work and existing finding IDs.

The objective remains a faithful Rust reference implementation of OGC API - Connected Systems. Evaluate the approved Goal, Guide, Roadmap, supporting evidence and implementation issues against that objective. Research mechanisms and peer implementations do not automatically become requirements.

The project lead controls iteration boundaries: one substantive batch per "proceed", then stop. Authorization for the review does not authorize implementation, issue changes, upstream filings, commits, pushes, software installation, or adopting recommendations.

You may maintain local review notes and a local source cache. Keep them outside tracked project documents or in an existing review-notes location. Do not introduce another project document system.

## What must change immediately

1. Stop spending whole iterations correcting or restating earlier reports. Incorporate corrections in the finding record while completing new coverage. An ordinary qualification is not a reason to restart a topic.
2. Keep one persistent coverage checklist, separate from the findings. Preserve it through compaction. Reuse existing notes instead of reconstructing the review from scratch.
3. Define the exact source sections or issue IDs for the next batch. Select a batch that fits the available context. Do not announce six reports and repeatedly finish a fraction of one.
4. Mark a unit complete only after its stated checks are performed. If interrupted, record the precise unread remainder and resume it next. Do not replace it with an unrelated easier topic.
5. Do not reread completed sources merely because another pass also uses them. Cross-reference the existing evidence; do the additional analysis without repeating retrieval. Reopen only for changed text, a specific contradiction, or a necessary unresolved check.
6. Treat shared issue boilerplate efficiently: retrieve/cache bodies once, identify truly identical sections mechanically, inspect those once, and inspect every variant. Read every issue's unique scope, exclusions, sources, prerequisites, acceptance criteria, independently expected results and constraints. Never discard customized test text as boilerplate.
7. Record potential findings during coverage; consolidate and settle them at the end. A finding needs a controlling obligation or clearly stated proposed improvement, concrete consequence, contrary evidence, and an affected implementation task. A keyword search miss is not proof of a missing design.
8. Distinguish a design defect, an unresolved implementation choice, an optional improvement and an unverified runtime claim. "Not adopted from research" is not itself a defect. A reference implementation does not need every research mechanism.
9. Do not invent additional full-report or historical-plan reading obligations. Follow the original section-based corpus review, plus the consequential deep reads already committed to below.
10. Stop expanding the review. A newly discovered concrete issue belongs in the findings with focused supporting evidence; it does not authorize a new research topic, new audit pass or wider standards ecosystem survey. A material extension requires the user's decision.
11. Keep progress reports short: work actually completed, exact remaining units, material new findings, and the next batch. Do not repeat all findings or governance language each turn. Do not claim a completion percentage from findings count, page count or pass labels.
12. If a cost/usage ceiling is supplied, honor it. None has been supplied here. Do not invent a dollar estimate, guarantee a turn count, or call incomplete coverage complete to meet an assumed budget. Report any real access or capacity blocker plainly.

## Baseline and work already paid for

Use the existing review baseline. Planning checkout verified for this handoff:
`DGIWG-P507/glaux` main at `a310eaee2e80bb861197822a3c5bb12164ca9ac3`.
Goal v1.8, Implementation Guide v1.3, Roadmap v1.18.
Use the server revision already recorded in the review; check only for relevant changes, not a wholesale restart.

Latest reviewer checkpoint: **Pass 3c, iteration 11**.

- Pass 1 baseline/document review is complete. Goal, Guide, Roadmap, README, CONTRIBUTING and issue template were read.
- Research synthesis and shared upstream evidence register were read.
- Topic reports recorded as fully read: IDR-011, 029, 031, 036, 041, 050, 052.
- Substantial reads with outstanding portions: IDR-008 sections 16-17; IDR-037 appendices; IDR-038 appendices.
- IDR-039: remaining section 7 and sections 21-22.
- IDR-040: remaining sections 9-21.
- IDR-042: sections 1-2, 5, 8, 11-14, 17-18 read. Finish sections 3-4, 6-7, 9-10, 15-16, 19-20; reconcile any other end matter against the actual headings.
- IDR-043: section 15 and sections 19-20 read; selected audit rows only from sections 6, 7, 11.4. Finish the other body sections, including 16-18 and validation/end matter as applicable; do not count grep matches as full reads.
- Additional deep reads already promised: IDR-030, 034, 039A, 055.
- There are 71 topic report files plus the final synthesis. The systematic key-section review of all 71 is not yet demonstrated complete.
- Issue bodies documented as fully read: #3, #15, #19, #21, #22, #56, #71, #72, #104, #156, #163, #164, #168, #174, #240. This is 15 of 286; the remaining 271 are not documented as fully read. Reading an issue is not the same as completing the whole-graph or test-quality assessment.
- The implementation issue set is #3 through #288 inclusive. PRs #1, #2 and #289 are not implementation issues. The local Roadmap contains exactly 286 distinct implementation-issue links.
- Findings F-01 through F-22 already exist. F-06 and F-16 are withdrawn. Preserve numbering and the latest qualifications; the remaining 20 are not all confirmed defects or coding blockers.

Keep detailed source references from prior reports. Missing evidence in this handoff is not a reason to reread everything; recover the relevant existing note first.

## Fixed remaining scope and completion criteria

These are the original review obligations, not additional passes. Work can satisfy more than one obligation at once when the actual checks are recorded.

| Original area | Finish condition |
|---|---|
| Pass 1: baseline | Reuse completed work. Check only relevant baseline changes. |
| Pass 2: standards | Close the six carried groups below and F-13's extension-rule question using the controlling pinned sources. |
| Pass 3: research | Account for key findings/recommendations/open questions of all 71 topic reports; reuse completed reads, finish the named consequential deep reads, and complete the remaining promised CS-GO/OSH source checks. |
| Pass 4: workflows | Systematically examine the approved capability workflows and relevant failures across Goal sections 5.1-5.9 and Guide workflows; record design contradictions or missing ownership, not runtime success claims. |
| Pass 5: verification | Assess whether planned checks independently detect plausible wrong behavior for those workflows and boundaries, including conformance interpretations, false-green execution, real database/HTTP/broker boundaries and fault recovery. |
| Pass 6: issues | Account for all 286 bodies, unique acceptance/verification content, scope coverage, dependency resolution/cycles, sequencing and task sizing. Complete graph analysis even for issues whose bodies were already read. |
| Final assessment | Consolidate supported findings, withdrawals, decisions, implementation timing and coverage limitations into one usable assessment. Review completion does not require implementing its recommendations. |

### Standards backlog: close it instead of carrying it indefinitely

1. SWE Common 3.0 `quality` semantics/schema claim.
2. `recordsAsArrays` and `vectorsAsArrays` interpretation.
3. Features Part 3/CQL2 class identifiers and `cql2.json` GeometryCollection `minItems`.
4. SensorML 3.0's four referenced conformance-class identifiers against 23-000.
5. Remaining IDR-011 section 14.3 abstract-test-suite discrepancy rows.
6. Part 2 Annex A.1 inheritance and its effect on class declarations.
7. F-13: extension rules relevant to `foi@id` versus `samplingFeature@id`; do not prescribe blanket rejection of otherwise allowed extensions.

### Research and peer checks

Use the actual 71-file inventory under:
`Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`.

For every topic report, identify its key findings, recommendations and unresolved-question sections from its actual headings. Record the applicable section references and whether consequential conclusions are reflected in the Guide/Roadmap, deliberately superseded, optional, or genuinely unresolved. Synthesis rows alone are not that examination. Already completed full reads count; do not repeat them.

Complete the explicitly owed deep-read portions listed above. Broader full reads require a concrete consequential question, not an expectation that every historic paragraph must be revalidated.

Finish the previously promised CS-GO/OSH spot checks from IDR-014A, 014B and 062 against the recorded repository pins. Reuse previous code checks. Do not expand this into another complete commit/PR-history study. State what claims were independently verified and what remains unsupported.

Historical plans are consulted for a specific acceptance/scope question only. "69 plans unread" is not automatically 69 new assignments.

### Workflows, tests and issues: reuse the same evidence

For the approved discovery, registration, observation/query/encoding, status, tasking/feasibility, publication, sampling/filtering, provenance/security and exchange/restore capabilities:

- Trace the requirement through Guide behavior, owning Roadmap/issue and planned verification.
- Check relevant rejection, concurrency, interruption, disclosure and recovery boundaries.
- Identify the independent expected answer, realistic wrong behavior, required test environment and whether the planned assertion would detect it.
- Do not require every possible failure permutation or a particular testing tool. Explain consequential missing checks.
- Distinguish normative conformance, documented project interpretations and experimental drafts.
- Record scenario/test conclusions while reading the owning issues so later passes do not repeat the same work.

For issue processing, fetch a bounded batch of bodies, not one remote request followed by a report for each issue. Cache locally where supported. Read unique content for every issue, and compare recurring boilerplate to catch exceptions. Retain source/version metadata.

Resolve each prerequisite to its leaf or explicitly defined group. Expand group dependencies according to the Roadmap before testing the complete graph for missing nodes, cycles and sequencing errors. Do not assume that issue numbering alone establishes a valid dependency order.

Record sizing risks without automatically splitting or editing issues. A task needing refinement is not proof that the server design is defective.

## Carry-forward qualifications: do not litigate them again

Preserve the latest detailed findings from the existing conversation. The following prevent known review mistakes from reappearing after compaction:

- F-01/F-02: license selection and enforceable repository checks are specific concerns; do not select a license, alter repository controls or impose mandatory human PR approval without authority.
- F-03/F-12: existing independent access-matrix and test-oracle provisions are real. Narrow any residual semantics/independence concern; do not claim that no tests or runner owner exists. A separate test crate is not automatically required.
- F-07 through F-11/F-14: preserve checked standard/schema conflicts and accepted project interpretations. Peer behavior is not normative. Distinguish an issue already documented in research from one not carried clearly into the Guide.
- F-13: a wrong association member and unknown-extension handling are distinct questions.
- F-15: the Guide explicitly permits unkeyed command POSTs and disclaims safe automatic retry/unique recovery. A mandatory-key option would be a new choice.
- F-16 remains withdrawn: bounded all-or-nothing requests are an explicit project choice with rollback tests.
- F-17: stale acceptance wording is an editorial matter where dated acceptance records establish the decision; do not reinvent an acceptance-precedence rule.
- F-18: distinguish `live=false` from `null`, Command from Feasibility, and schema annotations from behavioral ownership. Do not invent a new lifecycle.
- F-19: durability/atomicity do not themselves claim tamper evidence. INSERT privilege does not imply UPDATE/DELETE privilege. Distinguish serving, owner/migration and authorized-retention roles. Append-only recommendations must account for any permitted retention. Hash chains are not automatically required.
- F-20: #15/#22 provide relevant ownership; the question is an explicit denial-recording deliverable and failure behavior. Proposed denial categories remain suggestions. Audit failure cannot authorize a denied operation.
- F-21: distinguish source-local audit history from supplied provenance that can travel with resources. Audit replication is not automatically required; backup protects the captured recovery point, not every subsequent event.
- F-22: no explicit exporter-side audit floor was found in the checked materials. Extending that floor is a proposed strengthening. Generation, release/handoff and confirmed receipt are different outcomes. Audit actor/recipient/scope metadata can itself be sensitive.
- Offline JWT use is bounded by token, cached-trust and policy validity. A locally reachable identity provider is not universally required for every valid existing token; renewal/refresh is a separate condition.
- Atomic pre-dispatch evidence does not guarantee that an effect's outcome can be recorded after storage fails. Do not claim rollback of physical actions or exactly-once physical execution.
- Exchange fingerprints support consistency checks against retained evidence. They neither require a later resource change to detect every mismatch nor establish source authenticity.
- Written safeguards and planned tests are not executed verification. No production-readiness, accreditation or operational safety claim follows from this document review.

## How the next iteration starts

Do not spend an iteration acknowledging or rewriting this instruction.

1. Load the existing coverage notes and reconcile the status above without reopening completed work.
2. Start substantive closure of the oldest standards backlog. The first batch is items 1-2 (SWE quality and array flags), using already pinned sources and relevant Guide text. If time remains, continue item 3.
3. Record precise evidence, dispositions and the remaining standards items. Then continue through the fixed remaining scope, choosing exact outstanding units before each batch.
4. A newly found concern is recorded for consolidation; it does not send the entire review back to its beginning.

End each turn with at most: completed units, material new findings, blocked or unread remainder, next exact batch. "Proceed" authorizes that next batch only.

## The final deliverable and the finish line

Produce one final assessment when the promised coverage is complete. Include:

1. Plain-English readiness conclusion: whether implementation can begin, under what specific conditions, and which later tasks need clarification before their own execution.
2. Supported findings, each with evidence, consequence, contrary evidence, smallest appropriate response and affected issue/milestone. Separate confirmed defects, open decisions and optional improvements.
3. Withdrawn/superseded findings, without resurrecting rejected premises.
4. Coverage accounting: the seven completion rows above, report-section coverage, all 286 issue dispositions and whole-graph results.
5. Explicit limits: inaccessible evidence, unresolved interpretations and runtime behavior not tested. Do not present excluded/inaccessible work as checked.

Findings need not be fixed to finish the review. An unresolved question can be reported as unresolved once its bounded investigation and consequence are documented; it does not require an unlimited search for certainty.

If material is genuinely inaccessible, report the exact blocked unit and its consequence. Do not call the entire review complete while silently dropping it, and do not keep billing iterations to rediscover the same blocker.

After delivering the assessment, stop. Do not automatically start another review, change project documents, publish issues or begin implementation.

## Provenance of this handoff

This handoff consolidates the existing review, not a fresh technical audit.
Original six-pass commitment: user attachment
`<local-attachment>/pasted-text.txt`, lines 182-190.
Latest coverage: user attachment
`<local-attachment>/pasted-text.txt`.
The intervening supplied reports establish the preserved reads and finding corrections.


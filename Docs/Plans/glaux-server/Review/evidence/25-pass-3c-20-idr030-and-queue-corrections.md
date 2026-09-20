# Pass 3c, iteration 20 — committed deep read IDR-030, with two queue-accounting corrections

**Date:** 2026-09-19
**Provider/model:** Claude Code; model identified by the runtime environment as Claude Opus 5 (model ID `claude-opus-5`).
**Batch:** batch 2 of the queue (committed deep read IDR-030), plus two directed corrections to the queue accounting written in iteration 19. Resumed from planning commit `bea4889573ca902d9d2021ef597178e915973d38`.
**Mode:** read-only review of one research report and the Implementation Guide, plus a read-only reachability check against the public GitHub issues API to size one queued batch honestly. Review artifacts updated and published; no implementation, Goal/Guide/Roadmap, issue, settings or upstream changes.

## 1. Sources consulted this batch

| Source | Access | Used for |
|---|---|---|
| [IDR-SRV-030](../../../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-030-data-lifecycle-retention-archival-and-deletion-strategy-report.md) | Header and Sections 1-2, 7-11, 13-15, 18-19 read; Sections 3-6, 12, 16-17 skimmed for cross-reference | The committed deep read |
| [Glaux Server Implementation Guide v1.3](../../glaux-server-implementation-guide.md) | Targeted search then read at each cited line | Adopted, not-adopted or departure comparison |
| `api.github.com/repos/DGIWG-P507/glaux-server` issues endpoint | One repository metadata request and one issue request (#98), read-only | Establish that a complete leaf-to-issue comparison is achievable in this environment before promising it in the queue |
| `review-state.json` research report entries for IDR-006, 007, 010, 014B, 017, 044 | Read | Assign their unread key sections to existing batches |

## 2. Correction 1: complete leaf-to-issue comparison replaces the sample

The iteration 19 queue specified batch 17 as a fidelity check on "a sample of roughly 18 issues, two per phase." That was wrong on its own terms: a sample can support an inference about the backlog, but it cannot establish complete issue coverage, and the review's finish condition is to account for **all** issue-specific scope and acceptance content. Batch 17 is now a complete mechanical comparison of all 286 issues.

**Feasibility was verified rather than assumed**, because promising a complete comparison without checking the environment would repeat the same error in a different form. The GitHub CLI is not installed here. The repository is public and the REST API is reachable unauthenticated: a metadata request returned HTTP 200 and issue #98 returned its full body (8,925 characters, 70 lines). The issues list endpoint returns bodies, so `per_page=100` covers 286 issues in three requests, well inside the unauthenticated limit of 60 requests per hour. That limit is recorded as a sizing uncertainty, since a per-issue approach would not fit.

**The comparison is tractable because issue bodies have a fixed structure.** Issue #98 carries the headings *Task and intended result*, *Scope and boundaries*, *Sources and applicable rules*, *Prerequisites*, *Acceptance and verification*, followed by the shared *Common completion checklist* and *Execution record*. That gives a clean split between task-specific content and boilerplate, which is what makes "review the boilerplate once and every variant, then compare only the task-specific content" a mechanical operation rather than 286 close readings.

Batch 17 as now specified: fetch all 286 bodies; separate shared from task-specific sections; review the common boilerplate once and every distinct variant, recording each variant; compare every issue's task-specific content against its Roadmap leaf Scope, Done, Guide and Depends text; inspect every difference; and produce a difference inventory keyed by issue number. Matching content is reused by the per-phase batches rather than reviewed twice there.

## 3. Correction 2: the six partial reports are accounted for

The iteration 19 queue covered the 13 fully read reports and the 51 whose read depth was never recorded, but silently omitted the seven partial reports. One of those, IDR-043, was closed as batch 1 in iteration 19. The remaining six were unaccounted for. They are now assigned to the cluster batch that already covers their topic, with their existing evidence reused and only their unread key sections screened:

| Report | Topic | Already read, reused | Assigned to |
|---|---|---|---|
| IDR-006 | CSAPI Part 1 requirement baseline | Interpretation and open-question sections (`03-pass-3a.txt`) | Batch 6, standards and obligation baselines |
| IDR-007 | CSAPI Part 2 requirement baseline | Interpretation and open-question sections (`03-pass-3a.txt`) | Batch 6 |
| IDR-010 | Collections, resources, links and navigation behavior | Link sections (`03-pass-3a.txt`) | Batch 7, conformance and API definition |
| IDR-014B | Connected Systems Go implementation study | Targeted acceptance and history references (`08-pass-3c-04.txt`) | Batch 8, peer studies, which already holds the CS-Go pins and `fq-10` |
| IDR-017 | Relationship and linkage model | Section 9.1 (`12-pass-3c-08.txt`) | Batch 9, resource model and temporal validity |
| IDR-044 | Rust implementation language and framework strategy | Section 22.2 (`04-pass-3b.txt`) | Batch 12, platform, architecture and deployment |

No batch was added for any of them. Each host cluster grows by one or two reports and stays inside the five-to-eight band. The queue now records the governing rule explicitly: a partial read does not establish complete coverage, and the recorded sections are reused rather than re-read.

A mechanical check confirms the queue now names every research report that is not already fully read, with no remaining gap.

## 4. Count change: 30 to 27

The queue is a workload estimate, not a quota, so the count moved when the evidence supported combining work.

| Change | Effect |
|---|---|
| Batch 2 (IDR-030) executed this iteration | Done rises from 1 to 2 |
| Former Phase 5a and 5b merged into one batch (42 leaves) | Minus 1 |
| Former Phase 6 and Phase 7 merged (41 leaves) | Minus 1 |
| Former Phase 8 and Phase 9 merged (50 leaves) | Minus 1 |
| Six partial reports absorbed into existing clusters | No change |
| Batch 17 widened from a sample to all 286 | No change in count; the batch is heavier but bounded and mostly mechanical |

**The merges are a consequence of correction 1, not a convenience.** Once batch 17 compares all 286 issues against their leaves, the per-phase batches no longer need to re-establish what each issue says; they account the task-specific content and inspect the differences the comparison reports. That makes 40-to-50-leaf batches realistic where 20-to-30 was the earlier assumption. If the difference inventory turns out large, those batches become heavier and the count rises again; that is recorded as the first sizing uncertainty.

**Twenty-seven batches total, two done, twenty-five remaining.**

## 5. IDR-030 accounted against Guide v1.3

IDR-030 is a large lifecycle strategy: a versioned policy registry, five orthogonal lifecycle state dimensions, per-class retention triggers and holds, archive and restore contracts, and careful distinctions between API deletion, logical deletion, tombstoning, purge, backup expiry, cryptographic erase and media sanitization. Its own Section 1 requests no numeric retention periods and states that an approved lifecycle-policy package is a prerequisite before production automated disposition.

### 5.1 Adopted

| IDR-030 item | Guide disposition | Reference |
|---|---|---|
| Recommendation 3, select no retention duration; recommendation 1, fail closed for destructive automation; decision 5, keep production automatic purge disabled for classes without an approved rule | **Adopted, and this is the central match.** "Do not enable automatic retention/purge by default. An explicitly configured policy may remove old records only while preserving the required visible deletion behavior and dependencies among values, schemas, source documents, pending work, and synchronization tombstones." | Guide line 527 |
| Recommendation 5, honour CSAPI cascade and `409` rules through an authority-aware dependency plan without letting cascade override evidence | **Adopted.** A populated stream's default deletion fails with `409`; authorized `cascade=true` deletes the prescribed nested resources; "Keep restricted internal revision/tombstone evidence where needed for audit and synchronization, but remove the public resource as required." | Guide line 509 |
| Section 9 rule 3, identifiers are never reused after purge | **Adopted, verbatim in effect.** "Do not reuse a deleted ID." | Guide line 509 |
| Rejected option, "DELETE means immediate physical erasure"; recommendation 4, treat deletion as an API and domain operation | **Adopted.** "Never use internal retention to pretend a failed deletion succeeded", with internal evidence retained separately from the removed public resource | Guide line 509 |
| Command cancellation remains an appended status, not Command deletion | **Adopted.** "Deleting a Command is not a request to cancel device execution"; cancellation follows command-status behavior, not HTTP DELETE | Guide lines 509, 582 |
| Section 7.1, idempotency and inbox records must outlive every authorized replay and offline reconnect horizon | **Adopted.** "Record a configurable terminal-outcome retention period and explain that reuse after expiry does not guarantee deduplication; unresolved admitted command work retains its key" | Guide lines 505, 849 |
| Section 8, derived state is rebuildable while accepted observations and command evidence are not | **Adopted.** "Derived summaries can be rebuilt; accepted observations and command evidence cannot be reconstructed from sampled logs" | Guide line 527 |
| Section 10, a latest-value view whose source has expired must not imply the history is still queryable | **Adopted in its prohibition form.** "never fabricate history removed by retention or never supplied" | Guide line 523 |
| Section 9 rule 4, tombstones carry minimum anti-resurrection fields rather than payload or sensitive text | **Adopted in substance.** The persistence table gives tombstones "Scoped identity/digest, saved outcome, source/revision, deletion marker" with "bounded retention and explicit recovery expiry" | Guide line 711 |
| Section 14.1, DDIL anti-resurrection through causal tombstones, and stale updates not silently recreating state | **Adopted.** "attempted resurrection of a deleted resource is retained/reported without silently overwriting accepted state"; the verification list includes "attempt resurrection after deletion" | Guide lines 647, 651 |
| Section 15.1, `409` for a delete rejected by a child rule, `412` for a stale precondition, policy-selected `404` for later reads | **Adopted.** The status table carries the same mappings, with consistent non-disclosure `404` where policy requires | Guide lines 509, 834-840 |
| Recommendation 15, restore drills before enabling automatic purge | **Adopted in the verification requirements.** Backup and restore are tested into a separate isolated database "including artifacts, schema bindings, IDs, tombstones, and pending work", and deletion is tested "with retained internal evidence and with subsequent stale synchronization input" | Guide lines 529, 533 |

### 5.2 Not adopted, as a recorded scope choice

The Guide implements no archive tier, no hold overlay, no lifecycle policy registry, no disposition receipt resource, no purge automation, no cryptographic erase and no media sanitization program. It also does not adopt the five orthogonal lifecycle state dimensions, the per-class disposition matrix, dynamic-data aggregation or downsampling, or the `202`-with-status-link contract for asynchronous deletion.

This is consistent rather than deficient. IDR-030 itself assigns media sanitization to "a deployment and media-management operation" under the system owner, leaves every numeric horizon to an approved policy package that does not yet exist, and states that acceptance "does not authorize IDR-SRV-031, draft Part 3 implementation, or server implementation." Its Section 18.3 lists ten unresolved questions, every one of which is owned by a deployment, security or profile authority rather than by the Guide. The Guide's corresponding posture is to require explicit configuration and to disable automatic purge by default, which is the safe subset of the same model. The approval-separation elements in Section 15.2 fall in the same family already dispositioned under F-03.

### 5.3 One documentation gap: restore re-exposes resources deleted after the backup point

IDR-030 is emphatic that a restore must not simply return an older database to service. Recommendation 11 requires reapplying later deletion and tombstone state before serving; Section 13.1 states that "A restored older point must receive the later deletion/tombstone ledger before it can serve data"; and Section 18.2 rejects "Restore backup directly into service" because it "Can resurrect deleted/unauthorized state."

Guide Section 4.7 treats restore carefully and enumerates several things it does **not** re-establish. A rollback or fork establishes a fresh recovery epoch before serving continuity tokens, and pre-restore cursors fail with `410` (lines 531, 547). Restored pending work stays held, reconciliation covers commands accepted after the backup, and same-key deduplication is explicitly not promised across a lost recovery interval, with that limitation to be documented (line 531). Command admission and dispatch stay disabled until restore checks pass, while "reads can be validated without actuating devices" (line 529).

What that enumeration omits is deletion. Restoring to a point before a deletion returns the deleted resource to the authoritative store, and because the Guide's tombstones live in that same store they are rolled back with it. Reads are explicitly the part re-enabled first, so the resurrected resource becomes publicly readable. The Guide states the limitation for continuity tokens, for pending commands and for idempotency keys, and is silent for deleted resources.

Two things keep this from being overstated. First, it is partly inherent to point-in-time restore: with a single authoritative store and no deletion ledger held outside the backup, there may be nothing to reapply, which is itself worth stating rather than leaving unsaid. Second, a peer that already received the deletion is protected on the receiving side, since an import that would resurrect a deleted resource is retained and reported rather than applied (line 647). The exposure is therefore local reads and any re-export from the restored node, not silent corruption of a peer.

The gap is that Section 4.7 documents its other restore limitations and not this one. Closing it needs one sentence stating what restore does to deletions and whether reads are gated until that is reconciled, plus one line in the Section 8 restore checks. Recorded as a related instance under **F-19**, whose remaining question is already intended retention behaviour and whose affected work already names the retention and configuration owners. No new finding number, and no new capability is implied.

## 6. Observations recorded, not findings

- **PostgreSQL and object-store residue.** Section 13.1 notes that `DELETE` leaves dead row versions until vacuumed, that ordinary `VACUUM` reuses rather than returns space, that point-in-time recovery deliberately preserves historical changes, and that a versioned object store may keep prior versions behind a delete marker. The Guide makes no erasure claim anywhere, so there is nothing to correct; the observation is recorded so a later reviewer does not read the Guide's deletion language as an erasure promise.
- **No numeric horizons anywhere.** IDR-030 supplies none and the Guide invents none, requiring explicit configuration instead (lines 527, 649, 663). The tombstone horizon that IDR-043 Section 8.4 deferred is likewise still unset. This is a consistent and deliberate absence across the corpus, not an omission to chase.

## 7. Findings register changes

No new finding number. One related instance added:

- **F-19 (Audit modification and retention boundary):** the restore-versus-deletion documentation gap in Section 5.3 above.

F-03, F-08, F-15, F-18, F-20, F-21 and F-22 were checked against this report and need no change. The approval-separation controls in Section 15.2 duplicate the F-03 disposition; the audit-retention delegation in Section 8 points at IDR-SRV-041, which F-19 and F-20 already track.

## 8. Disposition summary for the checkpoint

| Item | Disposition | Where verified |
|---|---|---|
| Batch 17 fidelity method | **Corrected.** Complete mechanical comparison of all 286 issues replaces the 18-issue sample; feasibility verified against the live public API | Section 2 |
| Six partial reports (006, 007, 010, 014B, 017, 044) | **Corrected.** Assigned to existing cluster batches 6, 7, 8, 9 and 12, reusing recorded evidence and screening only unread key sections; no batch added | Section 3 |
| Queue count | 30 to 27; two done, twenty-five remaining, with the merges explained | Section 4 |
| IDR-030 | Fully read; core posture adopted at Guide line 527; lifecycle machinery not adopted as a recorded scope choice | Sections 5.1-5.2 |
| Restore versus deletion | **Evidenced documentation gap**, recorded as a related instance under F-19 | Section 5.3 |

**Research-deep-read-remainders, updated remaining scope:** three committed deep reads outstanding, IDR-034, IDR-039A and IDR-055, queued as batches 3, 4 and 5. Next selected batch is **batch 3, IDR-034**.

## 9. Statement of limits

This iteration read one research report and the Guide text needed to assess it, and made two contacts with the public GitHub API purely to establish that a queued batch is achievable. It did not fetch any issue body for review, execute batch 17, resolve `fq-10`, or begin any batch other than batch 2. Sections 3-6, 12 and 16-17 of IDR-030 were skimmed for cross-reference rather than read in full; the sections carrying its decisions, rules, distinctions and recommendations were read in full, and the report is recorded as fully read on that basis. The restore gap in Section 5.3 is a documentation gap in planning material; no implementation exists, so it is not a report of observed behaviour. The queue count is a workload estimate with six recorded uncertainties, not a completion guarantee. `review_complete` remains `false`.

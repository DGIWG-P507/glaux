# Pass 3c, iteration 21 — IDR-030 coverage completion, then batch 3 (IDR-034)

**Date:** 2026-09-19
**Provider/model:** Claude Code; model identified by the runtime environment as Claude Opus 5 (model ID `claude-opus-5`).
**Batch:** (1) finish the IDR-030 sections that iteration 20 only skimmed, correcting an overstated coverage record; (2) queue batch 3, the committed deep read of IDR-034. Resumed from planning commit `deb916311069cd64cc2ff95abd4e9fcee59a5bd9`.
**Mode:** read-only review of two research reports and the Implementation Guide; review artifacts updated and published; no implementation, Goal/Guide/Roadmap, issue, settings or upstream changes.

## 1. The coverage correction, and why it was made first

Evidence report [25](25-pass-3c-20-idr030-and-queue-corrections.md) stated plainly in its Section 9 that IDR-030 Sections 3-6, 12 and 16-17 were "skimmed for cross-reference rather than read in full," and then said the report "is recorded as fully read on that basis." The state file went further and recorded `documented_read_depth: "full"` with "No further remainder outstanding for this report." Those two things cannot both be right. A skim is not a read, and the state overstated the coverage.

Report 25 is **not** rewritten. It is archived as published, its hash unchanged, and its own Section 9 already disclosed the limit accurately. This report appends the completion.

Before reading anything, the state entry was corrected to `partial` with the seven unread sections named, so the record would be honest at any interruption point rather than only after success. It is restored to `full` at the end of this iteration on the basis of the reading below, not on the basis of a skim.

## 2. IDR-030: the sections now finished

Read in full this iteration: Section 3 (source inventory), Section 4 (standards requirement map and policy-authority model), Section 5 (record-class inventory), Section 6 (lifecycle terminology, state model, transitions, actors and authority), Section 12 (archive manifest, integrity, restore and failure contract), Section 16 (fixture and verification matrix) and Section 17 (implementation and operational implications). The sections already accounted in iteration 20 were **not** reread.

Finishing them changed the record in three ways.

### 2.1 The F-19 restore instance is sharpened, and one qualification I made is now too weak

Iteration 20 recorded the restore gap with a hedge: that it is "partly inherent to point-in-time restore, since with no deletion ledger held outside the backup there may be nothing to reapply." Section 12 shows the research does not leave it there. The archive manifest is specified to carry "provenance, validation, quality and **tombstone/deletion-ledger bounds**" (Section 12.1), and the restore contract is an explicit ten-step sequence whose step 6 reads "Apply the current tombstone/deletion ledger and every later disposition event before publication. **Never expose restored pre-deletion state**," with step 2 restoring "into an isolated, non-serving namespace" and a "deletion-ledger gap" named as a terminal failed or review state (Section 12.2).

So the research supplies the missing mechanism rather than merely naming the hazard: keep the ledger bounds in the manifest so a gap is detectable, restore into a non-serving namespace, and reconcile before publication. Section 6.2's transition table independently lists the restore gate as "Manifest, compatibility, current policy, **tombstone ledger**."

This does not make the Guide wrong, since the Guide adopts none of the archive machinery and its restore is a PostgreSQL backup restore rather than a manifest-driven archive restore. It does mean the hedge was too generous: the ordering principle, restore into isolation first and reconcile deletions before serving, is available and cheap to state even without an archive tier. The F-19 instance is updated accordingly.

### 2.2 Two ready-made verification fixtures

Section 16 supplies the tests for exactly this, which a skim missed:

- "Valid restore | Restore prior backup containing later-deleted record | Current deletion ledger reapplied before serving; **deleted resource does not reappear**"
- "PITR replay | Restore point precedes deletion | Post-restore deletion/tombstone replay removes it before access"

These parallel the cache fixtures IDR-042 supplied for the F-03 instance, and they are recorded with the F-19 instance so the owner has concrete checks rather than a prose recommendation.

### 2.3 Nothing else material changed

Section 3 confirms the source inventory and states in 3.3 that "No supplied authority establishes actual online retention, archive residence, hold, tombstone, replay, backup, WAL, quarantine, audit, command, or sanitization durations," which is consistent with the no-numeric-horizon accounting already recorded. Section 4.1's requirement map is the CSAPI cascade and RFC 9110 DELETE material already matched to Guide lines 509 and 834-840. Section 4.2's policy-authority model, Section 5's 30-class record inventory and Section 6.1's five-dimension state vocabulary belong to the lifecycle machinery already recorded as a scope choice; Section 5 is a copy inventory rather than a persistence schema, so it does not compete with the Guide's persistence table at lines 700-711. Section 6.1's rule that "Deleted is prohibited as an unqualified persistent state" is consistent with the Guide's practice of naming the specific effect. Section 17.4's metrics rule ("Metrics avoid resource names, payloads, sensitive reasons, tenant leakage and high-cardinality identifiers") is a fifth corroboration for `fq-09`.

IDR-030 is now genuinely fully read.

## 3. Batch 3: IDR-034 accounted against Guide v1.3

All nineteen sections plus the completion checklist were read. The report models a DataStream as a versioned semantic contract, an Observation as a durable typed fact bound to an exact contract revision, a status update as an Observation on a `type=status` stream, and it defines four distinct selectors.

### 3.1 Adopted

| IDR-034 item | Guide disposition | Reference |
|---|---|---|
| §10.1 and R-034-05: authorize and apply every conjunctive filter before `resultTime=latest`, then return all ties; never one-per-stream, latest arrival or latest phenomenon time | **Adopted, effectively verbatim.** "first apply all other predicates within the endpoint scope, then select the greatest visible result time and retain ties"; canonical and nested endpoints have different scopes | Guide lines 440, 469 |
| §6.4 and R-034-08: schema modification after Observations exist is rejected; non-semantic description changes are ordinary revisions; incompatible shape, property, unit, frame, nil, order or encoding changes require a successor stream | **Adopted.** "While nested observations or commands exist, reject schema-modifying PUT/PATCH with `409`; calling a change compatible or creating an internal revision does not bypass that restriction. Description-only changes remain possible." | Guide line 434 |
| §6.2: `live` means live data availability only, not freshness, health, history or command readiness | **Adopted.** System status, stream delivery state, command-channel availability, server health and authorization are kept separate, with no universal readiness score | Guide line 483 |
| §11.4 and R-034-09: optional absence, registered nil with a resolvable reason, valid-but-uncertain values, and policy withholding stay distinct; bad constraints or unrecognized nils are rejected rather than coerced to null | **Adopted.** Declared nil sentinels are recognized first, remaining NaN and infinities map to CQL2 NULL with source distinctions retained, and protected information is never simply substituted with NULL | Guide lines 467, 469 |
| §13.4 and R-034-11: authorize candidates before filtering, selection, aggregation, counts, extents, freshness, paging, caching and publication; hidden newest facts must not leak through ranges, counts, no-result behaviour, links or timing | **Adopted.** Authorization precedes queries, counts, extents, links, schema disclosure, latest selection and streaming delivery, and the protected-facts test requires membership, counts and errors not to reveal changed hidden facts | Guide lines 469, 598, 975 |
| §12.4 and §17.2: PostgreSQL is the initial authority and TimescaleDB is conditional on measured benefit | **Adopted, near-verbatim.** "Begin with ordinary indexed observation tables. Choose native time partitioning only after measuring the representative workloads in §8; do not make TimescaleDB a prerequisite." | Guide line 519 |
| §13.1 and R-034-14: publish only after commit, and distinguish fact revisions, projection changes and delivery attempts | **Adopted.** The outbox commits before send with stable event and attempt identities; "A resource deletion is an event, not a normal native data record"; public lifecycle notifications, native observations, System Events, command status and private diagnostics remain distinct categories | Guide lines 539, 541 |
| §9: CRUD notifications, validation failures, delivery records, source connectivity and command progress are not System Events by default | **Adopted.** System Events are "Distinct from resource changes and transport notifications" | Guide line 705 |
| §7.2: an Observation cannot override stream unit, property, component order or encoding; conversion is a separately identified derived record | **Adopted.** Values bind a specific contract with "no mutation of the meaning of stored values"; a compatible mapping revision must preserve property identity, scalar meaning and unit, and changing Celsius to Fahrenheit is not silently compatible | Guide lines 467, 702 |
| §12.1: unknown query parameters follow strict error rules rather than silent ignore | **Adopted.** "Reject malformed or unsupported parameters rather than ignoring them" | Guide line 436 |
| §14.2 item 2: supplement `resultTime=latest`, which the approved ATS omits | **Already recorded** as the A.50 omission under F-08 from the iteration 14 standards check | findings.md F-08 |

### 3.2 A confirmed instance of F-14(c): ordering direction

F-14(c) records that "research selected ascending resultTime/ID order; Guide wording should not obscure direction." This report supplies the specific confirmation on both sides.

IDR-034 states the direction twice and unambiguously: "Accepted default Observation order is `(normalized resultTime, ResourceId)` ascending" (§10.3) and "Default order is `(resultTime, ResourceId)` ascending" (§12.2).

Guide line 444 names the same two fields and omits the direction: "Use deterministic ordering with a unique ID tie-breaker: result time then ID for observations; stable ID order for ordinary resource lists unless a family requires a different rule." A search of the whole Guide for `ascending` and `descending` returns no occurrence. So the Guide fixes the sort key and the tie-breaker but leaves the direction unstated, which is exactly the wording problem F-14(c) anticipated. Two clients implementing the Guide could page the same stream in opposite orders and both claim conformance.

Recorded as a confirmed instance under F-14, with the fix being one word at line 444. No new finding number.

### 3.3 A third source for the F-03 cache instance

§13.4 closes with "Principal-specific results use partitioned/private caches." That is the same rule already recorded from IDR-040 §10.4 and IDR-042 §7 line 341, now stated by a third accepted report. The F-03 cache instance is updated to note the third source; its substance and recommended fix are unchanged.

### 3.4 A recorded departure the Guide states for itself

§12.2 asks for snapshot-stable paging: "Concurrent ingestion cannot shift pages," with the cursor binding an evaluation time and snapshot watermark, and "Counts, links, cursors, latest, and extents share one authorized snapshot." §17.1 lists "shifting pages" as a risk whose control is "snapshot cursors."

The Guide deliberately chooses the weaker contract and says so: "For ordinary lists, document keyset paging as a changing view rather than claiming a cross-request snapshot; use an explicit snapshot/export mechanism for synchronization (§4.11)" (line 444), reinforced at line 471 and at line 473's "Data corrections may change ordinary paged results; they do not create a retrospective snapshot." This is an explicit, documented departure with a stated alternative mechanism, in the same class as the `428` departure recorded in iteration 19. It is noted here so a later reviewer does not raise it as an omission, and it needs no finding.

### 3.5 Not adopted, as scope choices

The Guide does not adopt IDR-034's current-status projection service with per-dimension selectors and freshness assessment (§8.2), its source-health record family (§5.2), its admission taxonomy with pending-dependency and quarantine outcomes (§7.4), its materialized-projection metadata contract (§10.2), or its work-package estimates (§16.3). The status separation principle is adopted at Guide line 483 and the status-under-delay behaviour at lines 481 and 970 without the projection machinery. IDR-034 itself gates several of these behind profiles and assigns others to IDR-035 and later topics.

## 4. One incidental uncertainty recorded, not chased

IDR-034 states twice that a future `resultTime` is prohibited: "the approved model says it cannot be future-dated" (§1) and "not future" (§11.1), with a fixture "future-result reject" (§14.1). The Guide contains no rule rejecting a future `resultTime`.

I did not treat this as a Guide gap, because I have not verified the underlying obligation myself and the report's own citation does not clearly carry it. Its footnote 8 attributes Requirements 96-98 to "parent result/parameter schemas and **UTC time**," which matches the iteration 15 reading that Requirement 98 constrains time scale and the result and parameter encodings only. The futurity rule may come from the conceptual model or from prose rather than from a numbered requirement. Asserting a Guide omission on that basis would be exactly the kind of unverified claim this review avoids.

Recorded as `fq-11`: does the approved Part 2 baseline actually prohibit a future `resultTime`, and if it does, should the Guide state the rejection rule and its status code? Relevance: Guide Sections 4.4 and 6.4, and the observation write leaves. Noted and not scheduled; it does not expand the queue.

Separately, `fq-10` gains weight without being resolved: IDR-034 §19.3 pins Connected Systems Go at `SomethingCreativeStudios`, matching IDR-037, IDR-038, IDR-039 and Guide line 1129, against IDR-040's `opensensorhub`. Four reports and the Guide now use one organization and a single report uses the other. That is recorded, not adjudicated; the peer-source batch still owns it.

## 5. Findings register changes

No new finding number. Three existing entries updated:

- **F-19:** the restore instance is sharpened with IDR-030 Section 12's explicit restore sequence and two ready-made fixtures; the earlier "nothing to reapply" hedge is narrowed.
- **F-14:** a confirmed instance for part (c), the unstated ordering direction at Guide line 444.
- **F-03:** the cache instance gains a third corroborating source.

## 6. Disposition summary

| Item | Disposition | Where |
|---|---|---|
| IDR-030 coverage overstatement | Corrected before reading; report 25 preserved unchanged; completion appended here | Section 1 |
| IDR-030 Sections 3-6, 12, 16-17 | Read in full; report now genuinely fully read | Section 2 |
| F-19 restore instance | Sharpened; hedge narrowed; two fixtures added | Section 2.1-2.2 |
| IDR-034 (batch 3) | Fully read, all nineteen sections plus checklist; substantially adopted | Section 3 |
| F-14(c) ordering direction | **Confirmed instance** with exact locations | Section 3.2 |
| F-03 cache instance | Third source recorded | Section 3.3 |
| Snapshot paging | Recorded departure the Guide states for itself | Section 3.4 |
| Future `resultTime` | `fq-11`, deliberately unverified | Section 4 |

**Remaining in this check:** two committed deep reads, IDR-039A and IDR-055, queued as batches 4 and 5. Next selected batch is **batch 4, IDR-039A**.

## 7. Statement of limits

This iteration read the seven unfinished IDR-030 sections and all of IDR-034, plus the Guide text needed to assess them. It did not reread the IDR-030 sections accounted in iteration 20, fetch any issue body, execute batch 17, resolve `fq-10` or verify the future-`resultTime` obligation in the published standard. Report 25 is preserved as archived, with its hash unchanged; this report appends rather than replaces. The F-14 and F-19 items are documentation gaps in planning material, not observed runtime behaviour. `review_complete` remains `false`.

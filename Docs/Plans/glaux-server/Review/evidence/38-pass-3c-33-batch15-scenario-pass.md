# Pass 3c iteration 33 - queue batch 15, end-to-end scenario pass

**Date:** September 20, 2026
**Provider:** Claude Code
**Model:** Claude Opus 5 (model ID `claude-opus-5`), per the active-model identification supplied by the environment at the time of this iteration
**Authorization:** One user `proceed` for batch 15, with a correction to its saved references.
**Sources read:** Goal §5 Core Capability Scope, §§5.1-5.9. Guide §8.2 Representative end-to-end scenarios (lines 966-986), §8.4 Whole-guide walkthrough and Roadmap handoff (993-1009), and §9 Risks and Remaining Implementation Checks, §9.1 and §9.2 (1010-1046), as supporting material.

---

## 1. Saved reference correction, made before reading

The saved scope for this batch said to work "against Guide Section 9 walkthroughs and the Goal capability set." That is wrong about where the walkthroughs live. Guide §9 is **Risks and Remaining Implementation Checks**. The representative scenarios are **§8.2** and the whole-guide walkthrough is **§8.4**.

The correction was applied to `batch_queue` entry 15 and to `current_work.next_batch` before any source was opened, and it records what the earlier wording got wrong rather than silently replacing it. §9 is used here as the supporting risk and check material the user directed.

This is the second reference defect found in the review's own saved state, after the two differing screen-scope statements reconciled in iteration 28. Both were introduced when the queue was first written in iteration 19, before the sections they name had been read.

---

## 2. Coverage: Goal §5 against the scenarios and the walkthrough

| Goal §5 capability | Guide §8.2 scenario | Guide §8.4 walkthrough row |
|---|---|---|
| 5.1 Discovery and navigation | 1, register and discover | Register and discover a System |
| 5.2 Registration and description | 1; 7, experimental sampling descriptions | Register and discover a System |
| 5.3 Access and exchange | 2, publish and retrieve; 8, enhanced observation selection | Register a stream and ingest observations; Retrieve, filter and assess status |
| 5.4 Streaming and dynamic data | 4, live delivery and recovery | Stream and recover |
| 5.5 Tasking and control | 5, feasibility and command execution; the command-recovery checks | Evaluate feasibility and issue a command |
| 5.6 Status and availability | 3, status under delay | Retrieve, filter and assess status |
| 5.7 Security, authorization and trust | **cross-cutting**: revoked access in 4; protected related facts in 8; protected contributor, quality and label facts in 9; cursor scope and protected omissions in the delivery checks | **cross-cutting**: the failure-boundary column of every row, plus the closing paragraph |
| 5.8 Cross-environment and DDIL | 6, synchronization and restore | Exchange and restore; Stream and recover |
| 5.9 Validation, conformance and verification | §8.1 layers and §8.3 regressions carry this; §8.2 scenario 1 starts "a separate client" | The closing Roadmap-handoff paragraph |

**Every Goal §5 capability area is reachable from both a representative scenario and a walkthrough path.** Two areas, 5.7 and 5.9, have no dedicated row and are cross-cutting by explicit design. §8.4 states the choice: "Across these paths, provenance/quality disclosure follows §4.10 and the same write/query boundaries; it is not another server subsystem." That is a defensible reading of Goal §5.7's own requirement that security "not be treated as deployment afterthoughts," and treating it as a condition on every path is stronger than isolating it in one scenario. It is recorded as a design choice, not a gap.

Multi-principal coverage exists and is not merely notional. The fixture dataset at Guide line 944 requires "two permitted/denied source groups," and scenarios 8 and 9 both end by changing protected facts while requiring the permitted view, membership, counts and errors to stay unchanged.

---

## 3. No scenario over-promises

Each consequential assertion in §8.2 was checked against the detailed design section that has to deliver it. All are supported:

| Scenario assertion | Supporting design |
|---|---|
| "A cursor observed after the backup must fail in the new recovery epoch instead of skipping new events" (6) | 531, the recovery epoch, with old tokens requiring a fresh snapshot even when numerically ahead |
| "resubmitting a post-backup command's key must not silently redispatch an effect whose receipt was lost" (6) | 531 restored pending work held and not resubmitted; 505 idempotency-key scope and replay contract |
| "last-known state remains timestamped and does not become false current availability" (3) | 481 current status selected by meaningful time; 483 separate status axes |
| "Assert no current fallback" for historical geometry (8) | 975 the same rule; Goal §5.3, missing historical evidence is not replaced with present-day location |
| "A transformed payload must not inherit an unverified signature claim" (9) | 610 preserve supplied labels and binding evidence without claiming a transformed representation retains the original signature's validity |
| "neither data nor publication changed" on invalid input (2) | 511 a write rejected before durable admission leaves no canonical resource or outgoing event behind; 539 the outbox seam |
| "Feasibility is not actuation" (§8.4 row 5) | 843-851 admission; 588 successful feasibility can answer NO without actuation |

This is worth stating plainly because the opposite is a common defect in scenario sets: a walkthrough that asserts a guarantee the design never establishes. None was found.

---

## 4. Four findings gain a named remedy location

The scenario pass does not produce a new finding. It does something more useful for four existing ones: it identifies the exact scenario where each recorded remedy would sit, and confirms at a third independent place in the Guide that it is not there yet.

**F-03, the response-cache instance.** §8.2 contains no occurrence of `cache`, `ETag`, `conditional`, `If-None-Match`, `Vary` or `Cache-Control`. That is not new in itself: F-03 already records that the Guide "nowhere ... lists a cache channel among its disclosure-leak tests (lines 1019, 983)," and line 983 is inside §8.2. What the scenario pass adds is the positive half. Scenarios 8 and 9 each **end** with an indirect-disclosure clause, "membership, counts and errors must not reveal those facts" and "must not alter the permitted response or reveal them indirectly." Those two clauses are the natural home for a cache and validator clause, and they enumerate the channels the project decided to test. The Guide does test preconditions, but at line 511 under §4.6 writes, for concurrency rather than for binding a validator to the authorized view. So the remedy has a location: extend the existing indirect-disclosure clause in scenarios 8 and 9, rather than add a scenario.

**F-19, the audit modification and retention boundary.** `audit` appears nowhere in §8.2, and once in §8.4, only inside "resource/audit/outbox transaction" in the ingest row, which says audit is *written*, not that its behavior is verified. Scenario 6 restores a backup and verifies "resource meaning, identity, deletion handling, and held command work," and does not mention audit. Guide line 602 sets the audit floor and line 529 requires restore testing, so the design exists; scenario 6 is where the check would go.

**F-18, `live` and admission semantics.** The two occurrences of "live" in §8.2 are both "live delivery" and "live-only MQTT recovery," about transport rather than the `live` field. No scenario exercises `false` against `null`, which is the distinction this finding records. Scenario 4 is the natural home.

**F-11, relation spelling.** Scenario 1 requires following links from the root and recovering "the same identities and associations in each supported representation," which exercises navigation but not the spelling of the relations themselves. §8.2 contains no `ogc-rel` and no `rel:`. This finding's recorded next consideration is to "document the selected spelling/compatibility rule in existing fixtures," and scenario 1 is that fixture.

None of these changes a disposition. Each converts a general remedy into a specific edit at a named line, which is what a consolidation owner needs.

---

## 5. Guide §9 as supporting material

§9.1's risk register corroborates the boundaries above rather than adding new ones. Its leak row reads: "Resource/query authorization leaks through links, schemas, counts, latest selection, or events | Apply one access model before selection and test indirect disclosures and policy changes." That is the same enumeration as §8.2 and Guide 610, and the cache is absent from it as well, which is the third independent place. Its failure rows back scenarios 5 and 6: "Database and transport/device effects diverge under failure" and "Synchronization overwrites correct local state or resurrects deletions."

§9.2 matters for a different reason. It is the Guide's own record of what remains unproven, and it states the review's standing distinction in the project's words: "Selecting a contract does not establish interoperability or conformance; remaining executable proofs belong to implementation." Its nine rows each name a design disposition and a remaining implementation proof, and two of them correspond to findings this review holds: "Source contradictions" carries the qualification that "unresolved normative contradictions may still prevent an unqualified affected claim," which is the same position F-08 and the §13 rows take, and "Provenance/quality and Part 5" records "no new public lineage or Part 5 contract assumed," which matches the batch 9 and batch 14 accounting.

§8.2 and §8.4 both state their own limit: "These scenarios connect the sections of the guide; they do not replace the complete class tests," and "This records a design review, not executed integration tests." The scenario set is a design artifact, and this assessment is an assessment of that artifact.

---

## 6. Research reuse

No research report was reopened. The research area completed in iteration 32, and its recorded screen results were sufficient for every comparison above: the recovery-epoch and command-reconciliation rules from the IDR-049 screen, the Case 2 discriminator and `cql2-json` selection from the IDR-059 screen, the provenance disclosure rules from the IDR-061 screen, the status and `live` treatment from the IDR-020 screen, and the cache accumulation from the fourteen sources recorded under F-03. **No specifically needed unread section was identified**, so none was opened and none needs recording.

---

## 7. Disposition summary

| Item | Disposition | Where |
|---|---|---|
| Saved references | **Corrected before reading.** §8.2 holds the scenarios, §8.4 the walkthrough, §9 the risks and checks | §1 |
| Goal §5 coverage | **Complete.** All nine capability areas reachable from both a scenario and a walkthrough path | §2 |
| Security as cross-cutting | A stated design choice, not a gap; multi-principal coverage exists in the fixtures and in scenarios 8 and 9 | §2 |
| Scenario soundness | **No scenario over-promises.** Every consequential assertion is supported by a named design line | §3 |
| **F-03** | Remedy location identified: the indirect-disclosure clauses closing scenarios 8 and 9. Third independent confirmation that the cache channel is unlisted | §4 |
| **F-19** | Remedy location identified: scenario 6, which restores a backup without checking audit | §4 |
| **F-18** | Remedy location identified: scenario 4; the `live` field distinction is untested | §4 |
| **F-11** | Remedy location identified: scenario 1, the link-navigation fixture | §4 |
| Guide §9 | Corroborates rather than adds; §9.2 states the review's own research-versus-adoption distinction in the project's words | §5 |
| Research reuse | No report reopened; no unread section needed | §6 |
| New findings | None. No new finding number and no new follow-up question | |

**Remaining: 12 batches of 27.** Next selected batch is **batch 16 of 27**, the verification-quality pass, against Guide §8.

---

## 8. Statement of limits

This iteration read Goal §5 in full, Guide §8.2, §8.4, §9.1 and §9.2 in full, and the Guide lines needed to check each scenario assertion against its supporting design. It did not read Guide §8.1 or §8.3 beyond what the coverage matrix required, since batch 16 owns the verification-quality assessment; it did not reopen any research report; it did not open any implementation issue; and it did not begin batch 16.

This assesses a design artifact. Guide §8.2 and §8.4 both state that they record a design review rather than executed verification, and nothing here establishes that any scenario passes or that the implementation exists. The four remedy locations in §4 are observations about where an existing recorded remedy would go; none is a new defect and none changes a disposition.

The channel absences in §4 were established by searching §8.2 and §8.4 for the specific terms and by reading the surrounding clauses, not by inference from the Guide's general silence.

Evidence reports 31 through 37 are preserved unchanged.

No implementation, Goal, Guide, Roadmap, issue or upstream change was made. `review_complete` remains `false`.

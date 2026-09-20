# Pass 3c, iteration 18 — research deep-read remainders, slice 3: IDR-042 unread sections

**Date:** 2026-09-19
**Provider/model:** Claude Code; model identified by the runtime environment as Claude Opus 5 (model ID `claude-opus-5`).
**Batch:** `research-deep-read-remainders`, slice 3, from `current_work.next_batch` at planning commit `814f060f38851ebabb167b60ec08b545389ec9b9` (Pass 3c iteration 17 checkpoint).
**Mode:** read-only review of one research report and the Implementation Guide; review artifacts updated and published; no implementation, Goal/Guide/Roadmap, issue, settings or upstream changes.

## 1. Sources consulted this batch

| Source | Access | Used for |
|---|---|---|
| [IDR-SRV-042](../../../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-042-ddil-informed-server-semantics-report.md) Sections 3-4 (lines 125-204), 6 (251-316), 7 (317-342), 9 (383-416), 10 (417-442), 15 (565-633), 16 (634-650), 19-20 (704-765) | Full read | The committed remainder; Sections 1-2, 5, 8, 11-14 and 17-18 were already read (evidence/15-pass-3c-11.txt) |
| [Glaux Server Implementation Guide v1.3](../../glaux-server-implementation-guide.md) | Targeted search then read at each cited line | Adopted/not-adopted/departure comparison |
| findings.md F-03, F-08, F-15, F-18, F-19, F-20, F-22 and the iteration 17 F-03 cache instance | Read | Identify existing homes; execute the batch's explicit instruction to test the recorded cache gap against this report |

No issue bodies were fetched (optional per the batch's source scope). RFC 9111 was not re-retrieved; the three sections needed were quoted into [22-pass-3c-17-research-deep-read-slice2.md](22-pass-3c-17-research-deep-read-slice2.md) Section 5 during the previous iteration and are reused here.

## 2. The instructed check: does IDR-042 reinforce, qualify or bound the F-03 cache gap?

The recorded batch scope directed this iteration to test the response-cache-validator gap recorded under F-03 in iteration 17 against IDR-042 Sections 9-10, and to extend the existing instance rather than open a new one. The answer is that IDR-042 **reinforces** it, from a second and partly independent direction, and **adds one dimension** the earlier instance did not cover.

**Reinforcement.** IDR-042 §7 closes its resource-family table with a rule the Guide does not state anywhere: "'Cache-safe' means policy-partitioned, integrity/version checked and accompanied by correct validators; it does not mean publicly cacheable. Protected or principal-specific responses use appropriate private/no-store controls." (line 341, tagged **[N/A/P]**). That is the same requirement iteration 17 found absent — policy partitioning plus `private`/`no-store` for principal-specific responses — now asserted by a second accepted report, and partly as a normative-derived rather than purely project-chosen position. IDR-042 §3.1 lists RFC 9111 §§4.2-4.2.4 and 5.1-5.2 as a normative source in its own right (line 136). Two accepted reports therefore state the rule; Guide v1.3 still contains no `Cache-Control`, `private`, `no-store`, shared-cache or cache-partitioning statement, as confirmed again this iteration.

**Added dimension.** IDR-042 raises a second cache concern the iteration 17 instance did not address: the separation of HTTP cache freshness from domain freshness. §4.2's non-collapse rules include "HTTP cache freshness is not domain freshness" (line 197); §3.1's RFC 9111 row records the limitation "Cache freshness is representation freshness only" (line 136); §6.4 specifies that the assessment's `ageSeconds` "is domain-evidence age under the named rule, not HTTP `Age`" (line 313); and §15.1 supplies two dedicated API-layer fixtures — "HTTP cache fresh/domain stale → `Age` and domain assessment remain independent" and "HTTP cache stale/domain fresh → cache revalidation behavior does not rewrite domain time" (lines 577-578).

The Guide establishes the underlying *domain* distinction thoroughly: evidence is selected by meaningful time rather than arrival (lines 147, 481, 970), "absence of fresh evidence is unknown or stale, not proof of a failed device" (481), and "last-known is not current" (348). What it never states is the HTTP-layer expression of that distinction, because it states no HTTP caching rules at all. So this dimension is not a contradiction in the Guide; it is the same single omission — no response-caching rules — showing a second consequence.

**Net effect on the finding.** The F-03 related instance recorded in iteration 17 stands unchanged in substance and gains corroboration plus a second consequence and two ready-made test fixtures. No new finding number, and no qualification or narrowing of the earlier statement was found. The instance text in `findings.md` is extended accordingly.

## 3. Adopted (substance carried into the Guide)

| IDR-042 item | Guide disposition | Reference |
|---|---|---|
| §4.2 Non-collapse rules: reachability is not System status; server health is not source health; `live=true` is not data freshness or command readiness; valid history is not invalid because it arrived late; authenticated is not currently authorized; broker delivery is not commit, acceptance or physical effect | **Adopted across the Guide.** "System status remains separate from whether the server, broker or command channel is reachable" (147); "disconnected operation does not extend expired credentials or manufacture current sensor availability" (175); "Accepted is not executed; last-known is not current; transport success is not proof of a physical effect" (348); the status/stream/command-channel/health separation rule (483) | Guide lines 147, 175, 348, 483, 580 |
| §6.1-6.2 Canonical state vocabulary (valid, current projection, fresh, stale, last-known, delayed, unavailable, unknown, partial) and the rule that no single `status` field may collapse the axes | **Adopted in substance.** "Stale and delayed evidence must not become false current status" (232); "Keep System operational status, stream delivery state, command-channel availability, server health, and authorization separate. Do not invent a universal readiness score or a new mandatory status API." (483) | Guide lines 145, 147, 232, 348, 479, 481, 483, 618, 970, 1001 |
| §6.4 Freshness assessment is computed after authorization and policy filtering, so hidden facts cannot alter visible freshness, counts or watermarks | **Adopted.** The protected-facts test requires that with hidden facts changed and the permitted view unchanged, "membership, counts and errors must not reveal those facts" | Guide lines 975, 1001 |
| §6.4 Freshness rules are named, versioned and evaluated at a stated time | **Adopted in principle.** "Where freshness is assessed, document the configured age/source rule and evaluation time" | Guide line 481 |
| §9.1 `resultTime=latest` selects all Observations tied at the greatest result time, after route scope, authorization and filters; it is not latest arrival, one-per-stream, or fresh | **Adopted, effectively verbatim.** "first apply all other predicates within the endpoint scope, then select the greatest visible result time and retain ties" | Guide line 440 |
| §9.1/§9.3 Accept valid delayed facts into history; apply selection by accepted temporal rules, never reconnect arrival order; no backdating or "now" stamping | **Adopted.** "Preserve delayed samples as history without replacing newer evidence accidentally" (481); "not last-arrival-wins" (473); "Never infer conflict resolution from arrival order, largest UUID, or unsynchronized node clocks" (647) | Guide lines 473, 481, 647, 970 |
| §9.2 On contact loss, change source reachability; never write a synthetic sensor status asserting the represented System is unavailable | **Adopted.** "Configured freshness rules can identify old or missing evidence without inventing a failed-device state" (147); the disconnect test requires last-known state to stay timestamped and "not become false current availability" (970) | Guide lines 147, 481, 970 |
| §9.3 Delayed/replay input table: identical replay deduplicates; same ID with different bytes quarantines as conflict; sequence gap leaves completeness unknown | **Adopted.** "Identical accepted input is a no-op returning the recorded outcome"; "different content at the same revision is a conflict"; unknown dependencies stage or are rejected; no completeness checkpoint advances past unresolved records | Guide lines 635, 637, 641, 511 |
| §10.1 Authorized snapshot bound to a logical watermark and an opaque policy-bound cursor carrying route/scope/filter/profile/policy/position, with reauthorization on reconnect | **Adopted, near-verbatim.** The SSE `id` is "an opaque authenticated-encrypted cursor binding recovery epoch, position, caller, exact scope, policy/mapping version and expiry"; authority is rechecked during delivery | Guide line 547 |
| §10.2 MQTT session state is never the canonical cursor; transport guarantees do not establish recovery completeness | **Adopted, near-verbatim.** "Native MQTT data is live delivery without an application recovery cursor in this version ... not a claim of gap-free MQTT replay. QoS 1 alone does not establish recovery completeness." | Guide line 564 |
| §10.2 Broker disconnect must not cause the server to discard a committed resource change | **Adopted** through the outbox model: changes commit first and publication follows from the committed record | Guide lines 119, 539 |
| §10.3 Expired cursor or compacted range returns a typed resnapshot-required outcome; hidden events advance the cursor without leaking count or ID; reconnect storms are bounded | **Adopted.** Expired or retired recovery context "receives `410` before streaming, with fresh-snapshot guidance"; buffers are bounded and slow consumers disconnected to resume or resnapshot | Guide lines 547, 975 |
| §15.1 Temporary dependency failure yields a safe explicit status rather than a misleading empty success | **Adopted.** "Rate/capacity limit; temporary dependency failure | `429` or `503` as applicable, with safe retry guidance" | Guide line 840 |
| §15.2 DDIL assertions are project contract tests with requirement IDs, not new OGC conformance URIs; experimental profiles stay separately declared and pinned | **Adopted.** Declare a class only when the enabled implementation satisfies its requirements with adequate tests; extensions are "clearly labeled" and the SSE extension is "disabled unless configured" | Guide lines 362, 543, 907 |
| §15.3 Metrics carry bounded low-cardinality labels and never resource, principal or topology identifiers; telemetry is not authoritative audit | **Adopted.** "Avoid resource IDs or arbitrary query strings as metric labels" (667); "committed write/command accountability cannot depend on them" (602) | Guide lines 602, 667 |
| §3.4 Implementation lessons must not be promoted into guarantees the sources never established | **Adopted as review discipline.** Client findings are "evidence of interoperability behavior, not permission to change the standard contract" (991), and the CS-GO observation is labelled "source analysis, not an observed runtime defect" (1129) | Guide lines 991, 1129 |

## 4. Not adopted, as a recorded scope choice

- **`RepresentationAssessmentV1` (§6.4)** is not adopted as a named profile object or sidecar. The Guide adopts its posture instead: keep the axes separate, "do not invent a universal readiness score or a new mandatory status API", and "expose additional assessment metadata only through a documented extension where needed" (line 483), with the freshness rule and evaluation time documented where freshness is assessed (line 481). IDR-042 itself frames the object as an opt-in profile that must not mutate mandatory CSAPI semantics and states that clients which do not opt in rely on standard timestamps and `live` (line 315), so a deployment-level extension decision satisfies the report's own constraint.
- **The Glaux-specific time fields** `receivedAt`, `committedAt`, `cachedAt`/`validatedAt`, `lastContactAt`, `lastSynchronizedAt` and `assessedAt` (§6.3) are not enumerated in the Guide, though their functions appear: commit ordering (539), recovery epoch (531) and documented evaluation time for freshness (481). The standards-defined fields in that table map to Guide lines 440, 754 and 843.
- **The DDIL mode taxonomy and its hysteresis fixture** (§5, already read; §15.1 line 571) are not adopted, so the no-flap requirement has no adopted state machine to apply to. The Guide's corresponding posture is configuration-driven and per-capability rather than a global mode.
- **Numeric thresholds** are deliberately absent from both documents. IDR-042 §3.3 records that no public NATO DDIL profile with numeric freshness, authority or queue thresholds was established and leaves values to deployments; §15.3 states "No numeric pass threshold is invented here." The Guide likewise requires explicit bounded configuration without inventing service levels (649, 989).

None of these is a departure. IDR-042 §3.2 separates its own standards-derived facts from Glaux decisions explicitly, listing the mode names, `RepresentationAssessmentV1`, offline authorization classes, queued-write policy, cursor/log retention and source-connectivity assessments as project decisions rather than standards requirements, and §19 line 719 states the report "does not authorize IDR-SRV-043, implementation, production numeric thresholds, deployment topology, an operational policy/credential package, real command effects, or an OGC Part 3 conformance claim."

## 5. Consistent by silence, recorded so a successor does not re-raise them

- **206 Partial Content (§15.1 line 588).** IDR-042 requires a semantically partial query to return `200` with an advertised assessment and never `206`. The Guide never mentions `206` anywhere, so there is nothing to correct; the prohibition is satisfied by the Guide's status-code table not offering it (834-840).
- **Empty result versus source loss (§9.1 line 396, §15.1 line 589).** The rule that an empty authorized set must mean truly empty rather than unknown source completeness applies to architectures that read through to a live source. Glaux serves its own committed PostgreSQL store, so a read has no per-request source dependency; the analogous completeness obligation appears in the exchange path, where a checkpoint may not advance past unresolved records and a size limit "must cause an explicit incomplete/rejected export, never a falsely complete snapshot" (Guide 641, 649).

## 6. Findings register changes

No new finding number. One existing related instance was extended:

- **F-03 (Minimal authorization semantics), iteration 17 cache instance:** extended with IDR-042's corroboration (§7 line 341's policy-partitioned and `private`/`no-store` rule, tagged **[N/A/P]**, with RFC 9111 §§4.2-4.2.4 and 5.1-5.2 cited normatively at §3.1 line 136) and with the added cache-freshness-versus-domain-freshness dimension and its two ready-made fixtures (§15.1 lines 577-578).

F-08, F-15, F-18, F-19, F-20 and F-22 were each checked against this remainder and need no change. The §9.2 `live` and availability content sits in F-18's territory and is consistent with the Guide text that finding already discusses; the §15.3 telemetry content duplicates the F-20 and `fq-09` material already recorded.

## 7. Follow-up questions

No new follow-up question. `fq-09` (metrics/traces exposure) gains further supporting research: §15.3 line 624 requires bounded mode/dependency-class/outcome metric labels and states that "Telemetry is not public source/System status and is not authoritative audit," which the Guide adopts for label cardinality (667) and audit independence (602) while still not stating the exposure boundary for the metrics endpoint itself. That question's evidence list is extended.

## 8. Disposition summary for the checkpoint

| Item | Disposition | Where verified |
|---|---|---|
| IDR-042 Sections 3-4, 6, 7, 9, 10, 15, 16, 19-20 | Fully read; report now fully read | Sections 2-5 above |
| Instructed cache-gap check against Sections 9-10 | **Reinforced, not qualified or bounded.** A second report states the missing rule, and a second consequence (cache versus domain freshness) is added, with two fixtures | Section 2 above |
| IDR-042 freshness, latest, status, cursor, reconnect and transport semantics | Substantially adopted, several passages near-verbatim | Section 3 above |
| `RepresentationAssessmentV1`, Glaux time fields, DDIL mode taxonomy, numeric thresholds | Not adopted; recorded scope choice consistent with the report's own §3.2 and §19 | Section 4 above |
| `206` prohibition; empty-versus-source-loss rule | Consistent by silence or not applicable to the selected architecture | Section 5 above |

**Research-deep-read-remainders, updated remaining scope:** IDR-043's unread sections (Sections 1-5, 8-14, 16-18 and 21-22, plus completion of Sections 6, 7 and 11 beyond the rows already read; Sections 15 and 19-20 are already read per evidence/15-pass-3c-11.txt; roughly 625 lines), and the four committed deep reads not yet begun (IDR-030, IDR-034, IDR-039A, IDR-055). The next selected slice is **IDR-043's unread sections**, which closes the partial-report remainders and leaves only those four reports.

## 9. Statement of limits

This iteration read exactly the committed remainder sections of one report and the Guide text needed to assess them. It did not reread IDR-042's already-read sections, fetch issue bodies, re-retrieve RFC 9111, or resolve `fq-10`. IDR-042 §10.1 treats the IDR-SRV-035 streaming contract as controlling and §14 hands mandatory inputs to IDR-SRV-043; neither was reopened here, and IDR-043's own remainder is the next slice. The cache gap discussed in Section 2 remains a documentation and test gap in planning material, not a report of observed runtime behavior, since no implementation exists. `review_complete` remains `false`.

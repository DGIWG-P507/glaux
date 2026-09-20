# Pass 3c iteration 30 - queue batch 12, platform, streaming, architecture, deployment and configuration

**Date:** September 20, 2026
**Provider:** Claude Code
**Model:** Claude Opus 5 (model ID `claude-opus-5`), per the active-model identification supplied by the environment at the time of this iteration
**Authorization:** One user `proceed`, executing queue batch 12 as saved.
**Comparison target:** `glaux-server-implementation-guide.md` v1.3, 1242 lines.

---

## 1. Coverage comparison

Built from the section headings before reading, as the iteration 28 stop condition requires.

| Report | Recommendations | Risks, constraints, open questions | Decision register | Not read |
|---|---|---|---|---|
| IDR-035 streaming and event publication | §18, R-035-01 to R-035-12, and §18.1 planning estimate | §19.1, §19.2 | §16.1 Option analysis, §16.2 Final decision, §16.3 Exact deviations | §§1-15, 17, 20-21, Appendices; executive summary; §20 validation |
| IDR-044 Rust language and framework | §19 Recommended First-Implementation Stack, §19.1, §19.2 | §22.1, §22.2 | §4.1 Decision rules, §4.2 Recommendation statuses | §§1-3, 5-18, 20-21, 23; executive summary; §23 validation. §22.2 was already read (evidence/04-pass-3b.txt) and was **reused, then re-read in context** because §22.1 adjoins it |
| IDR-045 service architecture | §18, 12 items | §19.1, §19.2 | §4.2 Hard-to-reverse decisions | §§1-3, 5-17, 20+; executive summary; validation |
| IDR-046 reference deployment | §19, 14 items | §20.1, §20.2, §20.3 | §2.4 Prior-decision invariants, §5.2 First implementation decision, §8.1 Orchestration decision | §§1, 3-18, 21+; executive summary; validation |
| IDR-047 configuration and secrets | §18, 12 items | §19.1, §19.2, §19.3 | §4.1 Decision tests | §§1-3, 5-17, 20+; executive summary; validation |
| IDR-048 observability | §18, 12 items | §19.1, §19.2, §19.3 | Evidence and Decision Legend §22 | §§1-14, 16-17, 20+; executive summary; validation |

Every section the scope assigns was read. None of the six is recorded as fully read. IDR-044 moves from `partial` to `key_sections_screened`, which leaves **no partial research report outstanding**.

---

## 2. Acceptance, and F-17 grows again

All six are Complete and Accepted in the governance record, with dated acceptance headers: IDR-035, IDR-044 and IDR-045 on September 15, 2026; IDR-046, IDR-047 and IDR-048 on September 16, 2026.

**Three new F-17 instances**, in addition to IDR-035 which iteration 29 already listed:

- **IDR-044** line 705: "The deliverable remains **In Review** until project-lead acceptance," with line 773's checklist item "IDR-SRV-045 and implementation remain unauthorized pending acceptance."
- **IDR-045** line 635: "IDR-SRV-046 and all later topics remain unauthorized pending acceptance of this report," with the same checklist pattern at line 769.
- **IDR-046** line 905: the closing checklist item "IDR-SRV-047, deployment implementation and operational/accreditation claims remain unauthorized pending acceptance."

**IDR-047 and IDR-048 are clean**, and they show the correct practice: both end with the checklist item "Accepted by Glaux Project Lead."

### 2.1 The iteration 29 lower bound was real, and the scan is now converged

Iteration 29 recorded eleven instances and stated explicitly that the number was a lower bound because the search matched three phrasings. Batch 12 proved that immediately: IDR-044, IDR-045 and IDR-046 use wordings the earlier pattern did not match. Rather than discover more each batch, the scan was re-run over all 71 reports with every variant now known, including the IDR-043 and IDR-055 forms that the narrow pattern missed.

| | Count |
|---|---|
| Accepted reports scanned | 67 |
| **Carry stale acceptance wording** | **16** |
| Carry a correct closing acceptance statement | at least 9 |
| Make no closing status claim at all | the remainder |

The sixteen, with the lines carrying the wording: IDR-006 (968), IDR-010A (1100-1108), IDR-014E (681), IDR-028 (526), IDR-029 (534), IDR-030 (772), IDR-032 (495), IDR-033 (539), IDR-035 (606), IDR-036 (649), IDR-043 (704, 765), IDR-044 (705, 773), IDR-045 (635, 769), IDR-046 (905), IDR-055 (436), IDR-056 (585).

Five of those are new to the list since iteration 29: IDR-029, IDR-044, IDR-045, IDR-046 and IDR-056. The scan now covers every phrasing observed across three separate iterations, and the batch-12 reports produced no further variant, so the pattern has converged. The count remains a lower bound in principle; it is no longer one that has moved under a different wording.

### 2.2 The fix already exists in the corpus, with a model sentence

Two forms of correct practice were found. Seven reports end with a checklist item, "Accepted by Glaux Project Lead." **IDR-026 does better and supplies a sentence the project can copy**, as its final line:

> **Acceptance record:** Accepted by the Glaux Project Lead on September 14, 2026. IDR-SRV-027 was authorized as the next bounded single-topic iteration; no later topic, draft Part 3 implementation, or server implementation was authorized.

That is exactly what the sixteen stale closings were trying to say before acceptance, written for after it. It carries the date, the authorization that followed, and the limits of that authorization. The remedy for F-17 is therefore fully specified: replace the closing paragraph in sixteen named reports with this shape. No new convention, no judgement call, and no technical change.

### 2.3 Disposition unchanged

Still optional editorial cleanup at Low severity. No technical defect follows, and the dated acceptance headers remain the controlling facts in every one of the sixteen.

---

## 3. IDR-035 is the most completely adopted report in the review

The Guide does not merely follow this report's direction. It carries its specific artifacts, including every one of its five listed profile deviations.

| IDR-035 | Guide |
|---|---|
| Decision P-035-01, the experimental profile named `glaux-csapi-part3-exp/0.1` | 549, the same name |
| Based on official commit `6f529a15bfa63259febc3620378d3e5a06305333` | 268, the same commit |
| Deviation 1, Glaux supplies the missing MQTT binding and AsyncAPI discovery locally | 268 and 560 |
| Deviation 2, lowercase `parentid` for CloudEvents compliance; the draft's mixed-case `parentId` is not emitted | 558 "Include authorized `parentid` where applicable, correcting the draft's uppercase spelling"; §13 row at 1157 names it as a listed deviation |
| Deviation 3, retain the `org.ogc.api.consys` event namespace for peer compatibility; any change requires a profile version, never a silent alias | 558 emits `org.ogc.api.consys.<token>.<operation>` |
| Deviation 4, Resource Event `data` omitted by default | 558 "omit optional `data` summaries in this binding" |
| Deviation 5, one versioned topic family replacing incompatible peer layouts | 549-553, one topic family under the profile name |
| "SSE is a separate Glaux change-feed adapter, not falsely labelled an OGC Part 3 binding" | 123 and 547, "SSE is a Glaux extension, not the Part 3 binding" |
| R-035-04, publish no exactly-once claim; state atomicity, handoff, acknowledgement, replay and duplicates separately | 119 "not an exactly-once processing guarantee"; 568 "Do not claim global device-event order or exactly-once delivery" |
| R-035-01, transport-neutral publication core with atomic outbox first | 119, 539, 541 |
| R-035-05, generate and verify AsyncAPI 3.0 from the capability registry | 560, generated from enabled channels with drift tests |
| R-035-07, keep inbound MQTT off until tasking and security acceptance | 549, outbound only |
| Risk control "retain false, bounded expiry" | 549, "QoS 1, non-retained messages" |

Recorded because it is the clearest counterexample in the review to the idea that the Guide treats accepted research loosely. Where the research supplies a concrete, testable artifact, the Guide carries it verbatim, including a pinned upstream commit hash and a deliberate lowercase spelling correction.

One open question from this report bears on **F-11** and reinforces the iteration 28 correction: §19.2 asks "What exact external route/link relation represents the Glaux change-feed and snapshot contract? IDR-SRV-045/050." That is a second accepted report routing a Glaux relation-vocabulary question downstream rather than fixing it, which is why the Guide's choice of a URN at line 560 was recorded as a choice inside an open question.

---

## 4. Adoption accounting for the platform reports

| Recommendation | Guide | Disposition |
|---|---|---|
| IDR-047 recs 1, 5 and 10: one versioned typed configuration schema; reject unknown fields and environment keys; emit a canonical redacted effective manifest | 663 "Use typed configuration with unknown-key rejection and startup validation... Redact effective configuration diagnostics." | Adopted, three recommendations in one line |
| IDR-047 rec 5 second half, reject every registered unsafe combination before effects | 663 "Reject unsafe combinations such as public listeners with development authentication." | Adopted |
| IDR-047 rec 7, separate secret references from values, mounted files first | 663 read secrets from protected files, environment references or deployment secret providers, not checked-in examples | Adopted |
| IDR-048 recs 7 and 8: separate liveness, readiness and protected dependency detail; do not include external dependencies in liveness | 665 liveness reports whether the process can respond, readiness whether required storage, schema and configuration permit serving; optional broker failure degrades publication and is reported "through protected diagnostics, not false global readiness" | Adopted, near-verbatim |
| IDR-048 recs 4 and 6: bounded label registry; prohibit raw paths, queries and unbounded IDs | 667 bounded-cardinality metrics; "Avoid resource IDs or arbitrary query strings as metric labels" | Adopted, near-verbatim |
| IDR-048 rec 11, keep backends optional and vendor-neutral | 667 "an external dashboard or telemetry backend is optional" | Adopted |
| IDR-048 rec 1 and IDR-046 risk row, telemetry must not substitute for audit | 602 "Diagnostic logs may be sampled; committed write/command accountability cannot depend on them." | Adopted, near-verbatim |
| IDR-046 rec 6, treat public origin and proxy trust as API correctness; trust only named proxies | 364 build absolute links from a configured public API root; trust forwarded origin headers only from configured reverse proxies | Adopted, near-verbatim |
| IDR-046 rec 5, run migrations explicitly before readiness; never use first-volume initialization as migration | 529 immutable SQL migration files run by an explicit administrative command; startup checks compatibility rather than applying destructive upgrades silently | Adopted |
| IDR-046 rec 13, prove restore, not just backup creation | 529 "Test backup and restore into a separate, isolated database, including artifacts, schema bindings, IDs, tombstones, and pending work." | Adopted; this is the Guide text the **F-19** restore instance already concerns |
| IDR-046 recs 1 and 4, Compose as the reference environment rather than the production orchestrator; base stack broker-free | 213 and 657, a Compose example for the local reference deployment with an explicit optional service group for the experimental broker, and "A message broker is required only for the MQTT adapter, not for ordinary HTTP use" | Adopted |
| IDR-046 rec 8 and IDR-044's dependency policy, pin digests and run dependency and security checks | 669 "Pin build dependencies and container images, document licenses, and run dependency/security checks as normal implementation maintenance" | Adopted generically; the Guide names no specific tool |
| IDR-044's selected stack, Rust with Tokio, Axum, Serde, SQLx Postgres, PostGIS, tracing, rumqttc | 213, 517, 560 and §2.4 | Adopted in substance |

**Non-adoptions, all stated by the Guide for itself.** IDR-045 recommends a bounded initial package set and Guide line 1118 declines an "eight-package skeleton" by name, keeping instead "a small Rust workspace separating resource rules, standards/encoding logic and the running server" at line 213. IDR-047 rec 2 names Figment as the candidate configuration crate while IDR-044 names `config` plus `clap` and notes "simple loader may suffice"; the two accepted reports differ, and the Guide names neither, which leaves the choice to the implementation rather than settling a research disagreement in a design document. IDR-047 rec 3's eleven named runtime profiles do not appear in the Guide, which configures profiles implicitly through typed configuration at 663. None of these is a defect.

One minor divergence worth recording without raising it: IDR-044 selects `nextest` in its stack table, while Guide line 962 says "A suite runner such as nextest is optional, not evidence of quality by itself." The Guide takes the weaker position deliberately and says why.

---

## 5. `fq-09` gains a second accepted source

`fq-09` asks whether the Guide should state that the metrics and traces surface is internal-only, as IDR-039 §7.1 classifies it, since Guide §4.12 specifies what to collect without saying who may see it.

**IDR-048 rec 9** states it directly: "Put metrics/readiness/diagnostics on a private management boundary and expose no detailed public health." Its risk register adds the reason: "public aggregate leaks hidden activity | inference side channel | private raw metrics, curated minimum public view, policy/security review." Its §19.3 records the residual decision, "is a shallow public demo monitor endpoint needed? Default: no public health route unless deployment requires it."

The Guide is partly there. Line 665 requires reporting degraded publication "through protected diagnostics," so the protected notion exists. Line 667 then says to "Supply a simple way to inspect" metrics without stating the boundary. So the question is unchanged in substance and now rests on two accepted reports rather than one. It remains open and unscheduled, and it is not raised as a finding, because the Guide's silence concerns an exposure boundary that its own §4.10 authorization rules already govern generally.

---

## 6. `fq-12` holds

Of the six reports in this batch, four have Guide reference markers: R035, R044, R045 and R046. IDR-047 and IDR-048 have none, and both appear on the bounded list of 25 published in iteration 29. The list continues to predict correctly, which is what a bounded list should do. No change.

---

## 7. Disposition summary

| Item | Disposition | Where |
|---|---|---|
| Coverage | Six reports screened; comparison built before reading; none fully read. **IDR-044 leaves `partial`, so no partial research report remains** | §1 |
| Acceptance | All six accepted. **Three new F-17 instances**: IDR-044, IDR-045, IDR-046 | §2 |
| **F-17 rescan** | **16 instances, up from 11.** The iteration 29 lower-bound warning was correct; the scan is now converged across every observed variant | §2.1 |
| **F-17 remedy** | **Fully specified.** IDR-026's closing acceptance record is a model sentence the sixteen reports can adopt | §2.2 |
| F-17 disposition | Unchanged: optional editorial cleanup, Low | §2.3 |
| **IDR-035** | **The most completely adopted report in the review.** All five profile deviations, the profile name and the pinned upstream commit are carried into the Guide | §3 |
| F-11 | Second accepted report routing a Glaux relation question downstream, reinforcing the iteration 28 correction | §3 |
| Adoption | Close. Guide 663 alone matches three IDR-047 recommendations; Guide 665 and 667 match four IDR-048 recommendations | §4 |
| Non-adoptions | Package skeleton declined at Guide 1118; the two reports' configuration-crate disagreement left unsettled by the Guide. Neither a defect | §4 |
| F-19 | IDR-046 rec 13 and IDR-048 rec 1 both land on Guide text this finding already concerns. No change | §4 |
| `fq-09` | Second accepted source. Still open, still unscheduled | §5 |
| `fq-12` | Bounded list predicts this batch correctly. No change | §6 |
| New findings | None. No new finding number and no new follow-up question | |

**Remaining: 15 batches of 27.** Next selected batch is **batch 13 of 27**, the eighth key-section screen.

---

## 8. Statement of limits

This iteration read the decision-usable sections of six research reports, six acceptance rows in the governance record, and the Guide text needed for comparison. One mechanical whole-corpus scan was re-run with the full set of observed residue variants; it reads one line at a time and is complete for what it matches, and §2.1 states what that does and does not establish.

It did not read those reports' executive summaries, bodies, appendices or validation-against-plan sections; did not retrieve any external source; did not verify that any named crate version exists or is current, since the Guide deliberately defers version pinning to implementation; did not open any implementation issue; and did not begin any batch after 12.

The F-17 count published in iteration 29 is superseded here rather than corrected, because iteration 29 stated its own limit and this iteration met it. Evidence reports 31 through 34 are preserved unchanged.

No implementation, Goal, Guide, Roadmap, issue or upstream change was made. `review_complete` remains `false`.

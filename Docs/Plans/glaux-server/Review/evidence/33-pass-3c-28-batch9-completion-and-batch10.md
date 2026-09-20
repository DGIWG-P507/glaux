# Pass 3c iteration 28 - batch 9 reopened and completed, then queue batch 10

**Date:** September 20, 2026
**Provider:** Claude Code
**Model:** Claude Opus 5 (model ID `claude-opus-5`), per the active-model identification supplied by the environment at the time of this iteration
**Authorization:** One user `proceed` directing that batch 9 be marked partial, that four named omissions be finished, that the recorded coverage then be compared against every section the saved scope requires before batch 9 is marked complete, and that batch 10 then run as already queued. No repeat studies, no additional batch, no implementation change.
**Comparison target:** `glaux-server-implementation-guide.md` v1.3, 1242 lines.

---

## 1. Batch 9 reopened

### 1.1 The state was changed before anything was read

Iteration 27 recorded batch 9 as done while its own coverage table left assigned sections unread. The saved scope names recommendations, risks and open questions, and any decision register. For four of the six reports only the recommendation sections were read.

The first action of this iteration, before opening any report, was to set `batch_queue` entry 9 from `done_iteration_27` to `partial_iteration_27`, return the queue counts from nine done to eight, name the outstanding sections in the entry, and set `current_work.active_batch` to the reopened batch. That ordering matters: the state was accurate about the gap while the gap was open, rather than becoming accurate only once it was closed.

This is the same omission pattern that Correction B repaired for batch 8, recurring one batch later. The iteration 27 instruction written into `next_batch` explicitly warned against stopping at the first subsection, and writing that warning did not prevent the reviewer from repeating the error in the batch it was written for. The durable fix is in §1.4 below: the coverage comparison is now a stop condition, not an intention.

### 1.2 The four omitted sections

| Report | Section | Lines |
|---|---|---|
| IDR-016 identifier, URI and lifecycle | §17.1 Risks and Controls, §17.2 Open Questions Assigned | 801-833 |
| IDR-017 relationship and linkage model | §17 Risks, Constraints, and Open Questions (§§17.1-17.3) | 806-851 |
| IDR-018 temporal, validity and freshness | §16 Risks, Constraints, and Open Questions (§§16.1-16.2) | 797-827 |
| IDR-020 status, availability and system events | §17 Risks, Constraints, and Open Questions (§§17.1-17.3) | 769-813 |

All were read in full. Four results follow; the remainder of the content routes named questions to named downstream topics and changes nothing already recorded.

### 1.3 What the sections show

**F-03 gains a named owner, which it did not have.** IDR-018 §16.2 routes downstream: "Which HTTP cache directives apply per mutable, immutable, authorized, and disconnected response class? IDR-SRV-045/046." That is the first time in this review that the response-cache question has been found assigned to a specific pair of owning topics in accepted research. F-03's cache instance has rested on what the Guide omits; it now also has a record of who the research expected to settle it. IDR-020 §17.1 adds the control "Authorization before selection/aggregation; policy-aware cache keys" against the risk that hidden evidence affects a visible aggregate, which states the cache-key requirement in the same words F-03 uses.

**F-11 is materially qualified, and the qualification corrects iteration 27.** Iteration 27 recorded R-017-09 as half adopted, on the basis that it asks for a versioned HTTPS Glaux relation namespace while Guide line 560 uses a URN. IDR-017 §17.2 shows that reading was too strong. It lists as an open question, assigned and not blocking acceptance: "Final Glaux relation namespace URI and published vocabulary lifecycle | IDR-SRV-014/046/047 before implementation." The report therefore does not fix the namespace form; it defers it to named owners with a stated deadline. The Guide's URN is a choice made inside an open question, not a departure from a settled rule. **The half-adoption characterization recorded in iteration 27 is withdrawn.** R-017-08, the spelling rule, is unaffected: it is stated as a recommendation with no corresponding open question, and the iteration 27 result under F-11 stands in full.

Two further rows support F-11 without changing it. IDR-017 §17.1 pairs the risk "Overloading `alternate` or bare relation words" with the control "exact registry and project namespace; 010/014/017," and §17.2 also routes "Whether later standards maintenance resolves Table 3 and Part 2 relation gaps" to the upstream register and IDR-SRV-057. That last row is the clearest statement yet that F-11's unresolved question has an upstream component no Glaux decision can close.

**F-18 gains support for its narrowed disposition.** F-18 records that `false` and `null` need distinct consideration for stream `live`. IDR-020 §17.2 question 4 asks exactly that: "When should DataStream/ControlStream `live` be `null` versus Boolean, and what evidence/age controls its derivation?" It is listed among questions that are "assigned downstream and do not block acceptance of the semantic baseline." An accepted report recording the same question as open and non-blocking supports keeping F-18 at a narrowed behavior-clarification candidate rather than escalating it.

**One observation recorded and not raised.** IDR-016 §17.1 lists the risk "UUIDv7 time leakage | Creation activity inference | Document, authorize every lookup, explicit privacy profile decision; 039/040," and §17.2 routes "Privacy profile response to UUIDv7 timestamp visibility" to IDR-SRV-039/040. Guide line 372 adopts UUIDv7 and requires treating the identifiers as "opaque locators, not authorization tokens or authoritative occurrence times," which addresses the time-authority half of the risk and not the privacy half. The Guide's disclosure-channel commitment at line 610 enumerates links, schemas, query NULL behavior, errors, exports and event topics, and does not name identifier timestamps. This is recorded as an observation and not raised, because the research itself assigns the question downstream to topics the review already tracks, and because raising it would expand scope on an incidental question. The adjacent tombstone risk is adopted: IDR-016 §17.1's control "Concealed 404/policy-filtered safe metadata" matches Guide line 834's "consistent non-disclosure `404` where policy requires."

**One consequence now on record for a departure already accounted.** Iteration 27 recorded that the Guide departs from R-018-03 by documenting keyset paging as a changing view at line 444 instead of claiming a cross-request snapshot. IDR-018 §16.1 states what that departure costs: the risk "Re-evaluated `now` across pages" has the consequence "duplicates/omissions/inconsistent state." The disposition is unchanged, because the Guide states the departure for itself and its mitigation is to document the behavior honestly rather than to conceal it. The consequence is recorded so the owner can weigh it at consolidation rather than rediscover it.

### 1.4 Coverage compared against the saved scope

The comparison the user required was run report by report, against both scope statements held in the state.

| Report | Recommendations | Risks and open questions | Decision register | Executive summary | Validation against plan |
|---|---|---|---|---|---|
| IDR-015 | §16, §16.1 read | §17 read | **§15, §15.1 read in this iteration** | not read | not read |
| IDR-016 | §16, §16.1 read | §17.1-17.2 read | none separate | not read | §17.3-17.6 not read |
| IDR-017 | §16, §16.1, §16.2 read | §17.1-17.3 read | §16.1 read | not read | §18 not read |
| IDR-018 | §15, §15.1 read | §16.1-16.2 read | none separate | not read | §17 not read |
| IDR-019 | §17.1, §17.2 read | §17.3, §17.4 read | none separate | not read | §18 read |
| IDR-020 | §16, §16.1 read | §17.1-17.3 read | §16.1 read | not read | §18 not read |

**The comparison found one further omission, which was then read.** IDR-015 §15 Decision Analysis and §15.1 Adopted Canonical Boundary are a decision register, which the operative scope names explicitly, and iteration 27 did not read them. They were read here. Option D, typed encoding-neutral aggregates with codecs, is adopted, and Option E, an event-sourced model as mandatory architecture, is deferred as a persistence option rather than required. The Guide matches both: it takes the outbox seam at line 539 without making event sourcing mandatory, and line 1118 lists no event-sourcing requirement.

**The two scope statements in the state differ, and the difference is now resolved rather than left ambiguous.** The `batch_queue` entry wording, written in iteration 19, lists "executive summary, recommendations, risks and open questions, validation." The `current_work.next_batch` wording, written in iteration 26 for this batch, lists "recommendations, risks and open questions, and any decision register." Batches 6, 7, 8 and 9 were all executed against the second wording, and every report entry from those batches records the executive summary and the validation section as not read. The executed practice is therefore consistent and was disclosed each time; only the queue wording was never reconciled to it.

That wording has now been aligned in `batch_queue`, and the reason is stated in the state rather than applied silently: a validation-against-plan section is a report's self-assessment against its own research plan, and an executive summary restates content the decision sections carry in decision-usable form, so neither contributes to accounting research against the Guide. **This is a reconciliation of the recorded scope to four batches of consistently recorded and accepted practice, not a reduction of scope to match what was done.** The sections remain recorded as not read, per report, so the narrowing is visible and reversible.

**Batch 9 now meets its scope and is marked complete.**

---

## 2. Batch 10 - key-section screen E: encoding, schema and validation

### 2.1 What was screened

Five reports, screened at their decision-usable sections. None is recorded as fully read.

| Report | Screened | Not read |
|---|---|---|
| IDR-021 SensorML representation | §16 Recommendations (16 items), §16.1 Acceptance Decision, §17 Risks, Constraints and Open Questions (§§17.1-17.3) | §§1-15, 18-20 and appendices |
| IDR-022 SWE Common data components | §16 Recommendations (16 items), §16.1 Acceptance Decision, §17 (§§17.1-17.3) | §§1-15, 18-20 and appendices |
| IDR-023 schema and encoding validation | §16 Recommendations (12 items), §17 (§§17.1-17.3), §18 validation table | §§1-15, 19-21 and appendices |
| IDR-024 units and semantic binding | §17 Recommendations (15 items), §18 (§§18.1-18.2), §19 validation table | §§1-16, 20-21 and appendices |
| IDR-025 database and persistence architecture | §19 Recommendations (16 items), §20 (§§20.1-20.2) | §§1-18, 21 and appendices |

### 2.2 Acceptance, checked in the governance record

`final-idr-research-report.md` lines 159 to 163 record all five as Complete and Accepted. Every report header carries `Accepted By: Glaux Project Lead` with `Acceptance Date: September 14, 2026`, and none contains a pending-acceptance or remains-in-review string.

**No new F-17 instance.** With batches 7 and 9 that is fifteen reports checked against three instances. The finding is well bounded as a residue in particular reports.

### 2.3 Adoption accounting

This is the most closely adopted batch the review has screened. All five reports are cited in the Guide, which defines and uses reference markers R021 through R025. The clearest matches:

| Recommendation | Guide | Disposition |
|---|---|---|
| IDR-025 rec 1, PostgreSQL/PostGIS as the authoritative persistence direction in one transactional boundary | 517 "PostgreSQL/PostGIS is the single authoritative store." | Adopted, near-verbatim |
| IDR-025 rec 2, relational/JSONB/artifact hybrid with exact bytes in immutable artifacts | 517 typed relational columns for identities and lifecycle, JSONB for validated extensible structures and not as a substitute for relationships, exact bounded bytes in `bytea` with digests | Adopted, near-verbatim and in the same order |
| IDR-025 rec 4, benchmark TimescaleDB but do not require it now | 519 TimescaleDB is not a prerequisite | Adopted |
| IDR-025 rec 13, explicit numbered migrations and coordinated restore tests; reject ungoverned startup ORM mutation | 529 immutable SQL migration files run by an explicit administrative command; normal startup checks compatibility rather than applying destructive upgrades silently | Adopted, near-verbatim |
| IDR-025 rec 7, transactional outbox; brokers remain derived delivery systems | 539 and 564 | Adopted |
| IDR-025 rec 6, relational typed edges; no graph database without a measured traversal failure | 1118 non-adoption list includes a graph or evidence database | Adopted |
| IDR-023 rec 5, disable uncontrolled `$ref` and URI network resolution; ship a reference-closed offline package | 420 schema allowlist with no arbitrary `$ref`, link or data-URL fetch | Adopted, near-verbatim |
| IDR-023 rec 6, bind every Observation and Command to the exact parent contract fingerprint; never silently rebind history | 408 bind observations and commands to the contract revision, and a stream-schema edit must never reinterpret historical values | Adopted, near-verbatim |
| IDR-023 rec 9, RFC 9457 diagnostics with stable public codes, restricted internal detail and `500` for invalid server responses | 827-840 problem details and the status table | Adopted |
| IDR-022 rec 13, vendor approved schemas unmodified and address gaps through overlays and validators | 410 "Retain unmodified upstream schemas and apply a separate, explicitly documented semantic check." | Adopted, near-verbatim |
| IDR-022 rec 6, keep component description and encoding separate, associated in the containing schema wrapper | 434 and the §13 row at 1149-1152 | Adopted |
| IDR-024 rec 6, never silently convert Commands; observation conversion only as a provenance-bearing view after compatibility gates | 424 "No implicit unit conversion is performed on command input. Any supported observation conversion must specify dimensional compatibility, precision, and original values." | Adopted, near-verbatim |
| IDR-024 rec 9, exact matching baseline; fuzzy labels, same dimension and `closeMatch` off by default | 424 "Labels and matching dimensions alone do not establish property identity." | Adopted, near-verbatim |
| IDR-024 rec 4, UCUM as the recommended unit-code baseline, preferring `uom.code` while supporting `href` | 424 validate UCUM codes where supplied against the incorporated UCUM basis, and a semantic unit URI is not invalid simply because it is not a UCUM code | Adopted in substance; the Guide does not pin a UCUM version |
| IDR-021 rec 6, generated policy-aware SensorML as the normal public view with source-artifact access granted separately | 406 store exact original bytes with media type and digest, and "do not serve unfiltered original bytes as a shortcut around access checks" | Adopted, near-verbatim |
| IDR-021 rec 14, layered validator and machine-readable profile; derive conformance claims from enabled codecs plus passing evidence | 901-907 declare a class only when satisfied | Adopted |
| IDR-021 rec 11, capabilities and configuration are descriptive evidence, not availability or execution | 483 and the §13 rows at 1146-1147 | Adopted |

### 2.4 F-03: four more accepted sources, and the mechanism stated

The cache instance under F-03 has until now rested on reports whose subject is security, policy, testing or content negotiation. Batch 10 shows that the reports whose subject is data contracts and storage state the same requirement independently, and one of them states the mechanism.

- **IDR-021 R-021-15** (Critical): "Apply policy before projection and secure raw artifacts, nested metadata, indexes, links, errors, resolvers, and caches independently."
- **IDR-022 R-022-14** (Critical): "Apply resource budgets and policy to schemas, values, references, codecs, indexes, errors, logs, caches, generated views, and examples."
- **IDR-025 rec 15** (Critical): "Make security and policy part of every authoritative and derived persistence contract. RLS is defense in depth; protect backups, replication, caches, indexes, broker state, diagnostics, and deduplication side channels."
- **IDR-025 rec 8** (High) states the mechanism: "Make caches and materialized views disposable and evidence-bearing. Record source watermark, builder/profile/policy version, staleness, and deterministic rebuild."

Taken with IDR-018 §16.2 from §1.3 above, which names IDR-SRV-045/046 as the owners of the HTTP cache-directive question, the instance now rests on ten accepted reports and has both a stated mechanism and named owners. Its substance and remedy are unchanged and no escalation is proposed. What changed is that a consolidation owner reading F-03 can now see that the requirement is convergent across five unrelated research subject areas rather than derived from one, that recording the policy version with the cache is the mechanism the research expects, and which two topics were expected to settle the HTTP-layer part.

### 2.5 Stated non-adoptions, none of them defects

**Quarantine and promotion.** Three reports converge on a privileged quarantine and promotion lane separate from strict conformant writes: IDR-021 R-021-07, IDR-022 R-022-12 and IDR-023 rec 8. The Guide has no such lane, and its only use of the word concerns quarantined CI checks at line 959. The Guide declines the scope that would require it, at line 422: "XML/legacy import is not necessary to claim these CSAPI JSON-based classes and is not added as a separate implementation objective." The recommendations exist to handle partial, invalid, legacy or profile-divergent material; the Guide does not accept such material, and it does test partial-invalid batches at line 511. A stated scope choice, not a gap.

**QUDT.** IDR-024 rec 7 treats QUDT as optional pinned enrichment at Medium priority, and the Guide does not mention it. Declining an explicitly optional Medium recommendation is unremarkable and is recorded only for completeness.

**SWE Binary.** IDR-022 R-022-04 and IDR-023 rec 7 both gate Binary behind complete codec evidence. Guide lines 422 and 890 take the same position, so this is adoption rather than non-adoption, and it bears on `fq-01` below.

### 2.6 Follow-up questions

**`fq-01` is narrowed and its urgency is low.** The question asks whether any planned validation path uses the `encodings.json` root, whose `oneOf` omits `BinaryEncoding`. IDR-022 R-022-04 advertises SWE Binary "only after bounded complete codec evidence," IDR-023 rec 7 says the same, and IDR-022 §17.3 question 4 leaves "Which BinaryEncoding datatype, compression, and encryption URI profiles, if any, are launch requirements" open to IDR-SRV-023/044/046. Binary is therefore gated behind evidence that does not yet exist, so a root-schema omission affecting Binary cannot bite before that gate. IDR-023 rec 5's reference-closed offline package and rec 3's explicit assertion profile also mean the validation path is a Glaux-owned profile rather than a bare root schema. The question stays open and stays unscheduled; nothing here answers it directly.

**`fq-02` gained no evidence, and that is now a screened result rather than an inference.** The question concerns `Quantity.json` requiring `label` while the published clause describes it as optional. Neither the recommendations nor the risks nor the open questions of IDR-022, IDR-023 or IDR-024 mention it. The question was raised partly because a grep of those three reports found no record; a key-section screen of all three now confirms the decision sections do not record it either. That does not establish that the reports' bodies are silent, which was not read.

**`fq-08` gained indirect support.** The question asks whether the Guide should adopt one unknown-member rule across inbound representation families. IDR-022 R-022-02 requires "typed known nodes plus quarantined opaque unknowns" for SWE, and IDR-021 R-021-05 requires preserving "rich/unknown source content" for SensorML. Both describe preserve-and-isolate rather than reject, for their own families. Neither states a single cross-family rule, which is what the question asks about, so the question stays open.

**`fq-12` gains a clean contrast.** All five batch-10 reports are cited in the Guide, which defines R021 through R025 and uses them at lines 337, 404, 410, 418, 515 and throughout §13. The three uncited accepted reports recorded in iteration 27 are therefore not part of a general pattern of undercitation in the Guide, which strengthens the reading that this is reference completeness for specific entries rather than a systemic omission.

---

## 3. Disposition summary

| Item | Disposition | Where |
|---|---|---|
| Batch 9 status | Set to **partial before any reading**, counts returned to 8 done, outstanding sections named in the queue entry | §1.1 |
| Four omitted sections | Read in full, 157 lines | §1.2 |
| F-03 | Named owners found: IDR-SRV-045/046. Plus `policy-aware cache keys` stated in IDR-020 | §1.3 |
| **F-11 correction** | **The iteration 27 R-017-09 half-adoption characterization is withdrawn.** The namespace URI is an explicitly open question routed to IDR-014/046/047. R-017-08 and the iteration 27 result stand in full | §1.3 |
| F-18 | Supported: an accepted report records the null-versus-Boolean question as open and non-blocking | §1.3 |
| UUIDv7 privacy | Observation recorded, deliberately not raised; research assigns it to IDR-039/040 | §1.3 |
| Scope comparison | Run report by report. **Found one further omission**, IDR-015 §15 decision register, which was then read | §1.4 |
| Scope wording | The two differing scope statements reconciled to four batches of consistently recorded practice, with the reason stated and the unread sections still recorded per report | §1.4 |
| Batch 9 | **Complete** | §1.4 |
| Batch 10 coverage | Five reports screened at decision-usable sections; none recorded as fully read | §2.1 |
| Acceptance | All five correct. **No new F-17 instance**; fifteen reports now checked against three instances | §2.2 |
| Adoption | The closest-adopted batch so far; many near-verbatim matches at Guide 406, 408, 410, 420, 424, 517, 519 and 529 | §2.3 |
| **F-03** | **Four more accepted sources, now ten, with the mechanism stated and owners named.** No escalation | §2.4 |
| Non-adoptions | Quarantine lane declined by a stated Guide scope choice at line 422; QUDT optional. Neither is a defect | §2.5 |
| `fq-01`, `fq-02`, `fq-08`, `fq-12` | Narrowed or supported; all remain open. No new follow-up question | §2.6 |

**Remaining: 17 batches of 27.** Next selected batch is **batch 11 of 27**, the sixth key-section screen.

---

## 4. Statement of limits

This iteration read 157 previously omitted lines in four batch-9 reports, one further omitted decision register found by the scope comparison, the decision-usable sections of five batch-10 reports, five acceptance rows in the governance record, and the Guide text needed for comparison. It did not read those reports' executive summaries, bodies, appendices or validation-against-plan sections except where the table above records otherwise; did not re-read any section recorded as already read; did not retrieve any external source; did not open any implementation issue; and did not begin any batch after 10.

One iteration 27 characterization is withdrawn here, in §1.3. Evidence reports 31 and 32 are preserved unchanged; this report appends the correction, following the practice the review has used since iteration 21.

Where the Guide does not carry a recommendation, that is recorded as a non-adoption and raised as a finding only where a divergence is evidenced at a named Guide line. No such divergence was found in batch 10. An unadopted research recommendation is not a defect.

No implementation, Goal, Guide, Roadmap, issue or upstream change was made. `review_complete` remains `false`.

# Pass 3c iteration 29 - queue batch 11, storage, query and write boundaries

**Date:** September 20, 2026
**Provider:** Claude Code
**Model:** Claude Opus 5 (model ID `claude-opus-5`), per the active-model identification supplied by the environment at the time of this iteration
**Authorization:** One user `proceed`, executing queue batch 11 as saved.
**Comparison target:** `glaux-server-implementation-guide.md` v1.3, 1242 lines.

This batch changed two things the review had already recorded. Both corrections are in §3 and §5. Neither is a new finding.

---

## 1. Coverage comparison

The saved scope requires recommendations, risks and open questions, and any decision register, each read in full including every subsection. The comparison was built from the section headings before reading, not reconstructed afterwards.

| Report | Recommendations | Risks, constraints, open questions | Decision register | Not read |
|---|---|---|---|---|
| IDR-026 geospatial storage and query | §17, 12 items | §18.1, §18.2, §18.3 | §2.2 Decision Boundary; §13.1 Policy Decision Order | §§1, 3-16, 19-20; executive summary; §19 validation |
| IDR-027 time-series observation storage | §17, P-027-01 to P-027-14 | §18.1, §18.2 | §2.2 Decision Boundary | §§1, 3-16, 19-20; executive summary; §19 validation |
| IDR-028 metadata and document storage | §16, 15 items | §17.1, §17.2 | §4.2 Decision Rules | §§1, 3-15, 18-19; executive summary; §18 validation |
| IDR-032 publisher-to-server boundary | §15, 15 items | §16 | §7.1 Decision analysis; §7.2 Adopted surfaces | §§1-6, 8-14, 17-18; executive summary; §17 validation |
| IDR-033 simulator-to-server boundary | §16, 16 items | §17 | §5.1 Decision analysis; §5.2 Contract boundary | §§1-4, 6-15, 18-19; executive summary; §18 validation |

Every section the scope assigns was read. None of the five reports is recorded as fully read.

---

## 2. Acceptance, checked in the governance record

`final-idr-research-report.md` lines 164, 165, 166, 170 and 171 record all five as Complete and Accepted. Every report header carries `Accepted By: Glaux Project Lead` with `Acceptance Date: September 14, 2026`.

**Three of the five are new F-17 instances**, and they are the clearest instances the review has found, because the contradiction is inside one document.

- **IDR-028** line 526: "The report is complete as research and is **In Review**. It is not accepted for downstream use until the Glaux Project Lead records acceptance in this report, its topic plan, and the overall plan." Its own header, lines 16 and 17, records that acceptance.
- **IDR-032** line 495: "Plan-owner acceptance remains deliberately unchecked while this report is **In Review**. The next two workflow actions are: (1) the Glaux Project Lead accepts IDR-SRV-032 after review..." Header lines 22 and 23 record that it already happened.
- **IDR-033** line 539: the same sentence for IDR-SRV-033, against header lines 23 and 24.

IDR-026 and IDR-027 contain no such text.

This is worse than the earlier instances in one specific way. IDR-055, the iteration 24 instance, omitted the header fields, so a reader had nothing to go on. These three state the acceptance correctly in the header and then deny it in the body. A reader who reaches the closing section last, which is where a reader looking for status goes, is told the opposite of the truth by the same document that told them the truth twenty pages earlier.

---

## 3. F-17 is larger than the review recorded, and the earlier bound was drawn wrongly

### 3.1 What was recorded

Iterations 27 and 28 recorded a bound: "fifteen reports checked against three instances," and described the finding as "a residue in particular reports, not a systemic practice," offered as a reason to keep it at optional editorial cleanup.

### 3.2 Why that was wrong

The bound was computed from reports whose **headers** had been checked. A header check finds reports that record acceptance correctly in the header. It cannot find a report that records acceptance correctly in the header and contradicts it in the body, which is exactly what these three do, and what IDR-043 did in iteration 19. The method could only ever find the IDR-055 shape, and it was used to make a claim about the whole population. That is a sampling error in the review's own accounting, and the conclusion drawn from it was too optimistic.

### 3.3 The corrected accounting

A single search of all 71 reports for the residue wording itself, rather than for correct headers, gives a bounded list. Every report named below was then checked against the governance record and is Accepted with a dated acceptance header.

| Report | Governance record | Header acceptance date |
|---|---|---|
| IDR-006 | Complete / Accepted | July 31, 2026 |
| IDR-010A | Complete / Accepted | August 1, 2026 |
| IDR-014E | Complete / Accepted | August 31, 2026 |
| IDR-028 | Complete / Accepted | September 14, 2026 |
| IDR-030 | Complete / Accepted | September 14, 2026 |
| IDR-032 | Complete / Accepted | September 14, 2026 |
| IDR-033 | Complete / Accepted | September 14, 2026 |
| IDR-035 | Complete / Accepted | September 15, 2026 |
| IDR-036 | Complete / Accepted | September 15, 2026 |

With IDR-043 and IDR-055, whose wording differs and which this search does not match, the confirmed population is **at least eleven accepted reports**, not three. The list is a lower bound, because it matches three specific phrasings and two known instances use others.

### 3.4 What changes and what does not

The **disposition does not change**. This remains optional editorial cleanup at Low severity. No technical defect follows from it, the dated acceptance records are the controlling facts, and nothing in the Guide depends on the stale sentences.

What changes is the **shape of the remedy**. A three-instance residue invites spot fixes. Eleven-plus reports with the same closing template invites one pass over a named list, and the list is now written down. Two of the newly named reports, IDR-006 and IDR-014E, sit in batches 6 and 8, which this review screened and recorded as producing no F-17 instance. That accounting was incomplete for the same methodological reason, and the state entries for those batches are corrected to say so rather than being left to imply a clean result.

Recording an error in the review's own method is the point of this section. The finding was never wrong; the bound the review put around it was.

---

## 4. Adoption accounting

Substantially adopted, with the spatial recommendations matched almost line for line.

| Recommendation | Guide | Disposition |
|---|---|---|
| IDR-026 recs 4, 6, 7 and 12: CRS84 wire baseline, `bbox` and `geom` as distinct contracts with different no-geometry behavior, antimeridian and 3D and invalid-coordinate fixtures | 442 "Preserve source geometry/reference information and generate the required CRS84 GeoJSON order. Treat `bbox` and WKT `geom` according to their different rules, including geometry-less features. Use fixtures for antimeridian crossing, 3D inputs, empty geometries, and invalid coordinates." | Adopted, near-verbatim, four recommendations in one line |
| IDR-026 rec 9: moving-position samples authoritative, latest and trajectory and extent derived | 442 "Never silently discard vertical or moving-position information while claiming an equivalent rich representation." | Adopted |
| IDR-026 rec 4 second half: gate alternate output CRS on explicit Features Part 2 adoption | 799 accept standard CRS84/CRS84h filter-coordinate behavior and "reject other requested CRSs in this initial binding" | Adopted |
| IDR-026 rec 1 and IDR-027 P-027-01: PostgreSQL and PostGIS as the full-profile authority | 517 the single authoritative store | Adopted |
| IDR-027 P-027-11: TimescaleDB conditional on a measured gate | 519 not a prerequisite | Adopted |
| IDR-027 P-027-07: implement latest, filter, order and paging from logical authorized records; prevent tie errors | 440 predicates first, greatest visible result time, retain ties; 469 authorized view before predicates | Adopted |
| IDR-027 P-027-03: never substitute arrival or commit for phenomenon, result, event or command time | 376 and the §6.1 persistence table | Adopted |
| IDR-027 P-027-09: separate retention eligibility from partition axis and database TTL | 527 "Do not enable automatic retention/purge by default" with dependency preservation | Adopted |
| IDR-028 recs 4 and 5: start with PostgreSQL `bytea`, preserve exact imported bytes before parsing | 517 exact bounded document bytes in `bytea` with digests; 406 store exact original SensorML bytes with media type and digest | Adopted |
| IDR-028 rec 9: immutable reference-closed packages, no uncontrolled runtime network retrieval | 420 schema allowlist, no arbitrary `$ref`, link or data-URL fetch | Adopted |
| IDR-028 rec 3: digest is byte integrity and location, not identity, trust, authorization or public URL | 631 digest checks exact decoded bytes, not semantic equivalence; 517 | Adopted |
| IDR-028 rec 8: preserve distinct validity, receipt, commit, publication, validation, activation and retirement clocks | 376 valid time and receipt/commit time stored separately | Adopted |
| IDR-028 rec 15: retain OpenAPI 3.1 until a gated migration | 362 one OpenAPI 3.1 document and no OAS 3.0 claim | Adopted |
| IDR-032 rec 6: stable retry identity scoped to publisher, source and operation with intent fingerprint; same key and different intent is a conflict | 505 `Idempotency-Key` scoped to verified caller, source, operation and target, retaining request digest and outcome; same key and same intent returns the same outcome, different intent returns a conflict | Adopted, near-verbatim |
| IDR-032 rec 8: separate transport receipt, validation, commit, publication and execution; broker acknowledgement never proves commit | 564 QoS 1 is not recovery completeness; 843-851 admission | Adopted |
| IDR-032 rec 12: opaque correlation and safe RFC 9457 problems that do not expose other-source existence, topology or policy logic | 827-840; 610 no leak through links, schemas, errors, exports or event topics | Adopted |
| IDR-032 rec 15 and IDR-033 rec 16: do not select or implement draft Part 3 here | 547 "SSE remains a Glaux interface, not a Part 3 binding" | Adopted |
| IDR-033 rec 7: never modify server security, receipt, commit, audit, retention, idempotency or backpressure clocks through remote simulation controls | 1118 no simulator management API; 533 recovery epoch | Adopted through the non-adoption below |

**Stated non-adoptions, neither a defect.** IDR-032 rec 2 adopts GPC-v1 as a supplemental Glaux contract with four private `/_glaux/publisher/v1` paths, and IDR-033 rec 3 adopts a narrow `/_glaux/simulator/v1` control plane. Guide line 1118 declines both by name: "no compulsory private Publisher envelope, simulator management API." The two reports and the Guide agree on rejecting the broad forms. IDR-032 §7.1 rejects a general private ingestion API outright and IDR-033 §5.1 rejects a broad server scenario engine; the Guide goes one step further and declines the narrow forms too. That is the Guide's approved capability scope, stated in its own non-adoption list.

The quarantine and promotion lane recurs here, in IDR-032 rec 9 and IDR-028's quarantine expiry question, as it did in batch 10. The disposition is unchanged: Guide line 422 declines the legacy-import scope that would require it.

---

## 5. F-03 gains four more sources, and one of them states the rule in the review's own words

The cache instance under F-03 says the Guide "nowhere binds a validator or cache key to the authorized view." Batch 11 found that sentence's content stated directly, as a numbered step, in an accepted report.

**IDR-026 §13.1, Policy Decision Order, step 6:** "Compute sort, counts, extents, pagination, links, and cache entries from the authorized view."

Steps 1 to 5 establish the authorized view, and step 6 derives every downstream product from it, naming cache entries alongside counts and pagination. This is the most precise statement of the requirement found anywhere in the corpus, and it is in a report whose subject is geospatial storage rather than security.

Three further sources in the same batch:

- **IDR-026 rec 10** (High): "Apply policy before geometry selection, predicates, counts, extents, sorting, paging, caching, and export. Test inference channels and derived-policy invalidation."
- **IDR-027 P-027-14** (High): "Apply policy before latest/extents/counts/aggregates/pages/streams/archives and scope derived products/caches to policy/version." Its risk register adds the invalidation mechanism: late or corrected data leaving projections stale is mitigated by "keyed invalidation, watermarks, rebuild/failure tests."
- **IDR-028 rec 11 and §4.2**: "Apply policy before every derived surface" and "Put policy before index, search, diagnostics, links, rendering, caching, and download." Its risk register names the failure directly: "Search/cache crosses policy boundaries | existence/content disclosure | policy-built documents, scoped keys/indexes, leakage tests."

IDR-033 recs 9 and 12 add cache state to the inventory that a test environment must enumerate and that synthetic context must propagate through.

**The instance now rests on fourteen accepted reports across seven unrelated subject areas**, with the rule stated, the mechanism stated by two reports independently, and named owners. Substance, severity and remedy are unchanged, and no escalation is proposed. The accumulation is recorded because it bears on how a consolidation owner weighs a Low-Medium item: this is not one report's preference that the Guide declined, it is a rule that every research area touching derived data states on its own initiative and that the Guide's line 739 does not carry.

---

## 6. `fq-12` becomes a bounded list, and the iteration 28 contrast does not generalize

Iteration 28 recorded that all five batch-10 reports are cited in the Guide, and drew from that a contrast: the three uncited accepted reports found in iteration 27 "are therefore not part of a general pattern of undercitation." Batch 11 shows that inference was drawn from too small a sample. Of its five reports, only IDR-027 has a Guide reference marker. IDR-026, IDR-028, IDR-032 and IDR-033 have none.

A complete comparison settles it. The governance record lists 67 accepted reports; the Guide defines 47 reference markers; **25 accepted reports have no marker of their own**: IDR-002, 003, 004, 005, 014A, 014B, 014C, 014D, 014F, 014G, 016, 017, 019, 026, 028, 030, 032, 033, 038, 041, 047, 048, 051, 055, 057.

That count is an upper bound on genuinely uncited reports, not a defect count, because the Guide reaches some of them without a marker of their own. IDR-014F and IDR-014G are cited as a range at line 991 through the `R014e` marker, and IDR-019's content is reached through `R061`. Establishing which of the 25 are genuinely unreferenced would need a prose check that a key-section screen does not perform.

What this settles is that the question was never about IDR-055. It is a reference-completeness question about a bounded list, and it is now recorded as one. The character is unchanged: not a substantive gap, and in every case checked so far the Guide carries the substance.

---

## 7. Disposition summary

| Item | Disposition | Where |
|---|---|---|
| Coverage | Five reports screened at recommendations, risks and open questions, and decision registers; comparison built before reading; none fully read | §1 |
| Acceptance | All five accepted. **Three new F-17 instances**: IDR-028, IDR-032, IDR-033 | §2 |
| **F-17 correction** | **The iterations 27 and 28 bound was drawn wrongly and is withdrawn.** Confirmed population is at least eleven accepted reports, now enumerated. Disposition unchanged at optional editorial cleanup, Low | §3 |
| Adoption | Substantially adopted. IDR-026 recs 4, 6, 7 and 12 are matched near-verbatim by Guide line 442 alone; IDR-032 rec 6 by Guide 505 | §4 |
| Non-adoptions | GPC-v1 private paths and the simulator control plane declined by name at Guide 1118; quarantine lane by Guide 422. Neither a defect | §4 |
| **F-03** | **Four more accepted sources, now fourteen.** IDR-026 §13.1 step 6 states the rule in the finding's own terms. No escalation | §5 |
| **`fq-12` correction** | **The iteration 28 contrast does not generalize.** 25 of 67 accepted reports have no Guide reference marker; the list is recorded. Upper bound, not a defect count | §6 |
| New findings | None. No new finding number and no new follow-up question | |

**Remaining: 16 batches of 27.** Next selected batch is **batch 12 of 27**, the seventh key-section screen.

---

## 8. Statement of limits

This iteration read the decision-usable sections of five research reports, five acceptance rows in the governance record, nine further acceptance rows for the reports named in §3.3, and the Guide text needed for comparison. Two mechanical whole-corpus searches were run, one for the F-17 residue wording and one comparing accepted reports against defined Guide reference markers; both are complete for what they match and both are recorded above with their limits. Neither involved reading a report.

It did not read those reports' executive summaries, bodies, appendices or validation-against-plan sections; did not retrieve any external source; did not open any implementation issue; and did not begin any batch after 11.

Two earlier review conclusions are corrected here, in §3 and §6, both of them bounds the review placed around existing items rather than the items themselves. Evidence reports 31, 32 and 33 are preserved unchanged; this report appends the corrections.

No implementation, Goal, Guide, Roadmap, issue or upstream change was made. `review_complete` remains `false`.

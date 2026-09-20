# Pass 3c, iteration 25 — queue batch 7: key-section screen B, conformance, API definition and documentation

**Date:** 2026-09-20
**Provider/model:** Claude Code; model identified by the runtime environment as Claude Opus 5 (model ID `claude-opus-5`).
**Batch:** batch 7 of 27, key-section screen B over IDR-009, IDR-010A, IDR-012, IDR-013 and IDR-014, plus the unread key sections of IDR-010. Resumed from planning commit `15a346870e2a1926f7d42b39321e34c31941cc56`.
**Mode:** read-only review of six research reports, two governance documents and the Implementation Guide; review artifacts updated and published; no implementation, Goal/Guide/Roadmap, issue, settings or upstream changes. No recommendation was implemented, no batch was added and no scope was expanded.

## 1. Coverage, stated precisely

A screen, not a full read. No report in this batch is recorded as fully read.

| Report | Sections read this iteration | Reused, not re-read | Not read |
|---|---|---|---|
| IDR-009 | §14 Decision Analysis and Recommendations (including §14.1 options table and §14.3 acceptance scope) | — | §§1-13, 15-21 |
| IDR-010 | Appendix C Publication and Artifact Contradiction Register (17 rows) | The link sections at lines 433, 747 and 770, already read and recorded in `evidence/03-pass-3a.txt`, which carry the adopted relation rule | §§1-18, Appendices A, B, D, E |
| IDR-010A | Appendix C Decision and Evidence Register (9 rows) | — | §§1-18, Appendices A, B, D |
| IDR-012 | Appendix C Proposed Decision Register (17 rows); §7.6 content coding, caches and validators at lines 395-397; recommendation 9 at line 621; the cache/coding test row at line 556 | — | §§1-6, 8-18, Appendices A, B, D |
| IDR-013 | Appendix C Proposed Decision Register (18 rows) | — | §§1-18, Appendices A, B, D |
| IDR-014 | Appendix C Proposed Decision Register (20 rows) | — | §§1-18, Appendices A, B, D |

Acceptance status was checked in the governance record rather than inferred from report metadata, following the iteration 24 correction. The final research report's topic inventory lists **all six as Complete and Accepted**.

## 2. The material result: F-03's cache instance moves from unspecified to partially adopted

This is the most consequential finding of the batch, and it comes from the report that owns the exact Guide line involved.

**IDR-012 is the accepted content-negotiation report** (its Appendix C records "The Glaux Project Lead accepted every entry on August 2, 2026," and the final report inventory confirms it). Its §7.6 states:

> "`Vary` is not an authorization control. Until IDR-SRV-039/040 proves that a response is identical public content for all callers, authenticated or policy-dependent responses should default to **`Cache-Control: private, no-store`**. A later policy may permit shared caching only with a non-user-specific representation, an explicit public cache contract, and tests proving that authorization state cannot cross cache entries." (line 397)

Its numbered recommendation 9 restates this as a single instruction:

> "...use correct `Vary` and representation/coding-specific validators; and **default protected variants to `private, no-store`** until security/policy research permits otherwise." (line 621)

Its required test list includes "representation-specific ETags, conditional requests, **private/no-store defaults**" as one row (line 556), and §7.6 adds that strong entity tags "must never allow a cached SensorML, GeoJSON, or compressed response to satisfy a different variant accidentally" (line 395).

**Guide line 739 is the corresponding rule**, and it adopts the first half of that single recommendation while dropping the second: "Use appropriate `Vary` headers and representation-specific ETags."

This changes the character of the F-03 cache instance. Through iterations 17 to 23 it rested on general architectural principles drawn from security and policy reports, and the Guide's silence could fairly be described as unspecified. It is now a **partial adoption of one accepted, numbered recommendation**: the Guide took the `Vary` and per-representation-validator clauses and omitted the `private, no-store` clause that the same recommendation attaches to them. IDR-012's "`Vary` is not an authorization control" also states directly the point iteration 17 had to derive from RFC 9111 §4.1.

The instance now rests on six accepted reports (IDR-012, IDR-034, IDR-039A, IDR-040, IDR-042, IDR-055), one of which governs the Guide line in question. Its substance and recommended remedy are unchanged, and it remains recorded under F-03 rather than as a competing finding number. The change in evidentiary character is recorded there so the owner can weigh it at consolidation. IDR-013's accepted P-013-08, "Headers and conservative caching," is consistent and is not counted as a separate source.

## 3. F-11 confirmed at Guide v1.3, with the specific divergence identified

F-11 records that "The standard's tables, mappings and examples have conflicting relation spellings" and asks that the selected spelling and compatibility rule be documented. This batch establishes both sides at the current Guide version.

**The research's controlling interpretation.** IDR-010 Appendix C row C-12: "Encoding sentence/examples/ATS disagree with `ogc-rel:` mapping intent | **Prefixed table-first output** | Bare-input compatibility; named ATS adapter." Row C-13 adds that `controlStreams` and `controlstreams` differ lexically but must be treated "as one RFC-equivalent relation." IDR-010 is accepted.

**The Guide's position.** A search of the Guide finds no occurrence of `ogc-rel`, and no general statement of a relation-spelling or compatibility rule; §13 carries no row for it. That confirms at v1.3 what `evidence/03-pass-3a.txt` recorded earlier. The Guide nevertheless emits relations in three different forms in three places:

- a **bare** name: the System Event parent mapping is "an authorized `links` entry with `rel: \"system\"`" (line 485);
- a **Glaux URN**: the AsyncAPI description uses "extension relation `urn:glaux:rel:experimental-asyncapi`, never a fabricated OGC relation" (line 560);
- a **full OGC URI**: queryables use "the `http://www.opengis.net/def/rel/ogc/1.0/queryables` link" (line 774).

The bare form at line 485 is the one that diverges from C-12's controlling interpretation, under which a bare name is an accepted *input* compatibility form rather than the emitted output. Whether that divergence is a defect depends on which published reading controls, which is precisely the question F-11 records as unresolved; this batch does not resolve it. What it establishes is that the rule F-11 asks for is still absent at v1.3, that the Guide is internally inconsistent in the forms it emits, and that one of those forms contradicts the accepted research's stated interpretation. Recorded as a confirmed instance under F-11.

## 4. F-17 third instance, and the pattern is now visible

IDR-010A's Appendix C marks all nine of its decisions "**Pending plan-owner acceptance**," although the final research report lists IDR-SRV-010A as Complete and Accepted.

That is the third instance of F-17, after IDR-043 (iteration 19) and IDR-055 (iteration 24). Importantly, this batch also shows the correct practice in the same corpus: IDR-012 states "The Glaux Project Lead accepted every entry on August 2, 2026," and IDR-013 and IDR-014 both state "The Glaux Project Lead accepted every entry on August 31, 2026," each with a per-decision acceptance column and an acceptance owner.

So F-17 is not an isolated editorial slip. Three reports carry pre-acceptance residue while at least four (IDR-001, IDR-012, IDR-013, IDR-014) record acceptance correctly and in a reusable form. The remedy already stated under F-17 is unchanged; what is new is that the scale is now evidenced and a good template exists inside the corpus to copy.

## 5. Otherwise substantially adopted

| Accepted decision | Guide disposition | Reference |
|---|---|---|
| IDR-009 rec 3 and IDR-014 P-014-003/005: publish a Glaux-owned deployment API definition, not the official example, and make no OAS 3.0 class claim merely because a 3.1 document exists | **Adopted, near-verbatim.** "Generate one implementation-specific OpenAPI 3.1 description... Do not deploy the upstream example OpenAPI bundle unchanged, and do not claim OGC's separate OAS 3.0 class merely because a 3.1 document is available." | Guide line 362 |
| IDR-009 rec 4: generate `/conformance` from evidence-complete classes; keep planned or experimental state outside `conformsTo` | **Adopted.** "Declare a class only when the enabled implementation satisfies all its applicable requirements and prerequisites with adequate tests. Keep the declared class list, OpenAPI, actual routes, representations, and runtime configuration consistent." | Guide line 907 |
| IDR-010A P-010A-01: one stable root; release version is not a resource version | **Adopted, near-verbatim.** "Keep canonical URLs stable across routine software releases. An API-root path prefix is deployment configuration, not a version of a resource or SensorML schema." | Guide line 364 |
| IDR-010A P-010A-06: safe published-defect adapters, no unsafe redirects | **Adopted.** "Use `/controlstreams`, `/commands`, and `/systems/{id}/events` consistently; no automatic unsafe-method aliases." | Guide line 1142 |
| IDR-010 C-01 to C-04: plural paths are primary despite singular requirement text | **Adopted** through the Section 13 rows already verified in earlier iterations | Guide lines 1142-1143 |
| IDR-013 P-013-06: the `401`/`403`/concealed `404` boundary | **Adopted.** The status table carries `401` with challenge, `403`, "or consistent non-disclosure `404` where policy requires" | Guide line 834 |
| IDR-013 P-013-05 and P-013-12: the `400`/`415`/`422` and `429`/`503` boundaries | **Adopted** in the same status table | Guide lines 839-840 |
| IDR-013 P-013-07: an empty query is a success, not an error | **Adopted, near-verbatim.** "A well-formed identifier matching nothing produces an empty result, not a syntax error." | Guide line 436 |
| IDR-013 P-013-11: retry never implies unsafe replay | **Adopted.** A same-key retry "never dispatches again just because a response was lost"; unkeyed submissions make no deduplication promise | Guide lines 849, 851 |
| IDR-013 P-013-18: sanitized public detail with opaque correlation | **Adopted.** Problems carry "stable documented type identifiers, safe detail, and a request correlation value" | Guide line 827 |
| IDR-012 P-012-06 and P-012-03: exact media labels, UTF-8 JSON without charset, strict negotiation failure | **Adopted.** "Return `406` or `415` when negotiation/request format is unsupported; do not return a different explicit format silently" | Guide line 739 |
| IDR-012 P-012-09: canonical SWE tokens with transport aliases excluded from advertised `formats` | **Adopted** through the Section 13 media-type row | Guide line 737 |

## 6. Three non-adoptions the Guide states for itself

These are useful because they show the Guide distinguishing accepted research from mechanisms it adopts, explicitly and in its own words.

- **The deprecation calendar.** IDR-010A P-010A-03 proposes a retirement floor of two stable minor releases and twelve months. Guide line 857 addresses it directly and declines: "Do not adopt the research's proposed calendar deprecation duration as a new project obligation without an actual release-support decision." The surrounding obligations (document the affected behavior, reason, migration path and overlap period; preserve supported older client behavior through regression tests) are adopted.
- **The documentation renderer.** IDR-014 P-014-015 provisionally selects pinned self-hosted Redoc CE 3.x with Swagger UI as the test renderer and Scalar as fallback. Guide line 362 defers: "Select and pin the documentation renderer during implementation."
- **The registry framing.** IDR-009 rec 1 and IDR-014 P-014-006 both call for a typed capability or contract registry driving routes, links, the API description, conformance and tests. Guide line 360 adopts the function and narrows the framing: "Keep a small typed definition of each route's methods, resource family, parameters, representations, and conformance dependencies beside its handler. Use it to assemble the router, links, and deployment API description. **This is ordinary shared metadata, not a new registry service.**"

## 7. One cross-report reconciliation

Iteration 17 recorded that the Guide does not adopt IDR-040 §15.2's per-profile OpenAPI derivative documents, emitting instead one description of the enabled deployment. IDR-014's accepted P-014-018 resolves that within the corpus: "**No ordinary per-user OAD is generated;** materially different visibility uses a small set of named policy profiles." The Guide's single-document choice is therefore consistent with the later accepted decision, not merely narrower than an earlier report. The iteration 17 accounting stands and is strengthened.

## 8. Carry-forwards and follow-up questions

- **`fq-13` gains no new evidence here.** Nothing in this cluster addresses `Command.issueTime` semantics or `systemType`/`systemKind`. The question remains as recorded, for the batch that accounts Part 1 requirement coverage.
- **`fq-05` corroborated, not extended.** IDR-010 Appendix C row C-17 independently reports the Requirement 31(B) citation defect (`limit` and `datetime` cited only to Features §7.15.2 when `datetime` is §7.15.4), which iteration 14 recorded from a different source. Rows C-06 and C-07 likewise restate the A.35, A.36, A.40, A.42 and A.43 defects already confirmed. No new candidate is added.
- **No new follow-up question** was raised by this batch.

## 9. Disposition summary

| Item | Disposition | Where |
|---|---|---|
| Coverage | Six reports screened at their decision registers and equivalent sections; IDR-010 link sections reused; none recorded as fully read | Section 1 |
| F-03 cache instance | **Materially strengthened.** Sixth accepted source, and the first showing partial adoption of one accepted recommendation at the exact Guide line involved | Section 2 |
| F-11 | **Confirmed at Guide v1.3.** No relation rule and no `ogc-rel` occurrence; three different emitted forms; the bare form at line 485 diverges from the accepted controlling interpretation | Section 3 |
| F-17 | **Third instance** (IDR-010A), with four correctly recorded reports in the same corpus as a template | Section 4 |
| Remaining decisions | Substantially adopted across OpenAPI, conformance declaration, URL stability, status-code boundaries and error semantics | Section 5 |
| Three non-adoptions | Stated by the Guide for itself; the accepted-research-versus-adopted-mechanism distinction in action | Section 6 |
| IDR-040 per-profile OAD question | Reconciled by IDR-014 P-014-018; iteration 17 accounting strengthened | Section 7 |

**Remaining: 20 batches of 27.** Next selected batch is **batch 8 of 27, key-section screen C**, covering the peer and client implementation studies, which also absorbs the outstanding CS-GO and OSH peer-source spot checks and resolves `fq-10`.

## 10. Statement of limits

This iteration read the decision registers and equivalent decision-usable sections of six research reports, two governance documents for acceptance status, and the Guide text needed for comparison. It did not read those reports' bodies, re-read the IDR-010 link sections, resolve the published relation-spelling ambiguity underlying F-11, verify the `issueTime` or `systemType` obligations, fetch any issue body, or begin any batch other than batch 7. The F-03 and F-11 items are documentation matters in planning material, not reports of observed runtime behaviour. `review_complete` remains `false`.

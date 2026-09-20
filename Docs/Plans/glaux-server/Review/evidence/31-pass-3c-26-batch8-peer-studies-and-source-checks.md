# Pass 3c, iteration 26 — queue batch 8: key-section screen C and the peer-source spot checks

**Date:** 2026-09-20
**Provider/model:** Claude Code; model identified by the runtime environment as Claude Opus 5 (model ID `claude-opus-5`).
**Batch:** batch 8 of 27, combining the key-section screen of the eight peer and client implementation studies with the outstanding CS-GO and OpenSensorHub spot checks and the resolution of `fq-10`. Resumed from planning commit `9776e95492857c03f440ff4cbeec89359f06fc40`.
**Mode:** read-only review of eight research reports and the Implementation Guide, plus read-only retrieval from the public GitHub API and raw content endpoints for the pinned peer sources. Review artifacts updated and published; no implementation, Goal/Guide/Roadmap, issue, settings or upstream changes. No recommendation was implemented, no batch was added and no scope was expanded.

## 1. Coverage, stated precisely

A screen, not a full read. No report in this batch is recorded as fully read.

| Report | Sections read this iteration | Reused, not re-read | Not read |
|---|---|---|---|
| IDR-014A (OSH) | §16 Recommendations (14 items); §17 Risks, Constraints and Open Questions | — | §§1-15, 18-19, Appendices A-C |
| IDR-014B (CS-GO) | §16 Recommendations (10 items); §17 Risks and Open Questions | The targeted acceptance and history references already recorded in `evidence/08-pass-3c-04.txt` | §§1-15, 18-19, Appendices A-C |
| IDR-014C (pygeoapi) | §16 Recommendations (10 items); §17.1 Risks and Constraints | — | §§1-15, 17.2-19 and appendices |
| IDR-014D (SECD server) | §16 Recommendations (10 items); §17.1 Risks and Constraints | — | §§1-15, 17.2-19 and appendices |
| IDR-014E (OS4CSAPI client smoke tests) | §14 Recommendations (12 items); §15.1 Constraints | — | §§1-13, 15.2-16 and appendices |
| IDR-014F (SECD interoperability) | §14 Recommendations (10 items); §15.1 Constraints | — | §§1-13, 15.2-16 and appendices |
| IDR-014G (OS4CSAPI discussions) | §13 Recommendations (12 items); §14.1 Constraints | — | §§1-12, 14.2-15 and appendices |
| IDR-014H (draft Part 3) | §14 Recommendations (12 items); §15 Risks, Open Questions and Monitoring Triggers | — | §§1-13, 16-22 and appendices |

## 2. The spot checks, verified against the pinned sources

All four checks were run against live public endpoints and all four produced definite results.

### 2.1 `fq-10` is resolved: the IDR-040 citation is wrong

| URL | Result |
|---|---|
| `SomethingCreativeStudios/connected-systems-go` | **Exists.** Not a fork, not archived, last pushed 2026-09-02 |
| `opensensorhub/connected-systems-go` | **Not Found** |

IDR-040 §21.3 cites Connected Systems Go at `opensensorhub/connected-systems-go`, which does not exist. IDR-037 Appendix B, IDR-038 Appendix C, IDR-039 §22.2, IDR-034 §19.3, IDR-055 §19.3 and Guide line 1129 all cite `SomethingCreativeStudios/connected-systems-go`, which does. The question is answered: the single IDR-040 citation is the incorrect one.

This is now a verified non-resolving research source link, which is the subject of **F-10**. Iteration 17 deliberately declined to attach it to F-10 while it was unverified; that reservation is discharged and it is recorded there as a related instance. It is a citation defect in one accepted report, not a Guide defect, and the Guide's own citation is correct.

### 2.2 The recorded pins verify exactly

| Pin recorded across the corpus | Verified value | Result |
|---|---|---|
| CS-Go `v1.0.4` = `244f4dd586da685d4d9b75e43f73001028b5bd0e` | The `v1.0.4` ref is an annotated tag object `f157c551…`, which dereferences to commit `244f4dd586da685d4d9b75e43f73001028b5bd0e` | **Match** |
| OSH `v2.0.2` = `235c0eabf24b6d6137b499b4402943d2794b70e6` | The `v2.0.2` ref points directly at commit `235c0eabf24b6d6137b499b4402943d2794b70e6` | **Match** |

The second result independently corroborates a specific claim in IDR-014A §17.1, that OSH "`v2.0.2` is a lightweight tag." Its ref resolves straight to a commit with no intervening tag object, which is what a lightweight tag looks like, and the contrast with CS-Go's annotated `v1.0.4` makes the distinction concrete rather than asserted.

### 2.3 The Guide's own CS-GO citation verifies, including both of its claims

Guide line 1129 makes a specific, checkable claim about peer source code. It names commit `b1fd2e0e9bd69e222d05258d659a842ca24502cb` and two line ranges in `e2e/observations_test.go`, asserting that the latest-observation test "checks the expected ID, while the adjacent time-range test checks one returned item but not which one," and that "The latter could accept the wrong observation."

The commit exists, dated 2026-09-02. The file was retrieved at that commit and both ranges were read:

- **Lines 465-500**, `TestObservation_List_LatestResultTime_ReturnsMostRecent`, ends with `assert.Equal(t, newerID, obs["id"], "expected the most recent observation to be returned")`. It does check the expected ID.
- **Lines 506-539**, `TestObservation_List_ValidResultTimeRange_Filters`, seeds a 2025 and a 2027 observation, queries the 2025 window, and ends with `assert.Equal(t, 1, len(items), "expected only the 2025 observation to be returned")`. It asserts the count and never asserts which observation came back. The test would pass if the server returned the 2027 observation instead.

Both claims are accurate, and so is the Guide's qualification that this is "source analysis, not an observed runtime defect or a conclusion about the author's TDD process." No correction is needed.

### 2.4 One incidental verification worth recording

The same latest-observation test asserts `require.Equal(t, 1, len(items), "expected exactly one observation for ?resultTime=latest")`. That encodes a one-item-per-query assumption which IDR-034 §10.1 states is wrong, since `resultTime=latest` must return every observation tied at the greatest result time. The fixture happens to seed only one newest observation, so the assertion passes either way.

This directly corroborates, in the pinned source, two accepted research claims made from the outside: IDR-014B §17.2 open question 6 asks whether CS-GO's "broad `latest`" is an extension or a defect, and IDR-034 §14.4 records that CS-GO's "broad `latest`, query names, cursors, and incomplete SWE support are not standards behavior." The Guide's own rule at line 440 is the correct one and is unaffected. Recorded as peer-source evidence, not a finding.

## 3. Screen result: recommendations substantially adopted, several near-verbatim

| Study recommendation | Guide disposition | Reference |
|---|---|---|
| IDR-014B rec 4: use explicit versioned migrations; "do not depend on startup `AutoMigrate`" | **Adopted, near-verbatim.** "Migrations are immutable SQL files packaged with the server and run by an explicit administrative command. Normal startup checks compatibility rather than applying destructive upgrades silently." | Guide line 529 |
| IDR-014B rec 6: treat cursors as security-sensitive; bind version, query, ordering, principal or tenant, and snapshot or expiry | **Adopted, near-verbatim.** The SSE `id` is "an opaque authenticated-encrypted cursor binding recovery epoch, position, caller, exact scope, policy/mapping version and expiry" | Guide line 547 |
| IDR-014A R-005: derive exact conformance from enabled, tested capabilities; "never use a fixed aspirational list" | **Adopted.** "Declare a class only when the enabled implementation satisfies all its applicable requirements and prerequisites with adequate tests." | Guide line 907 |
| IDR-014A R-010 and IDR-014C rec 6: upstream standard examples are provenance and negative fixtures, never the deployed contract | **Adopted, near-verbatim.** "Do not deploy the upstream example OpenAPI bundle unchanged" | Guide line 362 |
| IDR-014C rec 4: versioned typed configuration; parse by schema, reject unknowns, reference secrets, publish a redacted effective view | **Adopted, near-verbatim.** "Use typed configuration with unknown-key rejection and startup validation... Read secrets from protected files/environment references... Redact effective configuration diagnostics." | Guide line 663 |
| IDR-014C rec 9: Basic authentication is a development boundary only | **Adopted.** Development identities are loopback-only, "clearly labeled and rejected by normal network-facing configuration" | Guide line 596 |
| IDR-014E rec 6: test query semantics with discriminating seeded data and negative assertions; never status-only criteria | **Adopted, near-verbatim.** "Seed distinguishable records and assert exact selected IDs and order, not just response shape." | Guide line 446 |
| IDR-014H R-03: require an outbox-capable transaction seam so a committed mutation cannot silently lose its publishable event | **Adopted.** The outbox stores committed changes and a worker moves them to the retained publication log, with handoff recorded transactionally | Guide lines 119, 539 |
| IDR-014H R-05: generated, configuration-true discovery tested against runtime configuration | **Adopted, near-verbatim.** AsyncAPI is generated with enabled channels and actual path parameters, "not an unbounded wildcard as an authorization promise" | Guide line 560 |
| IDR-014H R-11: separate delivery claims from MQTT QoS | **Adopted, near-verbatim.** "QoS 1 alone does not establish recovery completeness." | Guide line 564 |
| IDR-014H R-04: keep experimental capabilities disabled by default and independently gated | **Adopted.** The SSE extension is "disabled unless configured"; extensions are clearly labeled | Guide lines 362, 543 |
| IDR-014G rec 5: add an external reverse-proxy acceptance environment verifying emitted URLs and the authentication boundary | **Adopted.** "Test a path-prefixed reverse proxy, forged forwarding headers, empty collections, disabled capabilities, and a missing resource." | Guide line 366 |
| IDR-014G rec 9: route Records federation, materialization and BFF proposals outside core scope unless separately approved | **Adopted.** The exchange adapter is "not a universal federation API"; organizations retain the wider responsibilities | Guide lines 165, 620 |
| IDR-014F rec 1 and IDR-014D R-003: reject silent wrong-result behavior; an unsupported parameter returns a deterministic error | **Adopted.** "Reject malformed or unsupported parameters rather than ignoring them." | Guide line 436 |
| IDR-014F rec 8 and IDR-014A R-007: normalize errors; do not inherit a two-field error object or a `400` catch-all, and keep storage-engine detail from clients | **Adopted.** RFC 9457 problems with stable type identifiers and safe detail, over the specific status boundaries in the table | Guide lines 827, 834-840 |
| IDR-014A R-009: historical and live streaming separation with explicit backpressure, resume, disconnect and authorization-expiry semantics | **Adopted.** Replay order is preserved per subscriber, authority is rechecked during delivery with close on expiry or revocation, buffers are bounded and slow consumers disconnected | Guide line 547 |

The registry recommendation recurs in five of the eight studies (IDR-014A R-001, IDR-014B rec 1, IDR-014C rec 2, IDR-014D R-001, IDR-014E rec 2). The Guide adopts its function and states its own narrowing at line 360: "ordinary shared metadata, not a new registry service." That narrowing was already recorded in iteration 25 and is unchanged.

## 4. Two existing findings gain supporting material

**F-11.** Two further studies call for canonical typed relations, consistent with the IDR-010 row C-12 interpretation established in iteration 25. IDR-014F rec 3: "Use canonical typed link relations even where generic `data` is technically permitted; test link-order independence in external clients." IDR-014D R-004: "Preserve canonical relationship semantics. Use typed relation identifiers and verify root/nested paths resolve to the same resource identity." The Guide still states no relation-spelling rule. This adds weight to the instance confirmed in iteration 25 without changing its substance or remedy.

**F-01.** Two studies record licensing constraints on peer source reuse: IDR-014A R-014 warns to "review MPL-2.0 obligations before any direct code reuse," and IDR-014B §17.1 records that "absent license text blocks confident direct source or fixture reuse" for CS-GO. Neither is a Guide defect, and neither changes F-01, whose subject is the Glaux project's own licence selection. They are recorded as context because F-01's original wording notes that "dependency and bundled-artifact notices do not select one," and these are concrete examples of why reuse decisions will need that selection settled.

## 5. Disposition of the peer-source spot checks

The `peer-source-spot-checks` remaining check called for "Remaining promised CS-GO/OSH checks associated with 014A, 014B, 062 at the recorded pins; no new whole-history study." Its pin-verification obligation is now complete:

- both recorded pins verify exactly (§2.2);
- the one Guide citation that depends on those pins verifies in full, including both of its specific claims (§2.3);
- the outstanding repository-URL question is resolved with a definite answer (§2.1);
- IDR-014A's lightweight-tag claim is independently corroborated (§2.2).

IDR-062, the CS-GO engineering-practices and development-history study, has not been screened; it sits in cluster I, queue batch 14. It uses the same CS-GO repository whose pins are now verified, so no new pin check is expected from it, and its key-section screen is tracked under the key-section sweep rather than under this check. The check is therefore closed on that explicit basis, with no new whole-history study performed, as its own wording required.

## 6. Disposition summary

| Item | Disposition | Where |
|---|---|---|
| Coverage | Eight studies screened at recommendations and risk sections; IDR-014B's prior references reused; none recorded as fully read | Section 1 |
| `fq-10` | **Resolved.** `opensensorhub/connected-systems-go` does not exist; the IDR-040 citation is wrong and the other five reports and the Guide are correct. Recorded as a verified instance under F-10 | Section 2.1 |
| CS-Go and OSH pins | **Both verify exactly**; the OSH lightweight-tag claim is corroborated | Section 2.2 |
| Guide line 1129 citation | **Verifies in full**, including both specific claims and its own qualification | Section 2.3 |
| CS-GO `resultTime=latest` assumption | Verified in the pinned source; corroborates two accepted research claims; no finding | Section 2.4 |
| Screen result | Recommendations substantially adopted, several near-verbatim | Section 3 |
| F-11 and F-01 | Supporting material added; neither disposition changes | Section 4 |
| `peer-source-spot-checks` | **Closed** on the explicit basis stated | Section 5 |

**Remaining: 19 batches of 27.** Next selected batch is **batch 9 of 27, key-section screen D**, covering the resource model, temporal validity and status reports: IDR-015, IDR-016, IDR-018, IDR-019 and IDR-020, plus the unread key sections of IDR-017, whose §9.1 was already read and must be reused.

## 7. Statement of limits

This iteration read the recommendation and risk sections of eight research reports, the Guide text needed for comparison, and four pinned external sources through public read-only endpoints. It did not read those reports' bodies, screen IDR-062, execute any peer implementation, reproduce any peer test run, or begin any batch other than batch 8. The peer-source verifications establish that the recorded pins and the Guide's one source claim are accurate as of this retrieval; upstream repositories remain mutable and the studies' own temporal caveats continue to apply. `review_complete` remains `false`.

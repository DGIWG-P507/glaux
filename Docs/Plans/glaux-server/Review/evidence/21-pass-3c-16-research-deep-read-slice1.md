# Pass 3c, iteration 16 — bookkeeping correction plus research deep-read remainders, slice 1

**Date:** 2026-09-19
**Provider/model:** Claude Code; model identified by the runtime environment as Claude Sonnet 5 (model ID `claude-sonnet-5`). This is a provider change from the prior iterations (GitHub Copilot, self-reported as Claude Fable 5.1), following the reviewer's disclosed subscription-budget stop.
**Batch:** (1) bookkeeping correction for issues #98, #99, #130; (2) `research-deep-read-remainders`, slice 1, from `current_work.next_batch` at planning commit `74f717ce1f55dea029cf5ede450e41723c748c6e` (Pass 3c iteration 15 checkpoint).
**Mode:** read-only review of research reports and the Implementation Guide; review artifacts updated and published; no implementation, Goal/Guide/Roadmap, issue, settings or upstream changes.

## 1. Bookkeeping correction (not a new review iteration of these issues)

`review-state.json.implementation_issues` still marked #98, #99 and #130 as `full_body_read_reported: false` / `review_status: "not_recorded"`, although:

- Issues #98 (leaf 3.2.1) and #99 (leaf 3.2.2) were read in full via the GitHub API during Pass 3c iteration 15, recorded in [20-pass-3c-15-observation-extension-policy.md](20-pass-3c-15-observation-extension-policy.md), Section 1 ("glaux-server issues #98 ... and #99 ... Full bodies read via the GitHub API") and Section 6.3 ("Issues #98 and #99 (bodies read) contain no unknown-member or `foi@id` statement").
- Issue #130 (leaf 4.1.3) was read in full during Pass 3c iteration 12, recorded in [17-pass-3c-12-standards-swe.md](17-pass-3c-12-standards-swe.md), Section 3 ("Roadmap v1.18 leaves ... 4.1.3 (#130) ... issue #130 body read in full").

These three entries are corrected below to `full_body_read_reported: true` / `review_status: "targeted_prior_review"`, with their evidence pointers set to the reports that actually read them. This is **not** a claim that #98, #99 or #130 received a complete issue-specific scope/dependency/acceptance review (the standing `conclusion_or_remaining_question` wording for `targeted_prior_review` entries already carries that caveat, matching the other 15 entries at that status); the issues were not reread for this correction. The displayed "N of 286 issue bodies read" count in `findings.md` and `README.md` moves from 15 to 18.

## 2. Sources consulted this batch

| Source | Access | Used for |
|---|---|---|
| [IDR-SRV-008](../../../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-008-conformance-class-and-requirement-mapping-report.md), lines 752-989 (Sections 16-19: Validation Against the Research Plan; References; Next Steps and Handoff; Appendices A-E) | Full read | Committed remainder of a report otherwise already read in full (evidence/05-pass-3c-01.txt) |
| [IDR-SRV-037](../../../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-037-feasibility-and-asynchronous-tasking-strategy-report.md), lines 693-821 (Appendix A Lifecycle/Ownership Matrix; Appendix B Reproducibility Record; Appendix C Requirement-by-Requirement Disposition; Appendix D Completion Checklist) | Full read | Committed remainder of a report whose main body was already read (evidence/09-pass-3c-05.txt) |
| [IDR-SRV-038](../../../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-038-command-authorization-safety-and-audit-strategy-report.md), lines 675-776 (Appendix A Command Authorization, Safety, and Audit Matrix; Appendix B Event Catalog and Invariants; Appendix C Reproducible Evidence Record; Completion Checklist) | Full read | Committed remainder of a report whose main body was already read (evidence/11-pass-3c-07.txt) |
| [IDR-SRV-039](../../../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-039-authentication-authorization-and-api-security-threat-model-report.md), lines 242-274 (Section 7, API Surface and Trust-Boundary Inventory) and lines 625-688 (Sections 21-22, Validation Against Success Criteria; References) | Full read | Committed remainder of a report whose other sections were already read (evidence/12-pass-3c-08.txt) |
| [Glaux Server Implementation Guide v1.3](../../glaux-server-implementation-guide.md) | Targeted `grep` then read at each cited line | Adopted/not-adopted/departure comparison for every material item above |
| [Glaux Server Goal and Definition](../../glaux-server-goal-and-definition.md) | Targeted `grep` (`override`, `lease`, `operator approval`) | Confirm scope silence on command-override/lease machinery is not a controlling-document omission |
| findings.md F-03, F-08, F-15, F-18, F-19, F-20, F-22 | Read | Identify existing homes before treating any comparison as a new finding |

No issue bodies were fetched for this batch (optional per the batch's source scope; the bookkeeping correction in Section 1 reused issues already read for a different check).

## 3. IDR-008 Sections 16-19 (report lines 752-989)

Section 16 is a validation ledger (methodology-phase, success-criterion and deliverable-content tables, all "Complete"/"Met") and an independent-review reconciliation record. Section 17 is a reference list. Section 18 records that the Project Lead's August 1, 2026 acceptance authorized exactly one next topic (IDR-SRV-009); no later topic was authorized by that instruction. Section 19 (Appendices A-E) is the full grouped identifier catalog for every Part 1/Part 2 direct class (already summarized, not superseded, by the grouped tables in the report's §§5-9, which were read in full during the original pass, evidence/05-pass-3c-01.txt), a count/reproducibility record, a research-question-to-evidence map, and a completion checklist.

**Accounting.** None of this remainder contains a key finding, recommendation or open question that was not already carried into the report's earlier, previously-read sections. Specifically:

- Appendix A/B's exhaustive `/req/.../conf/...` identifier lists are the same 25-class, 233-requirement, 5-recommendation, 240-test baseline already synthesized into Guide Section 7's class/dependency tables when §§5-9 were read; this remainder introduces no identifier absent from that earlier synthesis. **Adopted** (via the prior, not this, read).
- Appendix C's "Part 2 0/24 class PURLs resolve" observation restates the risk row already present at report line 743 ("Part 2 and recommendation PURLs do not resolve | Emit exact standard identifiers; link humans to approved HTML; monitor separately | No | OGC #152"), itself part of the section range already read in the original pass. No new Guide-facing item.
- Appendix D and E are process/completeness records (research-question disposition, report checklist), not implementation-facing claims.
- Section 18's single-topic authorization is project-governance history superseded by the fact that 71 reports now exist under later, separately recorded authorizations; it raises no live question.

**Disposition:** No Guide correction; no new finding; no related instance added to any existing finding. IDR-008 moves from "substantial read" to fully read.

## 4. IDR-037 Appendices A-D (report lines 693-821)

Appendix A is a 14-row lifecycle/ownership matrix (private phase, public status, authoritative reporter, required evidence, retry/cancel/recovery rule) for synchronous/asynchronous tasking, Feasibility and Command. Appendix B is the pinned-source reproducibility record and a bulleted restatement of findings already in the main body (singular/plural path defects at Requirements 29/32/34; the missing nested `{id}` defect at Requirement 75; the A.105-adjacent ATS gap). Appendix C is a per-requirement disposition ledger (Requirements 25-39, 56-61, 71-77, 85-91, 99-105) naming each requirement's concrete Glaux behavior and verification approach. Appendix D is a completion checklist.

**Accounting against Guide v1.3:**

| Appendix A/C item | Guide disposition | Reference |
|---|---|---|
| Async admission required evidence (atomic resource/status/task/idempotency/outbox); same-key retry returns same resource | **Adopted.** Idempotency-Key contract scopes key to caller/operation/target, retains request digest and outcome atomically, and returns the same resource/outcome for the same key and intent | Guide lines 505, 849 |
| Sync admission: private durable work before dispatch, no public effect before durable stage | **Adopted.** "Retain a private durable admission/attempt record before dispatch, then publish ... atomically when the authoritative outcome is known" | Guide line 845 |
| Exactly nine terminal/interim status codes; "never invent public UNKNOWN" | **Adopted.** Guide names the same nine codes (`PENDING, ACCEPTED, REJECTED, SCHEDULED, UPDATED, CANCELED, EXECUTING, COMPLETED, FAILED`) and forbids inventing an additional value | Guide line 576 |
| Cancellation is authoritative-outcome-gated; "DELETE remains distinct" from cancellation | **Adopted.** "Cancellation follows the command-status behavior, not HTTP DELETE. A `CANCELED` report must reflect authorized, effective cancellation" | Guide line 582 |
| Command/Commands and Feasibility/`{id}`/status/result use **plural** paths (Req. 26-29, 32, 34; Req. 35-38) | **Adopted.** Guide's resource table lists `/commands`, `/commands/{id}/status` and `/result`; `/feasibility`, `/feasibility/{id}/status` and `/result` | Guide lines 731-732 |
| Requirement 32/34 "singular-defect test" (published Annex A text is internally inconsistent on singular/plural); Requirement 75 missing nested `{id}` in the ATS/OAD | **Adopted, and explicitly sourced to this report.** Section 13 rows: "Singular `/controlstream`, `/command`, isolated `/controls/{id}` ... | Use `/controlstreams`, `/commands` ... consistently" and "Nested feasibility replace/delete template omits the item ID ... | Implement explicit canonical `/feasibility/{id}`" | Guide lines 1142-1143, citing `[037][R037]` |
| Command/Feasibility POST response shape (`201` + canonical `Location` + status body + individual status `Content-Location`) | **Adopted**, labelled an explicit Glaux interpretation to verify with independent clients | Guide line 843, 1144 (also citing `[037][R037]`) |
| Terminal-status rules coexisting with report CRUD/PATCH and parent cascade | **Adopted.** "Keep execution state distinct from editable public report records ... reject an attempted forbidden lifecycle transition" | Guide line 584, 1145 |
| Unkeyed submissions and their consequence (no guaranteed retry identity) | **Adopted; matches F-15's existing disposition**, not a new item | Guide line 851 |
| Appendix B git pins (CS-Go v1.0.4, OSH v2.0.2, OS4CSAPI phase-9, SECD evidence) | Out of scope for this check; these pins belong to the separate `peer-source-spot-checks` remaining check (014A/014B/062) and are not chased further here | — |

**Disposition:** No contradiction found between Appendix A/C's disposition ledger and the Guide. No Guide correction; no new finding. IDR-037 moves from "main body read; appendices outstanding" to fully read.

## 5. IDR-038 Appendices A-C (report lines 675-776)

Appendix A is a 16-row command authorization/safety/audit matrix spanning discovery, Feasibility submit, Command submit, **operator approval**, **override**, **target/control lease**, dispatch authorization, direct/broker/gateway dispatch, target acceptance, execution monitoring, result append, cancellation, timeout/expiry, audit search/export/redaction, and reconciliation. Appendix B is a 37-name minimum audit-event catalog (B.1) and 12 numbered decision invariants (B.2). Appendix C is the pinned-source reproducibility record.

**Accounting against Guide v1.3 and the Goal:**

- **Operator approval, override and target/control-lease rows (and the corresponding parts of invariants 5-7)** describe a decision architecture — separation-of-duties approval, an independent override authority bound to non-overrideable rules, and an exclusive/coordinated control lease — that the Guide does not implement: `grep` for `lease`, `operator approv`, `separation of duties` and `override` (command-safety sense) returns no hits in the Guide, and the Goal is likewise silent. This is not a new gap: F-03 already dispositions this exact category ("research-specific approval/lease machinery are not automatically needed") after reviewing issue #22's authorization matrix. **Related instance recorded under F-03** (below); no new finding.
- **Sender-derived identity** (Command submit row, "`sender` derived, not trusted input"): **adopted**, "A submitter cannot impersonate another sender by writing a field" (Guide line 574).
- **Feasibility/transport-acknowledgement/target-reachability never substitute for authorization or safety** (invariant 4): **adopted** — Feasibility "neither authorizes nor performs the action" (Guide line 137); "transport ACK not acceptance" is an explicit Guide table entry (line 1037).
- **No mandatory-rule failure becomes a warning** (invariant 5, non-override half): no Glaux-specific "mandatory rule" catalog exists to check this against, because the override/rule-engine machinery itself is not adopted (see above); not a separate gap from the F-03 item.
- **No cancellation becomes `CANCELED` without configured authoritative semantics** (invariant 8): **adopted**, Guide line 582 as quoted in Section 4 above.
- **No status/result assertion changes projection until reporter authority, sequence/epoch, transition and schema validate** (invariant 9): **adopted**, Guide line 584 ("Derive execution/current-status facts from retained authoritative lifecycle evidence ... reject an attempted forbidden lifecycle transition").
- **No external publication precedes commit** (invariant 12): **adopted**, the outbox/publication-log model commits before any delivery attempt (Guide lines 539, 649).
- **Appendix B.1's named 37-event catalog**: **not adopted verbatim.** The Guide's audit floor is a field-level requirement, not a named-event taxonomy: "Record durable audit information for meaningful mutations and command attempts: verified actor/source, operation, target/revision, time, outcome, and correlation identifier" (Guide line 602), plus "Record relevant denials safely." This is the same gap F-20 already tracks ("Proposed event categories are suggestions ... Define selected categories, bounds and failure behavior if adopted"). **Related instance recorded under F-20**; no new finding.
- **Invariant 10** ("No correction, reconciliation, retention, redaction, or export rewrites original audit evidence") and the Appendix A "Audit search/export/redaction" row's "derived redaction never alters original": **not stated as a hard rule.** Guide line 602 states a durable-record floor and explicitly disclaims one specific architecture ("A mandatory tamper-proof ledger ... is not proposed") without stating whether correction, redaction or export may alter the original evidence versus only a derived view. This is the exact question F-19 already carries ("The explicit Guide floor covers durable transactional accountability, not a complete tamper-evidence architecture ... Separate serving access from owner/migration and authorized retention roles"). **Related instance recorded under F-19**; no new finding.
- **`audit_exported` as a distinct catalogued event, separate from generation/release**: consistent with, and does not exceed, F-22's existing "distinguish generation, release/handoff and confirmed receipt" recommendation. No additional instance needed beyond noting the consistency.
- Appendix C git pins: out of scope for this check, same as IDR-037 Appendix B (belongs to `peer-source-spot-checks`).

**Disposition:** Two related instances recorded (F-19, F-20) and one (F-03); no new finding number. IDR-038 moves from "main body read; appendices outstanding" to fully read.

## 6. IDR-039 Section 7 and Sections 21-22 (report lines 242-274, 625-688)

Section 7.1 classifies twelve API surfaces (landing/conformance, standards reads/writes, dynamic-data query, publisher ingestion, source registration, streaming subscription, Feasibility/command, administration, health/readiness/metrics/traces, conformance/test/reset, and outbound schema/reference/federation calls) by exposure default, principal boundary and primary threat. Section 7.2 states seven trust-boundary invariants. Sections 21-22 are the plan's success-criterion validation table and reference list.

**Accounting against Guide v1.3:**

| Section 7 item | Guide disposition | Reference |
|---|---|---|
| Outbound schema/reference/federation calls: "Allowlisted and mediated"; primary threat SSRF | **Adopted** (SSRF mitigation is not named as such, but the control is present verbatim): "Install required schema references locally and resolve them from an allowlist. Do not fetch arbitrary `$ref`, SensorML links, data URLs, or result URLs during a public request" | Guide line 420 |
| Health/readiness surface: liveness minimal, other signals reported through protected diagnostics | **Adopted.** "Expose liveness and readiness separately ... Report that condition through protected diagnostics, not false global readiness" | Guide line 665 |
| Health/readiness surface: **metrics/traces internal** (i.e., not on the same public/anonymous exposure as liveness) | **Not explicitly stated.** Guide line 667 describes what metrics to collect and that "Supply a simple way to inspect them" is optional, but does not say the metrics/traces endpoint itself must be non-public or access-restricted the way readiness diagnostics are called "protected" at line 665. This is a documentation-completeness gap, not an evidenced defect (no metrics endpoint has been built to inspect) | Guide line 667; **recorded as a new follow-up question (`fq-09`), not a finding** |
| Incoming identity/forwarding headers stripped unless from an authenticated allowlisted proxy | **Adopted.** "Trust forwarded origin headers only from configured reverse proxies" | Guide line 364 |
| Query results/derived metadata/caches/cursors/links/events/errors remain bound to the authorized view | **Adopted.** "No hidden contributor, relationship count or source artifact may leak through links, schemas, query NULL behavior, errors, exports or event topics" | Guide line 610 |
| External and federated content untrusted even when its transport peer is authenticated | **Adopted.** "imports re-evaluate local access and source authority rather than inheriting the sender's permission decision" | Guide line 635 |
| Authentication/policy-service failure never turns a protected route into an anonymous/cached allow | **Adopted, near-verbatim.** "no network-facing operation defaults to allow on policy-service failure" | Guide line 598 |
| A credential proves no more than the exact local mapping and policy decision | **Adopted in substance.** Token type/signature/issuer/audience/validity/scope checks before any grant | Guide line 594 |
| "Direct backend access cannot bypass edge controls" | Deployment-topology concern (WAF/edge placement); reasonably outside a single reference-server implementation guide's scope. Not treated as a gap |

Sections 21-22 (success-criterion validation, references) contain no material not already reflected in the sections read in the original pass (evidence/12-pass-3c-08.txt); no new item.

**Disposition:** One new follow-up question recorded (`fq-09`, metrics/traces exposure); no Guide correction required for the remainder; no new finding. IDR-039 moves from "sections 1-6, 8-20 read; 7 and 21-22 outstanding" to fully read.

## 7. Findings register changes

No new finding number was created. Related instances were added under three existing findings:

- **F-03 (Minimal authorization semantics):** IDR-038 Appendix A's operator-approval, override and target/control-lease decision types, and the corresponding parts of Appendix B.2's invariants 5-7, are the same category of research-recommended machinery F-03 already treats as optional, not omitted-and-required. The Guide's and Goal's continued silence on `lease`/`override`/`operator approval` confirms no departure has occurred since none was required.
- **F-19 (Audit modification and retention boundary):** IDR-038 Appendix B.2 invariant 10 and the Appendix A audit-search/export/redaction row's "derived redaction never alters original" sharpen the open question already recorded under F-19 into a specific candidate rule (original audit evidence is never rewritten by correction, reconciliation, retention, redaction or export) that Guide line 602's durable-record floor does not itself state.
- **F-20 (Denial-audit deliverable):** IDR-038 Appendix B.1's 37-name minimum event catalog is a concrete candidate for the "selected categories" F-20 already asks the owner to define; it is a suggestion the record can point to, not an adopted requirement.

No related instance was needed under F-08, F-09, F-15, F-18, F-21 or F-22 for this batch's content; each was checked and found either already consistent (F-15, F-18, F-22) or not implicated (F-08, F-09, F-21) by the sections read.

## 8. Follow-up question recorded, not scheduled

`fq-09-metrics-traces-exposure`: IDR-039 Section 7.1 classifies "Health/readiness/metrics/traces" with metrics/traces as internal-only signals, distinct from minimal public liveness. Guide Section 4.12 (line 667) specifies what to collect but does not state that the metrics/traces endpoint itself must be non-public or access-restricted, the way line 665 calls readiness detail "protected diagnostics." Should Guide Section 4.12 add one sentence stating metrics/traces exposure is internal/access-restricted by default, or is this left to deployment network topology (reverse proxy / separate listener), consistent with how Section 4.10 already treats administration as "internal/privileged, separate audience"? Relevance: Guide Section 4.12; no owning issue identified yet. Evidence: this report, Section 6.

## 9. Disposition summary for the checkpoint

| Check | Disposition | Where verified |
|---|---|---|
| Bookkeeping correction (#98, #99, #130) | Corrected; not a new issue review | Section 1 above |
| IDR-008 Sections 16-19 (lines 752-989) | Fully read; no Guide correction; no new finding | Section 3 above |
| IDR-037 Appendices A-D (lines 693-821) | Fully read; no Guide correction; no new finding | Section 4 above |
| IDR-038 Appendices A-C (lines 675-776) | Fully read; no Guide correction; three related instances (F-03, F-19, F-20) | Section 5 above |
| IDR-039 Section 7 and Sections 21-22 (lines 242-274, 625-688) | Fully read; no Guide correction; one new follow-up question (`fq-09`) | Section 6 above |

**Research-deep-read-remainders, updated remaining scope:** IDR-040 Sections 9-21 (report lines 344-686: Metadata/SensorML/SWE redaction findings through References; Sections 1-8 were already read, evidence/13-pass-3c-09.txt); IDR-042 and IDR-043 remaining sections (beyond the sections already read per evidence/15-pass-3c-11.txt); and the four committed deep reads not yet begun (IDR-030, IDR-034, IDR-039A, IDR-055). The next selected slice is **IDR-040 Sections 9-21**, a single-report, bounded read comparable in size to this iteration's four-report remainder slice.

## 10. Statement of limits

This iteration read exactly the committed remainder sections named in the batch and the Guide/Goal text needed to assess them; it did not reread any main body, rereview any standards check (all are closed), or fetch new issue bodies beyond reusing the two already read in iteration 15 for the bookkeeping correction. The `peer-source-spot-checks` pins observed in IDR-037 Appendix B and IDR-038 Appendix C (CS-Go v1.0.4, OSH v2.0.2, OS4CSAPI phase-9, SECD evidence) were noted but not chased; they remain the separate `peer-source-spot-checks` remaining check. `review_complete` remains `false`.

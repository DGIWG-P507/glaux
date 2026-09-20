# Pass 3c, iteration 19 — research deep-read remainders, slice 4 (IDR-043) and the complete remaining batch queue

**Date:** 2026-09-19
**Provider/model:** Claude Code; model identified by the runtime environment as Claude Opus 5 (model ID `claude-opus-5`).
**Batch:** two parts in one iteration, as directed: (1) map all remaining review coverage into a finite numbered batch queue inside the existing `review-state.json`; (2) execute the already-selected slice 4, IDR-043's unread sections. Resumed from planning commit `eb34935d9b157813121b4b1742b2baf94c6fd154` (Pass 3c iteration 18 checkpoint).
**Mode:** read-only review of one research report, the Implementation Guide and the Roadmap; review artifacts updated and published; no implementation, Goal/Guide/Roadmap, issue, settings or upstream changes. No new planning document was created and no scope was added.

## 1. Sources consulted this batch

| Source | Access | Used for |
|---|---|---|
| [IDR-SRV-043](../../../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-043-server-synchronization-and-conflict-handling-boundary-report.md) header and Sections 5-14, 16-18, 21-22 and the completion checklist (lines 1-22, 175-518, 538-635, 689-765) | Full read | The committed remainder; Sections 15 and 19-20 were already read (evidence/15-pass-3c-11.txt), and Sections 6, 7 and 11.4 were completed here beyond the rows previously sampled |
| [Glaux Server Implementation Guide v1.3](../../glaux-server-implementation-guide.md) | Targeted search then read at each cited line | Adopted/not-adopted/departure comparison |
| [Glaux Server Roadmap v1.18](../../glaux-server-roadmap.md) | Mechanical extraction of the phase/group/issue mapping | Sizing and functional grouping of the issue-review batches in the queue |
| findings.md F-03, F-08, F-15, F-17, F-18, F-19, F-20, F-21, F-22 | Read | Identify existing homes before treating any comparison as a new finding |

Sections 1-4 of IDR-043 (summary, scope, evidence base, methodology) were read as context for Sections 5-14 but contain no Guide-facing claim of their own.

## 2. IDR-043 accounted against Guide v1.3

### 2.1 Adopted, several passages near-verbatim

| IDR-043 item | Guide disposition | Reference |
|---|---|---|
| §5.3 Non-bypass invariant: synchronization is another write path passing the same or stricter authentication, authorization, source-trust, policy, validation, command-safety, transaction and audit gates; a trusted network, signed bundle, broker ACL, database replica or administrator role confers no resource authority | **Adopted.** "imports re-evaluate local access and source authority rather than inheriting the sender's permission decision"; "Imported command history is evidence; new physical action requires the local authorization and dispatch process" | Guide lines 635, 647 |
| §6 Scenario taxonomy: a deployment must name the scenario and authority model; "enabling a generic `/sync` route with unspecified semantics is prohibited" | **Adopted.** "begin with an administrative export/import adapter for configured peers or authorized files, not a universal federation API"; "No JSON canonicalization/signature scheme or general multi-master protocol is implied" | Guide lines 620, 633 (which cites `[R043]` directly) |
| §8.1 `SyncEnvelopeV1` envelope, domain-identity, revision/causality, content and range field groups | **Adopted in narrower form.** The manifest contract table carries `format`, `exchangeId`, `source`, `sourceEpoch`, `recipient`, `scope`, `scopeVersion`, `mode`, `snapshotId`, `baseCursor`, `endCursor`, `complete`, `records`, and per-record `kind`, `sourceId`, `revision`, `predecessor`, `dependencies`, `ancestry`, `operation`, `mediaType`, `payloadBase64`, `sha256` | Guide lines 624-631 |
| §8.2 Identity rules: resource ID, revision ID and message/operation ID are different objects; external canonical URLs and UIDs are aliases until mapping proves local scope; UUID order does not determine precedence | **Adopted.** "maps remote to local identities explicitly"; "Never infer conflict resolution from arrival order, largest UUID, or unsynchronized node clocks" | Guide lines 635, 647 |
| §8.4 A tombstone is an authoritative lifecycle assertion retained until stale resurrection is impossible; concurrent update versus tombstone is a protected conflict | **Adopted.** "Retain tombstones and exchange receipts for the configured recovery window"; "attempted resurrection of a deleted resource is retained/reported without silently overwriting accepted state"; "Do not reuse a deleted ID" | Guide lines 509, 635, 647 |
| §8.5 Inbox idempotency keyed by sender/source, scenario and operation identity, committing the inbox row and its effect atomically | **Adopted in substance with a narrower key** built from exchange identity, source, source epoch, record identity/revision and digest: "Identical accepted input is a no-op returning the recorded outcome," committed "with the exchange receipt in one transaction" | Guide lines 620, 635 |
| §9.2 Receive/apply pipeline: authenticate, authorize scope, verify digests, resolve pinned schemas without unapproved network retrieval, check inbox, validate, classify, persist atomically, return a durable outcome, advance only the eligible watermark | **Adopted in substance.** The import path validates the same resource semantics as HTTP, resolves references from a local allowlist, stages unknown dependencies, commits with the receipt in one transaction, and does not advance a completeness checkpoint past unresolved records | Guide lines 420, 635, 641, 649 |
| §9.3 Watermarks are scoped and policy-hidden records advance only an opaque authorized cursor without leaking count or identity | **Adopted.** Opaque authenticated-encrypted cursors bound to caller, scope, policy version and expiry; "A policy-filtered omission or changed scope is not a tombstone" | Guide lines 547, 645, 649 |
| §10.1 Classification table: identical scoped identity and digest deduplicates with the stored result; same identity with different bytes is a conflict; valid older fact is accepted history; missing predecessor holds or records an explicit gap; stale update against a tombstone is anti-resurrection | **Adopted, close match.** "Retained known predecessors are harmless repeats, different content at the same revision is a conflict, and missing ancestry requires dependency recovery or a fresh snapshot" | Guide lines 635, 637 |
| §10.2 and §12.4 Prohibited universal resolvers: arrival time, wall clock, latest UUID, broker order, generic last-write-wins, central-node preference | **Adopted for the principal cases, near-verbatim.** "Never infer conflict resolution from arrival order, largest UUID, or unsynchronized node clocks"; "There is no general force-overwrite or arrival-time-wins command" | Guide lines 641, 647 |
| §12.2-§12.3 Isolation of unaccepted candidates and an explicit resolution allowlist rather than silent merge | **Adopted in narrower form.** "Unknown dependencies stay in a restricted staging area ... they do not become partially valid public resources"; administrative `exchange inspect`/`exchange resolve` select keep-local or accept-incoming, require the expected current local revision and a reason, revalidate authority at resolution time and record the decision with audit | Guide lines 635, 641 |
| §13.2 Authorized view construction precedes counts, ranges, watermarks, latest selection and errors | **Adopted.** Authorization precedes queries, counts, extents, links, schema disclosure, latest selection and streaming delivery | Guide lines 469, 598 |
| §13.3 Federation needs a versioned partner profile; a public CSAPI read endpoint alone is not one; cross-domain transfer remains external with no CDS or accreditation claim | **Adopted.** Configured peers only, recipient/scope "not a client-supplied permission grant", and the explicit statement that organizations retain release authority and accreditation | Guide lines 165, 592, 620, 627 |
| §14 Commands are not general replicated mutable objects; synchronization cannot broaden command authority or bypass the pre-effect gate; loss of acknowledgement never creates a new Command ID or a blind new physical attempt; a late nonterminal status cannot replace a terminal one | **Adopted.** "Do not synchronize queued command intent into automatic dispatch"; the same-key retry "never dispatches again just because a response was lost"; "Late reports must not blindly overwrite a newer terminal state"; restored pending work is held for reconciliation | Guide lines 531, 582, 584, 647, 849 |
| §16.1 HTTP behavior for `412`, `409`, `400`, `413`, `429`/`503`, `401`/`403`/policy-selected `404`, and RFC 9457 problem details with stable type, safe detail and opaque correlation that exclude topology, hidden IDs, policy labels and internals | **Adopted.** The status table carries the same mappings including `413` and the consistent non-disclosure `404`; problems have "stable documented type identifiers, safe detail, and a request correlation value" | Guide lines 600, 827, 834-840 |
| §16.2 Commit precedes publication through the outbox; logical event IDs stay stable across retries; Part 3 remains outbound, experimental, version-pinned and disabled by default | **Adopted.** The outbox commits before send with stable event IDs; the SSE extension is "disabled unless configured" and extensions are clearly labeled | Guide lines 119, 362, 539, 543 |
| §16.3 Observability: high-cardinality resource, peer, tenant and command IDs are not unrestricted metric labels; health distinguishes process, store, audit path and optional dependencies, and one global healthy/unhealthy flag is insufficient | **Adopted.** "Avoid resource IDs or arbitrary query strings as metric labels"; liveness and readiness are separate and "Optional broker failure can degrade publication without stopping valid historical reads ... Report that condition through protected diagnostics" | Guide lines 665, 667 |
| §17.3 Security testing including SSRF through references, decompression bombs, parser differentials and replay floods | **Adopted.** Reference resolution is allowlisted with no arbitrary `$ref`/link/data-URL fetching, and recursion, decompression and total request cost are bounded; the security test list covers malicious references and parser exhaustion | Guide lines 420, 612 |

### 2.2 Not adopted, as a recorded scope choice

The Guide does not implement IDR-043's full machinery: the complete `SyncEnvelopeV1` field taxonomy including its authority, policy/trust, lifecycle and provenance groups (§8.1); the five orthogonal state machines for session, candidate, outcome, local record and continuity (§9.1); the `SyncConflictV1` protected conflict record (§12.1); a governed quarantine subsystem with encryption, tenant separation and retention (§12.2); the operator review workflow with claim leases, role separation and configured second approval (§12.5); and versioned federation partner profiles (§13.3).

In their place the Guide selects a deliberately narrower design stated at line 620: an administrative export/import adapter for configured peers or authorized files rather than a universal federation API, with a restricted staging area for unresolved input (635) and two administrative operations, `exchange inspect` and `exchange resolve`, over restricted staged conflicts (641). IDR-043's own §5 frames its scope as the server-owned contract while assigning topology, products and numeric horizons to later topics, and its §8.4 explicitly defers retention numbers. The approval and role-separation elements fall in the same family as the operator-approval machinery already dispositioned under F-03 in iteration 16. No new finding.

### 2.3 A recorded departure that is already dispositioned

IDR-043 §16.1 includes a `428` row for a missing required conditional revision. The Guide deliberately does not adopt mandatory preconditions: "The baseline permits ordinary standards writes without a mandatory `If-Match` header. This deliberately does not adopt IDR-029/031's stronger mandatory-header policy" (line 503), with "missing `If-Match` is not an automatic error in this baseline" in the status table (838) and the tradeoff recorded in Section 13 (1139). IDR-043's `428` belongs to that same already-recorded project choice, so it is noted here and not raised again.

## 3. One finding instance confirmed: F-17 stale acceptance wording

F-17 records that "Dated acceptance records exist for checked reports; some closing paragraphs or headers retain earlier In Review wording." IDR-043 supplies a concrete, verified instance with exact locations:

- The header records **Report Status: Final**, **Accepted By: Glaux Project Lead**, **Acceptance Date: September 15, 2026** (lines 4, 17-18).
- §21 closes with "The deliverable should remain **In Review** until project-lead acceptance." (line 704).
- The completion checklist's final item still reads "Category H and implementation remain unauthorized pending acceptance." (line 765).

The dated acceptance record is the controlling fact; the two trailing statements are pre-acceptance residue that the acceptance did not sweep. This is exactly the editorial pattern F-17 describes, now evidenced rather than reported. It remains a low-severity editorial cleanup for the owning report, not a technical defect and not a reason to treat the report as unaccepted. Recorded as a related instance under F-17.

## 4. Other observations, not findings

- **Cache mention.** IDR-043 §7 line 265 states that "Local database rows, queue leases, HTTP cache entries and metrics normally remain node-local even when the underlying immutable domain evidence is exchanged." This is a synchronizability statement rather than a disclosure statement, so it does not add to the F-03 cache instance beyond confirming that the research corpus treats HTTP cache entries as node-local state. No change to that instance.
- **`fq-09` corroboration.** §16.3's health-dimension requirement and metric-label discipline is the fourth research source bearing on the metrics/traces exposure question. The Guide adopts the label discipline (667) and the health-dimension split (665); the open part remains the exposure boundary of the metrics endpoint itself. Evidence list extended; the question stays noted and unscheduled.
- **`fq-10` not resolved.** IDR-043 §22 does not cite the CS-Go repository, so the URL discrepancy recorded in iteration 17 is untouched and still belongs to the peer-source spot checks.

## 5. The remaining batch queue

A finite numbered queue covering every remaining finish condition was written into `batch_queue` in `review-state.json`. It is a cursor structure inside the existing state file, not a new planning document, and it adds no scope: every entry traces to a finish condition already recorded in `review_areas`, `remaining_checks` or `backlog_analysis`.

**Sizing basis.** Each batch is sized against what the last four iterations actually absorbed: roughly 350-650 lines of source plus its Guide cross-checks. Research key-section batches group five to seven reports because a key-section screen (summary, recommendations, risks and open questions) is lighter per line than a deep-read accounting. Issue batches follow Roadmap phase and group boundaries, which are the functional units.

**Genuinely unreviewed work versus bookkeeping.** The queue distinguishes three states explicitly, because conflating them would overstate the work:

1. **Coverage not established** — 47 research reports whose read depth was never recorded. The state file's standing note is "Do not infer either a full read or that no incidental excerpt was ever seen." These need a key-section screen to establish coverage, which is cheaper than a cold read.
2. **Genuinely unreviewed** — the four committed deep reads, the end-to-end scenario pass, the verification-quality pass, and the issue-specific scope and acceptance accounting for 268 issues.
3. **Bookkeeping already corrected** — the issue read-count correction completed in iteration 16 for #98, #99 and #130; no further bookkeeping-only work is queued.

**A structural efficiency that reduces real issue work.** Every one of the 286 issues corresponds to a Roadmap leaf whose Scope, Done, Guide-reference and Depends text is already in Roadmap v1.18, a baseline document already read. Iteration 15 confirmed on #98 and #99 that the issue bodies carry their leaf faithfully. The issue batches therefore verify leaf-to-issue fidelity on a sample per phase and account the leaf text against the Guide and the findings register, rather than reading 268 issue bodies cold. Batch 17 establishes the shared boilerplate and template once so no later batch repeats it; if it finds that issues do **not** faithfully carry their leaves, the per-phase batches become substantially heavier and the count will rise.

**Queue shape.** Thirty batches total, one of which (batch 1) was executed in this iteration:

| Batches | Area | Content |
|---|---|---|
| 1 | research | IDR-043 remainder — **done this iteration** |
| 2-5 | research | The four committed deep reads: IDR-030 (841 lines), IDR-034 (728), IDR-039A (786), IDR-055 (437) |
| 6-14 | research | Key-section screen of the 47 reports whose coverage is not established, in nine topic clusters; batch 8 merges the peer-source spot checks and `fq-10` with the 014-series studies |
| 15 | scenarios | End-to-end scenario pass against the approved capability workflows |
| 16 | verification | Verification-quality pass, reusing batch 5's IDR-055 evidence |
| 17 | issues | Shared boilerplate, issue template and leaf-to-issue fidelity, checked once across all nine phases |
| 18 | issues | Whole-backlog dependency graph, coverage and sizing analysis |
| 19-29 | issues | Per-phase unique scope and acceptance accounting, split at group boundaries |
| 30 | final | Consolidated final assessment |

**Stated sizing uncertainties.** These are estimates for planning, not a completion guarantee:

- Batch 17's fidelity result is the single largest swing factor. Faithful issues keep batches 19-29 at eleven; unfaithful issues could add several.
- Phase 3 at 39 leaves in one batch is the tightest fit and may need to split, adding one.
- Any key-section screen that surfaces a substantive Guide question may need a targeted follow-on read, adding batches that cannot be predicted now.
- Batch 8's peer-source checks depend on external repositories whose availability and size are not known in advance.
- Batch 30 assumes the findings register is stable by then; a late material finding could require another consolidation pass.

**Twenty-nine batches remain after this iteration.**

## 6. Disposition summary for the checkpoint

| Item | Disposition | Where verified |
|---|---|---|
| IDR-043 Sections 5-14, 16-18, 21-22 and checklist | Fully read; report now fully read; all partial-report remainders are now closed | Section 2 above |
| IDR-043 invariants and prohibitions | Substantially adopted, several near-verbatim, with Guide line 633 citing this report directly | Section 2.1 above |
| `SyncEnvelopeV1` taxonomy, orthogonal state machines, `SyncConflictV1`, quarantine subsystem, review workflow, federation profiles | Not adopted; recorded scope choice stated at Guide line 620 | Section 2.2 above |
| IDR-043 §16.1 `428` row | Recorded departure already dispositioned at Guide 503/838/1139 | Section 2.3 above |
| F-17 stale acceptance wording | **Confirmed instance** with exact locations in IDR-043 (header lines 4/17-18 versus line 704 and line 765) | Section 3 above |
| Complete remaining batch queue | Thirty numbered batches written to `batch_queue` in the state file; twenty-nine remain | Section 5 above |

## 7. Statement of limits

This iteration read the committed remainder of one report and built the queue from the existing finish conditions and a mechanical extraction of the Roadmap's phase, group and issue mapping. It did not read any issue body, resolve `fq-10`, verify that the 286 issue bodies carry their Roadmap leaf text beyond the two confirmed in iteration 15, or begin any queued batch other than batch 1. The queue's sizing is an estimate derived from four completed iterations; it is not a completion guarantee, and Section 5 states the specific factors that could change the count. The F-17 instance is an editorial observation about a research report, not a technical defect in the Guide. `review_complete` remains `false`.

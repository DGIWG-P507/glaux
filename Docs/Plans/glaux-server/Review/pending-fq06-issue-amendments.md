# fq-06 issue amendments — delivery completed

These seven amendments belong to the fourteenth update in [the review action list](action-list.md#fourteenth-authorised-update--part-2-collection-exposure). All seven are **published to their issue bodies and verified**, completing all thirteen fq-06 amendments. Guide v1.16 / Roadmap v1.31 remain published at [664abdf](https://github.com/DGIWG-P507/glaux/commit/664abdf9de954d178d277aa1aaa7370207049c58). The six previously delivered amendments (#57/#80/#94/#100/#119/#121) were reconfirmed present once and open. This file retains its original path and saved texts as delivery history; it is no longer a pending queue.

**Verified delivery — 21 September 2026, 16:37:54 UTC:** following the lead's organisation-installation setup and new retry authorisation, the connected GitHub issue-edit operation succeeded. Each saved amendment below was prepended unchanged to its full live issue body after a pre-write comparison; exact complete-body readback verified preservation of earlier instructions and preparation pins. Titles, labels, assignees, milestones and open states are unchanged. Complete comment lists were checked before writing and at final verification and remained empty; no comment fallback was needed. No issue is complete or closed, and implementation/tests remain not started. Do not post these amendments again.

## Earlier access failures — historical record

On 21 September 2026, the execution policy blocked the second issue-update batch before publication. Subsequent readbacks found all seven bodies unchanged from the pre-edit snapshots. The connected GitHub update tool also returned `403 Resource not accessible by integration` for #122. No permission changes or alternate credential workarounds were attempted. No issue is closed or software implemented by these instructions.

## Earlier retry and comment fallback — 21 September 2026

The project lead explicitly authorised retrying body edits and using issue comments if edits are blocked, with sufficient time for responses. The connected GitHub tool completed both attempts on #122 with definitive `403 Resource not accessible by integration` responses: first the body edit, then comment creation. Neither was a timeout or an interrupted request. At **13:50:26 UTC**, readback of all seven issues and their comment lists confirmed unchanged bodies/open states and zero comments. No amendment was delivered by this retry, and no repeated comment writes were attempted on the other six after the permission denial.

Comment delivery was authorised as an alternative to prepending. The saved fallback instructions were to keep the substantive instructions and pinned links unchanged; immediately after the heading add: “This approved amendment is delivered as a comment at the project lead's request. It supplements the existing issue description and earlier amendments; preparation pins and completion checklists remain in place.” Replace the final paragraph with “Implementation and tests remain **not started**.” and omit the trailing divider. The original issue body would stay intact, with the exact verified comment URL recorded as delivery evidence. This alternative was not used; the issue-body deliveries above are complete.

## Retry safeguards used for the completed delivery

The completed retry read the current action list, each live issue and its complete comment thread; checked for an existing equivalent `664abdf` amendment; compared the live body immediately before writing; and verified the complete result afterwards. Saved text was added to the full body, never substituted for it. Each write's actual result was awaited. No access control was bypassed. Delivery method and issue URLs are recorded here and in the action list. The seven-item remainder is now empty; any next topic requires the next bounded authorisation, and this delivery does not authorise implementation.

## Issue #122 — [3.5.2] System Event and ordinary-JSON class cases

[Updated issue](https://github.com/DGIWG-P507/glaux-server/issues/122) — **delivered by body edit; exact readback verified 21 September 2026 at 16:37:54 UTC**. Preserved prepend text:

~~~~markdown
## Approved review-follow-up amendment — Part 2 collections (664abdf)

Guide v1.16 [§4.1.2](https://github.com/DGIWG-P507/glaux/blob/664abdf9de954d178d277aa1aaa7370207049c58/Docs/Plans/glaux-server/glaux-server-implementation-guide.md#412-part-2-collection-exposure) and [§7.2.1](https://github.com/DGIWG-P507/glaux/blob/664abdf9de954d178d277aa1aaa7370207049c58/Docs/Plans/glaux-server/glaux-server-implementation-guide.md#721-published-prerequisite-conflicts-and-qualified-test-results) control this bounded fq-06 clarification; [source record and action status](https://github.com/DGIWG-P507/glaux/blob/664abdf9de954d178d277aa1aaa7370207049c58/Docs/Plans/glaux-server/Review/action-list.md#fourteenth-authorised-update--part-2-collection-exposure). This selects the previously deferred collection exposure within existing task scopes and dependencies, not a new implementation iteration.

Include the implemented systemEvents/SystemEvent default in the Guide §4.1.2 and Part 2 requirement 44/A.44 evidence. Run the applicable A.2 inherited collection checks through the shared runner, retaining actual metadata/list/item outcomes and source qualifications. Use the family's ordinary JSON and occurrence-time/native-filter contract, not a generic GeoJSON/spatial substitute.

Independently seeded events across permitted Systems and a denied System must establish the exact type-wide default membership, narrower parent/custom views, metadata/links and individual collection-item/canonical identity. Check empty, missing and restricted cases, wrong marker and wrong target. Missing seeded members are setup/coverage gaps, not a passed empty canonical loop.

Retain the previous A.40 wrong-family adaptation and supplemental canonical-resource proof. Collection exposure does not resolve the event parent-rel qualification or close later command-dependent encoding obligations.

Implementation and tests remain **not started**. Preserve the original body, prior amendments, preparation pins and completion checklist below.

---

~~~~

## Issue #157 — [5.1.2] Expose ControlStream identities and associations

[Updated issue](https://github.com/DGIWG-P507/glaux-server/issues/157) — **delivered by body edit; exact readback verified 21 September 2026 at 16:37:54 UTC**. Preserved prepend text:

~~~~markdown
## Approved review-follow-up amendment — Part 2 collections (664abdf)

Guide v1.16 [§4.1.2](https://github.com/DGIWG-P507/glaux/blob/664abdf9de954d178d277aa1aaa7370207049c58/Docs/Plans/glaux-server/glaux-server-implementation-guide.md#412-part-2-collection-exposure) and [§7.2.1](https://github.com/DGIWG-P507/glaux/blob/664abdf9de954d178d277aa1aaa7370207049c58/Docs/Plans/glaux-server/glaux-server-implementation-guide.md#721-published-prerequisite-conflicts-and-qualified-test-results) control this bounded fq-06 clarification; [source record and action status](https://github.com/DGIWG-P507/glaux/blob/664abdf9de954d178d277aa1aaa7370207049c58/Docs/Plans/glaux-server/Review/action-list.md#fourteenth-authorised-update--part-2-collection-exposure). This selects the previously deferred collection exposure within existing task scopes and dependencies, not a new implementation iteration.

Expose /collections/controlstreams with exact itemType ControlStream under Guide §4.1.2 and Part 2 requirement 24. Reuse canonical identities and the existing collection machinery for a server-managed type-wide view across permitted parents; keep System/Deployment/custom views scoped independently.

From root-only discovery, verify inventory/detail consistency, items links and individual collection-item reads, required canonical links and exact IDs. Include permitted streams from different parents, a denied stream and an empty authorized view. Wrong-family IDs, an altered type marker or missing canonical link must be detected without leaking protected relationships.

Preserve descendant-association rules, applicable native endpoint contracts and actual supported-format advertising. Collection exposure does not require advanced filters or command execution early; those remain with their existing owners. No independent status/result collection or duplicate resource identity is added.

Implementation and tests remain **not started**. Preserve the original body, prior amendments, preparation pins and completion checklist below.

---

~~~~

## Issue #164 — [5.2.3] Expose asynchronous command POST and reads

[Updated issue](https://github.com/DGIWG-P507/glaux-server/issues/164) — **delivered by body edit; exact readback verified 21 September 2026 at 16:37:54 UTC**. Preserved prepend text:

~~~~markdown
## Approved review-follow-up amendment — Part 2 collections (664abdf)

Guide v1.16 [§4.1.2](https://github.com/DGIWG-P507/glaux/blob/664abdf9de954d178d277aa1aaa7370207049c58/Docs/Plans/glaux-server/glaux-server-implementation-guide.md#412-part-2-collection-exposure) and [§7.2.1](https://github.com/DGIWG-P507/glaux/blob/664abdf9de954d178d277aa1aaa7370207049c58/Docs/Plans/glaux-server/glaux-server-implementation-guide.md#721-published-prerequisite-conflicts-and-qualified-test-results) control this bounded fq-06 clarification; [source record and action status](https://github.com/DGIWG-P507/glaux/blob/664abdf9de954d178d277aa1aaa7370207049c58/Docs/Plans/glaux-server/Review/action-list.md#fourteenth-authorised-update--part-2-collection-exposure). This selects the previously deferred collection exposure within existing task scopes and dependencies, not a new implementation iteration.

Expose /collections/commands with exact itemType Command under Guide §4.1.2 and Part 2 requirement 30, using admitted canonical Commands. Its default membership is type-wide across permitted ControlStreams. Thus this default includes another parent's permitted Command; the original prohibition on unrelated-parent leakage still applies to parent-scoped lists and custom collections whose membership excludes it, not to this deliberately broader default.

Check root-to-inventory/detail/items/individual-item traversal, exact IDs/parameters and canonical links with independently specified multi-parent and denied fixtures. Exercise empty defaults and wrong-family items; detect a missing default, wrong marker or omitted access predicate. Keep status/result lists under their existing parents rather than inventing generic /collections families.

All admission/live, 201/Location/individual-status Content-Location and no-unintended-effect rules remain unchanged. Hold admitted work as this task already requires; no dispatch, advanced filters, synchronous work, later codecs or Feasibility implementation is pulled forward.

Implementation and tests remain **not started**. Preserve the original body, prior amendments, preparation pins and completion checklist below.

---

~~~~

## Issue #177 — [5.4.1] Admit and discover asynchronous feasibility requests

[Updated issue](https://github.com/DGIWG-P507/glaux-server/issues/177) — **delivered by body edit; exact readback verified 21 September 2026 at 16:37:54 UTC**. Preserved prepend text:

~~~~markdown
## Approved review-follow-up amendment — Part 2 collections (664abdf)

Guide v1.16 [§4.1.2](https://github.com/DGIWG-P507/glaux/blob/664abdf9de954d178d277aa1aaa7370207049c58/Docs/Plans/glaux-server/glaux-server-implementation-guide.md#412-part-2-collection-exposure) and [§7.2.1](https://github.com/DGIWG-P507/glaux/blob/664abdf9de954d178d277aa1aaa7370207049c58/Docs/Plans/glaux-server/glaux-server-implementation-guide.md#721-published-prerequisite-conflicts-and-qualified-test-results) control this bounded fq-06 clarification; [source record and action status](https://github.com/DGIWG-P507/glaux/blob/664abdf9de954d178d277aa1aaa7370207049c58/Docs/Plans/glaux-server/Review/action-list.md#fourteenth-authorised-update--part-2-collection-exposure). This selects the previously deferred collection exposure within existing task scopes and dependencies, not a new implementation iteration.

Expose /collections/feasibility with exact itemType Feasibility under Guide §4.1.2 and Part 2 requirement 39. This is a server-managed type-wide view across permitted ControlStreams, using the Command-shaped query/response contract for Feasibility resources, not executable Commands. Keep each parent's nested scope and custom memberships separate.

Verify root discovery, inventory/detail metadata, items links, individual collection-item reads and canonical /feasibility/{id} identity using independently expected admitted requests from two permitted parents and a denied request. Detect accidental itemType Command, a /commands target, a parent-only default query or leakage. Empty-but-implemented remains distinct from absent/unbuilt; subordinate status/result lists do not become independent collection entries.

Keep the earlier Command-live versus evaluator-admission distinction, atomic PENDING/analysis-work rules and zero Command/device effects. Running analysis, later filters, result publication and synchronous waiting remain outside this task.

Implementation and tests remain **not started**. Preserve the original body, prior amendments, preparation pins and completion checklist below.

---

~~~~

## Issue #196 — [5.5.16] Publish the independent tasking workflow

[Updated issue](https://github.com/DGIWG-P507/glaux-server/issues/196) — **delivered by body edit; exact readback verified 21 September 2026 at 16:37:54 UTC**. Preserved prepend text:

~~~~markdown
## Approved review-follow-up amendment — Part 2 collections (664abdf)

Guide v1.16 [§4.1.2](https://github.com/DGIWG-P507/glaux/blob/664abdf9de954d178d277aa1aaa7370207049c58/Docs/Plans/glaux-server/glaux-server-implementation-guide.md#412-part-2-collection-exposure) and [§7.2.1](https://github.com/DGIWG-P507/glaux/blob/664abdf9de954d178d277aa1aaa7370207049c58/Docs/Plans/glaux-server/glaux-server-implementation-guide.md#721-published-prerequisite-conflicts-and-qualified-test-results) control this bounded fq-06 clarification; [source record and action status](https://github.com/DGIWG-P507/glaux/blob/664abdf9de954d178d277aa1aaa7370207049c58/Docs/Plans/glaux-server/Review/action-list.md#fourteenth-authorised-update--part-2-collection-exposure). This selects the previously deferred collection exposure within existing task scopes and dependencies, not a new implementation iteration.

Add ControlStream, Command and Feasibility default-collection discovery to the existing independent tasking workflow under Guide §4.1.2. Verify controlstreams/ControlStream, commands/Command and feasibility/Feasibility descriptors, applicable items queries and individual collection-item/canonical identity using independent expectations. Run applicable A.2 and family collection tests over the actual candidate inventory; reuse earlier valid family/Property evidence without dropping any qualifying non-feature collection.

Use multi-parent permitted resources plus denied ones to separate type-wide defaults from parent/custom scopes. Check empty views, exact markers, native filters, paging and existing applicable formats. Feasibility's Command-shaped contract never changes its marker, canonical family or no-actuation boundary. Status/results and schemas remain on their existing routes, not independent generic collection entries.

Retain prior Req61 statusCode/history and supplemental canonical checks and all original/adapted labels. A missing selected default or wrong marker must fail, not merely remove a test iteration. Reuse the established suites, not a new test platform.

Implementation and tests remain **not started**. Preserve the original body, prior amendments, preparation pins and completion checklist below.

---

~~~~

## Issue #271 — [9.1.9] Reconcile declarations with candidate evidence

[Updated issue](https://github.com/DGIWG-P507/glaux-server/issues/271) — **delivered by body edit; exact readback verified 21 September 2026 at 16:37:54 UTC**. Preserved prepend text:

~~~~markdown
## Approved review-follow-up amendment — Part 2 collections (664abdf)

Guide v1.16 [§4.1.2](https://github.com/DGIWG-P507/glaux/blob/664abdf9de954d178d277aa1aaa7370207049c58/Docs/Plans/glaux-server/glaux-server-implementation-guide.md#412-part-2-collection-exposure) and [§7.2.1](https://github.com/DGIWG-P507/glaux/blob/664abdf9de954d178d277aa1aaa7370207049c58/Docs/Plans/glaux-server/glaux-server-implementation-guide.md#721-published-prerequisite-conflicts-and-qualified-test-results) control this bounded fq-06 clarification; [source record and action status](https://github.com/DGIWG-P507/glaux/blob/664abdf9de954d178d277aa1aaa7370207049c58/Docs/Plans/glaux-server/Review/action-list.md#fourteenth-authorised-update--part-2-collection-exposure). This selects the previously deferred collection exposure within existing task scopes and dependencies, not a new implementation iteration.

Reconcile the actual candidate against all six Guide §4.1.2 default IDs/itemType values, including inventory/detail/items/individual-item behavior and source-linked evidence. A candidate with a missing/unbuilt default is partial under this selected Glaux policy even though the standard's family collection availability is conditional. Do not remove a default merely to declare the test inapplicable.

Apply Part 2 A.2's literal itemType != feature selector to every actually exposed collection, including sosa:Property and any additional qualifying custom collection. Reuse evidence only when it is valid for this candidate/configuration and tested scope. Distinguish empty-but-advertised collections, unavailable/unbuilt families and caller-restricted visibility; seed authorized known members for item checks.

Detect a deliberately omitted default, wrong marker or qualifying collection left out of the inventory. Retain original/adapted/supplemental outcomes, inherited qualifications and earlier non-vacuous canonical evidence. This resolves the prior amendment's deferred exposure choice without erasing that historical record, introducing new classes or claiming certification.

Implementation and tests remain **not started**. Preserve the original body, prior amendments, preparation pins and completion checklist below.

---

~~~~

## Issue #288 — [9.4.3] Record the evidence-backed completion result

[Updated issue](https://github.com/DGIWG-P507/glaux-server/issues/288) — **delivered by body edit; exact readback verified 21 September 2026 at 16:37:54 UTC**. Preserved prepend text:

~~~~markdown
## Approved review-follow-up amendment — Part 2 collections (664abdf)

Guide v1.16 [§4.1.2](https://github.com/DGIWG-P507/glaux/blob/664abdf9de954d178d277aa1aaa7370207049c58/Docs/Plans/glaux-server/glaux-server-implementation-guide.md#412-part-2-collection-exposure) and [§7.2.1](https://github.com/DGIWG-P507/glaux/blob/664abdf9de954d178d277aa1aaa7370207049c58/Docs/Plans/glaux-server/glaux-server-implementation-guide.md#721-published-prerequisite-conflicts-and-qualified-test-results) control this bounded fq-06 clarification; [source record and action status](https://github.com/DGIWG-P507/glaux/blob/664abdf9de954d178d277aa1aaa7370207049c58/Docs/Plans/glaux-server/Review/action-list.md#fourteenth-authorised-update--part-2-collection-exposure). This selects the previously deferred collection exposure within existing task scopes and dependencies, not a new implementation iteration.

Carry Guide §4.1.2's six typed defaults and §7.2.1's actual non-feature collection inventory into the existing full-completion check. Reference #271's candidate-specific metadata/items/individual-item and applicable inherited evidence, including exposed Property/custom collections where the A.2 selector matches.

Do not call the full target complete when a selected default is absent, unbuilt, disabled in place of implementation or untested; distinguish that product requirement from the standard's optional exposure choice. Empty-but-implemented and access-restricted results are different cases, not blanket inapplicability. Preserve source adaptations and supplemental canonical checks rather than flattening them to an unqualified official-test pass.

Use the existing disposable missing-result/candidate-mismatch check to detect an omitted collection obligation. Record actual evidence and remaining owners without inventing runtime proof, new conformance classes or another evidence service. This supplements, rather than replaces, the earlier prerequisite-qualification amendment.

Implementation and tests remain **not started**. Preserve the original body, prior amendments, preparation pins and completion checklist below.

---

~~~~

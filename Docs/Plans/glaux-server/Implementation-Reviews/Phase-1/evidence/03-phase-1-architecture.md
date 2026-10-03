# 03 — Step 3: Phase 1 architecture

**Question:** Can the System write path, permission checks, HTTP boundary and per-issue proof programs grow into Phase 2 without copying everything?

**Date:** 3 October 2026<br>
**Authorised by:** the project lead's `proceed` after merging [PR #112](https://github.com/DGIWG-P507/glaux/pull/112), approving step 2. This authorizes step 3 only.<br>
**Performed by:** Codex (OpenAI), with separate source-analysis agents for application/storage and authentication/HTTP composition. Exact model version is not exposed in this session. Context was not fresh: the agents had earlier project context. A separate non-author agent reviews the actual documentation diff before publication; the delivery PR records the reviewed commit and outcome. This is not independent human expert review.<br>
**Examined:** server [`27955c1b9260cd811ad6bc08f85feab43ad65028`][Server]; planning [`145e0d40b869d56ae2568b485ffe8f50408ccf87`][Planning], including Guide v1.21 and Roadmap v1.40. Both checkouts were synchronized before inspection. All fifteen review gates were checked and remain open.

## Answer in brief

**The foundation is usable. This bounded source review found no blocking architecture defect and no reason to redesign Phase 1.** Identity verification, the HTTP envelope, source-byte storage and the pattern of making an accepted change in one database transaction are already reusable.

The implementation is not yet a multi-resource server underneath a System-shaped API. Its admission operations, revision bindings and retry records really are System-specific. That matches Phase 1's intended scope. Phase 2 must extend those pieces deliberately when it adds Procedures and Deployments, preserving the integrity and permission rules rather than copying the entire System implementation.

One low-severity maintenance finding is new: **P1-04, duplicated proof-program plumbing**. The creation and retrieval checks repeat process management and raw HTTP handling. Small test-only helpers would make future fixes less likely to diverge. This does not justify replacing the tests, adding a framework or starting a separate preparatory project.

**No code was changed or executed.** This review establishes source-level extension points, not runtime correctness, performance, full security assurance or conformance. Recommendations below are not adopted requirements.

## 1. What was checked

The sample followed the implemented path from runtime route assembly through authentication, resource admission, application transaction and durable records, then back through authorized retrieval. It inspected:

- the three production packages and their dependency direction;
- the relevant System POST/GET handlers, shared HTTP and discovery assembly, verified caller context and admission methods;
- create/update transaction orchestration, connection-scoped helpers and the identity, artifact, revision, audit, outgoing-work and retry schema bindings;
- the creation/read proof programs' fixture, child-process and wire-client helpers, their Python build/output wrappers, the restore wrapper and the retrieval fault-control runner;
- the approved Guide architecture and the Phase 2 tasks that will extend these paths.

This was a selected architectural read, not a full read of every source file, migration trigger, permission branch or proof assertion. The [step-2 audit](02-test-source-audit.md) remains the source-to-test assessment; no client study or completed research review was reopened. This step concerns shared server foundations, not a client application's preferred workflow.

## 2. What can already be reused

| Foundation | Evidence at the examined server commit | Assessment |
|---|---|---|
| Package boundaries | [Workspace and package manifests][Workspace]; [server modules][Modules]. | The intended three production crates exist. Domain has no HTTP/SQL dependency; standards depends on domain, not database writes or authentication. Application, persistence, HTTP and security remain server modules, as Guide §§2.2–2.3 directs. More packages are not needed merely because more families are coming. |
| Verified identity and shared policy | [Authentication][Authentication] lines 57–86, 216–217, 291–315; [authorization][Authorization] lines 106–159, 494–514; [configuration][Configuration] lines 144–156, 258–261. | Caller context has no public unchecked constructor. The authenticator can protect another router, and handlers share admission/policy state. Each new family need not implement token handling, policy parsing or its own denial limiter. |
| HTTP boundary | [Runtime][Runtime] lines 95–124; [HTTP boundary][Http] lines 308–382, 532–597; [System routes][SystemHttp] lines 77–95. | Runtime wraps the assembled routes once. Bounded parsing, safe errors and configured-root links are shared. The inspected System POST/GET paths go through admission before persistence/disclosure, not directly to trusted storage. |
| Atomic accepted writes | [Application][Application] lines 437–540, 680–764; [storage][Storage] lines 178–199; [revisions][Revisions] lines 100–114, 160–186. | One operation owns the transaction. Connection-scoped helpers let it commit identity, bytes, revision, audit, outgoing work and retry outcome together. The update path already reuses insertion helpers without forcing create and update into an artificial generic operation. |
| Canonical identity and original bytes | [Identity schema][IdentitySchema] lines 4–29; [artifact/revision schema][RevisionSchema] lines 1–26. | A shared resource identity precedes the family row. Original-byte artifacts have no System foreign key and can serve other families. A second unrelated identity namespace or Procedure-specific artifact store is unnecessary. The current family constraint still needs an explicit migration. |
| Authorized current view | [Authorized storage][AuthorizedStorage] lines 53–118, 173–224; [System response][SystemHttp] lines 393–406. | Visible-row selection applies permission rules before counts/limits; current retrieval selects related identity/artifact facts together. Extend that approach, while keeping each family's relationship rules. System-parent SQL is not automatically a Procedure or Deployment visibility rule. |

The low-level application and repository APIs are trusted internal interfaces, not authenticated entry points. Their public availability inside the server crate leaves a wiring obligation for future handlers; it does not establish a current bypass. The inspected System composition meets that obligation. Guide §5 already forbids privileged alternate ingestion paths.

**Transaction and pool check.** Create/update keep their accepted database changes inside their own transaction; the inspected blocks do not dispatch to an external device or broker. PostgreSQL normally retains acquired locks until transaction end, so extending these operations must not casually put remote work inside that interval ([PostgreSQL 18 locking documentation](https://www.postgresql.org/docs/18/explicit-locking.html)). Runtime lines 81–95 builds one bounded SQLx pool, currently two connections, shared with readiness checks. That is not a per-request connection factory, but neither this inspection nor that setting proves load capacity. No workload was measured and no new pooler or database service is recommended.

## 3. Where Phase 2 must extend, rather than clone

These are concrete implementation cautions within existing task ownership, not additional defect findings or new prerequisites.

| Extension point | What is specific today | Bounded recommendation and existing owner |
|---|---|---|
| Durable write records | `WriteReceipt.system_id`, `SystemRevision`, the System write head and create retry records are typed for Systems. [Application][Application] lines 57–88, 132–303, 387–423; [outgoing schema][OutgoingSchema] lines 45–60; [head schema][HeadSchema]; [retry schema][RetrySchema]. | Use the first Procedure path, **#50 / 2.3.2**, to extend the shared identity/artifact foundation and factor only genuinely common write/audit/outgoing/retry behavior. Carry the pattern into #51–#53; #59 and #67 own full replacement and conditional/retry integration. Preserve compound bindings proving that revision, artifact, accepted audit and event belong together; simply replacing typed keys with strings would weaken that protection. |
| Resource admission | `list_systems`, `get_system`, `current_system`, create/update admission and ownership selection are System operations. [Authorization][Authorization] lines 551–591, 661–683, 761–791; [authorized storage][AuthorizedStorage] lines 16–38. | At **#50/#51**, reuse caller/policy/denial plumbing but add typed family operations and relationship checks. Retain authorization before disclosure and before exposing a stored retry outcome. #56 later extends cross-family navigation. Do not bypass admission to reuse a convenient storage method. |
| Enabled routes and discovery | Shared route metadata exists, but runtime and discovery still separately assemble their surface from one `system_creation_enabled` choice. [Runtime][Runtime] lines 105–124; [discovery][Discovery] lines 44–86, 277–293, 341–345, 668–695. | As **#49/#50** expand the real surface, evolve the existing ordinary metadata so handler installation and API descriptions consume the same enabled-operation choices. Keep independent listener-versus-document tests. #68 owns complete method/OPTIONS/OpenAPI parity, not permission to advertise inaccurate routes until then. No new registry service is proposed. |

These recommendations fit [Roadmap][Roadmap] group 2.3's instruction to extend the persisted slice, not make a parallel store, and [Guide][Guide] §§2.2–2.3/§5. They do not call for a generic resource framework before Phase 2 starts. The current minimal System projection and limited representation support are intentional Phase 1 boundaries, not failures to implement already-promised Phase 2 behavior.

## 4. P1-04 — Share proof plumbing, not expected answers

There are fourteen `*-proof.rs` example programs. That is an inventory, not a quality score or a prediction of future CI duration. This finding rests on inspected duplication in two of them:

- [System creation proof][CreateProof] lines 37–186 and [System read proof][ReadProof] lines 39–189 repeat owned temporary files, private file permissions, child startup, bounded readiness/exit waits, output capture and shutdown cleanup.
- Their raw HTTP helpers ([creation][CreateProof] lines 214–272; [read][ReadProof] lines 208–271) repeat response bounds and parsing. There is a legitimate difference: the read helper handles HEAD's absent body separately. Any shared helper must preserve that behavior rather than erase it.
- The Python [create][CreateRunner], [read][ReadRunner] and [restore][RestoreRunner] wrappers repeat building a specific example and validating required ordered output markers. Sharing already exists: read reuses the independent signature fixture, restore reuses `require`, and disposable database lifecycle lives in [one harness][DatabaseHarness]. This is incremental cleanup, not a missing test infrastructure project.

**Why it matters:** a fix to child cleanup, timeout handling or wire parsing currently has multiple copies to find. More capability tests can multiply those maintenance points even when their actual assertions differ appropriately. No wrong test outcome or measured delay is attributed to this duplication.

**Recommendation:** when #49/#50 extend these API proofs, extract small test-only process, fixture and wire helpers where the behavior is truly shared. Keep expected fields, permitted identities, selection results and source-derived bytes explicit in the capability tests; never import production response types or serializers to decide what the answer should be. Share mechanics, not the answer generator. Preserve ordered completion markers, failure/unrun distinctions and the [retrieval fault controls][ReadFaults]. Update mutation targets and execution inventories coherently if a helper moves; an obsolete target must remain a failure, not become a passing control. #80 is the later conformance-runner integration point, not a reason to duplicate helpers until then.

This is a **Low, open recommendation**, not a new required issue or permission to reduce coverage. The existing CI lanes and required gate are unchanged. CI duration and broader mutation/security effectiveness belong to their already-defined review steps, not a new forecast from this source sample.

## 5. Conclusion and next step

Step 3 is complete at its stated source-level scope. Keep the approved architecture; extend the family-specific operations within their existing issues. Record P1-04 for the project lead's decision. No server fix, task edit, planning requirement, tool installation or gate closure was made here.

**Two review steps remain.** The next `proceed` is **step 4's bounded proposal for test strength and security scanning**. It is not authorization to install tools, run new campaigns or start Phase 2. Step 5 remains the separately authorized pinned OpenSensorHub comparison.

No human architecture/security expert was available. This OpenAI review does not complete the charter's task-by-task implementing-provider and cross-provider accounting; that remains part of the full gate review. It is not a basis to claim that OpenAI-authored work received a different-provider review. [Gate 1 (#339)](https://github.com/DGIWG-P507/glaux-server/issues/339) stays open, and only the project lead may close it after the remaining review work and recorded limitations are considered.

[Server]: https://github.com/DGIWG-P507/glaux-server/tree/27955c1b9260cd811ad6bc08f85feab43ad65028
[Planning]: https://github.com/DGIWG-P507/glaux/tree/145e0d40b869d56ae2568b485ffe8f50408ccf87
[Guide]: https://github.com/DGIWG-P507/glaux/blob/145e0d40b869d56ae2568b485ffe8f50408ccf87/Docs/Plans/glaux-server/glaux-server-implementation-guide.md
[Roadmap]: https://github.com/DGIWG-P507/glaux/blob/145e0d40b869d56ae2568b485ffe8f50408ccf87/Docs/Plans/glaux-server/glaux-server-roadmap.md
[Workspace]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/Cargo.toml
[Modules]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/src/lib.rs
[Authentication]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/src/authentication.rs
[Authorization]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/src/authorization.rs
[Configuration]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/src/configuration.rs
[Runtime]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/src/runtime.rs
[Http]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/src/http_boundary.rs
[SystemHttp]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/src/system_http.rs
[Application]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/src/application.rs
[Storage]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/src/storage.rs
[Revisions]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/src/revisions.rs
[AuthorizedStorage]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/src/authorization_storage.rs
[Discovery]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/src/discovery.rs
[IdentitySchema]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/migrations/0002_system_identity.sql
[RevisionSchema]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/migrations/0004_system_revisions.sql
[OutgoingSchema]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/migrations/0006_audit_outgoing_work.sql
[HeadSchema]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/migrations/0007_conditional_system_writes.sql
[RetrySchema]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/migrations/0008_system_create_retry.sql
[CreateProof]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/examples/system-create-proof.rs
[ReadProof]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/crates/glaux-server/examples/system-read-proof.rs
[CreateRunner]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/scripts/test_system_create.py
[ReadRunner]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/scripts/test_system_read.py
[RestoreRunner]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/scripts/test_system_restore.py
[DatabaseHarness]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/scripts/database_harness.py
[ReadFaults]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/scripts/test-system-read-failures.py

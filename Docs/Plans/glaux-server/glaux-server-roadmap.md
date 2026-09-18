# Glaux Server Roadmap

**Version:** 0.1<br>
**Date:** 18 September 2026<br>
**Effort:** Glaux Server<br>
**Status:** Draft — complete first drafting pass; review and finalization next<br>
**Depends On:** [Goal and Definition v1.7](glaux-server-goal-and-definition.md), Approved<br>
**Implements:** [Implementation Guide v1.0][Guide], Baselined<br>
**Implementation status:** Tasks below are planned, not verified complete. This iteration creates the Roadmap, not server software.

## 1. Purpose and Executive Summary

This Roadmap turns the existing server design into an ordered implementation plan. The Goal defines scope; the Guide defines behavior, architecture and verification; this document assigns that work to phases and tasks. It does not introduce another requirements specification, decision-record system or research program.

Build the full Rust CSAPI reference server in nine phases. Establish a working HTTP/PostgreSQL path first, then complete descriptions, observations and encodings, tasking, publication, the selected sampling/filtering additions, and recovery. Finish with integrated verification and a reproducible reference release candidate. Tests, authorization, source/schema preservation, documentation and independent-client checks accompany the capabilities; they are not all postponed to the final phase.

The complete target remains all 25 direct CSAPI Parts 1/2 classes and applicable prerequisites, experimental Part 3 publication, the selected static Part 4 Point/Curve/Surface types, and all six selected Features Part 3/CQL2 classes. Part 5 implementation remains deferred. Required SWE Binary is not deferred or replaced by Protobuf. Provenance and quality remain part of the existing model, validation and access paths, not a separate platform.

### 1.1 Phase overview

The numbered order is the default execution order. The dependency column identifies actual prerequisites, so independent work need not acquire artificial dependencies on unrelated phases.

| Phase | Main result | Prerequisites | Planned tasks | Status |
|---|---|---|---:|---|
| 1. Running foundation and first registration | Buildable service; authorized System creation/read through real storage; initial tests and restore proof | Goal/Guide baseline and approved local prerequisites | 5 | Planned |
| 2. Complete Part 1 resources | Descriptions, relationships, collections, representations, native filters and required writes | Phase 1 | 6 | Planned |
| 3. Observations, status and System Events | Schema-bound ordinary-JSON ingestion, retrieval and native querying | Phase 2 | 5 | Planned |
| 4. Complete SWE payload encodings | SWE JSON, Text and Binary observation paths with exact value preservation | Phase 3; shared component model from Phase 2 | 4 | Planned |
| 5. Commands and feasibility | Full synchronous/asynchronous tasking, status/results and required mutations | Phase 2 and Phase 4's shared contracts/codecs | 5 | Planned |
| 6. Live publication | Ordered committed log, authenticated SSE and experimental MQTT binding | Phases 3–5 for the covered resource families | 4 | Planned |
| 7. Experimental sampling and enhanced filtering | Three selected Part 4 types and six-class observation filtering | Phases 2–4; not dependent on tasking or MQTT | 4 | Planned |
| 8. Exchange, interrupted operation and restore | Bounded administrative exchange, complete recovery workflows and operational examples | Phases 5–7; log/SSE from Phase 6 | 4 | Planned |
| 9. Integrated verification and release readiness | Full-target evidence, external-client workflows and clean reference setup | Phases 1–8 | 4 | Planned |

There are 41 planning tasks. A task can require several implementation iterations; neither a task nor a phase is a promise of one `proceed`. Section 5 explains how to select a bounded next slice without creating another planning layer.

**Effort estimate:** Not yet calibrated. The template's phase/task effort fields are recorded as uncalibrated below because there is no measured implementation rate for this server. Complexity labels identify technical risk, not hours. Do not infer calendar duration, staffing, lines of code or a fixed number of assistant turns from task counts. Record observed effort from the first implementation slices and use it to add defensible ranges as work progresses; missing speculative estimates do not authorize scope reduction.

### 1.2 Scope boundaries

Implementation belongs in `DGIWG-P507/glaux-server`; this repository retains the planning documents. Inspect the server repository before implementation and preserve existing work. The Guide's earlier repository inspection is historical, not permission to recreate or overwrite a repository.

Provide the small three-package Rust workspace and PostgreSQL/PostGIS design already selected. A broker is needed only for the experimental MQTT adapter. Operational identity infrastructure, devices, federation agreements, hosting, accreditation and other Glaux products are not deliverables or prerequisites for a working reference example. A deterministic device adapter and synthetic data are sufficient for the planned reference tests.

## 2. Planning Assumptions and Constraints

- Goal v1.7 and Guide v1.0 control this draft. Research is supporting evidence; its proposed mechanisms are not additional tasks unless the Guide selected them.
- Each implementation task includes its tests, safe error/access behavior, API/example updates and relevant source/test identifiers. A passing happy path alone is not task completion.
- Preserve original standards/schema artifacts and the Guide's explicit interpretations. Do not silently adopt moving drafts or claim that a project interpretation resolved an upstream ambiguity.
- Use a real PostgreSQL/PostGIS test instance and independent HTTP checks from the first persistent slice. Keep expected values independent of server serializers and query code.
- Pin actual build dependencies and test tools when they compile and pass the relevant checks. Research-time versions are not a tested lockfile.
- Software installation on the project lead's company laptop requires the appropriate organizational process. Inspect availability; document missing prerequisites and seek direction rather than installing them implicitly. Docker is an example-deployment option, not the only possible way to supply PostgreSQL/PostGIS.
- The ordinary reference setup must work without another Glaux application, an operational sensor or a production identity provider. Network-facing authentication still requires the configured verified-token contract; development identities remain explicitly limited.
- A limited increment may be useful without being conformant to a complete class. Its advertised routes, formats, documentation and declarations must describe only what is enabled and verified.
- No production performance target or calendar deadline has been supplied. Measure representative workloads and state the environment; do not manufacture a service-level commitment.

## 3. Dependencies and Blocking Conditions

### 3.1 Execution dependencies

Follow the phase overview by default and the task prerequisites below when scheduling parallel work. Shared foundations precede their consumers:

- Operation-aware validation, typed identities, access checks, migrations, audit and transactional outgoing work precede the first public mutation.
- The shared SWE component model and immutable schema binding precede storing values under those contracts. JSON-only increments do not eliminate Text/Binary work.
- Native filtering accompanies each resource family. Enhanced CQL2 filtering is additional; it cannot substitute for either CSAPI Advanced Filtering class.
- Tasking depends on durable work and shared codecs, not on SSE or MQTT. No broker is placed in the command-execution path.
- SSE and MQTT consume the ordered committed log. Their implementations can proceed independently once that log exists. SSE snapshot bootstrap is completed in Phase 8, not assumed from live/replay success in Phase 6.
- Experimental Part 4 types and enhanced observation filtering share generic SamplingFeatures but are not prerequisites for one another. Task 7.2 can use approved generic SamplingFeatures without waiting for 7.1.
- Backup/restore starts in Phase 1 and grows with persistent capabilities. Phase 8 completes cross-capability recovery; it is not the first restore test.

### 3.2 Blocking conditions

Block only work that depends on a failed prerequisite: unavailable approved tools/storage, a dependency that fails its required semantic proof, a standards contradiction that invalidates the selected implementation, or a security/recovery defect that makes the next action unsafe. Record the specific affected task and the smallest next action. Continue independent authorized work where possible.

Do not bypass failed validation, replace exact values with rounded ones, repeat uncertain device actions, or declare a class merely to pass a phase. If a bounded implementation proof disproves a selected design, update the affected Guide section and task; do not automatically reopen the research collection. A new capability or changed project scope requires the project lead's direction and the existing Goal/Guide change process.

## 4. Phase Plan

All tasks initially have **effort estimate: uncalibrated**. Each numbered task includes scope/deliverables, verification and prerequisites; its complexity is stated beside the title. Guide section references identify the controlling technical design, not a second specification.

### Phase 1: Running foundation and first registration

**Goal:** Prove a useful, authorized request-to-database path before building the whole model.<br>
**Dependencies:** Approved Goal and baselined Guide; existing repository and approved tools inspected.<br>
**Estimated effort:** Uncalibrated; five tasks.<br>
**Complexity:** High because later work depends on these contracts.<br>
**Guide:** §§2–4.3, 4.6–4.7, 4.10, 4.12, 7–8.

1. **1.1 Establish the build and test environment — Medium.**
   - Scope/deliverables: Inspect existing server work; establish `glaux-domain`, `glaux-standards`, `glaux-server`, the Rust toolchain/lockfile, database test configuration and CI formatting/lint/build/test checks. Record dependencies, licenses and prerequisites without installing unapproved software.
   - Verification: Reproduce the documented build, run a real database connectivity/migration check, and distinguish tests run from unavailable checks.
   - Dependencies: Existing repository/prerequisite inspection; no other implementation task.
2. **1.2 Prove shared validation and value primitives — High.**
   - Scope/deliverables: Package pinned standards/schema artifacts with offline allowlisted resolution; establish typed IDs, exact number/time handling and operation/direction validation. Create the independently specified fixture/test foundation, including recursive SensorML/SWE examples and source-conflict fixtures.
   - Verification: Prove recursive resolution without network/file escape, generated versus writable fields, malformed/deep input limits, exact numeric/time boundaries and calendar/offset/leap-second handling before committing to storage types.
   - Dependencies: 1.1. Full component-family implementation follows in 2.1; a simple parser pass is not that proof.
3. **1.3 Establish the transactional storage boundary — High.**
   - Scope/deliverables: Add initial migrations and shared identity, revision/artifact, relationship, audit, retry and outbox/work storage. Implement short application transactions, conditional-write handling and rollback; add family tables with their later capabilities rather than creating empty subsystems.
   - Verification: Real-database tests establish atomic resource/revision/audit/outbox commit, no effect after rejected input, identity constraints, stale preconditions and concurrent/repeated writes. A transport worker is not needed to inspect durable outgoing work.
   - Dependencies: 1.1–1.2.
4. **1.4 Establish HTTP, discovery and access enforcement — High.**
   - Scope/deliverables: Implement Axum routing, safe origin/link construction, problems/negotiation, initial landing/API/conformance documents, typed configuration and health endpoints. Add verified JWT caller context, explicit loopback development identities and the configured action/resource policy interface.
   - Verification: Independent HTTP tests cover authentication failures, two permitted/denied sources, unsafe configuration rejection, unavailable identity/storage, forged forwarding headers and accurate partial declarations. No unverified route becomes public by default.
   - Dependencies: 1.1–1.3; domain-specific access cases grow with later tasks.
5. **1.5 Deliver the first persisted System workflow — Medium.**
   - Scope/deliverables: Implement a standards-shaped System POST/GET slice through the actual validation, authorization and storage boundary; add root-to-resource examples and an isolated backup/restore procedure for the data already present.
   - Verification: An independent HTTP client creates a System, follows its canonical link, reads it after restart and after isolated restore; invalid/denied input leaves no resource or outgoing work. No complete Part 1 class is claimed merely from this slice.
   - Dependencies: 1.2–1.4.

**Phase exit:** The documented build and first real workflow pass with safe negative cases and an initial restore proof. Exactness/recursive-validation failures affecting subsequent storage are resolved or explicitly block that dependent work. The server remains an accurately described partial implementation.

### Phase 2: Complete Part 1 resources

**Goal:** Complete connected-system descriptions, relationships, representations and required Part 1 operations.<br>
**Dependencies:** Phase 1.<br>
**Estimated effort:** Uncalibrated; six tasks.<br>
**Complexity:** High.<br>
**Guide:** §§4.1–4.3, 4.6–4.7, 4.10, 6.1–6.3, 7.1, 13.

1. **2.1 Complete the shared SWE component model — High.**
   - Scope/deliverables: Implement all applicable scalar/range, record/vector, choice, array/matrix and geometry descriptions, references, units, constraints, nils and quality. Compile immutable contracts with bounded traversal and retain source artifacts.
   - Verification: Positive/negative fixtures cover every family, nested combinations, order, exact values, local dynamic-quality references, all four quality types, missing/nil distinctions and recursive abuse. Unknown-member acceptance is not semantic validation.
   - Dependencies: 1.2–1.3; execution slices should group related component families with their tests.
2. **2.2 Complete SensorML and GeoJSON representation mappings — High.**
   - Scope/deliverables: Implement applicable SimpleProcess, PhysicalSystem, Deployment and DerivedProperty mappings/dependencies, source preservation and supported input/output bindings. Keep Procedure and Property representation differences explicit.
   - Verification: Independent expected meanings establish alternate-representation identity/relationship preservation, allowed extensions, rich source retention and documented mapping limitations; no request-time external dereference.
   - Dependencies: 2.1 and the Phase 1 request/response validation boundary.
3. **2.3 Complete Part 1 resources, hierarchy and collections — High.**
   - Scope/deliverables: Extend Systems and implement Subsystems, Deployments/Subdeployments, Procedures, generic SamplingFeatures and Properties with canonical/nested routes and typed relationships. Implement collection metadata/views and the required member operations without adding an unrelated collection-management API.
   - Verification: Register the Guide's connected fixture set; prove canonical identity across views, UID conflicts, parent/cardinality/cycle rules, recursion and authorized membership.
   - Dependencies: 2.2; extend the real persisted slice, not a parallel resource store.
4. **2.4 Complete required writes and representation-safe updates — High.**
   - Scope/deliverables: Implement Part 1 POST/PUT/PATCH/DELETE and collection-member semantics using the selected transaction draft. Cover ignored body-local IDs, protected domain fields, writable projections, lossless cross-format replacement and retained private revisions/tombstones.
   - Verification: Test all required resource families, null/array patches, omitted-field removal, lossy replacement rejection, conditional writes, retry-key conflicts and authorized deletion effects atomically.
   - Dependencies: 2.3 and 1.3; do not confuse collection removal with resource deletion.
5. **2.5 Complete Part 1 native filtering and paging — High.**
   - Scope/deliverables: Implement family-specific ID/keyword, hierarchy/relationship/property, spatial and temporal predicates, Unicode matching, scoped counts and deterministic next links. Add measured indexes where supported by plans.
   - Verification: Assert exact independently expected IDs, recursive versus direct results, property derivation, geometry-less cases, antimeridian/height inputs, non-ASCII text, paging changes and protected-data non-disclosure.
   - Dependencies: 2.3; query fixes and access tests accompany each resource implementation, not only this completion task.
6. **2.6 Verify the full Part 1 workflow — Medium.**
   - Scope/deliverables: Complete applicable inherited/direct tests, root-to-resource API documentation, examples and initial OS4CSAPI/independent-Python-client checks. Extend backup/restore to these resources and artifacts.
   - Verification: Exercise all 13 Part 1 classes and applicable dependencies; record genuine failures/interpretations without marking skips as passes. Confirm setup does not require another Glaux application.
   - Dependencies: 2.1–2.5. Advertise a class only when its own complete obligations pass.

**Phase exit:** Part 1 capabilities, writes and native filters have their complete test coverage and documented interpretations. Full shared component support is present; payload codec delivery remains explicitly assigned to Phase 4.

### Phase 3: Observations, status and System Events

**Goal:** Deliver usable schema-bound observation workflows and temporal status/event meaning in ordinary CSAPI JSON.<br>
**Dependencies:** Phase 2.<br>
**Estimated effort:** Uncalibrated; five tasks.<br>
**Complexity:** High.<br>
**Guide:** §§4.3–4.7, 4.10, 6–8, 13.

1. **3.1 Register and manage DataStreams and expose schemas — High.**
   - Scope/deliverables: Implement nested DataStream registration with one write-only schema, compiled contracts, derived formats/summaries and schema discovery. Provide required stream PUT/PATCH/DELETE and canonical/nested/collection and associated-feature navigation.
   - Verification: Minimal valid requests need no generated fields; schema-selector/media errors remain distinct; generated responses and equivalent schemas describe actual available codecs. Test stream mutations, with populated-stream restrictions and cascade integration completed in 3.2. Do not advertise not-yet-implemented Phase 4 formats.
   - Dependencies: 2.1–2.4.
2. **3.2 Implement observation storage and required writes — High.**
   - Scope/deliverables: Add observation identity, immutable contract revision, producing/feature context, exact phenomenon/result times and ordinary-JSON values. Implement required creation/replacement/patch/deletion, bounded supported multi-record ingestion and source/quality context preservation.
   - Verification: Invalid schema/value/parent or denied writes have no mutation/event; successful writes commit the complete context together. Test retries, schema freezing after data exists, correction history, populated-stream deletion and authorized cascade.
   - Dependencies: 3.1 and shared transaction/access primitives.
3. **3.3 Implement observation and stream queries — High.**
   - Scope/deliverables: Add required native temporal, spatial/relationship and observed-property queries, recursive `foi`, scoped latest selection/ties, summaries and changing-view pagination.
   - Verification: Assert exact IDs before pagination, delayed/tied times, local/external reference handling and authorized counts/latest results; inspect real database plans without assuming a page limit bounds all work.
   - Dependencies: 3.2 and 2.5. Enhanced CQL2 is separate Phase 7 work.
4. **3.4 Implement status observations and System Events — High.**
   - Scope/deliverables: Use ordinary datastreams for status and dynamic feature properties; preserve meaningful evidence time/freshness. Implement System Event representations, parent links, queries and required writes under the Guide's selected mapping.
   - Verification: Fresh, delayed, stale, missing and conflicting evidence never turns API uptime into device availability. Real events, HTTP audit and resource notifications remain distinct; test event media/field interpretations and access.
   - Dependencies: 3.2–3.3 and Part 1 relationships.
5. **3.5 Verify JSON observation workflows and continuity — Medium.**
   - Scope/deliverables: Complete known-dataset HTTP/external-client examples, applicable conformance tests, observation/provenance disclosure checks and backup/restore for the expanded data. Add bounded representative ingestion/query measurements.
   - Verification: Independent clients discover the stream schema, publish/read known values and retrieve exact native-filter results; restore preserves identity, time, contracts and source artifacts. Record any client subset limitation separately from server correctness.
   - Dependencies: 3.1–3.4. Part 2 classes spanning commands cannot close until Phase 5.

**Phase exit:** Ordinary-JSON observation, status and System Event workflows pass with actual storage and negative tests. This is not full Part 2 completion or a permanent JSON-only scope.

### Phase 4: Complete SWE payload encodings

**Goal:** Complete the required JSON, Text and Binary payload codecs and observation integration.<br>
**Dependencies:** Phase 3 and Phase 2's complete shared component model.<br>
**Estimated effort:** Uncalibrated; four tasks.<br>
**Complexity:** High.<br>
**Guide:** §§4.3–4.4, 4.7, 6.2, 7.1, 8, 13.

1. **4.1 Implement SWE JSON payloads — High.**
   - Scope/deliverables: Implement record/schema wrappers, object/array representation rules and all applicable component-value behavior distinct from ordinary CSAPI JSON wrappers.
   - Verification: Independently authored fixtures cover nested structures, choices, optional/positional nulls, nil reasons, large numbers and the Guide's array-flag interpretation.
   - Dependencies: 2.1 and 3.1–3.2.
2. **4.2 Implement SWE Text payloads — High.**
   - Scope/deliverables: Implement applicable delimiters, ordering, scalar formatting, composite/variable records and framing from compiled contracts; retain the actual media type rather than renaming it CSV.
   - Verification: Decode/encode known fixtures independently, including malformed/truncated input, numeric/time precision, optional/nil fields and bounded parser costs.
   - Dependencies: 2.1 and 3.1–3.2; not dependent on the JSON serializer's output as an oracle.
3. **4.3 Implement SWE Binary payloads — High.**
   - Scope/deliverables: Implement the applicable descriptor and value operations, byte order, sizes, choices, framing and supported encoding features across the complete target. Keep allocation/decompression bounded.
   - Verification: Use independently specified byte fixtures, boundary/truncation/length cases and composite values. Descriptor acceptance alone cannot pass this task or its class.
   - Dependencies: 2.1 and 3.1–3.2; no Part 5/Protobuf dependency.
4. **4.4 Integrate all codecs and format negotiation — High.**
   - Scope/deliverables: Wire formats into observation reads/writes and schema discovery, derived format lists and shared command-parameter interfaces. Implement declared SWE media aliases consistently with the Guide, preserving source/quality and exact meaning across supported conversions.
   - Verification: Compare independent expected values across all formats, schema wrappers and selected response media; test unsupported conversions/errors, JSON patch restrictions on opaque content and malformed bodies. Record measured codec limits.
   - Dependencies: 4.1–4.3 and 3.5. Full command-side HTTP evidence follows in Phase 5 before the three Part 2 SWE classes close.

**Phase exit:** All applicable codecs and observation wire paths are implemented and tested, not merely stubs. Difficult component/encoding combinations remain required; any failed conformance obligation is an explicit unresolved implementation task.

### Phase 5: Commands and feasibility

**Goal:** Complete tasking as a first-order server capability using deterministic reference equipment.<br>
**Dependencies:** Phase 2 and Phase 4 shared contracts/codecs; durable work/access boundary from Phase 1. No transport/broker prerequisite.<br>
**Estimated effort:** Uncalibrated; five tasks.<br>
**Complexity:** High.<br>
**Guide:** §§4.6–4.7, 4.9–4.10, 6.2–6.4, 7–8, 13.

1. **5.1 Implement ControlStreams and command contracts — High.**
   - Scope/deliverables: Add registration, required ControlStream PUT/PATCH/DELETE, canonical/nested discovery, schemas and the actual supported formats; bind parameters and applicable result contracts. Implement schema-freeze and default-versus-authorized-cascade deletion rules. Separate submitter identity/authority from authorized reporting.
   - Verification: Test minimal creates, stream mutations/deletion rules, omitted `cmdFormat`, explicit unsupported selectors, schema freezing, all applicable payload formats and derived metadata without client impersonation. Exercise populated-stream races with tasking in 5.3/5.5.
   - Dependencies: 2.3–2.4 and 4.4.
2. **5.2 Implement durable asynchronous command execution — High.**
   - Scope/deliverables: Add command/attempt storage, deterministic adapter, initial durable status, allowed lifecycle/report handling and results. Recheck authority/contract/deadline before dispatch; retain uncertainty instead of guessing an external effect.
   - Verification: Exact status codes/time rules, duplicate/concurrent retries, reporter denial, crash boundaries and uncertain-effect reconciliation pass against real transactions and the reference adapter.
   - Dependencies: 5.1 and 1.3; no live operational device required.
3. **5.3 Implement synchronous submission and recovery — High.**
   - Scope/deliverables: Add private durable admission, bounded HTTP waiting, one authoritative terminal outcome/publication and the Guide's exact success/conflict/error response contract. Integrate private admission with the ControlStream cascade rules from 5.1 and retry identity without requiring it from all CSAPI clients.
   - Verification: Independent HTTP clients check `201`, Location/Content-Location and status body; disconnect, timeout, same-key rejoin, lost response, unkeyed ambiguity and concurrent cascade do not duplicate dispatch or fabricate failure.
   - Dependencies: 5.2; share the executor boundary rather than implementing a second task engine.
4. **5.4 Implement feasibility with its own outcomes — High.**
   - Scope/deliverables: Implement synchronous/asynchronous Feasibility resources, parameters, status/results, discovery and reference-adapter analysis using existing machinery.
   - Verification: A successful analysis can return an infeasible answer without actuation; failed analysis, forbidden statuses, supported result contracts and exact response paths pass. Feasibility never grants execution authority or reserves a device implicitly.
   - Dependencies: 5.1–5.3.
5. **5.5 Complete tasking mutations, filtering and end-to-end proof — High.**
   - Scope/deliverables: Complete required Command/Feasibility/status/result PUT/PATCH/DELETE, native command/status/controlled-property filtering, cancellation and cascade handling. Extend clients, API docs, backup/restore and conformance tests across every required encoding and resource family.
   - Verification: Terminal-report correction/deletion does not reopen execution; cancellation differs from deletion; parent/schema races do not resurrect resources. Restore holds uncertain work and accounts for missing post-backup receipts. All Part 2 target/dependency tests now have implemented paths, including subordinate writes.
   - Dependencies: 5.1–5.4, 3.3–3.5 and 4.4.

**Phase exit:** Full command and feasibility workflows pass through a deterministic adapter and independent HTTP checks, with all required formats/mutations. Public lifecycle, internal execution evidence and uncertain physical effects remain distinct. No universal device-safety or accreditation claim follows.

### Phase 6: Live publication

**Goal:** Deliver committed events through the selected SSE and experimental MQTT contracts.<br>
**Dependencies:** Phases 3–5 for covered resource families; transactional outbox from Phase 1.<br>
**Estimated effort:** Uncalibrated; four tasks.<br>
**Complexity:** High.<br>
**Guide:** §§4.8, 4.10–4.12, 6.5, 7.3, 8.

1. **6.1 Implement the ordered retained publication log — High.**
   - Scope/deliverables: Implement atomic outbox handoff, immutable event/revision references, serialized replay positions, bounded workers/backoff and finite retention with recovery epochs.
   - Verification: Crash/restart before and after handoff/send proves no skipped committed position and tolerable duplicates within retained history. A database sequence alone is not commit-order proof.
   - Dependencies: 1.3 and existing mutation paths; complete producer coverage after Phase 5.
2. **6.2 Implement authenticated SSE and retained replay — High.**
   - Scope/deliverables: Implement the Guide's exact scope selector, event/checkpoint output, authenticated-encrypted cursors, policy rechecks, bounded subscriber buffers and pre-stream error behavior. Provide an authenticated-fetch browser example.
   - Verification: Scope/caller binding, tampering, expiry, policy changes, slow consumers, deletes and ordered resume pass. Snapshot bootstrap remains explicitly assigned to 8.2/8.4; subscription is not a snapshot.
   - Dependencies: 6.1 and shared access rules.
3. **6.3 Implement the pinned experimental MQTT binding — High.**
   - Scope/deliverables: Pin/test rumqttc and the Mosquitto example; implement the Guide's topics, format/event tokens, CloudEvents/native payloads, outbound QoS 1/non-retained delivery and generated AsyncAPI discovery. Keep the adapter disabled by default.
   - Verification: Actual messages/channels match the description; broker outage/reconnect and duplicates are safe; whole-topic audiences, credentials, wildcard/expiry/revocation and queued-data behavior are enforced. No inbound writes/commands or approved Part 3 claim is introduced.
   - Dependencies: 6.1, 4.4 and Phase 5 resource events; independent of SSE implementation.
4. **6.4 Verify transport isolation and live examples — Medium.**
   - Scope/deliverables: Add end-to-end examples for observations/status, System Events and resource lifecycle notifications; document delivery limits and optional broker startup/diagnostics.
   - Verification: Ordinary HTTP works without a broker; broker acknowledgement is not recipient processing; SSE is not represented as Part 3. Test live MQTT interruption with the currently available authorized HTTP recovery path; complete snapshot recovery is checked in Phase 8.
   - Dependencies: 6.2–6.3; security checks use the same permitted/denied fixtures as HTTP.

**Phase exit:** Ordered retained publication, SSE replay and the actual experimental MQTT adapter pass their defined tests. No claim of disconnected MQTT completeness is made, and SSE's administrative snapshot bootstrap is not marked complete until Phase 8.

### Phase 7: Experimental sampling and enhanced filtering

**Goal:** Complete the approved bounded sampling and spatial/value-query additions without changing native CSAPI semantics.<br>
**Dependencies:** Phases 2–4; no tasking or broker dependency.<br>
**Estimated effort:** Uncalibrated; four tasks.<br>
**Complexity:** High.<br>
**Guide:** §§1.4, 4.2.1, 4.4.1, 6.3.1, 7.4, 8, 13.

1. **7.1 Implement static Part 4 Point/Curve/Surface types — Medium.**
   - Scope/deliverables: Add the pinned/adapted specialized validators and canonical type handling through existing SamplingFeature routes and writes. Preserve generic approved behavior.
   - Verification: All three aliases/full URIs, matching non-null geometry, coordinate precision/height, relationships, CRUD/PATCH, native spatial results and authorization pass. Generic JSON acceptance is not specialization support.
   - Dependencies: 2.2–2.5; no observation-query or mobile/derived-volume implementation is implied.
2. **7.2 Implement scoped observation queryables and value mappings — High.**
   - Scope/deliverables: Add protected queryables discovery and the direct, phenomenon-time sampling-geometry view; bind single-stream scalar aliases to immutable contracts, units and missing/nil/non-finite rules. Keep authoritative correction and unresolved-conflict behavior explicit.
   - Verification: Discoverable names match executable mappings; direct versus ancestor and current versus historical geometry differ as expected. Missing history is unavailable, not reconstructed; projection updates cannot lose true matches.
   - Dependencies: 2.3, 3.2–3.3 and 4.4. Generic SamplingFeatures suffice; 7.1 is not a prerequisite.
3. **7.3 Implement the complete selected CQL2 JSON language — High.**
   - Scope/deliverables: Implement typed/bounded parsing, selected basic and spatial operators/literals, three-valued evaluation and allowlisted parameterized SQL. Combine with native filters, latest selection and scoped paging exactly as the Guide specifies.
   - Verification: All selected operators and spatial literal families, calendar/type errors, singleton GeometryCollection interpretation and empty/short-circuited branches pass independent expected-result tests. No extra text language, arithmetic, joins or arbitrary field expansion is added.
   - Dependencies: 7.2 and established query/access primitives.
4. **7.4 Verify all six filtering classes and combined workflows — High.**
   - Scope/deliverables: Complete Features/CQL2 prerequisite tests, integration examples and query-cost measurements using both generic and selected specialized sampling features.
   - Verification: Check exact IDs for spatial/value/native combinations, latest ties, NULL/negation, mixed units/types, compatible revisions, mapping cursor retirement and protected facts. No cross-stream universal result value or authorization-by-NULL workaround is accepted.
   - Dependencies: 7.1–7.3; all six classes, not parser acceptance alone.

**Phase exit:** The three selected static types and all six filtering targets/dependencies pass with the Guide's explicit mapping and source qualifications. These remain separate from the 25 CSAPI classes and from whole-Part-4 claims.

### Phase 8: Exchange, interrupted operation and restore

**Goal:** Complete the Guide's bounded recovery and administrative exchange responsibilities.<br>
**Dependencies:** Phases 5–7; shared revisions/tombstones/access and Phase 6's committed log.<br>
**Estimated effort:** Uncalibrated; four tasks.<br>
**Complexity:** High.<br>
**Guide:** §§4.7–4.12, 6.1, 6.4, 8, 9.2.

1. **8.1 Implement bounded administrative export/import — High.**
   - Scope/deliverables: Generate the Guide's versioned manifest schema/fixtures and implement source/recipient/scope validation, artifact/schema dependencies, exact-byte integrity, source revision/ancestry checks, atomic receipts and restricted conflict staging/resolution.
   - Verification: Exact replay is a no-op; altered identity/content, missing dependencies, UID collisions and attempted resurrection cannot silently overwrite accepted state. Imported command history never dispatches work.
   - Dependencies: Existing application write boundary, Phase 5 execution evidence and 6.1 log/revisions; no universal federation service.
2. **8.2 Implement consistent snapshots and catch-up — High.**
   - Scope/deliverables: Implement bounded repeatable-read snapshots, publication watermark/overlap ancestry, scoped replacement reconciliation and compatible SSE resume tokens. Enforce retention/expiry and unresolved-record completeness rules.
   - Verification: Snapshot r5 with delayed r1–r4 then r6 cannot roll back or skip data; protected omissions do not become deletes; mismatched SSE/exchange tokens fail; peers beyond retention require a fresh snapshot.
   - Dependencies: 8.1, 6.1–6.2 and retained source/revision context.
3. **8.3 Complete migration, restore and runtime operations — High.**
   - Scope/deliverables: Extend the existing isolated restore procedure to all resources, source artifacts, pending work, publication/exchange records and recovery epochs. Complete retention controls, safe migrations, typed limits/secrets, readiness/metrics and bounded shutdown/resume examples.
   - Verification: Restore/fork invalidates old continuity without silently skipping newer events; missing post-backup command receipts do not authorize replay. Validate old-instance fencing, held work, rollback/restart and no implicit database reset or automatic purge.
   - Dependencies: Earlier restore tests, Phase 5 safety boundaries and 8.1–8.2.
4. **8.4 Verify interrupted workflows and publish reference limits — High.**
   - Scope/deliverables: Finish disconnected-source, replay/conflict, SSE bootstrap and live-MQTT HTTP/snapshot recovery examples. Measure and document finite parser/query/queue/export limits and recovery/deduplication windows on the reference setup.
   - Verification: Revoke/expire authority during recovery; inject transaction/handoff/import failures; prove no false completeness, protected-context disclosure, unintended actuation or unbounded staging. Validate historical/local reads under remote outages.
   - Dependencies: 8.1–8.3 and 6.3–6.4; measurements are reference configuration, not production guarantees.

**Phase exit:** All adopted recovery paths, including the Phase 6 snapshot-bootstrap follow-through, work with explicit limits and safe failure. Network/federation operation, policy ownership and production accreditation remain deployment responsibilities.

### Phase 9: Integrated verification and release readiness

**Goal:** Establish full-target completion from executable evidence and an independently usable reference setup.<br>
**Dependencies:** Phases 1–8.<br>
**Estimated effort:** Uncalibrated; four tasks.<br>
**Complexity:** High.<br>
**Guide:** §§4.12, 5, 7–10, 13.

1. **9.1 Close full conformance and interpretation coverage — High.**
   - Scope/deliverables: Run the complete direct/inherited and selected filtering tests against the candidate configuration; reconcile annotations, interpretations, actual API behavior and declarations. Re-run experimental/extension tests separately.
   - Verification: All 25 CSAPI targets, applicable prerequisites and six filtering classes have passing evidence or explicit unresolved failures; failures cannot be called full completion. No skipped test, permissive schema, draft URI or library workaround is mistaken for conformance.
   - Dependencies: Phases 1–8; tests already exist from capability work, not a new final-only harness.
2. **9.2 Complete independent-client and clean-environment workflows — Medium.**
   - Scope/deliverables: Exercise pinned OS4CSAPI and independent Python/HTTP clients, the nine Guide scenarios and the build/configure/migrate/sample/serve/test instructions. Cover all required representations with appropriate independent tools, not an unsupported client feature claim.
   - Verification: A separate setup reproduces discovery, writes, exact queries, streaming, feasibility/tasking and restore using only documented prerequisites; examples require no other Glaux product or operational hardware.
   - Dependencies: 9.1's candidate and existing incremental client fixtures; a client limitation must be distinguished from a server defect.
3. **9.3 Complete security, failure and workload checks — High.**
   - Scope/deliverables: Repeat the cross-family access matrix, dependency/license review, fault injection and representative load/codec/query/recovery measurements with the complete implementation and optional adapters.
   - Verification: Test protected indirect facts, malformed/expensive input, identity/policy outages, queue pressure and shutdown/restore. Record machine, data, versions, limits, correctness and results; add justified indexes/partitioning only through measured evidence and the existing design process.
   - Dependencies: Complete feature/test corpus and 9.2 reference setup; not permission to defer earlier security/failure tests.
4. **9.4 Prepare the reference release candidate and completion summary — Medium.**
   - Scope/deliverables: Produce reproducible build/container artifacts, complete README/API/examples, pinned configuration/test results and an honest capability/limitations summary. Update Roadmap task status against actual evidence.
   - Verification: Guide §10's full-completion conditions pass, including experimental Parts 3/4, all encodings/tasking and selected filtering. A partial candidate remains labeled partial. No production deployment, OGC certification or accreditation is implied.
   - Dependencies: 9.1–9.3; unresolved correctness/safety obligations return to their owning tasks before a full-completion claim.

**Phase exit:** The full reference implementation has reproducible evidence and documentation matching its actual behavior. This is the completion target, not the current state of this drafting iteration.

## 5. Implementation Iterations and Task Size

Use the existing `proceed` workflow. Select the next dependency-ready task, or a named bounded portion of it, and state the intended code change and verification before work. Finish that slice, run the appropriate checks, update its status/evidence and push the completed iteration; then pause. Do not interpret one `proceed` as authorization to run every remaining phase.

Some tasks deliberately cover a family of closely related cases. Split them where needed into concrete slices—such as one SWE component family plus its tests, one resource's complete mutation path, or one command crash boundary. Keep the parent task open until its full scope passes. Do not create separate phase guides, execution-unit catalogs or new approval forms merely because a task is large.

During Roadmap drafting, use two iterations on this same file: this complete first draft, then feedback/dependency/coverage review and finalization. The next `proceed` authorizes that second drafting pass, not Phase 1 implementation.

## 6. Coverage, Milestones and Deliverables

### 6.1 Direct CSAPI class completion ownership

Class names and authoritative requirement mappings remain in Guide §7.1. This table assigns their delivery; it is not a duplicate requirements specification. A phase assignment never waives prerequisites or authorizes an early declaration.

| Part | Class | Owning completion task(s) |
|---|---|---|
| 1 | `api-common` | 1.4 foundation; 2.6 complete inherited/core proof |
| 1 | `system` | 2.3–2.6 |
| 1 | `subsystem` | 2.3–2.6 |
| 1 | `deployment` | 2.3–2.6 |
| 1 | `subdeployment` | 2.3–2.6 |
| 1 | `procedure` | 2.2–2.6 |
| 1 | `sf` | 2.3–2.6; experimental types remain separate |
| 1 | `property` | 2.2–2.6 |
| 1 | `advanced-filtering` | 2.5–2.6 |
| 1 | `create-replace-delete` | 2.4–2.6 |
| 1 | `update` | 2.4–2.6 |
| 1 | `geojson` | 2.2–2.6 |
| 1 | `sensorml` | 2.1–2.6 |
| 2 | `api-common` | 3.1–3.5; family-wide confirmation in 5.5 |
| 2 | `datastream` | 3.1–3.5 |
| 2 | `controlstream` | 5.1–5.5 |
| 2 | `feasibility` | 5.4–5.5 |
| 2 | `system-event` | 3.4–3.5 |
| 2 | `advanced-filtering` | 3.3–3.5 and 5.5; also depends on Part 1 filtering |
| 2 | `create-replace-delete` | 3.1–3.5 and 5.1–5.5 |
| 2 | `update` | 3.1–3.5 and 5.1–5.5 |
| 2 | `json` | 3.1–3.5 and 5.1–5.5 |
| 2 | `swecommon-json` | 4.1/4.4 observation proof and 5.1–5.5 command proof |
| 2 | `swecommon-text` | 4.2/4.4 observation proof and 5.1–5.5 command proof |
| 2 | `swecommon-binary` | 4.3/4.4 observation proof and 5.1–5.5 command proof |

Final cross-class declaration verification belongs to 9.1. Applicable Common/Features Core, JSON/GeoJSON and the pinned transaction-draft tests begin in 1.4/2.4 and complete with their consumers. SensorML and SWE component prerequisites belong to 2.1–2.2; value-encoding prerequisites to 4.1–4.4. Incorporated HTTP, linking, time, schema, spatial and Unicode obligations are exercised in 1.2, 1.4, 2.5 and the relevant downstream operations. The exact dependency baseline stays in Guide §1.2.

### 6.2 Additional targets and implementation proofs

| Target or remaining Guide proof | Assigned work |
|---|---|
| Features Part 3 `queryables`, `filter` | 7.2–7.4, including Common/JSON discovery dependencies |
| CQL2 `basic-cql2`, `basic-spatial-functions`, `basic-spatial-functions-plus`, `cql2-json` | 7.3–7.4 and applicable incorporated dependencies; final evidence in 9.1 |
| Experimental Part 3 MQTT; separate SSE extension | 6.1–6.4; snapshot recovery follow-through in 8.2/8.4 |
| Experimental static Part 4 Point/Curve/Surface | 7.1/7.4 with pinned adapted-schema fixtures |
| Buildable dependencies, recursive schema, exact time/number and Unicode behavior | 1.1–1.2, 2.1/2.5, 4.1–4.4; limitations assessed before affected conformance claims |
| Earlier transaction draft, validation direction, cross-format writes and schema selectors | 1.2–1.3, 2.4, 3.1–3.2, 4.4, 5.1/5.5 |
| Command response, timeout, retry, terminal edits and cascade interpretation | 5.2–5.5; restore/missing-tail proof in 8.3 |
| MQTT client/broker pins, AsyncAPI, topic access and delivery limits | 6.3–6.4 and 8.4 |
| Exchange schema/ancestry, snapshot overlap, recovery epochs and limits | 8.1–8.4, extending earlier backup/restore tests |
| Queryable mappings, unavailable values, corrections and literal validation | 7.2–7.4 |
| Provenance, quality and independent disclosure | 2.1–2.2, 3.2/3.5, 5.1–5.5, 6.4, 7.4, 8.1/8.4; final cross-family check in 9.3 |
| All Guide §13 interpretations | Source fixtures introduced with each owning capability; original/selected behavior retained in tests; claim qualification checked by 9.1 |
| Part 5 | Deferred under Guide §1.5; no placeholder implementation, dependency or milestone |

### 6.3 Milestones and deliverables

Milestones describe observable results, not extra approval meetings:

- **After Phase 1:** a developer can run the first authorized persisted System workflow and its tests.
- **After Phase 3:** the described systems can produce schema-bound observations and status/event information that independent clients retrieve correctly.
- **After Phase 5:** complete intended Parts 1/2 implementation paths, including Text/Binary and full tasking, have capability-level tests; full-project completion still requires subsequent adopted capabilities and integration evidence.
- **After Phase 8:** publication, selected sampling/filtering and recovery behavior are implemented together with explicit limits.
- **After Phase 9:** the complete reference release candidate meets Guide §10, or remaining failures are identified without a false completion claim.

Deliverables are ordinary server-repository outputs: Rust source, migrations, bundled standards artifacts and explicit adaptations, independent fixtures/tests, OpenAPI/AsyncAPI where applicable, runtime configuration/examples, README/build/run/backup instructions and normal CI/test summaries. Planning status stays here. No separate evidence platform or requirements document is required.

## 7. Quality Gates

The phase exit criteria above are the template's progression gates; do not duplicate them as a second approval system. Their common rule is: dependencies are real, implementation and tests exist, relevant negative/access/failure cases pass, and documentation/declarations match the enabled capability. Preserve existing known unrelated failures explicitly; do not silently call them passes.

For a partial reference release, apply Guide §10's partial-release conditions and identify omitted capabilities/classes. For full completion, all nine phases' adopted deliverables and the complete class/dependency target must have evidence. Deployment disablement is not implementation, JSON-only is not full encoding support, SSE is not the MQTT experiment, and a fake adapter test is not universal physical-device safety.

Failure of a prerequisite blocks dependent completion, not unrelated work. Fix the owning task and rerun affected downstream tests. No phase passes merely because its code was written, its estimate was exhausted or its documentation says “complete.”

## 8. Verification and Reporting Cadence

Run unit/codec and applicable real-database/HTTP tests with each implementation slice. Expand the independent standards runner, synthetic dataset, source-conflict fixtures and client scenarios as endpoints arrive. At phase completion, rerun the combined affected workflows; Phase 9 repeats the complete set in a clean reference environment.

Each implementation handoff states what changed, what was actually run, failures or limitations, commit/evidence location and the next dependency-ready slice. Record task status as planned, in progress, blocked or complete here when execution begins, with the relevant server commit/test reference. Completed drafting does not mark implementation tasks complete. Update estimates only from stated evidence and assumptions; do not translate assistant response time into developer labor automatically.

Use the server repository's ordinary test results and a small summary recording build/commit, configuration, dependency/schema/fixture pins, executed cases and outcomes. Distinguish not run, unsupported client feature, implementation failure and source interpretation. The project lead's `proceed` remains the normal iteration control; no special acceptance phrase is required.

## 9. Execution Risks and Contingencies

| Risk | Response within this plan |
|---|---|
| Required toolchain/database is unavailable on the company laptop | Inspect in 1.1; document exact missing prerequisites and use the approved IT process. Do not install implicitly or claim unrun checks passed |
| Rust/schema/time libraries cannot preserve required semantics | Prove the risky behavior in 1.2/2.1 before dependent storage and codecs; revise the affected Guide choice rather than weakening the standard |
| Full component/codec or tasking work is hidden inside a “basic server” milestone | Keep explicit Phases 2/4/5 ownership and full-target completion table; split execution slices while retaining the parent obligation |
| Public standard/artifact contradictions affect conformance | Carry the Guide's selected interpretation and both source fixtures; obtain a defensible disposition before an unqualified affected claim |
| Authorization or source context is added too late | Require it with the first write/query and repeat indirect-disclosure tests as schemas, filters, provenance, topics and exports appear |
| Delivery/physical effects diverge from database state | Test transaction/log/admission crash boundaries at introduction; hold uncertain commands and account for rollback-lost receipts |
| Synchronization appears complete despite missing ancestry or protected omissions | Use 8.1–8.4's explicit scope, conflict and completeness tests; require a new snapshot when continuity is not established |
| Final verification becomes the first real integration attempt | Run real HTTP/database/client workflows incrementally; Phase 9 combines and repeats them rather than inventing the foundations |
| Early measurements do not support a proposed operating limit or index | Publish measured reference limits, revise the affected implementation and remeasure; do not fabricate production throughput guarantees |
| Research or late requests expand the project unnoticed | Keep Goal/Guide scope authority explicit; treat Part 5 and broader sampling/querying/provenance options as unadopted unless the project lead changes scope |

Replan only when a dependency/design proof fails, measured task size requires a different sequence, an authoritative correction changes behavior, or the project lead changes priorities/scope. Record the reason and affected tasks in this document; a new planning framework is not a contingency.

## 10. Change Control and Next Iteration

Version 0.1 is the complete first Roadmap draft authorized by the project lead's `proceed` after agreeing to two drafting iterations. Goal v1.7 and Guide v1.0 remain unchanged. No software installation, server implementation or production action is performed by drafting this document.

The next `proceed` authorizes the second pass on this Roadmap: incorporate feedback, verify full coverage and dependency order, resize unclear tasks and finalize the document for implementation. Another drafting pass requires a concrete unresolved issue, not a default extension of the process. Implementation starts only after that handoff and the next authorized iteration.

Use version updates for material sequencing or scope changes. Technical design changes belong in the Guide; mission/scope changes belong in the Goal first. Keep task-to-Guide/test connections current without copying the standards into a separate requirement list. Historical research acceptance and findings remain unchanged.

## 11. References and Example Use

- [Approved Goal and Definition](glaux-server-goal-and-definition.md)
- [Baselined Implementation Guide][Guide], especially §§1, 7–10 and 13
- [Roadmap template](../../Governance/roadmap-template.md) and [Initial Planning Guidance](../../Governance/initial-planning-guidance.md)
- [Final research synthesis and supplements](../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/final-idr-research-report.md), used through the Guide's selected findings
- [OS4CSAPI implementation Roadmap][ExampleMain] and [Phase 5 parser-completion Roadmap][ExampleParser], pinned to the inspected example revision

The examples inform concrete task boundaries, dependency order, deliverables and tests accompanying implementation. Their historical hours, code-volume estimates, client-only scope, tolerant parsing rules and restrictions on live/performance testing are not server requirements. The standards baseline and existing interpretations remain those linked by the Guide; this scheduling pass does not repin standards or conduct another research study.

[Guide]: glaux-server-implementation-guide.md
[ExampleMain]: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/planning/ROADMAP.md
[ExampleParser]: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/planning/phase-5/P5-ROADMAP.md

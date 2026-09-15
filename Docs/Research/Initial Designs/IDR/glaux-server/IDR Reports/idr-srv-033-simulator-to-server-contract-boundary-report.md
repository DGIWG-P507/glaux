# Section 033: Simulator-to-Server Contract Boundary - Research Report

**Topic ID:** IDR-SRV-033<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-033 Simulator-to-Server Contract Boundary](../IDR%20Plans/idr-srv-033-simulator-to-server-contract-boundary.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** Five core questions and all detailed questions concerning simulator roles, data/control planes, capability and lifecycle, identity, deterministic replay, virtual time, reset safety, tasking, faults, DDIL, isolation, progress, evidence, fixtures, and handoffs<br>
**Methodology Used:** Authority-ranked analysis of approved OGC API - Connected Systems behavior, the accepted IDR-SRV-001 through IDR-SRV-032 server model, commit-pinned Glaux Simulator and implementation/test evidence, HTTP and provenance standards, and official disposable-environment/observability guidance; followed by use-case, operation, identity, time, reset, failure, and verification mapping<br>
**Research Time:** Approximately 15 hours of AI-assisted research and synthesis, September 14, 2026<br>
**Primary Source(s):**
- [OGC API - Connected Systems - Part 1: Feature Resources](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2: Dynamic Data](https://docs.ogc.org/is/23-002/23-002.html)
- [Glaux Simulator `main` commit `fc5dd86d3be9aa583e911d5ebbf50e0156e80638`](https://github.com/DGIWG-P507/glaux-simulator/tree/fc5dd86d3be9aa583e911d5ebbf50e0156e80638)
- [IDR-SRV-032 Publisher-to-Server Contract Boundary](idr-srv-032-publisher-to-server-contract-boundary-report.md)
**Supporting Resources:**
- [IDR-SRV-018 Temporal, Validity, and Freshness Model](idr-srv-018-temporal-validity-and-freshness-model-report.md)
- [IDR-SRV-019 Provenance, Lineage, Quality, and Trust Metadata Model](idr-srv-019-provenance-lineage-quality-and-trust-metadata-model-report.md)
- [IDR-SRV-029 Transaction, Consistency, Idempotency, and Concurrency Strategy](idr-srv-029-transaction-consistency-idempotency-and-concurrency-strategy-report.md)
- [IDR-SRV-030 Data Lifecycle, Retention, Archival, and Deletion Strategy](idr-srv-030-data-lifecycle-retention-archival-and-deletion-strategy-report.md)
- [IDR-SRV-031 Server Write and Ingestion Model](idr-srv-031-server-write-and-ingestion-model-report.md)
**Document Purpose:** Establish a safe, deterministic simulator data/control contract that exercises the production write boundary without exposing broad reset, clock, validation-bypass, or fault-injection powers<br>
**Author(s):** OpenAI Codex, for the Glaux Project<br>
**Accepted By:** TBD until controlling-plan owner acceptance<br>
**Acceptance Date:** TBD until accepted<br>
**Date:** September 14, 2026<br>
**Last Updated:** September 14, 2026

---

This report labels evidence as **[N] normative standard**, **[A] accepted project baseline**, **[P] project recommendation/decision proposed for acceptance**, **[I] informative implementation/test evidence**, and **[X] documented conflict, absence, or unresolved gap**. Combined labels preserve rather than raise the authority of each source class.

## Table of Contents

1. Executive Summary
2. Scope, Terminology, and Source-Authority Statement
3. Simulator Use-Case and Capability Inventory
4. Standards-Derived Versus Glaux-Specific Behavior Classification
5. Data-Plane/Control-Plane Contract Decision
6. Scenario, Dataset, Run, Session, Clock, Source, and Resource Identity Model
7. Capability Discovery and Session/Control Lifecycle
8. Synthetic Data, Provenance, Policy, and Isolation Model
9. Replay, Determinism, Checkpoint, Resume, and Completion Semantics
10. Virtual-Time and Temporal Mapping Semantics
11. Reset, Cleanup, Ownership, Safety, and Recovery Model
12. Dynamic-Data, Fault-Injection, DDIL, and Tasking-Loop Behavior
13. Authentication, Authorization, Audit, Errors, Diagnostics, Progress, and Backpressure
14. Scenario Manifest, Verification, and Interoperability Matrix
15. Downstream Handoff Matrix
16. Recommendations and Explicit Project Decisions
17. Risks, Contradictions, Assumptions, and Unresolved Questions
18. Validation Against This Plan's Success Criteria
19. References

---

## 1. Executive Summary

Glaux Simulator should appear to Glaux Server as **two separately authorized roles**: an ordinary GPC-v1 publisher on the data plane and a test controller on a small private control plane. The same authenticated principal may receive both roles in a dedicated test deployment, but the permissions, credentials or token scopes, routes, audit decisions, and revocation remain separable. Simulated Systems, descriptions, stream definitions, Observations, status data, System Events, Commands, Feasibility records, statuses, and results use the same advertised CSAPI operations, validation, source authority, transaction, persistence, provenance, policy, error, outbox, and backpressure behavior as other publishers. No simulator data path writes canonical storage directly or bypasses validation. **[A/P]**

The private control plane should be production-disabled and namespaced at `/_glaux/simulator/v1`. It manages server-visible sessions, runs, manifests, progress/checkpoints, completion, and guarded reset. It does **not** turn Glaux Server into a scenario engine, direct the simulator's internal generator, globally pause ingestion, change the server clock, or expose a remotely callable “accept invalid data” switch. Generator pause/resume/step/rate, source disconnection, packet loss/reordering, and transport faults are performed by the simulator, harness, or test proxy; their declared control/checkpoint evidence is correlated to the run. **[P]**

Glaux Simulator's current repository is mission-only evidence. At `main` commit `fc5dd86d3be9aa583e911d5ebbf50e0156e80638` (June 7, 2026), it contains a README and no interface, scenario schema, executable source, tag, release, or test corpus. The README defines it as a STANAG 4789-aligned telemetry generator for evaluation, stress testing, and demonstrations without live tactical feeds.[^1] Therefore all detailed control operations in this report are explicitly Glaux project proposals rather than observed Simulator behavior. **[I/X/P]**

The recommended control resources are:

- `GET /_glaux/simulator/v1/capabilities`;
- `POST /_glaux/simulator/v1/sessions`, `GET /_glaux/simulator/v1/sessions/{sessionId}`, and `POST /_glaux/simulator/v1/sessions/{sessionId}/closures`;
- `POST /_glaux/simulator/v1/sessions/{sessionId}/runs` and `GET /_glaux/simulator/v1/sessions/{sessionId}/runs/{runId}`;
- `POST /_glaux/simulator/v1/sessions/{sessionId}/runs/{runId}/checkpoints` for state/clock/action checkpoints and explicit finalize/fail/cancel declarations;
- `POST .../runs/{runId}/reset-previews` and `POST .../runs/{runId}/resets`; and
- `GET /_glaux/simulator/v1/operations/{operationId}` for asynchronous reset/large control work.

The server assigns session, run, operation, and canonical ResourceIds. Scenarios and datasets have stable author-supplied identity, immutable version and content digest. A seed is insufficient for reproducibility: the manifest also pins generator algorithm, implementation/build, configuration, mapping/profile, action order, artifacts/digests, identifier policy, and permitted nondeterminism. The default identifier mode remaps logical fixture keys into run-local server IDs; ordinary simulator POST cannot choose canonical ResourceIds. Stable cross-run identifiers are allowed only in an isolated clean namespace with an explicit collision/reset policy. **[A/P]**

Virtual time applies only to declared simulated source/domain times. Server receipt, ingest, commit, transaction sequence, publication/delivery, authentication, authorization, audit, lease, idempotency retention, backpressure, and lifecycle/retention clocks remain trusted wall/monotonic server time. Tests of “now,” freshness, retention, or timeout policy use an in-process injectable clock only in a disposable test server or explicit query times; the remote simulator API never changes a shared server clock. Time shifting is a new transformation/run, not exact replay. **[A/P]**

Reset is the highest-risk operation. For CI, conformance, security, performance, and destructive fault testing, **destroying and recreating a disposable isolated deployment is preferred** because it produces a stronger boundary than graph-selective deletion. Official Testcontainers for Rust documentation describes programmatic creation and cleanup of container dependencies, while Docker documents that teardown scope differs for containers, networks, named/anonymous volumes, and external resources; orchestration must therefore enumerate storage rather than assume “down” erased everything.[^9][^10] For persistent test/training deployments, an application reset is a two-step, asynchronously monitored operation: generate an ownership/dependency preview, then confirm its signed/opaque preview token with strong preconditions and reset scope. It may affect only the transitive closure of exclusively run-owned resources and derived effects. Shared, operational, foreign-authority, held, or ambiguously owned state blocks reset. Reset preserves an immutable minimal receipt, provenance/audit, and anti-resurrection evidence and never claims physical purge unless the IDR-SRV-030 disposition contract proves it. **[A/P]**

The central safety result is simple: a structurally valid synthetic record never becomes operational merely because it passed schema validation, and an intentionally invalid fixture never earns a bypass merely because it is useful. Environment, tenant/security domain, source grants, immutable synthetic/run context, namespace, broker/object-store paths, policy, and export/federation controls jointly enforce isolation. Production deployments do not register these routes or advertise their capability and should respond as if absent.

## 2. Scope, Terminology, and Source-Authority Statement

### 2.1 Scope and exclusions

In scope are wire-visible server behavior for simulator publication; control capability discovery; session/run lifecycle; scenario/dataset/seed/clock/source/resource identity; deterministic replay and resume; temporal mapping; progress/completion; reset preview/execution/recovery; synthetic provenance; fault and invalid-data test treatment; tasking loops; authorization/isolation; errors/audit/observability/backpressure; manifests; fixtures; and downstream handoffs.

Out of scope are Simulator code, UI, scenario authoring, generator algorithms, hardware models, operator workflows, final dynamic-data semantics, streaming/broker selection, Command/Feasibility lifecycle and safety rules, final security architecture, DDIL synchronization, test-framework selection, deployment implementation, and numeric limits. This report describes what the Server must observe and enforce, not how the Simulator produces traffic.

### 2.2 Precise terms

| Term | Meaning | Must not be confused with |
|---|---|---|
| Simulation scenario | Versioned logical definition of actors, sources, resources, actions, assertions, and cleanup intent | One execution or server session |
| Dataset | Immutable versioned artifact set used by a scenario, identified by digest | Generated canonical state or mutable directory |
| Seed | Input to a named generator algorithm/build/configuration | A reproducibility guarantee by itself |
| Simulation session | Server-assigned isolation/control context grouping one or more authorized runs | Authentication session, database transaction, MQTT session, or tenant |
| Simulation run | One execution of one pinned scenario/dataset/identifier/clock policy within a session | Scenario definition or replay delivery attempt |
| Simulator instance | GPC publisher instance/software deployment from registration | Scenario, run, represented synthetic source, or principal |
| Test controller | Principal/role authorized for private session/run/reset operations | Publisher authority to submit data |
| Synthetic source | Registered non-operational represented source generating or emulating data | Simulator software, physical source, or trusted source |
| Logical fixture key | Stable manifest-local name used to correlate resources/actions across runs | Canonical ResourceId or necessarily a UID |
| Virtual/source clock | Scenario-defined mapping used to generate source/domain timestamps | Server receipt/commit/security/retention clock |
| Checkpoint | Durable server record of manifest/action progress and outcome digest at a run revision/watermark | Receipt-only acknowledgement or arbitrary simulator offset |
| Exact replay | Re-submission of identical original operation identity/content/domain time to test stored outcome/deduplication | New run intended to create equivalent new state |
| Regeneration | New generation activity under pinned algorithm/build/config/seed | Byte-for-byte replay |
| Shifted replay | Transformation that changes domain timestamps according to a recorded mapping | Exact replay |
| Reset | Authorized operation that restores an isolated simulator scope to a declared baseline/empty state | Unqualified delete, physical purge, or production wipe |
| Disposable recreation | External destruction and re-creation of a fully isolated test deployment and all enumerated stores | Application reset inside a shared deployment |

### 2.3 Source authority and evidence hierarchy

Approved standards control CSAPI behavior. Accepted IDR-SRV-015 through 032 control identity, time, provenance, status, validation, persistence, transaction, lifecycle, write, and publisher behavior. The Simulator repository defines only mission intent. Implementation/test repositories demonstrate possible patterns or failure cases but do not define standards obligations. Private control operations, fields, and state machines below are Glaux project decisions. **[N/A/I/P]**

The controlled AEP baseline remains `AC/224(JCGISR)D(2026)0005`, dated 27 April 2026, SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`. It is not redistributed. Accepted reports establish the need for evaluation, stress, demonstration, interpretable context, secure handling, and server-facing Simulator writes; no controlled finding establishes these private routes, reset semantics, or a global virtual clock. **[A]**

Simulation source authority is deliberately narrow. The principal authenticates; the Simulator publisher instance conveys; the registered synthetic source asserts; the scenario/run describes test context; and the server validates/authorizes/persists. None proves physical truth. A Simulator may emulate a target for tasking tests but does not acquire operational command-target authority. **[A/P]**

## 3. Simulator Use-Case and Capability Inventory

| Use case | Needed server behavior | Classification | Initial availability posture |
|---|---|---|---|
| Local development/demo | Seed small descriptions/data, inspect and clean run | Ordinary data plane + optional control plane | Dedicated local deployment |
| CI integration | Create deterministic namespace/run, execute assertions, dispose environment | External orchestration + ordinary data plane | Prefer disposable server/container set |
| CSAPI conformance | Submit/query standard resources without hidden behavior | Ordinary standards data plane | Control plane may seed/clean externally but cannot affect tested response |
| Client interoperability | Stable graph, media, pagination, time, errors, lifecycle fixtures | Ordinary data plane + manifests/checkpoints | Isolated reproducible server data |
| Stress/performance | Sustained/burst/load shapes, measured acceptance/publication, cleanup | Ordinary data plane + progress; external load controller | Dedicated deployment; reset by recreation |
| Security/adversarial | Unauthorized, malformed, oversize, duplicate, cross-tenant, fault cases | Ordinary boundary + external proxy/harness | Disposable isolated deployment only |
| DDIL/store-and-forward | Disconnect, backlog, epoch/sequence/gap/replay/conflict | Ordinary GPC data plane | Isolated test source/tenant; semantics handed to 042/043 |
| Tasking closed loop | Receive Command, return Feasibility/status/result/failure/timeout | Ordinary CSAPI/GPC paths + external simulator script | Separate command target/report grants; safety stays normal |
| Temporal/freshness tests | Preserve/shift/scale/burst/step source times and explicit query instants | Generator/orchestration + data plane | No server-global clock mutation |
| Fault injection | Delay/drop/reorder/duplicate/disconnect/malformed/invalid | Simulator/proxy before boundary; ordinary server rejection | No public or shared server bypass hook |
| Run progress/reproducibility | Counts, action outcomes, checkpoints, completion digest | Glaux private control plane | Explicit simulator capability |
| Reset/cleanup | Restore run-exclusive simulation scope | Prefer disposable recreation; guarded control fallback | Absent production; blocked on ambiguous ownership |
| Operational training | Realistic synthetic task/data without real feeds | Dedicated training deployment and policy | Never mixed by default with operational authority |

The use-case inventory does not justify server-side scenario scheduling or generator control. The Server needs enough context to isolate, validate, correlate, measure, and clean effects. The Simulator/harness remains responsible for generating actions, pacing them, and injecting transport/source faults. **[P]**

Required control capabilities are separately discoverable: `sessions`, `runs`, `checkpoints`, `run-progress`, `reset-preview`, `reset-execute`, and optional `application-reset`. Data families, media, GPC version, quotas, and write methods remain in the ordinary deployment contract/capability registry. Capability presence never grants authorization. **[A/P]**

## 4. Standards-Derived Versus Glaux-Specific Behavior Classification

| Behavior | Authority/status | Consequence |
|---|---|---|
| CSAPI resources, paths, representations, links, query and claimed write classes | Approved OGC standards, with accepted documented gaps **[N/A]** | Simulator data must be indistinguishable in protocol correctness, not in provenance/authority |
| HTTP methods/status/conditional requests and Problem Details | RFC 9110/RFC 9457 **[N]** | Reuse `201`, `200`/`204`, guarded `202`, ETag/preconditions, safe problems[^2][^3] |
| GPC-v1 principal/publisher/source/message/idempotency boundary | Accepted IDR-SRV-032 **[A]** | Simulator is a registered publisher for data-plane effects |
| Canonical ResourceId and revision assignment | Accepted IDR-SRV-016 **[A]** | Simulator cannot force deterministic server IDs through ordinary POST |
| Domain/source versus receipt/commit/publication clocks | Accepted IDR-SRV-018 **[A]** | Virtual time cannot replace server evidence/security clocks |
| Provenance Entity/Activity/Agent grammar | W3C PROV and IDR-SRV-019 **[N/A]** | Scenario/dataset/run/generation/replay/reset are typed evidence, not payload labels[^4] |
| Status/Event/Command distinctions | CSAPI Part 2 and IDR-SRV-020 **[N/A]** | Simulator health/control events cannot masquerade as System status/events |
| Transaction/idempotency/outbox/inbox | IDR-SRV-029/031 **[A]** | Test traffic gets no weaker atomicity or duplicate rules |
| Lifecycle/deletion/hold/purge distinctions | IDR-SRV-030 **[A]** | Reset must state exact logical/copy scope and preserve accountability |
| Simulator sessions, runs, manifests, checkpoints, reset previews | No CSAPI standard behavior found **[X/P]** | Versioned private Glaux control plane only |
| Pause/resume/step/rate generator behavior | Simulator/orchestrator function **[P]** | Server records declared checkpoints and observed data; it does not drive generator internals |
| Network/source fault injection | Harness/proxy/simulator function **[P]** | Server continues normal handling and measurement; no remotely exposed bypass |
| Disposable environment recreation | Official tool capability, project testing choice **[I/P]** | Preferred cleanup for destructive test lanes; not a server API requirement |

No simulation control is advertised as OGC conformance. Conversely, use of a private session header or internal synthetic marker does not excuse a malformed CSAPI response. Standards tests must be able to observe the same public route behavior with the control plane disabled from the assertion path. **[P]**

The upstream-history register was consulted at Version 1.11 and September 14 pins. No simulator-specific CSAPI issue or newly material upstream delta was found. The draft Part 3 publication surface remains owned by IDR-SRV-035 and is not required by this control plane. **[X]**

## 5. Data-Plane/Control-Plane Contract Decision

### 5.1 Decision analysis

| Option | Benefits | Costs/risks | Decision |
|---|---|---|---|
| Treat Simulator as ordinary publisher only | No private server surface | Cannot safely correlate ownership/progress or support bounded cleanup; orchestration metadata becomes external and fragile | Insufficient alone |
| Add a broad scenario engine and validation/fault/reset hooks to Server | Central orchestration | Large unsafe test API, production attack surface, second processing model, clock contamination | Reject |
| Ordinary GPC/CSAPI data plane + narrow session/run evidence/control plane | Realistic writes, server-known ownership/progress, bounded reset | Requires private schema and hard deployment gates | **Adopt** |
| No application reset; always recreate deployment | Strongest isolation and cleanup | Expensive for persistent demos/training and cannot selectively reuse a test server | Preferred where feasible, not universal |

### 5.2 Contract boundary

The data plane uses the exact GPC-v1 and CSAPI operations accepted in IDR-SRV-032. Simulation adds logical context fields `simulationSessionId`, `simulationRunId`, `scenarioId/version/digest`, `datasetId/version/digest`, `logicalActionId`, and `syntheticOrigin`. The HTTP binding initially uses `Glaux-Simulation-Session-Id`, `Glaux-Simulation-Run-Id`, and `Glaux-Simulation-Action-Id`; scenario/dataset/clock identity is resolved from the immutable run record rather than repeated/trusted on every data request. These fields are required only for the simulation profile and are not CSAPI members. **[P]**

The authenticated publisher instance and represented synthetic source still come from GPC credential/registration and `Glaux-Source-Id`. A run-context header cannot authorize a source, path, resource family, or operation. The server validates that session/run is active for admission, belongs to the same tenant/security domain, pins the submitted contract/profile, and permits the action ID under the manifest. **[A/P]**

The control plane is not a data write shortcut. It contains metadata/evidence and orchestrates cleanup of previously admitted effects. It must not accept arbitrary CSAPI payloads, arbitrary SQL/filter deletion, arbitrary URLs, policy/credential changes, a global wall-clock value, direct queue manipulation, or “expected invalid therefore accept” directives. **[P]**

### 5.3 Required simulator operation matrix

| Use case | Data-plane/control-plane classification | Operation | Required capability | Authenticated role/scope | Scenario/session state | Request inputs | Idempotency/concurrency rule | Affected resource ownership | Temporal behavior | Acknowledgement/progress behavior | Failure/retry behavior | Audit/provenance requirement | Isolation guard | Expected postcondition | Cleanup/reset behavior | Verification scenario | Standards status | Unresolved issue |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Discover controls | Control | `GET /_glaux/simulator/v1/capabilities` | simulator capability registry | Authenticated test viewer/controller; own environment | Any | Desired version/media | Safe GET; validator/cache scoped to deployment and caller | None | Server wall/evaluation time | `200` exact enabled capabilities or absent route | `404` if disabled; `403`/concealment if ungranted | Access audit as policy requires | Production route unregistered; capability filtered | Caller learns only enabled/authorized controls and limits | None | Production absence, cross-scope filtering, registry parity | Glaux-specific **[P]** | Final representation/API description in 044–050 |
| Create session | Control | `POST /_glaux/simulator/v1/sessions` | `sessions` | `simulator.session.create` in test tenant/environment | None → created | scenario/dataset refs, purpose, isolation class, expiry request, contract/identifier policy | Required idempotency key; server-assigned ID; duplicate returns original; bounded optimistic state | Empty reserved simulation namespace/security scope | Real server creation/lease times | `201` + `Location`, ETag and session representation | Validation/authorization permanent; overload retry same key | Principal, controller instance, scenario/dataset digests, policy/grant, requested isolation | Exact non-operational tenant; no production source grants | Durable `created` session with immutable pins | Empty session can close/expire; no data deletion implied | duplicate create, prod denial, invalid artifact digest | Glaux-specific **[P]** | Numeric lease policy in deployment/security topics |
| Close session | Control | `POST /_glaux/simulator/v1/sessions/{sessionId}/closures` | `sessions` | `simulator.session.close`; session ownership | Created/active → closing → closed | reason and expected session ETag | Required key and `If-Match`; one closure effect; duplicate returns original | Does not delete canonical data; all runs must be terminal or separately cancelled | Real server transition/lease times | `200`/`202` plus session/operation `Location`; terminal ETag | Active-run conflict `409`; ambiguous response queries then retries same key | Actor, session/run terminal inventory, policy and outcome | Same tenant/session; cannot close another controller's scope or imply reset | Session rejects new runs/control except status/reset already authorized | Run effects remain until explicit reset/recreation; evidence retained | active run, stale close, retry, close then reset | Glaux-specific **[P]** | Exact abandoned-session retention in lifecycle/deployment policy |
| Prepare run | Control | `POST /_glaux/simulator/v1/sessions/{sessionId}/runs` | `runs` | `simulator.run.create`; session ownership | Session created/active | Manifest artifact/digest, seed+generator descriptor, clock/ID policy, expected completion/reset scope | Required key; `If-Match` session; immutable manifest after prepared | Reserves run-local namespace; no canonical effect yet | Pins clock mapping; server time remains real | `201` + run `Location`/ETag; validation findings | Correct manifest/new key; transient overload same key | Manifest/dataset/generator/controller/profiles and validation activity | Same session/tenant; declared caps intersection | `prepared` run with validated action/ownership plan | Can cancel/expire without canonical cleanup if empty | bad digest/capability, concurrent run, cross-session manifest | Glaux-specific **[P]** | Manifest encoding/media type in API-definition work |
| Publish simulated metadata/data | Data | Registered CSAPI POST/PUT/PATCH/DELETE routes under GPC-v1 | Ordinary write capability + `simulation-data` profile | Publisher/source grant; active run; family/operation scope | Prepared/running or explicit backfill state | Canonical payload, GPC fields, simulation session/run/action IDs | Ordinary scoped key/message/ETag plus unique manifest action; no bypass | New run-owned resources by default; shared/preexisting mutation forbidden unless separately profiled | Payload domain time from run mapping; receipt/commit real | Ordinary HTTP result plus run counters after commit | Same GPC retry; expected rejection still terminal action outcome | Full GPC evidence plus scenario/run/action/generator/clock mapping and synthetic origin | Hard tenant/source/namespace/run validation and no cross-links | Real canonical effect or normal rejection correlated to action | Run-owned effect included in reset closure; rejected action has evidence only | valid/invalid/cross-source/duplicate/concurrency matrix | CSAPI/HTTP data + Glaux context **[N/A/P]** | Dynamic semantics 034–038 |
| Declare run state/checkpoint | Control | `POST /_glaux/simulator/v1/sessions/{sessionId}/runs/{runId}/checkpoints` | `checkpoints` | `simulator.run.checkpoint`; matching session/run | Prepared/running/paused/draining as transition allows | checkpoint ordinal, declared state/action range, virtual-clock segment, simulator counters, expected outcome digest | Required key and `If-Match` run; monotonic checkpoint ordinal; server recomputes observed counters | No canonical mutation except evidence/run state | Declared virtual mapping plus real receipt/commit | `201` checkpoint/`200` transition; ETag; server-observed counters/watermark | Stale `412`, invalid transition `409`; retry same key | Controller/simulator, manifest, declarations versus server observations, discrepancies | Cannot claim actions outside manifest/run or overwrite observed truth | Durable checkpoint with both declared and observed progress | Preserved through reset according to evidence policy | stale checkpoint, forged counters, disconnect/restart/resume | Glaux-specific **[P]** | Exact progress representation in observability/API topics |
| Pause/resume/step/change rate | External orchestration + control evidence | Simulator/harness operation; then checkpoint POST | `checkpoints`; no server generator-control capability | External simulator controller plus checkpoint scope | Running ↔ paused; prepared/running per manifest | External command and resulting clock mapping/checkpoint | Orchestrator-defined command identity; checkpoint uses server key/ETag | No server canonical data mutation merely from pace change | Piecewise virtual mapping changes; wall clocks unchanged | Server acknowledges checkpoint, not generator actuation | Generator failure is external; checkpoint discrepancy/timeout visible | Control activity and generator instance/build; no false server execution claim | No global ingestion/clock gate; run-only context | Subsequent actions carry declared mapping; server records observed arrival | Reset unaffected; evidence preserved | pause with in-flight data, rate change, step boundary, lost control response | External/project behavior **[P]** | Simulator product API remains out of scope |
| Finalize/drain run | Control | Checkpoint with `draining`, then terminal `completed`/`failed`/`cancelled` declaration | `run-progress`/`checkpoints` | Run controller | Running/paused → draining → terminal | Expected action set, completion profile, outcome digest/reason | Required key/ETag; terminal immutable; server verifies action ledger | No new canonical effect; waits on owned pending work | Real deadline; virtual time cannot extend security/operation lease | `202` + operation/status if draining; terminal `200/201` checkpoint when conditions hold | Retry same key; incomplete action/publish obligations keep draining or fail | Final manifest/result digest, counts by state, discrepancies, watermarks/outbox threshold | Only own run status; no unrelated queue disclosure | Every action accounted under declared commit/publication completion profile | Terminal run becomes reset-eligible subject to holds/ownership | completion with pending item/outbox, cancel race, restart | Glaux-specific **[P]** | Publication-complete threshold owned by 035 |
| Preview application reset | Control | `POST /_glaux/simulator/v1/sessions/{sessionId}/runs/{runId}/reset-previews` | `reset-preview` | `simulator.reset.preview`; terminal/paused run | Paused/completed/failed/cancelled/expired | requested scope/baseline, reason, expected run ETag | Required key and `If-Match`; snapshot-bound preview token/digest and expiry | Enumerated exclusive run-owned closure and derived/copy categories | Real evaluation time; domain/virtual time irrelevant | `201` preview with safe counts/categories/blockers, closure digest, expiry, token | Recompute after change; blockers permanent until resolved; overload retry | Requester, policy/hold/authority graph, closure manifest, decision | Fail closed on shared/foreign/operational/ambiguous/held edges; redacted detail | No mutation; exact candidate and blockers recorded | Preview evidence expires; no cleanup | shared edge, hidden resource, hold, stale ETag, empty closure | Glaux-specific **[P]** | Risk-based approval separation in 039–041 |
| Execute application reset | Control | `POST /_glaux/simulator/v1/sessions/{sessionId}/runs/{runId}/resets` | `reset-execute` | Separate high-risk reset scope; optional approval | Valid preview; run paused/terminal | preview token/digest, confirmation, reason, desired postcondition | Required key + preview/run preconditions; one durable job; conflicting intent `409` | Only previewed exclusive closure; audit/evidence/tombstones excluded from broad deletion | Real transaction/job times; no virtual trigger | `202` + operation `Location`; per-scope progress; terminal receipt | Retry same key/status query; recoverable saga; irreversible boundary explicit | Full disposition transition, actor/approver, copy/dependency outcomes and residual limits | Revalidate ownership/holds/policy before every destructive step; route absent production | Declared baseline restored or explicit partial/failed state; never silent success | Preserve minimum receipt/audit/provenance/anti-resurrection; report physical-copy limits | crash each phase, new edge/hold after preview, retry, residual copy | Glaux-specific using accepted lifecycle **[A/P]** | Exact physical transaction/saga in architecture implementation |
| Recreate disposable environment | External orchestration | Destroy/recreate complete enumerated test deployment | Environment manager, not Server | Infrastructure test role outside API | Any after evidence capture | Deployment/image/config/dataset digests and complete store inventory | Orchestrator run identity; repeatable immutable inputs | Entire dedicated environment only | New wall-clock environment/run; no continuity implied unless manifest imports it | Orchestrator outcome plus new server readiness/capability proof | Failed teardown blocks clean claim; inspect containers/volumes/external stores | Environment/build/config/store manifests and teardown receipt | Dedicated credentials/network/stores; no external shared resources | Fresh declared environment with no prior canonical test effects | Preferred destructive cleanup; explicitly remove governed volumes/stores | leaked volume/object/broker state, pinned-image recreation | Tool/project behavior **[I/P]** | Deployment technology/final tooling 044–049/052 |
| Fault/invalid-data test | Data + external proxy/simulator | Ordinary API submission or transport fault before boundary | Ordinary write + test isolation; no validation bypass | Test publisher/source; proxy control external | Running | Manifested invalid bytes/semantics/auth case or delay/drop/reorder plan | Normal keys; duplicate tests explicitly preserve/change identity by case | None on rejection; valid fault survivors run-owned | Declared source skew/delay; server times actual | Normal problem/no response/retry/status; action ledger expects outcome | Same production classification; ambiguity uses same key | Exact fixture digest, injection point, expected/observed result; sensitive data bounded | Disposable/adversarial environment, quotas, no production routes/credentials | Safeguard behaves normally and deterministically | Cleanup valid effects through run reset/recreation; rejection evidence retained | malformed, unauthorized, overload, delay/reorder/drop/duplicate | Standard boundary + external test **[N/A/P]** | Network tooling selection out of scope |
| Tasking closed loop | Data + external simulator behavior | Ordinary Commands/Feasibility and status/result routes; future stream receive | CSAPI tasking write/read/stream capabilities | Separate command target/report source and controller scopes | Running | Manifested command expectations, response action IDs, timing/fault script | Normal command/status idempotency and state preconditions | Run-owned synthetic target/command graph only | Domain issue/report/execution times follow resource rules; server times real | Ordinary task status/result plus run action counters | Normal timeout/race/retry; no synthetic safety bypass | Principal, emulated target/source, command, generator script, outcomes | No operational target IDs/grants; command capability separate from reset | Real server task state machine exercised | Run closure/reset after terminal or explicit safe cancellation | every state, timeout, duplicate, invalid transition, cancellation race | CSAPI data + project harness **[N/A/P]** | Detailed lifecycle/safety 036–038 |
| Query run progress | Control | `GET /_glaux/simulator/v1/sessions/{sessionId}/runs/{runId}` or `GET /_glaux/simulator/v1/operations/{operationId}` | `run-progress` | Own-run viewer/controller/reset operator | Any | Conditional GET/authorized projection | Safe GET, ETag; no caller counter overwrite | None | Real evaluation time; returns pinned virtual mapping as data | Counts received/authenticated/validated/rejected/quarantined/duplicate/committed/publication-pending/published, checkpoints and discrepancies | `404` concealment; transient `503`; no retry mutation | Access evidence; counters derived from authoritative ledgers | Row/tenant scope; safe buckets suppress topology/payload | Reproducible progress/terminal summary | Evidence retained by policy after reset | cross-tenant query, counter reconciliation, pagination | Glaux-specific **[P]** | Exact metric cardinality/retention 041/046–048 |

## 6. Scenario, Dataset, Run, Session, Clock, Source, and Resource Identity Model

### 6.1 Identity ledger

| Identity | Supplier/authority | Stability and uniqueness | Relationship |
|---|---|---|---|
| Scenario ID + version + digest | Scenario author; server validates registry/artifact | Stable logical name; immutable content at one version/digest | Used by many sessions/runs |
| Dataset ID + version + digest | Dataset publisher/release authority | Immutable artifact set; digest covers manifest-listed bytes | Used by scenarios/runs; never mutable “latest” during a run |
| Generator descriptor | Simulator supplies; server records | Algorithm name/version, implementation commit/build/image digest, config digest, platform-relevant mode | Seed is interpreted only with this descriptor |
| Seed | Scenario/run manifest | Exact typed bytes/string and derivation; not assumed portable across algorithms/builds | Input to generation activity |
| Session ID | Server-assigned UUIDv7 | Unique server-local control resource | Contains authorized runs and isolation scope |
| Run ID | Server-assigned UUIDv7 | Unique execution identity | Pins scenario, dataset, generator, seed, identifier and clock policy |
| Principal/test controller | Authentication authority/server observation | Credential/session dependent | Requests control actions; not source truth |
| Simulator publisher instance | GPC registration | Stable software deployment identity | Submits on behalf of source; may differ from controller |
| Represented synthetic source | GPC source registration | Stable within security domain/authority grant | Origin claim for canonical data |
| Logical action ID/ordinal | Immutable scenario manifest | Unique within manifest/run; stable across resume | Correlates input, expected outcome, server submission/result |
| GPC message/idempotency identities | Simulator under registered contract | Scoped by publisher/source/operation; semantics depend on replay mode | Protect individual effects/retries |
| Resource logical key | Scenario manifest | Stable comparison key | Maps to one server ResourceId per run in default mode |
| Canonical ResourceId/revision | Server | Server-assigned per IDR-SRV-016; immutable identity/revision rules | Mapping ledger links logical key and source UID |
| Virtual clock mapping/version | Run manifest and checkpoints; server records | Immutable piecewise segments/revision | Explains generated domain/source times |
| Checkpoint/operation/reset IDs | Server | Durable unique control/evidence identities | Correlate progress, resume, cleanup and audit |

### 6.2 Identifier modes

The default is **isolated remap**: every run uses a run-local source/UID namespace and a manifest mapping from logical keys to server-assigned ResourceIds. Equivalent runs compare logical graph, values, relations, and outcomes rather than demanding equal ResourceIds. This supports concurrency and preserves the accepted rule that ordinary clients cannot choose ResourceIds. **[A/P]**

**Stable-semantic mode** keeps source UID/logical entity identity across runs only in a clean, exclusively scenario-owned namespace where the manifest declares create-versus-update semantics, expected prior state, and collision policy. It is unsuitable for concurrent runs in one namespace. **Duplicate-test mode** deliberately reuses original GPC message/idempotency identity within the same scope to test deduplication. **New-run equivalence mode** derives new run-scoped keys/messages so equivalent data creates the intended new isolated effects. **Captured replay mode** preserves captured source IDs/times/digests but maps them through a non-operational source and explicit sanitization/authority profile. **[P]**

Resource UID namespaces must visibly denote synthetic authority without relying only on a human-readable label. Run-local IDs cannot link to operational/shared targets. A scenario that needs common reference data uses immutable read-only fixture references copied or provisioned into the same simulation namespace; those references are part of the manifest and ownership policy, not arbitrary production links. **[P]**

## 7. Capability Discovery and Session/Control Lifecycle

### 7.1 Capability discovery

`GET /_glaux/simulator/v1/capabilities` returns a policy-filtered private representation containing control-contract version, supported session/run/checkpoint/reset capabilities, allowed isolation classes, clock/identifier modes, manifest media/profile versions, data-plane GPC versions, enabled resource/data families by reference to ordinary capabilities, size/count/rate/concurrency/storage limits, maximum lease/run/reset-operation classes, and links to schemas/status resources. It is generated from the same deployment registry used by routing and authorization; documentation, route, and runtime parity is tested. **[A/P]**

In production, simulator controls are absent at route registration and omitted from OpenAPI/capability documents. A request receives `404` or a concealment-equivalent response; setting a header or possessing publisher credentials cannot enable them. A test deployment still requires authentication and authorization—“test mode” is not anonymous or all-powerful. **[P]**

### 7.2 Session and run lifecycle

Session states are `created → active → closing → closed`, with side terminal `failed` and `expired`. Creating/preparing the first run activates the session; `POST .../closures` moves it to `closing`, rejects new runs, and reaches `closed` only when every run has a terminal execution state. Closure does not reset data. A session records lease/expiry in real server time. Lease expiry prevents new runs/control and moves active runs according to policy (normally `paused` or `failed/expired`), but it does not delete data. Reconnecting or reauthenticating does not create a new session; the caller reads state and resumes through preconditions. **[P]**

Run states are:

`created → prepared → running ↔ paused → draining → completed`

with `failed`, `cancelled`, and `expired` terminal execution outcomes, followed where authorized by `resetting → reset` cleanup state. `resetting/reset` describes the cleanup dimension and does not rewrite the terminal execution outcome. A failed reset yields `reset-failed` with exact completed/residual scope; it never changes a failed run into completed. **[P]**

The simulator/harness requests or declares create, prepare, started, paused, resumed, draining, complete, fail, or cancel checkpoints. The server controls admission acceptance, stale-transition rejection, draining completion, expiry, reset eligibility, and final reset result. State mutation requires scoped idempotency plus strong `If-Match`; terminal execution outcomes are immutable except a separately recorded correction by authorized administration. **[A/P]**

Disconnect has no magical lifecycle meaning. The server continues already committed work, lets leases/status policies apply, and records the absence. Server restart reconstructs session/run state from durable control and admission ledgers; no in-memory counter or timer is authoritative. Abandoned sessions expire but retain evidence and require ordinary authorized cleanup. **[P]**

## 8. Synthetic Data, Provenance, Policy, and Isolation Model

Every simulation data effect carries immutable internal context: `originClass=synthetic|captured-sanitized-test`, session/run/scenario/dataset IDs and digests, simulator/publisher/source identities, controller where applicable, generator descriptor/seed/config, logical action, clock mapping, identifier mode, mapping/profile/contract versions, exact input digest, expected outcome classification, and validation/transaction/output correlations. These are provenance/evidence fields, not invented CSAPI members. W3C PROV supplies the Entity/Activity/Agent pattern: scenario/dataset/input/output are entities; generation/replay/transformation/admission/reset are activities; controller/simulator/publisher/source/server are separately role-qualified agents.[^4] **[N/A/P]**

Synthetic marking is enforced through multiple independent controls:

- dedicated deployment is preferred for destructive, security, conformance, performance, and DDIL lanes;
- otherwise a dedicated non-operational tenant/security domain, source registry, identifier namespace, database partition/schema or equivalent authority boundary, object-store prefix/key, broker subject/ACL namespace, and credentials are required;
- simulation sources cannot obtain operational source, federation/export, cross-boundary release, or real command-target grants by default;
- relationships and collection membership are constrained to the same simulation isolation domain;
- caches, search indexes, exports, backups, outbox messages, metrics, and logs retain the isolation label and cannot be merged into operational projections;
- control routes are production-disabled; and
- every read/write/reset is reauthorized rather than trusting a run header.

Coexistence of simulated and operational content in one logical tenant is rejected as the baseline. A shared physical deployment may host hard-separated simulation tenants only if security research and tests prove fail-closed row/object/broker/cache isolation. Demonstrations and training should use dedicated deployments because human interpretation and export risks remain even when row security is correct. **[P]**

Intentional invalid input uses the normal public/GPC boundary and expects rejection, quarantine only if the ordinary registered profile authorizes it, or another normal failure. The manifest marks the expected outcome; it cannot change the validator or policy. Server-internal fault hooks, if used by unit/component tests, are compile/build/profile restricted, not remotely reachable, and make affected results non-conformance evidence. **[A/P]**

Policy labels on simulated data are still validated. A synthetic label cannot downgrade classification/releasability, and a policy-invalid fixture does not enter canonical state. Captured operational data must be separately approved, sanitized, provenance-marked, and isolated; “simulation” does not authorize copying live tactical content into test stores. **[P]**

## 9. Replay, Determinism, Checkpoint, Resume, and Completion Semantics

### 9.1 Replay modes

| Mode | Preserved | Changed | Intended server result |
|---|---|---|---|
| Exact retry/replay | Body bytes, route, source/message/key, domain times, contract/profile | New transport attempt/receipt evidence only | Original stored outcome; no second canonical effect |
| Duplicate adversarial test | Selected same message/key with same or conflicting fingerprint | Explicit fixture case | Same-intent duplicate or `409` conflict/security signal |
| New isolated repeat run | Scenario/dataset/logical actions/expected semantics | Run/session IDs, run-local source/message/keys and ResourceId mapping | Equivalent new isolated graph/outcomes |
| Deterministic regeneration | Scenario, dataset, generator algorithm/build/config, seed | Generation activity and run-local identities | Equivalent logical corpus; byte equality only when manifest promises it |
| Captured/backfill replay | Original source IDs, domain times, message/digest when authorized | New ingestion/replay activity and simulation isolation binding | Historical facts or duplicate outcomes under declared policy |
| Shifted-time replay | Logical values/order plus explicit mapping | Domain timestamps and therefore content digest/identity as specified | New transformed facts with derivation provenance; never called exact |
| Faulted replay | Base action plus deterministic fault plan | Delay/drop/reorder/duplicate/corrupt behavior | Expected normal accept/reject/ambiguity/backpressure outcomes |

Determinism is scoped. The manifest must distinguish byte determinism, semantic determinism, identifier determinism, action-order determinism, server-state determinism, and timing/performance tolerance. A seed without algorithm/build/config and input artifact digests does not meet any complete reproducibility claim. Server-assigned UUIDv7 values, actual receipt/commit times, trace IDs, scheduling, and performance measurements are normally permitted nondeterminism; expected results compare logical mappings, constraints, ordering partial order, and bounded tolerances. **[A/P]**

### 9.2 Checkpoint and resume

A checkpoint includes server-assigned checkpoint ID, run revision, manifest/dataset digests, highest contiguous terminal manifest ordinal, explicit noncontiguous terminal bitmap/ranges where parallelism is allowed, per-state authoritative server counts, per-action result/digest references, pending actions/jobs, server transaction watermark, publication threshold/watermark if required, declared simulator offset/state and virtual-clock segment, discrepancy list, and real receipt/commit times. **[P]**

The server derives action outcome from idempotency/admission/canonical/job/outbox ledgers; caller counters are evidence only. A manifest action is checkpoint-terminal when it has a declared expected terminal outcome (`committed`, `duplicate`, `rejected`, `quarantined`, or another named state) and all required evidence exists. Receipt/authentication alone is not terminal. A contiguous checkpoint cannot skip an earlier unresolved action even when later actions finished, unless the manifest explicitly defines independent lanes and records each lane frontier. **[A/P]**

Resume uses the same session/run, exact manifest/dataset/contract/clock/identifier versions, and original per-action idempotency/message identities. The caller first reads run state/checkpoint, reconciles server truth, and resubmits only unresolved actions with their original keys. A changed manifest or mapping starts a new run or a versioned correction; it cannot silently continue. Server restart does not lose checkpoint truth. **[P]**

### 9.3 Completion

Completion is explicit, never inferred from disconnect or elapsed time. The manifest declares one completion profile:

- **commit-complete**: every expected action has a terminal server admission/commit outcome and all canonical transaction effects are durable; or
- **publication-complete**: commit-complete plus every required run-owned outbox item reached the named publication terminal/watermark.

The default is commit-complete. Publication-complete remains contingent on IDR-SRV-035 semantics. `draining` accepts no new ordinary actions except previously admitted retries/reconciliation and waits for declared conditions. Final summary contains manifest/result digest, state counts, action discrepancies, last checkpoints, resource mapping/ownership closure summary, transaction/publication watermarks, warnings, permitted nondeterminism evaluation, and cleanup eligibility. **[P]**

## 10. Virtual-Time and Temporal Mapping Semantics

### 10.1 Clock rule

Virtual time is **input data and provenance**, not a replacement server clock. The accepted IDR-SRV-018 taxonomy remains controlling: phenomenon, result, valid/effective, issue, execution, report, and event time are domain/source concepts; receipt, ingest, commit/transaction, publication, delivery, synchronization, lifecycle, evaluation, lease, cache, and retention times have separate owners. **[A]**

| Mode | Generator/domain mapping | Server treatment | Principal test use |
|---|---|---|---|
| Preserve | Keep original domain timestamps | Validate normally; record new receipt/commit | Exact/captured history/replay |
| Shift | `virtual = original + declared offset` | New transformed payload/activity/digest | Move a fixture into selected domain epoch |
| Scale | Piecewise `virtual = vAnchor + rate × (orchestratorWall - wallAnchor)` | Records mapping; validates submitted values; no pacing guarantee | Accelerated/slow scenario production |
| Pause | Rate zero at declared segment; generator stops new timed actions | Server continues real-time work; already in-flight requests proceed | Inspect stable run frontier |
| Step | Explicit next virtual instant/action boundary | Each generated record still normal data-plane request | Deterministic lifecycle progression |
| Burst | Preserve/compute domain times but submit without wall pacing | Backpressure and receipt times show actual burst | Throughput/backlog testing |
| Backfill | Original older domain times arrive now | Valid history if allowed; no arrival overwrite | DDIL/catch-up/late data |
| Clock-skew fault | Deliberately invalid/future/regressed source/domain time | Normal validation/reject/quarantine behavior | Temporal safeguard testing |

A mapping is immutable piecewise segments with `mappingVersion`, source clock domain, virtual anchor, orchestrator wall anchor (evidence only), rational rate, pause/step discontinuities, precision/uncertainty, and transform algorithm. Every affected action records the mapping segment. Rate changes create new segments rather than rewriting prior times. **[P]**

Server receipt/commit ordering uses actual server time plus monotonic transaction sequence. Authentication expiry, authorization/grant effective times, session lease, idempotency/replay retention, rate limiting, job timeout, audit time, retention/disposition eligibility, and outbox retry use server clocks and policy. Virtual time cannot prolong a credential, accelerate purge, evade throttling, or order unrelated sessions. **[A/P]**

Tests requiring deterministic “now,” freshness-boundary, lifecycle deadline, or retention behavior use explicit `datetime`/as-of inputs where standards allow, or an in-process injected clock in a disposable server build/harness. Such a clock is isolated to the test process/deployment, not remotely mutable through `/_glaux/simulator/v1`, and results are labeled project/component evidence rather than public conformance when the tested standard requires actual HTTP-time behavior. **[P]**

Time shifting may make a formerly valid record invalid—for example, future Observation result time—and the server must reject it normally. Command issue/report/execution semantics remain governed by IDR-SRV-036/037; the manifest cannot redefine legal task transitions by choosing a convenient clock.

## 11. Reset, Cleanup, Ownership, Safety, and Recovery Model

### 11.1 Reset hierarchy and decision

1. **Disposable recreation (preferred):** capture results, destroy the complete dedicated server/database/object/broker/cache/search environment, and recreate from pinned images/config/migrations/fixtures.
2. **Run-scoped application reset (conditional):** remove/tombstone only the proven exclusive closure of one run and restore its declared baseline in a persistent non-production test tenant.
3. **Session/namespace reset (exceptional):** allowed only when the entire namespace is exclusively owned by that session and no shared/foreign/held content exists.
4. **Global/shared/production reset:** prohibited through the simulator contract.

Testcontainers for Rust supports programmatic lifecycle and cleanup of container dependencies, which makes disposable integration/smoke environments feasible.[^9] Docker Compose teardown does not remove every volume by default and never removes external volumes; a reset harness must explicitly enumerate and verify PostgreSQL data, object versions, broker persistence, caches/indexes, exports, backups/PITR/WAL, and external resources rather than equating container deletion with data erasure.[^10] **[I/P]**

### 11.2 Ownership proof and preview

Every canonical effect created through a simulation run receives immutable internal ownership attribution. Reset eligibility is not a caller label or query. The server computes a dependency/copy graph containing canonical resources/revisions, relationships/memberships, observations/status/events, commands/feasibility/status/results, validation/raw/quarantine/staging artifacts, current/latest/materialized projections, idempotency/inbox/outbox/delivery records, caches/indexes, jobs, exports and other managed copies. Each candidate is classified exclusive-run-owned, shared immutable fixture, foreign/operational, held/restricted, derived rebuildable, evidence-retained, or unknown. **[A/P]**

A reset preview records run/session/policy revisions, requested postcondition, graph snapshot/watermark, closure digest, safe counts/categories, shared/foreign/hold blockers, derived rebuild plan, physical-copy limitations, estimated stages, irreversible boundary, expiry, and an opaque confirmation token. Preview makes no mutation. Any graph, ownership, hold, policy, or run revision change invalidates it. Unknown ownership blocks rather than broadens scope. **[A/P]**

Updates/deletes to preexisting/shared resources are prohibited in the default simulator profile precisely because “undo” is not safely equivalent to resetting a run. A specialized isolated fixture may mutate a run-created resource or a copied baseline resource owned exclusively by the run. Server before-images alone are insufficient where concurrent actors, external publication, or derived state exist. **[P]**

### 11.3 Execution, recovery, and postconditions

Reset is a durable asynchronous operation. The server rechecks authorization, ETag, token/digest, ownership, dependencies, holds, and policy under the applicable transaction immediately before each destructive stage. Small visibility changes may be atomic; large copy cleanup is a recoverable saga with durable stage state. New foreign/shared edges or holds after preview stop the operation with an explicit residual state. Retrying the same key returns/resumes the same job; a different intent under the key is `409`. Cancellation is allowed only before the declared irreversible boundary. **[A/P]**

The required postcondition names a baseline:

- `empty-run`: no run-owned canonical resource/effect is visible or publishable;
- `manifest-initial`: the run's pinned baseline resources/relationships exist at their declared initial revisions and no action effects remain; or
- `closed-evidence-only`: canonical test effects are absent while minimum run/reset provenance, audit, result digest, tombstone/anti-resurrection and disposition evidence remains.

Reset rebuilds or invalidates latest/current/extents/caches/indexes from surviving authoritative facts, reconciles outbox/publication work, and reports every residual/uncontrolled copy. It does not rewrite history to pretend the run never occurred. “Reset” never implies archive purge, backup expiry, cryptographic erase, or media sanitization. Those terms retain IDR-SRV-030 meanings and authorities. **[A]**

If complete postcondition proof cannot be obtained, state is `reset-failed` or `reset-partial` with completed stages, residual categories, retry/administrator action, and no clean-environment claim. A later Server restore must reapply the reset/tombstone ledger before serving, preventing a simulation run from reappearing. **[A/P]**

## 12. Dynamic-Data, Fault-Injection, DDIL, and Tasking-Loop Behavior

### 12.1 Data and tasking

Simulated metadata, DataStreams, Observations, status samples, System Events, ControlStreams, Commands, Feasibility resources, Command/Feasibility statuses and results use their ordinary routes and parent contracts. Simulator health reports use the GPC private health-evidence path and remain separate from System operational status. All schema, semantic, unit, temporal, geospatial, relationship, authority, policy, concurrency, transaction, provenance, and error checks remain enabled. **[N/A]**

The closed-loop Simulator may represent a synthetic controlled System and report its outcomes only under explicit source/target/status-result grants. Receiving a Command via query or future stream does not authorize reporting for it. Scripted acceptance, rejection, latency, timeout, duplicate, invalid transition, out-of-order result, and cancellation race are simulator actions whose submitted records face the normal server state machine. IDR-SRV-036 through 038 determine final legal transitions, Feasibility behavior, safety, dispatch, and audit. **[P]**

### 12.2 Fault placement

| Fault | Injection point | Server expectation |
|---|---|---|
| Malformed bytes/media/encoding | Simulator before HTTP/broker boundary | Bounded normal parse/media failure; no canonical effect |
| Invalid schema/semantics/units/time/reference/policy | Ordinary data request | Normal safe rejection/quarantine only if already authorized |
| Unauthorized source/path/command target | Test credential/grant matrix | Normal denial/concealment; no protected existence leak |
| Duplicate/conflicting identity | Reuse key/message according to manifest | Original result or conflict/security signal |
| Delay/reorder/drop/disconnect | Simulator or network proxy | Actual receipt/order/timeout behavior; retry same identity |
| Backlog/burst | Simulator pacing/spool | Normal 429/503/backpressure and fair admission |
| Partial batch | Manifested mixed items | Independently item-atomic complete ledger |
| Server crash/restart | Disposable harness/infrastructure | Durable commit/checkpoint recovery; no in-memory authority |
| Database/broker/object/cache failure | Component/integration harness | Transaction/outbox/reset saga failure contracts |
| Clock skew/regression | Submitted domain/source times | Normal temporal validation and explicit evidence |

A remote server fault-control endpoint is rejected. It would introduce an unusually powerful production-shaped attack surface and make evidence ambiguous about whether normal safeguards were actually exercised. Unit/component-only fault injection may use internal dependency seams under a test build; integration faults use disposable infrastructure/proxies. **[P]**

### 12.3 DDIL

The simulator can disconnect, buffer, reconnect, resume from checkpoints, replay original GPC message/key/source epoch/sequence, create gaps, reset epochs, and submit conflicting branches. It must preserve immutable contract/mapping/data identity and obey current authorization/backpressure on admission. The server never interprets run completion or a higher sequence as proof of loss-free continuity. Valid late records may enter history; current-state and reconciliation rules remain IDR-SRV-034/042/043. **[A/P]**

Application reset cannot be used to erase a DDIL conflict or duplicate-security signal before expected assertions are collected. A run checkpoint names the source frontiers and conflict states. Cleanup begins only after terminal evidence or an explicitly authorized cancellation/exception. **[P]**

## 13. Authentication, Authorization, Audit, Errors, Diagnostics, Progress, and Backpressure

### 13.1 Roles, scopes, and gates

Separate scopes are required for capability view, session create/read/close, run create/checkpoint/finalize, simulation data publication by family/source, task target/status/result emulation, reset preview, reset execute, reset approval where policy requires, operation read/cancel, and protected evidence inspection. A principal may hold multiple scopes, but authorization decisions and audit roles remain separate. Reset credentials should not be sent by the publisher data process when architectural separation is feasible. **[P]**

Three independent gates must all pass: deployment registers simulator routes; tenant/environment is explicitly non-operational and permits the capability; authenticated principal/instance has object/action scope. Network segmentation, separate issuer/audience, and sender-constrained credentials are Category G candidates. Production defaults to no registered route, no capability advertisement, no simulation source grants, and no reset executor. **[P]**

### 13.2 Errors, diagnostics, and audit

Control failures use RFC 9457 problems: stable type, safe status/detail, instance/correlation, run/session/operation ID only when authorized, current ETag/state, retry class, and safe next action.[^3] Use `400` malformed control/manifest, `401` authentication, `403`/concealed `404` authorization, `404` absent/disabled resource, `409` invalid transition/ownership/hold/key conflict, `412` stale precondition, `413` bounded size, `415` media/profile, `422` structurally valid but invalid manifest/control semantics, `428` required precondition, `429` caller/scope throttling, and `503` global unavailability. Asynchronous acceptance uses `202` only with durable operation `Location`. **[N/A/P]**

Diagnostics disclose own run action/state/counts, safe validator pointers, supported authorized capabilities, retry advice, and closure blocker categories. They do not expose other tenants/runs, operational resource existence, graph topology, policy/hold identities, credentials, payload fragments, SQL/object/broker details, or security detection logic. Expected invalid fixtures are referenced by digest/action rather than copied unbounded into logs. **[P]**

Audit covers capability access as policy requires; every session/run/checkpoint/terminal/reset operation; authentication/authorization denial; synthetic source/grant changes; manifest/dataset/generator/clock/identifier decisions; ordinary data attempts and outcomes through GPC; discrepancies between caller and server counters; duplicate/conflict/fault/overload; task outcomes; cleanup stages/holds/residuals; and disposable-environment evidence where imported. Provenance and audit cross-reference but do not duplicate unrestricted content. **[A/P]**

### 13.3 Progress and observability

Run progress derives from authoritative ledgers and reports at least `declared`, `received`, `authenticated`, `validated`, `accepted-for-commit`, `committed`, `rejected`, `quarantined`, `pending-dependency`, `duplicate`, `retryable-failed`, `unknown/reconcile`, `publication-pending`, and `published` counts; action/independent-lane frontiers; pending jobs; latest checkpoint; transaction/publication watermarks; discrepancies; reset eligibility/state; and bounded rate/backlog observations. Aggregate partial state never conceals item outcomes. **[A/P]**

OpenTelemetry trace context may correlate request flows, but trace IDs/baggage are not durable run identity. Official guidance warns that incoming context can be forged, outgoing values can expose internals, and baggage has no built-in integrity; Glaux therefore stores its own opaque run/action/submission IDs and sanitizes propagated context.[^11] Metrics use bounded dimensions and must not place unbounded scenario/action/resource IDs in high-cardinality labels. Exact telemetry schemas/cardinality remain IDR-SRV-046–048. **[I/P]**

Backpressure applies per principal, publisher, source, session, run, operation class, resource family, batch, staging/quarantine, and deployment. Simulation priority cannot be self-asserted. A performance run uses configured dedicated capacity but still records throttling and resource limits; bypassing quotas would fail to exercise production admission. `429`/`503` and `Retry-After` follow the GPC baseline. **[A/P]**

## 14. Scenario Manifest, Verification, and Interoperability Matrix

### 14.1 Minimum scenario manifest

| Required field | Contract |
|---|---|
| Scenario and dataset version | Stable IDs, immutable versions, release authority and exact content/manifest digests |
| Seed | Typed exact seed plus derivation; paired with generator algorithm/version/build/image/config/platform-relevant mode |
| Contract/profile versions | CSAPI/SensorML/SWE/GPC/simulation control/mapping/media/schema/policy expectation pins |
| Required capabilities | Exact data operations/media and private controls/identifier/clock/reset/completion modes; no inference from global conformance |
| Initial-state assumptions | Empty/baseline state, required immutable fixture graph, tenant/environment, source/grant setup, expected absence and server capability digest |
| Resource/identifier mapping | Logical keys, source UID policy, parent/relationship references, run-local/stable/duplicate mode, expected server-ID mapping rules |
| Clock mode and mapping | Clock domains, preserve/shift/scale/pause/step/burst/backfill/skew mode, piecewise mapping/precision/uncertainty, server-clock invariants |
| Ordered actions or input artifacts with checksums | Stable action IDs/ordinals/dependencies/parallel lanes, operation/path/media, artifact/content digest, source/message/key derivation, expected injection point |
| Expected responses/state/events | Exact status/problem class, headers/links, canonical graph/state/query results, validation outcomes, outbox/events and allowed alternatives |
| Checkpoints | Required action/lane frontiers, state counts, result digests, transaction/publication watermarks, resume behavior |
| Completion criteria | Commit- or publication-complete profile, terminal action accounting, permitted warnings/discrepancies, result-summary digest |
| Cleanup scope | Preferred disposable environment or exact run/session reset baseline, ownership expectations, evidence retained, copy/store inventory and postconditions |
| Permitted nondeterminism | Server IDs/times/traces/scheduling/performance tolerances/order partial sets explicitly named; everything else deterministic or failed |

The manifest is immutable after run preparation. Corrections create a new manifest version/run. Checksums cover exact bytes; semantic equivalence uses explicit canonical comparison rules and cannot substitute for integrity. Secret credentials and unrestricted captured sensitive payloads do not belong in the manifest. **[P]**

### 14.2 Verification and interoperability matrix

| Scenario | Preconditions/actions | Expected response/state/evidence | Cleanup assertion | Test class/handoff |
|---|---|---|---|---|
| Production absence | Production registry/config and ordinary publisher credential | No route/capability; header cannot enable; audit/concealment policy holds | None | Security/deployment 039/044/055 |
| Role separation | Publisher-only, controller-only, reset-only principals | Each can perform only its scoped plane/action; no privilege union by run ID | Empty session or disposable teardown | Security 039/039A |
| Deterministic metadata graph | Pinned scenario/dataset/seed/build, isolated remap | Same logical graph/links/values across runs; server IDs mapped, not assumed equal | Each run closure empty; evidence retained | Fixture/interoperability 053/056 |
| Manifest mutation | Prepare then alter digest/action/clock policy | Stale/conflict/new run required; original remains immutable | No unexpected data | Contract 050/052 |
| Exact retry/duplicate conflict | Replay same key/same intent then changed bytes | Original result/no effect, then `409`/audit signal | One canonical effect only | Transaction 029/052 |
| Checkpoint resume | Crash after selected commits/responses, restart Server/Simulator | Server counters/watermark authoritative; unresolved same-key retry; no skip/duplicate | Final closure/recreation | Recovery 044/052 |
| Pause/rate/step | External generator changes mapping with in-flight requests | Server times real; old/new segments recorded; no global ingestion pause | Reset all run effects | Temporal 034/042 |
| Preserve/shift/backfill/skew | Same base dataset in four clock modes | Exact vs transformed provenance; valid late history; invalid future/regression normal rejection | No stale current projection after reset | Dynamic 034/042 |
| Invalid-data corpus | Malformed/media/schema/unit/reference/policy/auth cases | Normal problems/no canonical effect; no bypass or leak | Rejection evidence per policy | Validation/security 023/053/055 |
| Batch partial | Mixed valid/invalid/duplicate items | Complete independently atomic item ledger and correct aggregate | Valid effects removed; invalid absent | Ingestion/fixture 032/052/053 |
| DDIL replay | Disconnect, spool, epoch reset, gap, late/conflict, reconnect | Same identities, explicit gaps/conflicts, backpressure, no arrival overwrite | Preserve terminal evidence before cleanup | DDIL 042/043 |
| Task lifecycle driver | Script every state, failure, timeout, duplicate/out-of-order/cancel race | Ordinary Command/Feasibility status/result rules and audit; no target-authority confusion | Safe terminal/cancel then reset/recreate | Tasking 036–038 |
| Reset preview blockers | Add shared/foreign edge, hold, hidden resource, stale ETag | Safe blocker categories; no mutation; preview invalidates after change | Original state remains | Lifecycle/security 030/039/041 |
| Reset crash matrix | Fail before/after visibility, derived, outbox and copy stages | Durable resumable job; exact residual; no false clean claim | Eventually baseline or explicit failure | Architecture/recovery 044/052 |
| Disposable teardown leakage | Persist DB volume, object version, broker state, cache/index | Harness detects residual state; recreation not declared clean | Enumerated stores empty/new identity | Deployment 045/052 |
| Performance burst | Dedicated environment, pinned load/actions | Measured accept/commit/publish, 429/503/backoff, no correctness loss | Recreate complete environment | Performance 054 |
| OSH/CS-Go/pygeoapi/client variants | Feed pinned positive/negative media/route/task examples | Standards-correct Glaux behavior; implementation quirks remain fixtures | Self-contained isolated data | Interoperability 056 |

The OS4CSAPI evidence shows why state-rich fixtures are necessary: historical servers and tests often exercised only empty or `COMPLETED` states, and status-only success could not prove query, dynamic, or task semantics. It recommends deterministic seeded self-hosted fixtures, known hit/miss values, full lifecycle states, immutable captures, and isolated mutation cleanup.[^7] SECD evidence further demonstrates that cleanup behavior, route asymmetry, and apparently successful writes can mislead unless persisted state and relationships are asserted; its mutations should be replayed only in disposable isolated data.[^8] **[I/P]**

OSH and Connected Systems Go provide useful handler, dynamic-data, asynchronous, batch, MQTT, and test scenario precedents, but neither pinned implementation supplies this full simulator session/reset/clock/evidence contract. Their behaviors remain compatibility fixtures, not the basis for weakening Glaux invariants.[^5][^6]

## 15. Downstream Handoff Matrix

| Topic/owner | Fixed input from IDR-SRV-033 | Decision still owned downstream |
|---|---|---|
| IDR-SRV-034 Dynamic update semantics | Run/source/action/clock mapping, late/backfill/replay modes, no arrival overwrite, reset projection rebuild | Observation/status current/latest/watermarks/correction specifics |
| IDR-SRV-035 Streaming/events | Simulator/broker uses normal pipeline; run completion can optionally require publication watermark; no Part 3 selection here | Protocol, subscriptions, event identity, replay, ACK and publication-complete threshold |
| IDR-SRV-036/037 Task lifecycle | Closed-loop source/target identity, scripted states/times/failures, normal routes/validation | Legal Command/Feasibility states, transitions, async/timeout/result contract |
| IDR-SRV-038 Command safety | Simulator authority separate from operational target authority; no test bypass; destructive lanes isolated | Safety interlocks, command authorization, dispatch and emergency behavior |
| IDR-SRV-039/039A Security | Separate data/controller/reset scopes; triple gate; production absence; hard isolation/anti-enumeration | Credential/token/network architecture, sender constraint, threat model and zero-trust enforcement |
| IDR-SRV-040 Policy | Synthetic is immutable context, not downgrade; captured data needs policy/sanitization | Label/profile syntax, releasability and cross-boundary controls |
| IDR-SRV-041 Audit | Required session/run/action/reset/task/fault/discrepancy events and correlations | Audit schema, retention, review and protected export |
| IDR-SRV-042/043 DDIL/sync | Replay modes, epoch/gap/checkpoint behavior, same-key resume, virtual/server-clock boundary | Freshness, conflict/reconciliation, disconnected auth and synchronization protocol |
| IDR-SRV-044–049 Architecture/deployment | Private paths/resources, durable state/counters/jobs, injectable clock only in disposable tests, recreation store inventory | Components, physical stores, APIs/config, observability, deployment gates and technology |
| IDR-SRV-050–056 Verification | Operation matrix and manifest/fixture suite in §§5.3/14 | Executable conformance, Rust, fixture, performance, security and interoperability harnesses |

## 16. Recommendations and Explicit Project Decisions

1. **Treat the Simulator as two roles: GPC-v1 publisher and separately scoped test controller. [A/P]** One principal may hold both only by explicit deployment policy; reset authority should remain separable.
2. **Reuse ordinary CSAPI/GPC data-plane operations and the one common write pipeline. [A]** Synthetic traffic gets normal validation, policy, transaction, persistence, provenance, error and outbox behavior.
3. **Adopt the narrow `/_glaux/simulator/v1` control resources in §1 and reject a broad server scenario engine. [P]** The Server registers, observes and cleans runs; Simulator/orchestrator generates and paces actions.
4. **Make simulator controls absent from production by default. [P]** Route registration, capability advertisement, environment/tenant, network and object authorization all fail closed.
5. **Adopt immutable scenario/dataset manifests and server-assigned session/run/resource IDs. [A/P]** Default to run-local logical-key mapping; never force deterministic ResourceIds through ordinary POST.
6. **Define reproducibility beyond a seed. [P]** Pin generator algorithm/build/image/config, data/artifact digests, versions, identifier/clock policy, action graph, expected outcomes and permitted nondeterminism.
7. **Keep virtual time limited to source/domain data and recorded mapping. [A/P]** Never modify server security, receipt, commit, audit, lease, retention, idempotency or backpressure clocks through remote simulation controls.
8. **Use durable authoritative checkpoints and exact same-key resume. [A/P]** Caller counters are claims; server admission/transaction/outbox ledgers determine progress and completion.
9. **Prefer disposable environment recreation for CI, conformance, security, performance and destructive faults. [I/P]** Enumerate and verify all databases, volumes, objects, broker state, caches, indexes and recovery copies.
10. **Allow application reset only through preview plus high-risk confirm against exclusively run-owned closure. [A/P]** Block shared, operational, foreign, held or ambiguous ownership and preserve evidence/anti-resurrection state.
11. **Reject remote validation-bypass and server fault-injection controls. [P]** Submit invalid records normally; inject transport/component faults through isolated simulator/proxy/test-harness seams.
12. **Keep synthetic, captured-test, training and operational authority separate. [P]** Do not rely on payload text; propagate immutable context through storage, search, cache, broker, export, metrics and cleanup.
13. **Use ordinary tasking resources and separate command target/report grants. [N/A/P]** Simulated target possession or run control never implies operational command authority.
14. **Adopt commit-complete as the default run completion profile. [P]** Publication-complete remains an optional IDR-SRV-035-defined threshold.
15. **Use the scenario manifest and verification matrix in §14 as mandatory handoff. [P]** Every operation gets positive, negative, authorization, failure/retry and cleanup assertions.
16. **Do not implement or advertise draft Connected Systems Part 3 through this topic. [P]** Simulator streaming remains an input to IDR-SRV-035.

## 17. Risks, Contradictions, Assumptions, and Unresolved Questions

| Item | Current resolution | Owner/review trigger |
|---|---|---|
| Simulator repository has mission text but no implementation/interface/release | Pin `fc5dd86`; define server proposal independently; do not claim compatibility | Recheck when Simulator adds design/code/tag |
| Broad reset could destroy shared/operational state | Prefer recreation; app reset requires preview, exclusive closure, preconditions and high-risk scope | Security/lifecycle/architecture 039–045 |
| Container teardown may leave volumes/external stores | Complete store inventory and post-teardown probes required | Deployment/test 045/052 |
| Deterministic ResourceIds conflict with server identity authority/concurrency | Default logical mapping/run-local source UID; stable mode only isolated clean namespace | API/fixture 044/053 |
| Same idempotency key suppresses new run; new key defeats duplicate test | Replay mode explicitly controls scope/key derivation and expected outcome | Transaction/fixture 052/053 |
| Virtual clock could corrupt retention/security/task time | No remote/global server clock; domain-only mapping; injectable clock disposable/in-process | Dynamic/security/DDIL 034/039/042 |
| Simulator pause cannot stop in-flight requests atomically | Pause is generator/orchestrator state; checkpoint records observed frontier/discrepancy | Simulator product plus fixture assertions |
| Shared physical deployment isolation may leak through caches/brokers/metrics | Dedicated deployment preferred; hard tenant test required before coexistence | Security/architecture/observability 039/044–048 |
| Captured operational data may remain sensitive | Separate approval/sanitization/provenance and non-operational storage; synthetic label not enough | Policy/security 040/055 |
| Application reset cannot prove physical purge/backup removal | Use exact IDR-030 copy/disposition states and residual report; no erase claim | Lifecycle/deployment 030/045 |
| Completion may wait on undefined publication semantics | Default commit-complete; publication profile deferred | Streaming 035 |
| Dynamic latest/freshness and task state rules incomplete | Preserve clocks/actions and avoid preempting legal semantics | 034, 036–038, 042 |
| Exact control schemas, media types, leases and numeric limits unset | Behavioral contract fixed; generated OAD/config/performance work supplies values | 044–050/054 |
| Part 3 and broker behavior mutable | No selection/implementation; normal pipeline and run correlation are transport-neutral | 035 and upstream delta review |

Assumptions are limited to accepted GPC-v1, durable server admission/control ledgers, an enforceable non-operational isolation domain, and deployment ability to omit control routes. If exclusive ownership cannot be represented or proven, application reset is blocked rather than approximated. This report neither requires the Simulator to expose a particular control API nor selects Testcontainers/Docker as production dependencies; they demonstrate the feasibility and caveats of disposable test environments.

## 18. Validation Against This Plan's Success Criteria

| Topic plan success criterion | Status | Evidence |
|---|---|---|
| Classify every simulator use case as data, control, orchestration, unsafe or unresolved | Met | §3 and operation matrix §5.3 |
| Reuse write/publisher contracts absent evidence-backed separate mechanism | Met | §§4–5; recommendations 1–3 |
| Distinguish scenario, dataset, seed, run, session, simulator, source, principal, clock and generated-resource identity | Met | §§2.2, 6 |
| Define lifecycle, idempotency, concurrency, disconnect, restart and expiry | Met | §7 and §5.3 |
| Make replay modes, identifiers, order, checkpoints, resume, completion and evidence explicit | Met | §9 and manifest §14.1 |
| Separate virtual/source/domain time from server receipt/commit/publication time | Met | §10 |
| Reset ownership, authorization, preview, transaction, recovery, audit and postconditions protect unrelated state | Met | §11 and reset rows §5.3 |
| Synthetic provenance/environment isolation prevents operational confusion | Met | §8 |
| Invalid/fault scenarios exercise normal safeguards | Met | §§8, 12 |
| Tasking, DDIL, streaming, security, observability, fixture, performance and interoperability handoffs explicit | Met | §§12, 14–15 |
| Every recommended operation has positive, negative, authorization, failure and cleanup verification | Met | §§5.3 and 14.2 |
| Evidence, project decisions, assumptions and unresolved issues visibly distinguished/reproducible | Met | Labels throughout; §§2.3, 17, 19 |
| Polished, recommendation-first, independently readable and self-contained | Met for review | §1 plus complete 19-section report, exact operation matrix and manifest contract |

Research phases 1–6, report drafting, and author review are complete. Plan-owner acceptance remains deliberately unchecked while this report is **In Review**. The next two workflow actions are: (1) the Glaux Project Lead accepts IDR-SRV-033 after review; and (2) in the same instruction, authorizes execution of exactly IDR-SRV-034. The combined response pattern is **`accept IDR-SRV-033 and proceed`**; under the established workflow, a later bare **`proceed`** may express that combined action. This report neither records its own acceptance nor starts IDR-SRV-034.

## 19. References

### Standards and project sources

1. [OGC API - Connected Systems - Part 1: Feature Resources, OGC 23-001, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html).
2. [OGC API - Connected Systems - Part 2: Dynamic Data, OGC 23-002, Version 1.0](https://docs.ogc.org/is/23-002/23-002.html).
3. [OGC API - Connected Systems `v1.0.0`, commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2).
4. [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html).
5. [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html).
6. [W3C PROV-O, Recommendation 30 April 2013](https://www.w3.org/TR/prov-o/).
7. Project-controlled AEP baseline `AC/224(JCGISR)D(2026)0005`, 27 April 2026, SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`; not redistributed; used through accepted project findings.
8. [OGC API - Connected Systems Upstream Standards-History Evidence Register, Version 1.11](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md).
9. [IDR-SRV-016 Identifier, URI, and Resource Lifecycle Strategy](idr-srv-016-identifier-uri-and-resource-lifecycle-strategy-report.md).
10. [IDR-SRV-018 Temporal, Validity, and Freshness Model](idr-srv-018-temporal-validity-and-freshness-model-report.md).
11. [IDR-SRV-019 Provenance, Lineage, Quality, and Trust Metadata Model](idr-srv-019-provenance-lineage-quality-and-trust-metadata-model-report.md).
12. [IDR-SRV-020 Status, Availability, and System Event Model](idr-srv-020-status-availability-and-system-event-model-report.md).
13. [IDR-SRV-023 Schema and Encoding Validation Strategy](idr-srv-023-schema-and-encoding-validation-strategy-report.md).
14. [IDR-SRV-029 Transaction, Consistency, Idempotency, and Concurrency Strategy](idr-srv-029-transaction-consistency-idempotency-and-concurrency-strategy-report.md).
15. [IDR-SRV-030 Data Lifecycle, Retention, Archival, and Deletion Strategy](idr-srv-030-data-lifecycle-retention-archival-and-deletion-strategy-report.md).
16. [IDR-SRV-031 Server Write and Ingestion Model](idr-srv-031-server-write-and-ingestion-model-report.md).
17. [IDR-SRV-032 Publisher-to-Server Contract Boundary](idr-srv-032-publisher-to-server-contract-boundary-report.md).

### Simulator, tooling, observability, and implementation/test evidence

18. [Glaux Simulator README, `main` commit `fc5dd86d3be9aa583e911d5ebbf50e0156e80638`, June 7, 2026](https://github.com/DGIWG-P507/glaux-simulator/blob/fc5dd86d3be9aa583e911d5ebbf50e0156e80638/README.md).
19. [Testcontainers for Rust documentation, accessed September 14, 2026](https://rust.testcontainers.org/).
20. [Testcontainers for Rust repository, `main` commit `02ebb999af039888afa538b4b6e395cdc22c28d7`, accessed September 14, 2026](https://github.com/testcontainers/testcontainers-rs/tree/02ebb999af039888afa538b4b6e395cdc22c28d7).
21. [Docker Compose `down` documentation, accessed September 14, 2026](https://docs.docker.com/reference/cli/docker/compose/down/).
22. [OpenTelemetry context propagation guidance, accessed September 14, 2026](https://opentelemetry.io/docs/concepts/context-propagation/).
23. [OpenTelemetry baggage guidance, last modified February 22, 2026; accessed September 14, 2026](https://opentelemetry.io/docs/concepts/signals/baggage/).
24. [IDR-SRV-014A OpenSensorHub implementation study; OSH `v2.0.2` commit `235c0eabf24b6d6137b499b4402943d2794b70e6`](idr-srv-014a-osh-csapi-server-implementation-study-report.md).
25. [IDR-SRV-014B Connected Systems Go implementation study; CS-Go `v1.0.4` commit `244f4dd586da685d4d9b75e43f73001028b5bd0e`](idr-srv-014b-connected-systems-go-csapi-server-implementation-study-report.md).
26. [IDR-SRV-014E OS4CSAPI client smoke-test study; `phase-9` commit `754411897173c2ec4debaa9bcf4ed9e0f8a9e230`](idr-srv-014e-os4csapi-client-smoke-test-findings-study-report.md).
27. [IDR-SRV-014F SECD interoperability study; evidence commit `f018fd129bf0d0d1ce75e68198e3ab4d99d937a0`](idr-srv-014f-secd-interoperability-findings-study-report.md).
28. [52North Connected Systems pygeoapi proof of concept](https://github.com/52North/connected-systems-pygeoapi).

[^1]: The pinned Glaux Simulator repository contains only its README across two commits and has no tag/release or interface/test implementation. The README states the mission summarized in §1. Accessed and commit-verified 2026-09-14.
[^2]: RFC 9110 controls HTTP methods, status semantics, strong validators/preconditions, retry-relevant behavior and `Retry-After`. Accessed 2026-09-14.
[^3]: RFC 9457 defines reusable problem details and cautions against exposing implementation details that create security risk. Accessed 2026-09-14.
[^4]: PROV-O defines Entity, Activity, Agent, generation, derivation, association and related provenance concepts and permits domain specialization. The Glaux simulation mapping is a project specialization. Accessed 2026-09-14.
[^5]: Accepted IDR-SRV-014A pins OSH `v2.0.2` at `235c0eab...` and identifies useful typed/asynchronous/test patterns but no controlling Glaux simulator contract.
[^6]: Accepted IDR-SRV-014B pins Connected Systems Go `v1.0.4` at `244f4dd...`; its REST/MQTT/batch implementation is informative and does not define reset, virtual-time or safe simulation isolation.
[^7]: Accepted IDR-SRV-014E pins the OS4CSAPI `phase-9` test corpus at `75441189...` and finds deterministic seeded, state-rich, known-outcome fixtures necessary because status-only and thin live data do not prove semantics.
[^8]: Accepted IDR-SRV-014F pins SECD evidence at `f018fd12...`, documents controlled mutation/cleanup and semantic failures, and recommends disposable isolated write tests rather than shared public mutation.
[^9]: Testcontainers for Rust describes programmatic creation and cleanup of container-based dependencies for automated integration/smoke tests. It is feasibility evidence, not a selected Glaux dependency. Accessed 2026-09-14.
[^10]: Docker documents that Compose `down` removes defined containers/networks by default, does not remove anonymous volumes by default, and never removes external networks/volumes; `--volumes` changes part of that scope. This supports explicit store inventory and verification. Accessed 2026-09-14.
[^11]: OpenTelemetry documents that propagated context can be forged or reveal internals, that sensitive baggage can reach unintended services, and that baggage has no built-in integrity. Glaux therefore treats trace/baggage only as observability context. Accessed 2026-09-14.

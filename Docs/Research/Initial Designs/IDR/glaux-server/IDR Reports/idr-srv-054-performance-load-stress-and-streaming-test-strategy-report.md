# Section 054: Performance, Load, Stress, and Streaming Test Strategy - Research Report

**Topic ID:** IDR-SRV-054<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-054 Performance, Load, Stress, and Streaming Test Strategy](../IDR%20Plans/idr-srv-054-performance-load-stress-and-streaming-test-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** performance scope and non-goals; taxonomy; external, internal and resource metrics; threshold and regression governance; tools; deterministic datasets and workload models; reference profiles; API/query/geospatial/time-series, ingestion/validation, streaming/replay/backpressure, command/policy, DDIL/synchronization behavior; stress, endurance and recovery; CI, evidence and downstream handoffs<br>
**Methodology Used:** accepted-requirement extraction; current primary-tool and database documentation review; workload/envelope modeling; measurement-point and clock analysis; open/closed load and saturation analysis; statistical/noise and gate analysis; safety/privacy review; implementation/community lesson reconciliation; downstream synthesis<br>
**Research Time:** Approximately 46 hours of AI-assisted execution on September 16, 2026<br>
**Accepted Verification Baseline:** IDR-SRV-050 through IDR-SRV-053 independent conformance, normalized traceability, multi-layer Rust TDD and governed deterministic corpus strategies<br>
**Current Tool Evidence:** k6 `v2.2.0`, Criterion `0.8.2`, Vegeta `v12.13.0`, hey `v0.1.5`, Locust `2.46.5`, JMeter `5.6.3`; PostgreSQL 18/PostGIS 3.6 reference candidate; Prometheus/OpenTelemetry primary documentation checked September 16, 2026<br>
**Document Purpose:** Define a reproducible performance-measurement and stress strategy without implementing tests, asserting production capacity, treating latency as conformance, enabling physical commands, or selecting operational infrastructure/SLOs<br>
**Author:** OpenAI Codex<br>
**Date:** September 16, 2026<br>
**Last Updated:** September 16, 2026

---

## Evidence and Decision Legend

- **[N] Normative:** approved external standard or normatively incorporated artifact.
- **[A] Accepted project baseline:** accepted Glaux report or governing project decision.
- **[D] Direct documentation:** official tool, runtime, protocol, database or observability documentation.
- **[I] Implementation evidence:** another implementation, client, public endpoint or community result; informative only.
- **[T] Test evidence:** reproducible measurement with frozen target, environment, workload, data and method.
- **[E] Analysis:** reasoned synthesis from identified evidence.
- **[P] Project recommendation:** proposed Glaux decision pending acceptance of this report.
- **[X] Explicit boundary:** excluded claim or later-topic responsibility.

A valid response can be too slow; a fast response can be wrong; a green conformance run does not establish capacity; and a performance result outside its exact environment/workload envelope is not transferable by assumption. **[N,A,E]**

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Performance Requirement Extraction Methodology
5. Performance Test Taxonomy
6. Metrics, Thresholds, and Evidence Model
7. Tooling Evaluation
8. Workload and Data Set Strategy
9. Deployment Profile and Reset/Teardown Findings
10. API, Query, Geospatial, and Time-Series Performance Findings
11. Ingestion, Validation, and Source-Trust Performance Findings
12. Streaming, Event, Replay, and Backpressure Performance Findings
13. Command, Control, Security, and Policy Performance Findings
14. DDIL, Synchronization, and Conflict Performance Findings
15. CI, Regression, Reporting, and Artifact Findings
16. Downstream Topic Handoff Matrix
17. Recommendations
18. Risks, Constraints, and Open Questions
19. Validation Against This Plan's Success Criteria
20. References

---

## 1. Executive Summary

Glaux should implement a **correctness-gated, envelope-bound performance program**. Every run first proves that its requests, events and state transitions remain valid; only then may latency, throughput or capacity be interpreted. Every result is bound to a versioned workload, corpus digest, deployment/profile/configuration, server image and migration state, load-generator/tool version, hardware/cgroup resources, database state, telemetry settings, network topology, clock method and warm/cold condition. Results from a developer laptop, shared cloud runner, fixed reference host and public demo are separate evidence classes. **[A,E,P]**

Use k6 `v2.2.0` as the first external HTTP load orchestrator because it provides scenarios, open arrival-rate executors, protocol/custom metrics, tags, thresholds and machine-readable outputs. Arrival-rate tests expose overload instead of slowing the offered rate as server responses lengthen; `dropped_iterations` is a generator-capacity or target-saturation signal, never silently discarded. Core k6 WebSocket support is available, but accepted Glaux streaming starts with SSE. Use a small Rust streaming probe as the authoritative SSE cursor/replay/correctness and timing client; evaluate the official `xk6-sse` extension only after pinning its binary and comparing it with that probe. Optional MQTT performance may later use a pinned official `xk6-mqtt` extension plus an independent Rust subscriber. **[A,D,P]**

Use Criterion `0.8.2` for pure codec, canonicalization, validation, filter, cursor and state-machine microbenchmarks on a stable dedicated runner. Shared virtual CI is too noisy for merge-blocking time comparisons; PR jobs compile and exercise the benchmark mechanics, while fixed nightly/reference jobs perform statistical comparisons. A regression becomes blocking only after the benchmark is stable and the change exceeds both a practical effect budget and statistical/noise criterion, with a confirmatory rerun. Microbenchmarks diagnose components and never substitute for API, database or streaming load. **[D,E,P]**

PostgreSQL/PostGIS testing combines externally observed query latency with protected server spans/metrics, connection-pool state, `pg_stat_statements`, operating/cgroup resources and controlled `EXPLAIN (ANALYZE, BUFFERS, WAL, SETTINGS, FORMAT JSON)` evidence. Plans are compared by named invariants and measured costs, not exact golden text; planner/version/statistics/cache state are evidence. `EXPLAIN ANALYZE` actually executes the statement, so writes run only in an isolated disposable database with explicit rollback/reset semantics. Native PostgreSQL time-series remains the baseline; a TimescaleDB comparison occurs only if accepted adoption gates are reached. **[A,D,P]**

The measurement model records offered, started, completed, successful and dropped work; goodput; p50/p90/p95/p99 and maximum diagnostic latency; errors/timeouts/invalid results; bytes; CPU/throttling, memory, I/O and connections; pool wait; locks/temp/WAL/query plans; queue depth and oldest age; outbox/ingest/stream lag; event gaps/duplicates; reconnect/catch-up/recovery; and telemetry/export drops. Percentiles require their sample count and method. With fewer than 10,000 successful observations, p99 is informational because its tail contains fewer than roughly 100 samples. Histograms use stable units/buckets or supported native histograms; precomputed quantiles are never averaged across instances. **[D,E,P]**

Thresholds have four meanings that must not be conflated: correctness/safety gates; generous sanity limits that catch hangs; relative regression budgets on controlled hardware; and absolute readiness objectives inside a declared profile. PR smoke runs on a small deterministic dataset for roughly three minutes after warm-up and blocks on incorrect responses, unexpected errors/timeouts, dropped offered work, unbounded queue/resource growth or failure to drain—not on noisy p99 changes. Nightly/reference runs establish stable baselines and use paired/confirmatory regression decisions. Stress tests locate saturation and verify bounded degradation/recovery; they do not “pass” because the server survived an uncontrolled assault. **[P]**

This report defines a provisional public-demo acceptance envelope so the phrase “demo ready” is testable without implying production readiness: one declared 4-vCPU/8-GiB single-node Compose host with PostgreSQL/PostGIS and TLS proxy; the public fixture pack expanded to 100 systems, 1,000 datastreams and 1 million observations; 25 browsing users, 10 offered HTTP requests/s, 2 observation ingests/s and 25 SSE subscribers receiving 2 events/s for 30 minutes. Within that exact envelope, unexpected request/event loss is zero, HTTP error/timeout rate is at most 0.1%, simple-read p95 is at most 500 ms, bounded query p95 at most 1 s, commit-to-SSE p95 at most 1 s/p99 at most 2 s, no load iterations are dropped, memory remains below 80% of its limit without sustained post-warm growth, and a two-minute 2× burst drains its oldest backlog within 30 seconds. These are initial project guardrails to validate and revise with evidence, not operational SLOs. **[P,X]**

Nightly workloads use small and medium generated grids; manual/release capacity and endurance use parameterized large grids. Large data is generated/hydrated under IDR-SRV-053 manifests, never committed as an opaque database dump. Workloads distinguish cold/warm cache, item/list/query sizes, spatial/temporal selectivity, ingestion validity/replay mix, subscriber filters, policy visibility, command simulation and DDIL/sync backlog. A single aggregate “requests per second” result is rejected because it hides data scale, response size and work mix. **[A,P]**

Streaming performance is sequence- and checkpoint-based. Publisher readiness barriers replace sleeps; each event binds commit/outbox position, publication and subscriber receipt; slow consumers read at a controlled rate; reconnect occurs at a named sequence and resume proves allowed duplicates, no gaps and bounded catch-up. Same-host monotonic clocks are preferred. Cross-host latency is reported only with measured clock synchronization/skew; otherwise sequence progress and interval durations remain valid while one-way latency is inconclusive. SSE is measured first, MQTT only in the default-off experimental profile, and Kafka/NATS/WebSocket remain unselected absent measured need. **[A,P]**

No destructive stress, soak, command or fault test targets a public endpoint. Command performance uses only the effect-recorder simulator and reports validation, feasibility, authorization, persistence, event and simulated-gateway stages separately. Policy tests report visible goodput and protected aggregate cost without revealing hidden counts/labels. Security abuse and final command assurance remain IDR-SRV-055; named client performance expectations remain IDR-SRV-056. **[A,X]**

No unresolved issue blocks the strategy. Operational workloads, hardware, user populations and SLOs require stakeholders and field evidence not present in IDR, so the future operational-reference tier deliberately has no numeric guarantee. Acceptance authorizes none of IDR-SRV-055 through IDR-SRV-057, performance implementation, production capacity claims, public stress, physical effects or accreditation/readiness claims. **[X]**

## 2. Scope and Plan Alignment

### 2.1 Completed Scope

This report completes all six authorized phases by:

- converting accepted API, storage, query, ingestion, streaming, command, policy, DDIL, synchronization, observability, deployment, fixture and test findings into measurable workloads;
- defining microbenchmark, baseline, load, spike, stress, endurance, scalability, streaming and recovery taxonomy;
- fixing metric definitions, clocks, denominator and evidence semantics;
- evaluating all tools named by the plan and selecting a bounded first stack;
- defining deterministic dataset grids, workload mixes, reference profiles and reset/teardown;
- defining functional-area and streaming/backpressure tests;
- specifying PR, nightly, weekly/manual, demo and release-candidate thresholds/gates; and
- producing the required matrix, implementation proofs, recommendations and explicit handoffs.

### 2.2 Explicit Boundaries

This report does not:

- implement load scripts, benchmarks, dashboards, datasets or infrastructure;
- assert that CSAPI, SensorML, SWE Common, HTTP or a deployment profile imposes a numeric latency/throughput requirement where it does not;
- establish operational user populations, mission workload, hardware sizing, SLO/SLA, availability, RTO/RPO or accreditation targets;
- make a conformance assertion from performance or a capacity assertion from functional correctness;
- select Kafka, NATS, WebSocket, TimescaleDB, managed load service or production observability backend;
- test public/live third-party systems destructively or use uncontrolled feeds/operational data;
- permit real command endpoints, physical effects, inbound Part 3 or unrestricted broker publication; or
- authorize IDR-SRV-055 or later research.

### 2.3 Research Question Coverage

| Plan theme | Status | Evidence |
|---|---|---|
| functions, workload classes and non-goals | Complete | Sections 4–5, 10–14 |
| performance/load/stress/soak/streaming taxonomy | Complete | Section 5 |
| metrics, units, clocks, thresholds and evidence | Complete | Section 6 |
| tool evaluation and selections | Complete | Section 7 |
| datasets, deterministic generation and workload mixes | Complete | Section 8 |
| deployment profiles, reset and teardown | Complete | Section 9 |
| API/query/geospatial/time-series | Complete | Section 10 |
| ingestion/validation/source trust | Complete | Section 11 |
| streaming/event/replay/backpressure | Complete | Section 12 |
| command/security/policy | Complete with security depth deferred | Section 13 |
| DDIL/synchronization/conflict | Complete | Section 14 |
| CI/nightly/manual/RC/operational tiers | Complete | Sections 6, 15 |
| observability/reporting/artifacts | Complete | Sections 6, 15 |
| implementation/community lessons | Complete and non-normative | Sections 3, 10–12 |
| downstream handoffs | Complete | Section 16 |

## 3. Evidence Base and Authority Classification

### 3.1 Primary and Accepted Sources

| Source | Version/status checked | Authority/use | Limitation |
|---|---|---|---|
| CSAPI Parts 1/2, OGC 23-001/23-002 | published 1.0; artifact tag `v1.0.0` | valid API/resource/dynamic-data behaviors and payloads | no project numeric capacity objective inferred |
| HTTP RFC 9110, SSE HTML standard, WebSocket RFC 6455, MQTT 5, CloudEvents | published/current | transport semantics and measurement boundaries | optional transports remain profile decisions |
| PostgreSQL | 18 reference candidate; current docs checked 2026-09-16 | EXPLAIN, runtime statistics, locks, I/O, query measurement | plans depend on version/config/stats/data/cache |
| PostGIS | 3.6 reference candidate | spatial operators/index behavior | query shapes and datasets remain project-specific |
| IDR-SRV-025–049 | accepted project baseline | persistence, query, lifecycle, streaming, command, security, DDIL, sync, architecture, deployment and observability | no numeric SLO imported where explicitly deferred |
| IDR-SRV-050–053 | accepted verification baseline | evidence separation, traceability, test tiers and deterministic corpus | performance thresholds were intentionally deferred here |

### 3.2 Current Tool and Observability Evidence

| Source | Pin observed 2026-09-16 | Relevant capability | Decision caveat |
|---|---|---|---|
| k6 | `v2.2.0` | scenarios, open arrival rate, checks, thresholds, HTTP/WebSocket metrics and result outputs | SSE/MQTT require separately pinned official extensions or independent probe |
| Criterion.rs | `0.8.2` | statistical Rust microbenchmarks, confidence/change reports | noisy shared virtualization undermines timing gates |
| Vegeta | `v12.13.0` | simple constant-rate HTTP attack/report model | less suitable than k6 for multi-stage stateful mixes/streaming |
| wrk | repository/no current GitHub release | high-rate HTTP benchmarking | Lua/closed-loop and limited evidence/orchestration; not baseline |
| hey | `v0.1.5` | quick HTTP request sanity | limited workload and evidence model |
| Locust | `2.46.5` | Python user workflows/distribution | second language/runtime and user-count closed model not needed first |
| JMeter | `5.6.3` | broad protocol/plugin ecosystem | heavy runtime/GUI/plugin complexity; no first-use advantage |
| Prometheus | current docs checked | counters, gauges and aggregatable histograms | bucket/resolution/cardinality governance required |
| OpenTelemetry | current docs checked | trace/metric/log correlation and sampling | telemetry is diagnostic, not audit or load truth |
| nextest | accepted IDR-SRV-052 pin policy | functional test timing and orchestration | test duration is diagnostic, not service benchmark |

Prometheus recommends native histograms when supported and explains that summaries' precomputed quantiles cannot be meaningfully aggregated across instances; classic histogram error depends on buckets. k6 explicitly exposes `dropped_iterations` for arrival-rate executors. Criterion documentation warns that virtualized cloud CI noise can generate misleading changes. These behaviors materially shape the strategy. **[D,E]**

### 3.3 Informative Implementation Lessons

- OSH and CS-Go demonstrate broad graphs and live/streaming behavior, but mutable public endpoints cannot produce reproducible capacity evidence. **[I]**
- CS-Go's typed storage/query/event paths and tests offer candidate workload shapes; its hardware, database contents and implementation results are not a Glaux target. **[I]**
- Client/SECD/Explorer work shows that document size, nested traversal, link behavior and page strategy affect perceived responsiveness; named clients still require IDR-SRV-056 measurement. **[I]**
- pygeoapi and broader OGC API implementations show that configuration/cache/plugin choices change results enough that environment/profile facts are indispensable. **[I]**

### 3.4 Evidence Hierarchy

Normative sources define valid behavior; accepted reports define Glaux architecture/profile/safety; official tool documentation defines mechanisms; controlled Glaux measurements establish performance; implementation examples suggest workloads only. A dashboard screenshot, single benchmark, public endpoint observation, average latency, fastest run or vendor default is not sufficient evidence. **[E,P]**

## 4. Performance Requirement Extraction Methodology

Each workload derives from an assertion-level behavior or risk using:

`requirement/decision → user/system operation → data shape/selectivity/state → offered-work model → concurrency/dependency/fault condition → correctness oracle → external/internal/resource observation points → envelope → metric/statistical method → threshold meaning → evidence/trace IDs → downstream owner`

This prevents a generic “500 users” test from obscuring what those users do, what data they access, whether the target remains correct and why the result matters. **[A,E,P]**

### 4.1 Workload Dimensions

- operation and route/use-case class, not raw unbounded URL;
- read/write/ingest/event/command/sync mix and payload/response bytes;
- dataset graph width/depth, observation volume and spatial/temporal selectivity;
- valid/invalid/duplicate/denied/hidden outcome mix;
- open offered arrival rate, closed concurrency or controlled subscriber count;
- cold/warm application/database/filesystem/cache state;
- connection reuse/TLS/proxy/compression/media/encoding settings;
- telemetry/log/trace sampling actually used by the profile;
- steady, spike, ramp, saturation, fault, recovery or endurance phase;
- target topology/resources and load-generator separation; and
- required correctness, safety, backlog and recovery invariants.

### 4.2 Measurement Validity Gates

A run is invalid—not a server pass or fail—when the load generator saturates unexpectedly, clocks are unsuitable for the claimed one-way metric, setup/reset or fixture digests differ, target/config/resource facts are absent, correctness checks are insufficient, collection loses material samples, external interference exceeds the declared policy or the run aborts before its minimum measurement window. Invalid runs are retained diagnostically but cannot update baselines. **[P]**

### 4.3 Functional Scope Classification

| Area | First implementation | Expanded/reference |
|---|---|---|
| landing/conformance/OAS/docs | smoke and document-size/cold-warm baseline | proxy/cache/compression and burst behavior |
| item/list/query/pagination | primary API baseline/load | capacity, combined filters and adversarial selectivity |
| spatial/time-series/latest | real PostgreSQL/PostGIS baseline | large manual grids, plan/capacity comparisons |
| ingestion/validation/trust | single/batch steady and backpressure | high rate, mixed invalid/replay/quarantine and endurance |
| outbox/SSE/replay | primary stream slice | fan-out, slow consumers, reconnect storm and optional MQTT |
| command | recorder-only stage latency smoke | simulated churn/fault; security assurance in 055 |
| policy/auth | representative visible/denied/hidden overhead | adversarial/rate-limit work in 055 |
| DDIL/sync | deterministic recovery/backlog smoke after feature exists | two-node capacity/endurance and field-informed future tier |

## 5. Performance Test Taxonomy

| Type | Question answered | Load/data shape | Gate/cadence |
|---|---|---|---|
| pure microbenchmark | did a hot deterministic function materially regress? | in-memory fixed vectors/throughput units | dedicated nightly; stabilized cases may block |
| component benchmark | what is codec/validator/filter/repository operation cost? | bounded component plus real dependency where authoritative | nightly/advisory then reference gate |
| single-user baseline | what latency decomposition occurs without contention? | one operation, cold and warm | PR smoke/nightly evidence |
| steady load | can the declared envelope sustain offered work correctly? | open arrival rate plus fixed stream population | nightly/demo/RC gate |
| spike | does a short burst remain bounded and drain? | step/ramp 2× or declared multiplier | nightly/manual |
| stress/saturation | where is the throughput/latency/error/backlog knee? | staged increasing offered load with stop conditions | isolated manual; characterization |
| scalability | how do resources/replicas/DB size change capacity? | same workload over one changed factor | manual/reference; no causal claim if multiple factors change |
| endurance/soak | do memory, handles, connections, queues or latency drift? | hours of representative steady cycles | weekly/RC |
| query-plan regression | did plan/index/buffer/temp behavior materially change? | fixed statistics/data/query parameters | PR invariant/nightly measured |
| ingestion/backpressure | where does accepted goodput diverge from offered records? | valid/invalid/replay/batch mixes | PR smoke/nightly/manual |
| stream/fan-out | how do delivery latency, goodput and resources scale by subscribers? | fixed event rate, subscribers/filters/payload | nightly/manual |
| replay/reconnect | how quickly/correctly does a named backlog recover? | sequence checkpoint, disconnection and catch-up | nightly/RC |
| fault/degraded recovery | does a dependency fault remain bounded and recover? | controlled DB/broker/network/worker fault | manual/RC; DDIL profiles |
| command-simulation latency | which safe lifecycle stage dominates? | effect recorder only | nightly/manual |

Performance baselines measure; load holds an intended envelope; stress increases beyond it to locate a boundary; soak holds long enough to expose cumulative failure; scalability varies a controlled resource; resilience injects a named fault. The labels are not interchangeable. **[P]**

### 5.1 Required Performance Strategy Matrix

| Workload/test ID | Functional area | Test type | Deployment profile | Data set/fixture | Tooling | Metrics | Threshold/gate tier | CI tier | Observability signals | Evidence artifact | Downstream topic handoff | Notes / unresolved issues |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `PERF-MICRO-001` | codec/canonicalization/validation | microbenchmark | dedicated native | micro vectors/`FX-SWE-*` | Criterion | time, bytes/elements/s, allocation diagnostic | >10% practical + statistical confirmed regression after stabilization | nightly | optional profiler; no service metrics | Criterion raw/report + host manifest | 057/implementation | shared PR timing nonblocking |
| `PERF-HTTP-001` | landing/conformance/OAS | baseline/burst | `ci-core`/reference | `FX-META-0001` | k6 | TTFB/total/bytes/goodput/error | PR correctness/sanity; nightly trend | PR/nightly | route latency/count; CPU/memory | k6 summary + metric snapshot | 056 | cold/warm and compression separate |
| `PERF-READ-001` | item/list/traversal | steady load | reference/demo | `FX-GRAPH-*`, S/M grid | k6 | p50/p95/p99, goodput, bytes, errors | demo/readiness and nightly regression | nightly/RC | HTTP/use-case/DB/pool | run bundle | 056 | open arrival model |
| `PERF-QUERY-001` | filter/sort/page | load/plan | reference | `FX-QUERY-0001`, S/M/L | k6 + PostgreSQL JSON EXPLAIN | latency, result bytes, buffers/temp/plan/pool | named query budgets + plan invariants | PR invariant/nightly/manual | query-class spans; DB stats | summary + protected plan | 055,056 | selectivity/ties/policy explicit |
| `PERF-GEO-001` | spatial | load/scalability | reference | generated spatial S/M/L | k6 + PostGIS/EXPLAIN | latency/goodput/buffers/index/temp | nightly trend; manual capacity | nightly/manual | spatial operation class, DB/I/O | run + plan/grid manifest | 056 | geometry/selectivity distribution fixed |
| `PERF-TIME-001` | observations/latest/range | load/scalability | reference | `DATASET-PERF-0001` | k6 + DB evidence | latency/rows/bytes/buffers/WAL | nightly trend; RC envelope | nightly/RC | repository/pool/cache | run bundle | 056 | native PostgreSQL baseline |
| `PERF-INGEST-001` | ingest/validation/trust | steady/batch/load | `ci-core`/reference | `SCN-INGEST-0001`, S/M | k6/publisher driver | offered/accepted/rejected records/s, end-to-end/commit latency | zero semantic loss; backlog/latency envelope | PR/nightly | ingest stages, pool, inbox/outbox | driver ledger + state proof | 055 | attempts and records denominators distinct |
| `PERF-INGEST-002` | invalid/replay/quarantine | mixed load | reference | governed invalid/duplicate mix | k6/publisher driver | classification goodput, validation CPU/latency, quarantine lag | correctness gate; resource trend | nightly/manual | reason classes, queue age | protected aggregate report | 055 | no invalid payload in public artifact |
| `PERF-SSE-001` | live events | stream/fan-out | streaming reference/demo | `SCN-STREAM-0001` | Rust SSE probe + k6 producer | commit-to-receive, subscribers, events/s, gaps/dups | demo envelope; zero prohibited loss | nightly/RC | outbox age, active streams, delivery/disconnect | sequence ledger + metrics | 055,056 | same-host monotonic preferred |
| `PERF-SSE-002` | reconnect/replay | recovery | streaming reference | named backlog/checkpoint | Rust SSE probe | reconnect, catch-up time/rate, duplicate/gap, oldest lag | correctness + bounded drain | nightly/RC | replay attempts, log/outbox, pool | checkpoint/sequence evidence | 055,056 | no sleeps as oracle |
| `PERF-SSE-003` | slow consumer | backpressure/stress | isolated streaming | controlled reader rate | Rust probe | queue/memory, disconnect reason, last cursor, server goodput | bounded memory; correct disconnect/resume | nightly/manual | slow-consumer count, queue/inflight | probe + resource series | 055 | no durable event silently discarded |
| `PERF-MQTT-001` | experimental Part 3 outbound | broker load/recovery | `streaming-mqtt-exp` | event/replay corpus | pinned xk6-mqtt + Rust oracle | publish/receive/ack, backlog, reconnect, broker resources | profile-specific/advisory then RC | manual | adapter/broker/outbox | pair evidence | 055,056 | default-off, not OGC conformance |
| `PERF-CMD-001` | command lifecycle | stage latency/churn | command simulation | `SCN-CMD-0001` | k6/driver + effect recorder | validate/auth/feasibility/commit/event/status/cancel | correctness and bounded simulated envelope | nightly/manual | stage spans, audit/outbox | transition/effect ledger | 055 | physical adapter impossible |
| `PERF-POLICY-001` | auth/policy/filter/redaction | comparative load | protected reference | `SCN-POLICY-0001` | k6 | visible goodput, decision/query latency, errors | correctness; overhead diagnostic | nightly | protected decision histograms, DB | redacted aggregate bundle | 055 | hidden count never public |
| `PERF-DDIL-001` | degraded/cache/backlog | fault/recovery | `ddil-single` | `SCN-DDIL-0001` | k6/probe + fault control | cached latency, queue age, replay goodput, recovery | correctness + drain envelope | manual/RC | health posture, queues, audit | fault timeline/run bundle | 055 | simulation, not field readiness |
| `PERF-SYNC-001` | two-node sync/conflict | backlog/scalability/recovery | `sync-two-node` | `SCN-SYNC-0001`, generated histories | sync driver | apply/dedupe/conflict/s, convergence, queue/age, resources | lineage/correctness + bounded recovery | manual/RC | per-node inbox/outbox/conflict | two-node ledger | 055,056 | no DB replication oracle |
| `PERF-SOAK-001` | representative mixed service | endurance | fixed reference | M grid + mixed workload | k6 + probes | latency/goodput/errors, memory/handles/connections/backlog slopes | no correctness failure/leak; resource bounds | weekly/RC | full bounded registry | time-series bundle/report | 057 | six/eight-hour initial RC proof |
| `PERF-STRESS-001` | system saturation | staged stress | isolated reference | M/L grid | k6 + probes/fault limits | offered/goodput, latency/error, saturation, recovery | characterize knee; safety stop/recovery gates | manual | CPU/throttle/memory/I/O/pool/queues | stage-by-stage capacity curve | 055,057 | never public endpoint |

## 6. Metrics, Thresholds, and Evidence Model

### 6.1 Work and Outcome Counters

Every run distinguishes:

- **offered:** work the schedule intended to start;
- **started:** work the generator actually began;
- **completed:** terminal responses/events observed;
- **successful/correct:** completed work satisfying the semantic oracle;
- **expected rejected:** intentionally invalid/denied work correctly classified;
- **unexpected failed/timed out:** target or protocol failure;
- **dropped:** scheduled work not started or data intentionally shed under the declared policy; and
- **goodput:** correct completed useful work per unit time.

Throughput without these denominators can make overload appear faster by dropping, rejecting or returning incorrect work. HTTP request rate, ingestion batches, ingestion records, events and subscribers use different units and are never summed. **[D,E,P]**

### 6.2 Latency Definitions

| Metric | Start | End | Notes |
|---|---|---|---|
| HTTP end-to-end | client schedule/send under declared connection model | complete validated response body | break out connect/TLS/TTFB/transfer where tool permits |
| server request | accepted route boundary | response completion/handoff | internal diagnostic; not client experience |
| ingestion accept | client send | durable accepted/rejected response | batch and per-record denominators distinct |
| source/ingest lag | semantically valid acquisition/event time | durable commit | unknown when source time is absent/untrusted |
| event commit-to-visible | authoritative commit/outbox position | adapter-visible/published position | durable pipeline metric |
| commit-to-receive | commit marker | complete subscriber message received | cross-host requires clock proof |
| replay catch-up | authorized resume accepted | subscriber reaches named watermark | includes backlog count/rate |
| command stage | stage-specific accepted input | durable stage outcome | simulated effect only |
| DDIL/sync recovery | fault cleared/reconnect barrier | backlog drained and invariant/convergence checkpoint | never infer solely from health green |

All durations use seconds in telemetry and nanosecond-capable monotonic clocks internally; presentation may use milliseconds. UTC timestamps correlate artifacts but do not replace monotonic elapsed measurement. **[A,D,P]**

### 6.3 Metric Inventory

- latency distributions: p50/p90/p95/p99, count, sum and max diagnostic;
- rates: offered, started, completed, goodput, expected rejects, unexpected failures/timeouts and drops;
- response/event bytes and compression/encoding class;
- application in-flight work, task/runtime saturation and telemetry drops;
- CPU time/utilization/throttling, RSS/working set/limit, disk/network I/O, file descriptors and connections;
- database pool total/in-use/idle/waiters/wait duration, transaction retries, locks, temp bytes, buffers, WAL and statement/plan facts;
- ingestion/outbox/stream/sync queue depth, oldest age, claim/retry/dead-letter and last-success;
- active connections/subscriptions, delivery/filter/replay/slow-consumer/disconnect counts;
- gap, duplicate, out-of-order and checkpoint/convergence correctness;
- recovery/catch-up duration and drain goodput; and
- protected policy/command/audit stage durations without sensitive labels.

Metrics follow IDR-SRV-048's bounded label registry. Raw paths, resource/source/user/command/subscriber IDs, tokens, topics, filters, policy labels and error text remain prohibited labels. **[A,P]**

### 6.4 Percentile and Histogram Rules

- Record sample count, window, success/error inclusion, aggregation method and tool.
- Do not calculate a percentile over mixed workload classes whose distributions answer different questions.
- Do not average per-instance or per-run p95/p99 values; aggregate raw observations/compatible histograms or report per-run distributions and a governed comparison statistic.
- Prefer supported native histograms; otherwise design classic buckets around decision boundaries and version them.
- Treat p99 as informational below 10,000 successful samples and max as diagnostic, not a stable objective.
- Keep failures/timeouts separate and also represent their effect on user-visible completion; dropping slow failures from latency is prohibited.
- Retain raw/granular data only within size/sensitivity policy; canonical summarized evidence includes enough buckets/counts to reproduce decisions.

### 6.5 Threshold and Gate Classes

| Class | Meaning | Typical use |
|---|---|---|
| correctness/safety | zero missing/prohibited effects, valid state and evidence | all tiers; always blocking |
| sanity/time bound | catches hang, deadlock or gross runaway; deliberately loose | PR smoke |
| regression budget | controlled comparison to accepted baseline | fixed nightly/reference runner |
| readiness objective | absolute value inside exact declared profile/envelope | demo and release candidate |
| capacity characteristic | observed saturation knee and recovery curve | manual stress/scalability; not binary production guarantee |
| informational | noisy, low-sample or early-stage measurement | development/shared CI |

### 6.6 Initial Tiered Gates

**PR smoke (`ci-core`):** after 30 seconds of readiness/warm-up, run 90 seconds of 5 HTTP arrivals/s plus 1 ingest record/s against the S grid and a short 5-subscriber SSE checkpoint where supported. Block on any incorrect response/state/event, unexpected timeout/error, dropped iteration, unsafe effect, queue/resource monotonic runaway, failure to drain backlog within 30 seconds or per-operation 2-second sanity timeout. Percentiles and CPU/memory are recorded but do not block for ordinary shared runners. **[P]**

**Dedicated nightly reference:** run at least 15 minutes after warm-up, with enough samples for the declared percentile. Compare target and accepted baseline on the same pinned runner/profile. A stabilized metric is a candidate regression when it worsens by more than 15%, exceeds its absolute envelope where defined and the confidence/noise interval excludes the configured tolerance; repeat twice and block only if two valid confirmatory runs reproduce it. Correctness/safety failures block immediately. Baseline promotion requires review; a changed runner/tool/dataset establishes a new series rather than rewriting the old. **[P]**

**Public-demo readiness:** use the exact 4-vCPU/8-GiB, dataset and 30-minute envelope stated in the executive summary. Simple-read p95 ≤500 ms; bounded query p95 ≤1 s; commit-to-SSE p95 ≤1 s and p99 ≤2 s; unexpected HTTP error/timeout ≤0.1%; zero semantically missing durable events/prohibited effects/dropped iterations; memory <80% of its cgroup limit with no sustained post-warm upward trend; two-minute 2× burst oldest backlog drains ≤30 s. TLS proxy, normal metrics, production-like logging and configured trace sampling stay enabled. **[P,X]**

**Release-candidate/endurance:** the project first calibrates a fixed reference envelope; then run one hour steady plus an initial eight-hour soak at representative mix. Require all correctness/readiness budgets, zero unexplained process restart/OOM/connection/file-descriptor exhaustion, final rolling memory median no more than 10% above the post-warm baseline and below 80% limit, pools/queues/connections return within 5% of the pre-burst steady median after drain, and no sustained latency/backlog trend. A threshold exception is a reviewed deviation, never a silent retry/pass. **[P]**

**Future operational reference:** numeric values remain unset until owners provide user/mission workload, hardware/topology, availability and risk requirements and controlled measurements validate them. Demo or RC values must not be relabeled operational. **[X]**

### 6.7 Performance Run Evidence

The canonical `PerformanceRunV1` package contains:

- run/workload IDs, source revision, server image and dependency locks;
- exact profile/config redacted fingerprint, migrations, capability and telemetry settings;
- dataset/scenario/generator IDs, parameters, seed and digests;
- target/load-generator hardware, OS/kernel, CPU/memory/cgroup, container/runtime, storage and topology facts;
- tool/extension/container digests and script digest;
- clock sources, synchronization/skew checks and warm/cold/reset state;
- phase schedule, offered model, connection/TLS/compression settings and safety stop conditions;
- correctness ledger and offered/started/completed/goodput/error/drop counts;
- raw/summarized load outputs, protected metrics, trace samples, resource series and database plan/stat evidence;
- threshold decisions with calculation/sample/confidence method;
- anomaly/invalid-run/deviation record and artifact redaction/classification; and
- links to requirement, risk, fixture, test, build and downstream evidence IDs.

Charts and Markdown are derived views. A screenshot or dashboard URL is not the canonical evidence package. **[A,P]**

## 7. Tooling Evaluation

### 7.1 External Load Tools

| Tool | Strength | Material limitation | Decision |
|---|---|---|---|
| k6 `v2.2.0` | code-reviewed scenarios, open arrival rate, checks/thresholds/tags, HTTP/WS, outputs, container distribution | JavaScript test layer; SSE/MQTT extension pin; generator resource planning | **Select primary external HTTP orchestrator** |
| Vegeta `v12.13.0` | simple constant-rate HTTP, library/CLI and concise reports | less expressive stateful scenarios, streaming and multi-phase evidence | retain optional independent HTTP spot check |
| wrk/wrk2 | high request generation; wrk2 historically addresses coordinated omission | limited semantic checks/evidence; closed-loop/script complexity; maintenance/pinning concerns | do not select baseline |
| hey `v0.1.5` | quick portable HTTP sanity | simple concurrency/request model and limited scenario/evidence | developer diagnostic only |
| Locust `2.46.5` | readable Python user behavior and distributed runners | second runtime/ecosystem; user-count model can hide offered-rate collapse; streaming extensions needed | reject first stack; reconsider for human workflow need |
| JMeter `5.6.3` | broad protocols/plugins and established UI | JVM/plugin/GUI complexity, heavier reproducibility burden | reject first stack |

k6 test scripts live with stable workload IDs, validate environment/configuration, import no unpinned remote modules and emit a resolved archive/script digest. Use open arrival-rate executors for rate claims and record preallocated/max virtual users plus dropped iterations. Closed concurrency is appropriate only when the workload question is explicitly “N simultaneous sessions/subscribers”; it must not masquerade as a fixed offered request rate. **[D,P]**

### 7.2 Streaming Drivers

The first SSE performance tool is a small Rust black-box probe independent of server production models. It opens named subscriptions, validates frames/events/cursors, records sequence/checkpoints with monotonic time, controls read rate, disconnects at exact sequence barriers and emits a machine-readable ledger. The probe and load producer may share corpus IDs/protocol schemas but not server parsers or assertion implementations. **[A,P]**

The official `xk6-sse` extension may add fan-out generation after an equivalence proof against the Rust probe; `xk6-mqtt` may drive the optional experimental MQTT profile. Each custom/auto-resolved extension is pinned by source/version and executable/container digest, included in SBOM/provenance and disabled from network-time resolution in evidence runs. k6's core WebSocket metrics are useful only if a later accepted profile selects WebSocket. **[D,P]**

### 7.3 Rust and Component Benchmarks

Criterion `0.8.2` is selected for pure, deterministic component benchmarks. Benchmarks use `std::hint::black_box`, distinguish setup from measured work, declare throughput units and verify outputs outside the timed path. Async benchmarks identify runtime flavor and avoid measuring runtime construction unless that is the subject. Allocation/profiling tools may diagnose but are not silently added to a comparable timing series. **[D,P]**

Do not use nextest wall time as an application benchmark. Test suites change, parallelism/resource contention vary and assertions/setup dominate. nextest timing is useful for CI duration, slow-test ownership and sharding only. **[A,D,P]**

### 7.4 Database Tools

- `EXPLAIN (ANALYZE, BUFFERS, WAL, SETTINGS, FORMAT JSON)` for isolated representative statements;
- `pg_stat_statements` for aggregate normalized statement count/time/rows/block/temp/WAL observations where enabled;
- PostgreSQL activity, database/table/index/WAL/background/checkpoint/lock views appropriate to pinned version;
- server/pool spans and metrics for application wait/transaction boundaries; and
- host/container I/O and memory evidence for cache/spill/resource context.

Plan assertions target durable facts: correct indexes/operators for the declared scale/selectivity, absence of catastrophic row-estimate/error or unintended temp spill, bounded buffers/rows and no regression to a materially worse plan/cost/latency. Exact node order/cost text is not a golden. Run `ANALYZE` deterministically after loading, record statistics target/extensions/settings, separate cold/warm trials, and never issue write `EXPLAIN ANALYZE` outside an isolated transaction/database. **[D,P]**

### 7.5 Observability Stack

The base runner consumes machine outputs plus the protected application scrape and cgroup/PostgreSQL statistics directly; Prometheus/Grafana/OTel Collector are optional reference-profile conveniences, not correctness dependencies. Normal operational telemetry remains enabled for readiness measurements. A telemetry-off comparison measures overhead diagnostically but cannot become the published capacity number. Trace sampling configuration is evidence; retain errors/slow exemplars where safe without exporting every high-volume span. **[A,D,P]**

## 8. Workload and Data Set Strategy

### 8.1 Generated Scale Grid

| Grid | Systems | Datastream/control streams | Observations | Use | Storage |
|---|---:|---:|---:|---|---|
| `micro` | 1–10 | 10–100 | ≤1,000 | micro/component and correctness neighbors | committed small fixture |
| `S` | 100 | 1,000 | 100,000 | PR smoke and local baseline | deterministic generation/cache |
| `M` | 10,000 | 50,000 | 10,000,000 | dedicated nightly/reference | generated/hydrated by digest |
| `L` | 100,000 | 500,000 | 100,000,000 | isolated manual capacity/stress | content-addressed generated output |

These are initial comparable grid points, not expected operational inventory or required server limit. Each recipe fixes graph fan-out/depth, field/payload sizes, time distribution, geometry distribution, source count, visibility classes and index/statistics preparation. Tests may interpolate parameters, but every result declares exact values. **[A,P,X]**

### 8.2 Query Data Distributions

- item existence: 90% hit/10% miss for browse mix; dedicated miss/denial workloads separate;
- page sizes: small default, declared medium and maximum allowed; never average across them;
- temporal selectivity: latest, narrow, medium and broad ranges with boundary/tie cases;
- spatial selectivity: point/small bbox/medium bbox/broad geometry plus boundary/antimeridian where supported;
- combined filters: explicitly named selectivity and sort/projection; and
- policy: public visible, authorized restricted, redacted and hidden sets with protected true cardinality.

Random IDs that mostly miss caches/indexes do not model browsing unless that miss rate is intended. Generator distributions and realized summary statistics are evidence. **[P]**

### 8.3 Workload Archetypes

| Workload | Mix | Purpose |
|---|---|---|
| `browse-read-v1` | 45% item, 25% list/page, 15% latest, 10% bounded temporal/spatial, 5% metadata/docs | public/demo and general API baseline |
| `query-heavy-v1` | 20% item, 25% temporal, 25% spatial, 20% combined sort/page/projection, 10% count/extent | DB/query/index pressure |
| `ingest-follow-v1` | offered records plus concurrent latest/range reads and SSE followers | end-to-end write/outbox/read visibility |
| `stream-fanout-v1` | fixed event rate across unfiltered/filtered subscribers and payload sizes | adapter/filter/fan-out cost |
| `recovery-v1` | create named backlog while consumer/dependency unavailable, then reconnect/catch up | replay/drain/recovery |
| `policy-mix-v1` | visible/denied/hidden/redacted reads at fixed proportions | safe policy overhead/side-channel preparation |
| `sync-catchup-v1` | deterministic two-node divergent histories, duplicates/conflicts and reconnect | sync apply/dedupe/conflict/convergence |

Percentages are project benchmark definitions, not claims about users. A run never changes mix dynamically without a new version/resolved manifest. **[P]**

### 8.4 Ingestion and Event Rates

The schedule declares offered batches/s, records/batch, records/s, payload bytes/s, valid/invalid/duplicate/replay/policy-blocked proportions and producer count. Accepted goodput is measured only after the declared durable boundary. Publication declares source commit rate, event family/payload, subscriber/filter counts, replay backlog and acknowledgment/checkpoint semantics. “Messages per second” without these facts is rejected. **[A,P]**

### 8.5 Determinism and Large Data

All grids/workloads use IDR-SRV-053 generator IDs, versions, seeds, logical clocks, namespaces, parameters, digests, validation and statistics. Large generated outputs are ignored or hydrated by digest; no opaque SQL dump becomes truth. Database load may use an optimized bulk mechanism only after a small semantic equivalence proof shows the same canonical state/index preparation as public/domain ingestion, and its use is recorded so ingestion cost is not accidentally included or excluded. **[A,P]**

## 9. Deployment Profile and Reset/Teardown Findings

### 9.1 Performance Profiles

| Profile | Topology | Purpose | Gate status |
|---|---|---|---|
| `bench-native` | release-mode benchmark process on fixed dedicated host | Criterion/component isolation | nightly/reference; no API capacity claim |
| `ci-perf-smoke` | accepted `ci-core`: server + fresh PostgreSQL/PostGIS, isolated project network | correctness under small offered load and mechanics | PR blocking on correctness/sanity |
| `perf-reference` | release image, PostgreSQL/PostGIS, load generator on separate pinned resources; protected direct metrics | nightly/API/DB/ingestion capacity baseline | trend then controlled gate |
| `perf-observed` | reference plus Collector/Prometheus/Grafana and normal sampling | correlate bottlenecks and telemetry overhead | diagnostic/readiness |
| `demo-perf` | accepted TLS proxy and `demo-public` shape, synthetic public data | exact public-demo readiness envelope | manual/RC blocking before publication |
| `stream-perf` | base SSE/outbox/log plus probe/producer | live, replay, fan-out, slow consumer | nightly/manual/RC |
| `mqtt-perf-exp` | stream reference plus pinned MQTT 5 broker/adapter | experimental Part 3 outbound | manual/advisory until profile release |
| `command-perf-sim` | isolated effect recorder, physical adapter absent | simulated lifecycle stages/churn | manual/security handoff |
| `ddil-perf` | self-contained node plus controlled fault boundary | degrade/backlog/recovery | manual/RC simulation |
| `sync-perf` | two isolated nodes/databases plus sync/fault controller | catch-up/conflict/convergence | manual/RC |

No public demo endpoint is a load target. The demo readiness topology is reproduced in an isolated environment or maintenance clone with explicit owner approval. **[A,P]**

### 9.2 Environment Manifest

Record CPU model/count/governor where visible, memory/swap, cgroup limits and throttling, kernel/OS, container engine/Compose, storage filesystem/device class, network layout/latency shaping, database/PostGIS/extensions/settings, pool sizes, worker/runtime settings, proxy/TLS/compression, telemetry/log level/sampling/export, load-generator resources and background/interference checks. Public cloud instance name alone is insufficient. **[P]**

### 9.3 Reset and Trial Protocol

1. resolve and verify build/config/corpus/workload/tool manifests;
2. provision an isolated project and assert no prohibited egress/effect adapter;
3. migrate and load/generate the exact grid;
4. run deterministic `ANALYZE`/preparation and capture pre-state/resource health;
5. establish cold or warm state according to workload—never ambiguously clear host caches;
6. complete readiness barriers, clock check and generator dry run;
7. warm up until defined metrics stabilize or for the declared bounded interval;
8. measure fixed phases with correctness checks and safety stops;
9. stop offered work, drain to the named checkpoint and capture post-state;
10. collect/redact/hash evidence, then destroy containers/volumes/secrets or reset from recipe; and
11. mark invalid/interfered trials without using them to update baselines.

At least one trial order is randomized or alternated when comparing builds to reduce systematic warm/cache/time bias. Comparable runs never reuse a contaminated database or generator connection pool accidentally. **[P]**

### 9.4 Safety Stop Conditions

Abort offered work while preserving diagnostics if memory reaches 90% limit, swap/OOM/throttling makes interpretation invalid, disk free space crosses the declared reserve, oldest durable backlog exceeds scenario safety bound, unexpected error/incorrectness rate exceeds 5%, load generator drops work, physical-effect/config invariant fails, or target dependency leaves its intended fault phase. Abort is not a pass; analyze recovery only when safe. **[P]**

## 10. API, Query, Geospatial, and Time-Series Performance Findings

### 10.1 Metadata and Documents

Landing, conformance and OpenAPI tests separate origin/proxy/TLS, conditional/cache, compression and cold/warm behavior. Record body bytes and validation; a cached truncated or stale document is not goodput. OpenAPI generation cost is measured separately from serving a prebuilt immutable artifact. Metadata endpoints receive PR smoke and demo burst coverage but are not the primary high-scale data capacity claim. **[A,P]**

### 10.2 Item, Collection, Navigation, and Pagination

Measure item hits/misses, list page sizes, next traversal, nested relationships, projections and response bytes. Keyset/cursor and other accepted pagination semantics are exercised with stable tie sets; the test never substitutes high offsets if the API contract does not. Each page is semantically validated, traversal checks duplicate/missing IDs and total/extent assertions follow their accepted optional semantics. Report first page and full traversal separately. **[A,P]**

### 10.3 Query Plan and Selectivity

Every named query has parameters/selectivity, expected result cardinality, index/statistics assumptions and cold/warm label. External latency, repository span and JSON plan/buffers are joined by run/query class. Plan regression checks avoid forcing an index universally: small tables and broad predicates may validly prefer sequential scans. The gate instead detects unexplained material changes at the scale/selectivity where an accepted index strategy should help. **[D,E,P]**

### 10.4 Spatial Workloads

Spatial grids record geometry types, coordinate ranges/CRS, validity, distribution/clustering, bounding-box sizes, intersection complexity and realized selectivity. Compare index-assisted bbox/narrow predicates, exact geometry refinement and broad-result transfer separately. CPU-heavy geometry processing, database buffers and response serialization are distinct stages. Antimeridian/boundary correctness cases remain small; they should not be treated as common traffic unless the workload says so. **[A,P]**

### 10.5 Time-Series and Latest Values

Test latest per stream/system, exact time boundaries, narrow/medium/broad ranges, ordering/page continuation, multi-field SWE results, late/corrected observations and large response protection. Metrics include rows examined/returned, bytes, buffers/temp/WAL where relevant, pool wait, serialization and transfer. Downsampling/aggregation is tested only if the capability exists. Native PostgreSQL is the reference; TimescaleDB or materialized acceleration requires the accepted decision gate and side-by-side semantic/performance evidence. **[A,P]**

### 10.6 Cache Comparisons

Warm/cold is a workload fact, not an optimization boast. Application cache, PostgreSQL shared buffers and OS cache are different. Required evidence says which can be controlled; host cache eviction may require privilege and can destabilize neighbors, so use isolated hosts/restarts or label the state “uncontrolled” rather than claiming cold. Cache hit/miss correctness and invalidation remain functional prerequisites. **[P]**

## 11. Ingestion, Validation, and Source-Trust Performance Findings

### 11.1 Stage Model

Measure receive/body limits, authentication/source identity, parse/decode, schema/profile validation, semantic normalization, trust/policy, idempotency/inbox, transaction/write, raw-reference/quarantine, audit/outbox and response. End-to-end and stage spans must reconcile without creating per-record telemetry explosions. **[A,P]**

### 11.2 Workload Families

- single-record latency at no contention;
- small/medium batch throughput with payload bytes and record count;
- open-rate multi-producer ingest with controlled source identities;
- validation-heavy complex SensorML/SWE payloads;
- mixed structurally invalid, semantically invalid and profile-invalid cases;
- exact duplicate, replay with same content and key collision with changed content;
- trust/policy denied, quarantine and raw-reference paths; and
- concurrent read/latest/SSE observation of committed data.

Expected rejection is successful classification, not an error-rate improvement. Denial/invalid work receives its own goodput and resource cost so cheap failure cannot mask slow valid ingest or vice versa. **[A,P]**

### 11.3 Backpressure and Rate Limits

Increase offered records until accepted goodput plateaus, queue age grows, pool waits rise or declared rate limits/load shedding activate. Verify bounded request/body/queue memory, explicit 429/503/problem behavior where designed, `Retry-After` semantics if used, idempotent safe retry, no unrecorded accepted data and complete quarantine/audit references. The generator must have spare capacity; its dropped work invalidates the target saturation conclusion. **[A,P]**

### 11.4 Durability and Visibility

An ingest acknowledgment's timing ends at the accepted durable boundary, not at later stream delivery unless the API contract says so. Separately measure commit-to-latest, commit-to-query and commit-to-event visibility. After the run, reconcile attempted/accepted/rejected/duplicate/quarantined records, canonical rows, audit and outbox facts. A high throughput number with missing or double-applied records fails correctness. **[A,P]**

## 12. Streaming, Event, Replay, and Backpressure Performance Findings

### 12.1 First Protocol and Measurement Pipeline

SSE over the durable event log/outbox is the first required live-data performance slice; it avoids making an external broker a baseline dependency. The producer records the authoritative commit/outbox position. The server exposes bounded aggregate metrics. The independent probe records complete validated events and cursors. Join these by protected evidence IDs/sequence, not high-cardinality metric labels. **[A,P]**

### 12.2 Deterministic Orchestration

- wait for explicit target and subscription-ready barriers;
- start the logical publisher schedule from an acknowledged barrier;
- generate exact event count/rate/payload/family with a monotonic scheduler;
- identify watermarks and permitted duplicate semantics;
- disconnect or change consumer rate at a named sequence, never after an arbitrary sleep;
- resume with the exact last completed cursor and reauthorize;
- finish only after the terminal watermark or declared timeout; and
- reconcile producer commits, durable log, adapter publication and subscriber ledger.

Tokio scheduling and network arrival order are observations, not the oracle. Partial order/happens-before rules from IDR-SRV-053 apply. **[A,P]**

### 12.3 Streaming Metric Definitions

Measure handshake/subscription authorization, time to first live event, commit-to-publish, publish-to-receive, commit-to-receive, delivered/filtered bytes/events, active connections/subscriptions, CPU/memory per population, outbox/log/adapter backlog count and oldest age, inflight, retries, duplicates/gaps/out-of-order, slow-consumer disconnect, last completed cursor, reconnect/resume and backlog catch-up time/rate. Keepalive frames are traffic/resource observations, not application event goodput. **[A,P]**

### 12.4 Fan-Out and Filters

Vary subscriber count, filter complexity/selectivity, representation/payload size and event rate one controlled dimension at a time before testing mixes. Identical filter sharing/caching, if implemented, is an optimization whose isolation/policy correctness must be proved. Report server serialization once/per subscriber behavior and network bytes. A filtered subscriber's hidden candidate count cannot enter its public metrics or response. **[A,P]**

### 12.5 Slow Consumer and Backpressure

The probe reads at a precise event/byte rate or pauses after a named cursor. Verify bounded connection memory/queue, correct priority/limit behavior, explicit disconnect reason, retained last completed cursor and successful resume. Durable events remain in the authoritative log; silent drop to keep latency attractive is prohibited. Other healthy subscribers' latency/goodput is measured to reveal head-of-line blocking. **[A,P]**

### 12.6 Reconnect Storm, Replay, and Broker Fault

Create a known backlog, reconnect a declared population with bounded jitter and measure authorization, connection, database/pool pressure, replay rate, live-transition watermark and time to steady state. Verify no thundering-herd bypass of admission control and no gap at snapshot/replay/live boundaries. MQTT tests separately inject broker outage/restart/session expiry/QoS duplicate behavior; broker acknowledgment is not application acceptance, and MQTT packet/session IDs are not Glaux cursors. **[A,P]**

### 12.7 Timing and Clock Safety

Same-host producer/server/probe can use a shared monotonic reference through the orchestrator. Across hosts, record NTP/PTP state and pre/post skew/RTT estimation; subtracting unsynchronized wall clocks is prohibited. If skew uncertainty is material relative to the objective, report round-trip/phase durations and sequence lag, and classify one-way commit-to-receive latency inconclusive. **[P]**

## 13. Command, Control, Security, and Policy Performance Findings

### 13.1 Command Simulation Only

Measure validation, authentication/authorization/safety, feasibility, durable submission, audit/outbox, simulated-gateway receive/response, status ingestion/publication and cancellation as distinct stages. Use `SCN-CMD-0001` and the effect recorder; the profile must fail startup if a physical adapter or resolvable real endpoint is configured. Load never changes this safety invariant. **[A,P]**

Test modest lifecycle churn, duplicate/idempotent submissions, timeout/unknown outcomes and status bursts. Correct state transitions and exactly the permitted simulated effects precede latency. IDR-SRV-055 owns adversarial authorization/safety, TOCTOU, rate-limit and gateway security assurance. **[A,X]**

### 13.2 Authentication and Policy Cost

Compare public, authenticated-visible, redacted, hidden and denied operations at identical data/selectivity where possible. Separate token/signature verification, policy decision, filtered query and response-redaction cost through protected spans. Caches are exercised for hit/miss/expiry/revocation correctness. Do not publish subject/source/policy labels, hidden candidate counts or timing distributions fine-grained enough to expose protected topology without IDR-SRV-055 review. **[A,P]**

### 13.3 Rate Limiting and Abuse Boundary

This report measures expected rate-limit/load-shed behavior in isolated profiles and ensures it protects resources without corrupting semantics. IDR-SRV-055 defines adversarial identities, distributed bypass, amplification, expensive denied-query and command-abuse tests. Performance and security share workload/metric IDs but maintain separate result types: throughput is not security assurance, and a scanner/abuse pass is not capacity evidence. **[P,X]**

### 13.4 Sensitive Diagnostics

Raw plans, traces, policy decisions, query parameters and command timelines are protected artifacts. Public reports contain workload classes and aggregates only. Normal redaction/cardinality controls stay enabled during measured runs; disabling them to improve performance invalidates readiness comparison. Test identities/credentials are injected from ephemeral providers and never written into scripts/results. **[A,P]**

## 14. DDIL, Synchronization, and Conflict Performance Findings

### 14.1 DDIL Phases

`steady connected → controlled dependency degradation/partition → bounded local work/backlog → reconnect barrier → replay/reconciliation → drained/converged steady state`

Measure local cached/read latency and correctness, stale/last-known provenance, failed-dependency response, queue count/oldest age, disk/memory growth, command-disabled posture, reconnect/replay goodput and time to named recovery checkpoint. Fault start/clear are orchestrator events; health status alone neither begins nor completes recovery. **[A,P]**

### 14.2 Synchronization Workloads

Use deterministic two-node histories with declared new/duplicate/delayed/collision/conflict/tombstone/policy/trust proportions and payload sizes. Measure exchange bytes, inbox/apply/dedupe/conflict classification goodput, transaction/pool/CPU/I/O, backlog age, retries, audit/output growth and time to convergence/unresolved-conflict checkpoint. Correct lineage and conflict preservation precede speed. Database replication metrics cannot substitute for domain synchronization. **[A,P]**

### 14.3 Fault Matrix

- dependency unreachable, high latency, limited bandwidth, loss, duplicate and reordering;
- worker pause/restart and process restart at named checkpoint;
- database pool contention/slow statement, without corrupting the database;
- full/near-full bounded queue and protected disk-reserve stop;
- expired cursor/session/policy/credential verification material; and
- simultaneous reconnect population.

Each test changes one primary fault first, declares fault-tool/version/parameters and verifies removal. Chaotic multi-fault exploration is manual and cannot create a causal performance claim. **[P]**

### 14.4 Operational Boundary

DDIL and synchronization results are lab simulations on declared topology. Field link distributions, device resources, authority policies, deployment sizes and recovery objectives are unknown. Therefore no tactical/field/operational readiness or RTO claim follows even when the lab envelope passes. **[X]**

## 15. CI, Regression, Reporting, and Artifact Findings

### 15.1 Cadence and Gates

| Tier | Maximum intended cost | Contents | Blocking semantics |
|---|---|---|---|
| developer | seconds/minutes | selected micro/single operation/small probe | informational except correctness |
| PR | about 3–5 minutes incremental | mechanics, S-grid HTTP/ingest/SSE smoke, plan invariants | correctness/safety/sanity only |
| nightly fixed reference | 15–45 minutes per partition | micro statistics, S/M API/query/ingest/stream, controlled trends | correctness immediately; stabilized confirmed regressions |
| weekly/manual | hours | L grid, saturation, spike, fan-out, faults, soak | capacity characterization; safety/recovery gates |
| demo readiness | 30+ minutes isolated clone | exact demo envelope/TLS/telemetry/burst | blocks demo publication/update |
| release candidate | one-hour steady + initial eight-hour soak and selected recovery | fixed reference envelope and complete evidence | blocks release claim for declared profile |
| future operational | stakeholder-defined | representative hardware/topology/workloads | undefined until separately approved |

Expensive jobs are partitioned by clean environment and cannot share a saturated database. Test duration is bounded by workload phase plus graceful drain; arbitrary retries are prohibited. **[P]**

### 15.2 Regression Decision Workflow

1. verify run validity and correctness;
2. compare the same workload/environment/baseline series;
3. examine effect size, uncertainty/sample count and absolute objective together;
4. rerun target/baseline in alternated order on the same runner when regression candidate exceeds 15%;
5. correlate client metric, server stage, database/resource and telemetry evidence;
6. classify code regression, data/plan change, environment interference, tool drift or intended tradeoff;
7. fix, accept a versioned reviewed baseline/objective change or record a time-bounded deviation; and
8. never make a green retry erase failed evidence.

Baseline updates require reason, affected workloads/metrics, before/after evidence and reviewer. A faster but incorrect or less secure implementation cannot be promoted. **[A,P]**

### 15.3 Artifact Set and Retention

Store the compact signed/hash-manifested `PerformanceRunV1`, k6/driver summaries, correctness sequence/state ledger, Criterion machine output, protected JSON plans/stat snapshots, bounded Prometheus/OpenMetrics samples, selected trace exemplars, cgroup/host series, sanitized logs and generated Markdown/graphs. Large granular series use controlled content-addressed artifact storage and retention class; Git stores workload source and compact accepted baselines, not raw multi-gigabyte runs. **[A,P]**

Retention preserves accepted baseline/RC evidence and failing/anomalous runs long enough for comparison; exact periods remain operations policy. Public artifacts omit endpoints/topology/secrets/source/identity/policy/command details and hidden counts. Evidence manifest records redaction and any missing signal; missing data cannot become a pass. **[A,P]**

### 15.4 Observability Validation and Perturbation

Before gating, prove required counters/histograms/gauges reconcile with driver facts within declared scrape/export semantics, labels remain bounded and collection does not drop unnoticed. Measure normal telemetry overhead against a diagnostic reduced-telemetry run on the same environment, but keep normal settings for official evidence. Stress telemetry cardinality/export queues separately; do not count audit as telemetry or infer application completion from a scrape alone. **[A,P]**

### 15.5 Implementation Proofs Required

1. k6 open-arrival HTTP smoke with offered/started/completed/goodput/drop reconciliation;
2. fixed-host Criterion comparison demonstrating practical-plus-statistical gate and shared-CI noise boundary;
3. `PerformanceRunV1` environment/workload/corpus/tool manifest and canonical digest;
4. external HTTP, internal route/use-case/DB span and cgroup metric correlation without high-cardinality labels;
5. PostgreSQL JSON plan/buffer invariant across S/M grid with safe write-plan handling;
6. deterministic ingestion offered/accepted/rejected/duplicate/quarantine/outbox reconciliation;
7. SSE commit/sequence/receipt timing and zero-gap replay through a named disconnect checkpoint;
8. slow-consumer bounded-memory disconnect/resume while healthy subscribers remain within envelope;
9. load-generator saturation/dropped-iteration test proving an invalid run is rejected;
10. public-demo readiness clone exercising TLS, normal telemetry, workload and 2× burst/drain budgets;
11. command simulation proving effect-recorder-only behavior under churn; and
12. DDIL or two-node backlog recovery proving phase barriers, drain/convergence and correct evidence classification.

These are implementation entry gates, not claims that this research executed the benchmarks. **[X]**

## 16. Downstream Topic Handoff Matrix

| Downstream owner | Inputs fixed here | Decisions retained downstream | Non-substitution rule |
|---|---|---|---|
| IDR-SRV-055 security/authorization/command-control tests | policy/auth/denial workload classes; protected metrics; rate-limit/load-shed mechanics; command stage timings; effect-recorder-only profile; performance artifact sensitivity | adversarial threat cases, authorization matrix, timing/size side-channel depth, scanner/abuse tooling, distributed bypass, command safety assurance and security gates | low overhead or successful denial under load is not security assurance |
| IDR-SRV-056 external-client interoperability | demo/reference envelope; per-client measurement boundaries; public canonical corpus; no public stress; streaming/replay metrics and named-pair evidence fields | client/server/version pairs, workflows, client timeout/retry/page/stream behavior and pair-specific expectations | one client result is not conformance or universal capacity |
| IDR-SRV-057 final synthesis | selected tools, workload grids, metrics, tiered thresholds, profiles, matrix, proofs and explicit operational boundary | reconcile all accepted design/verification work into implementation roadmap and claim language | strategy/provisional budgets are not executed readiness evidence |
| implementation roadmap | first stack, proof order, dataset grid, workload manifests and gate progression | schedule, owners, actual dedicated hardware and automation implementation | do not activate regression gates before stable baseline proof |
| operations/deployment | environment manifest and future operational-tier inputs | production topology, workloads, SLO/SLA, alerting, retention, capacity planning and scaling | demo/reference results do not size production |

### 16.1 Required Governance Handoff

The next two governance actions are plan-owner acceptance of this report and authorization of exactly one next eligible topic, `IDR-SRV-055: Security, Authorization, and Command-Control Test Strategy`. The formal combined response is:

`accept IDR-SRV-054 and authorize IDR-SRV-055`

Under the established conversational shorthand, a subsequent bare `proceed` may express that combined action. Until then, this report remains in review and IDR-SRV-055 remains unauthorized. **[X]**

## 17. Recommendations

1. Adopt correctness-gated, envelope-bound performance evidence; reject any result missing workload, data, environment, tool, clock or semantic facts. **Priority: critical.**
2. Select pinned k6 as the first HTTP load orchestrator and use open arrival-rate executors for offered-rate claims; treat dropped iterations as invalidating evidence unless intentionally part of the workload. **Priority: high.**
3. Build an independent Rust SSE performance/correctness probe before scaling with `xk6-sse`; use sequence/checkpoint barriers and a complete ledger. **Priority: critical.**
4. Select Criterion for deterministic component benchmarks on a stable runner; shared cloud timing stays informational, and blocking requires practical effect plus statistical/noise and confirmatory evidence. **Priority: high.**
5. Join external observations with accepted route/use-case/database metrics, protected traces, PostgreSQL plans/statistics and cgroup resources; no one signal is the performance truth. **Priority: high.**
6. Standardize offered/started/completed/goodput/error/drop denominators and latency start/end points before writing load scripts. **Priority: critical.**
7. Use the `micro`/S/M/L generated scale grid and versioned workload archetypes; label all quantities project benchmark definitions rather than operational predictions. **Priority: high.**
8. Keep PR performance work to short S-grid correctness/sanity smoke; place statistical regressions on fixed nightly hardware and stress/soak/fault work in isolated manual/RC lanes. **Priority: high.**
9. Adopt the provisional public-demo envelope/budgets in Section 6.6, validate them on the declared clone and revise only through reviewed evidence—not by weakening the test after failure. **Priority: high.**
10. Measure query latency with dataset selectivity, result bytes, application/pool stages and protected JSON plan/buffer evidence; avoid exact plan goldens or universal index assertions. **Priority: high.**
11. Reconcile ingestion attempts, outcomes, durable state, audit and outbox; report batch/record/byte denominators separately and measure visibility stages separately from acknowledgment. **Priority: critical.**
12. Stress streaming through fan-out, filters, slow consumers, reconnect storms and backlog recovery while proving bounded memory, permitted duplicate semantics and zero silent durable-event loss. **Priority: critical.**
13. Keep MQTT performance default-off and profile-pinned; do not select Kafka, NATS, WebSocket or TimescaleDB without measured accepted need. **Priority: high.**
14. Run command performance only against the effect recorder with physical adapter/startup impossibility, and hand adversarial assurance to IDR-SRV-055. **Priority: critical.**
15. Implement `PerformanceRunV1`, all twelve proofs and baseline-promotion review before claiming a regression gate, demo readiness or release capacity. **Priority: high.**

## 18. Risks, Constraints, and Open Questions

### 18.1 Risk Register

| Risk/constraint | Consequence | Control |
|---|---|---|
| load generator saturates | target appears capped/healthy while offered work disappears | separate generator resources, dry run, dropped-work invalidation |
| closed model/coordinated omission | response slowdown reduces offered work and hides tail | open arrival rate for rate claims; record schedule delay/drop |
| shared CI noise | false regressions/flaky gates | correctness-only PR, dedicated reference, effect+uncertainty+confirmation |
| insufficient tail samples | meaningless p99 | record count; p99 informational below 10,000 successes |
| average/averaged quantiles | tail and multi-instance error hidden | compatible histograms/raw aggregation; never average quantiles |
| workload/data drift | incomparable baseline | immutable workload/corpus/config/tool digests and series boundaries |
| cache ambiguity | misleading cold/warm result | explicit controllable state or label uncontrolled |
| exact plan golden | harmless planner change fails; bad plan may pass text check | semantic plan/buffer/latency invariants with version/stats context |
| telemetry disabled to win | unrealistic readiness and blind failures | normal telemetry gate; off comparison diagnostic only |
| stream sleeps/races | flaky replay/latency result | readiness/sequence barriers and monotonic timing |
| unsynchronized clocks | false one-way event latency | skew proof or sequence/phase duration only |
| silent event/drop/load shed | high throughput but wrong state | end-to-end ledger and goodput/correctness gates |
| stress damages public/external service | availability/security incident | isolated clone only; safety stops and owner-controlled endpoints |
| resource runaway/OOM/disk fill | lost evidence or host damage | cgroup/disk/backlog stops and graceful capture/recovery |
| policy metrics leak hidden facts | side channel in reports | protected aggregates, bounded classes and 055 review |
| command load reaches real target | unsafe effect | compile/profile adapter exclusion and recorder invariant |
| demo budget treated as production SLO | invalid operational promise | exact claim labels and future operational tier unset |
| large raw artifacts expose topology/data | disclosure/storage burden | classified redacted manifest, bounded summaries and protected retention |

### 18.2 Resolved Plan Questions

| Plan question | Resolution |
|---|---|
| primary load tool | pinned k6 for HTTP/open-arrival orchestration; independent Rust SSE probe; official extensions conditional |
| PR blockers | correctness/safety, no unexpected errors/drops, bounded queues/resources and loose timeout only—not noisy percentile regression |
| initial public demo threshold | exact provisional 4-vCPU/8-GiB/30-minute workload and budgets in Sections 1 and 6.6 |
| deterministic reconnect/replay | explicit ready/disconnect/watermark/resume barriers, sequence ledger and monotonic clocks |
| large datasets | S generated/cache, M/L generated or content-addressed; no opaque committed DB dump |

### 18.3 Open Implementation/Operational Decisions

- Select the actual dedicated benchmark host(s), storage device class and reservation policy; new hardware creates a new baseline series.
- Prove whether the selected Rust/Prometheus stack supports native histograms with acceptable interoperability; otherwise version classic buckets around decisions.
- Fix artifact backend/retention and raw-granular sampling after measuring volume and sensitivity.
- Calibrate the 15% regression candidate and demo budgets with initial implementation evidence; acceptance here makes them provisional defaults, not immutable numbers.
- Decide if official `xk6-sse`/`xk6-mqtt` extensions add enough scalable value after pinning/equivalence/security review.
- Define operational workloads, topology and SLOs only when stakeholders and representative evidence exist.

None blocks this research strategy. **[E]**

## 19. Validation Against This Plan's Success Criteria

| Topic Plan Success Criterion | Status | Evidence |
|---|---|---|
| performance/load/stress/soak/stream scope with source anchors/prior traceability | Met | Sections 3–5 |
| taxonomy, metrics, threshold tiers, evidence artifacts and gates | Met | Sections 5–6 |
| tooling evaluated/recommended for first/full scope | Met | Section 7 |
| workloads, datasets, generation, profiles, reset/teardown | Met | Sections 8–9 |
| all named functional performance areas | Met | Sections 10–14 and matrix |
| PR/nightly/manual/RC/future operational tiers | Met | Sections 6.6 and 15.1 |
| observability integration/reporting | Met | Sections 6.3, 7.5 and 15 |
| implementation/community lessons incorporated non-normatively | Met | Sections 3.3 and 10–12 |
| recommendations decision-usable/bounded | Met | Sections 17–18 |
| downstream handoffs explicit | Met | Section 16 |
| references explicit/reproducible | Met | Section 20 |

### Report Completion Checklist

- [x] Topic ID and research-plan linkage match the overall index
- [x] All core questions are covered or explicitly retained by the owning later topic
- [x] Normative, accepted, documentation, implementation, test, analysis and recommendation evidence are distinguished
- [x] Current mutable tools identify observed versions/date; implementations must pin exact digests
- [x] Correctness, conformance, performance, security and interoperability evidence remain distinct
- [x] The required 13-column performance strategy matrix is present
- [x] Offered/started/completed/goodput/drop and latency boundaries are explicit
- [x] Workload/data/environment/clock/reset and invalid-run controls are explicit
- [x] PR, nightly, demo, manual, RC and operational threshold meanings are explicit
- [x] Streaming sequence/replay/backpressure and command safety controls are explicit
- [x] Twelve implementation proofs and later-topic handoffs are explicit
- [ ] Plan-owner acceptance and acceptance date recorded

## 20. References

### Governing and Accepted Project Sources

- Glaux Server Overall IDR Research Plan: `Docs/Research/Initial Designs/IDR/glaux-server/IDR Plans/overall-idr-research-plan.md`
- IDR-SRV-054 topic plan: `Docs/Research/Initial Designs/IDR/glaux-server/IDR Plans/idr-srv-054-performance-load-stress-and-streaming-test-strategy.md`
- Glaux Server Goal and Definition: `Docs/Plans/glaux-server/glaux-server-goal-and-definition.md`
- IDR-SRV-025 through IDR-SRV-049 accepted reports: `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`
- IDR-SRV-050 Conformance Harness Strategy: `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-050-conformance-harness-strategy-report.md`
- IDR-SRV-051 Requirement-to-Test Traceability Strategy: `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-051-requirement-to-test-traceability-strategy-report.md`
- IDR-SRV-052 Rust Test-Driven Architecture and Multi-Layer Test Strategy: `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-052-rust-test-driven-architecture-and-multi-layer-test-strategy-report.md`
- IDR-SRV-053 Test Data, Fixtures, Golden Files, and Scenario Corpus Strategy: `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-053-test-data-fixtures-golden-files-and-scenario-corpus-strategy-report.md`

### Performance and Load Tool Sources

- k6 documentation: https://grafana.com/docs/k6/latest/
- k6 `v2.2.0` release: https://github.com/grafana/k6/releases/tag/v2.2.0
- k6 scenarios: https://grafana.com/docs/k6/latest/using-k6/scenarios/
- k6 arrival-rate concepts: https://grafana.com/docs/k6/latest/using-k6/scenarios/concepts/arrival-rate-vu-allocation/
- k6 built-in metrics: https://grafana.com/docs/k6/latest/using-k6/metrics/reference/
- k6 thresholds: https://grafana.com/docs/k6/latest/using-k6/thresholds/
- k6 WebSocket documentation: https://grafana.com/docs/k6/latest/using-k6/protocols/websockets/
- k6 official extensions, including SSE/MQTT: https://grafana.com/docs/k6/latest/extensions/explore/
- Criterion.rs `0.8.2`: https://github.com/criterion-rs/criterion.rs/releases/tag/criterion-v0.8.2
- Criterion.rs documentation: https://bheisler.github.io/criterion.rs/book/
- Criterion.rs CI/virtualization caveat: https://bheisler.github.io/criterion.rs/book/faq.html
- Vegeta `v12.13.0`: https://github.com/tsenart/vegeta/releases/tag/v12.13.0
- wrk: https://github.com/wg/wrk
- hey `v0.1.5`: https://github.com/rakyll/hey/releases/tag/v0.1.5
- Locust `2.46.5`: https://github.com/locustio/locust/releases/tag/2.46.5
- Apache JMeter `5.6.3`: https://github.com/apache/jmeter/releases/tag/rel%2Fv5.6.3
- cargo-nextest: https://nexte.st/

### Database, Runtime, and Deployment Sources

- PostgreSQL 18 documentation: https://www.postgresql.org/docs/18/
- PostgreSQL `EXPLAIN`: https://www.postgresql.org/docs/18/sql-explain.html
- PostgreSQL using `EXPLAIN`: https://www.postgresql.org/docs/18/using-explain.html
- PostgreSQL cumulative statistics: https://www.postgresql.org/docs/18/monitoring-stats.html
- PostgreSQL `pg_stat_statements`: https://www.postgresql.org/docs/18/pgstatstatements.html
- PostGIS documentation: https://postgis.net/documentation/
- Docker documentation: https://docs.docker.com/
- Docker resource constraints: https://docs.docker.com/engine/containers/resource_constraints/
- Docker Compose: https://docs.docker.com/compose/
- Rust benchmarking: https://doc.rust-lang.org/cargo/commands/cargo-bench.html
- Tokio: https://tokio.rs/
- Axum: https://docs.rs/axum/
- Tower: https://docs.rs/tower/
- Hyper: https://docs.rs/hyper/
- SQLx: https://docs.rs/sqlx/

### Observability, Streaming, and Protocol Sources

- Prometheus metric types: https://prometheus.io/docs/concepts/metric_types/
- Prometheus histograms and summaries: https://prometheus.io/docs/practices/histograms/
- Prometheus instrumentation practices: https://prometheus.io/docs/practices/instrumentation/
- OpenTelemetry documentation: https://opentelemetry.io/docs/
- OpenTelemetry signals: https://opentelemetry.io/docs/concepts/signals/
- OpenTelemetry sampling: https://opentelemetry.io/docs/concepts/sampling/
- CloudEvents: https://cloudevents.io/
- MQTT 5.0: https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html
- Server-Sent Events: https://html.spec.whatwg.org/multipage/server-sent-events.html
- WebSocket RFC 6455: https://www.rfc-editor.org/rfc/rfc6455
- NATS documentation: https://docs.nats.io/
- Apache Kafka documentation: https://kafka.apache.org/documentation/

### Standards and Informative Sources

- OGC API - Connected Systems Part 1: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems Part 2: https://docs.ogc.org/is/23-002/23-002.html
- Official Connected Systems `v1.0.0`: https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- SensorML 3.0: https://docs.ogc.org/is/23-000/23-000.html
- SWE Common 3.0: https://docs.ogc.org/is/24-014/24-014.html
- HTTP Semantics, RFC 9110: https://www.rfc-editor.org/rfc/rfc9110
- OpenSensorHub: https://github.com/opensensorhub
- Connected Systems Go accepted release baseline: https://github.com/SomethingCreativeStudios/connected-systems-go/tree/244f4dd586da685d4d9b75e43f73001028b5bd0e
- pygeoapi: https://github.com/geopython/pygeoapi
- OS4CSAPI client/testing corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/

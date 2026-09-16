# Section 048: Observability, Logs, Metrics, and Health Check Strategy - Research Report

**Topic ID:** IDR-SRV-048<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-048 Observability, Logs, Metrics, and Health Check Strategy](../IDR%20Plans/idr-srv-048-observability-logs-metrics-and-health-check-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** Signal taxonomy; structured logging; metrics, labels and cardinality; traces, context and correlation; startup/liveness/readiness/dependency/degraded checks; administrative diagnostics; functional-area instrumentation; redaction and disclosure; Rust tooling; profile behavior; test evidence and downstream handoffs<br>
**Methodology Used:** Accepted-requirement extraction; current primary-source review; signal/consumer and functional-area matrixing; correlation and trust-boundary analysis; health-state and failure-mode analysis; implementation-lesson reconciliation; verification and handoff synthesis<br>
**Research Time:** Approximately 38 hours of AI-assisted execution on September 16, 2026<br>
**Accepted Runtime Baseline:** IDR-SRV-046 eleven profiles and portable deployment contract; IDR-SRV-047 versioned typed configuration, explicit profile constraints, classified redaction, effective-configuration fingerprint and restart-first behavior
**Observability Evidence Freeze:** OpenTelemetry Specification 1.61.0; OpenTelemetry Rust 0.32.0; tracing 0.1.44; tracing-subscriber 0.3.23; tracing-opentelemetry 0.33.0; tracing-appender 0.2.5; metrics 0.24.6; metrics-exporter-prometheus 0.18.3; tower-http 0.7.1; OpenTelemetry Collector 0.160.0 deployment baseline; official documentation checked September 16, 2026
**Standards Baseline:** OGC API - Connected Systems Parts 1 and 2 Version 1.0; SensorML 3.0; SWE Common 3.0; RFC 9110 and 9457; W3C Trace Context; accepted Glaux IDR-SRV-001 through IDR-SRV-047
**Document Purpose:** Define portable, safe and testable telemetry and health contracts without implementing them, selecting a production backend, setting operational SLOs, or authorizing later work
**Author:** OpenAI Codex<br>
**Date:** September 16, 2026<br>
**Last Updated:** September 16, 2026

---

## Evidence and Decision Legend

- **[N] Normative:** approved external standard.
- **[A] Accepted project baseline:** accepted Glaux report or governing decision.
- **[D] Direct documentation:** current official product/specification documentation.
- **[I] Implementation evidence:** another implementation's source, deployment or behavior; informative only.
- **[T] Test/observation evidence:** reproducible test or observation, bounded to its conditions.
- **[E] Analysis:** reasoned synthesis from identified evidence.
- **[P] Project recommendation:** proposed Glaux decision pending acceptance of this report.
- **[X] Explicit boundary:** excluded claim, product selection or later-topic responsibility.

“Healthy,” “ready,” “observable,” “durable” and “operational-reference” are bounded design terms, not SLO attainment, production approval, accreditation, complete incident detection or tactical readiness. **[X]**

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Observability Requirement Extraction Methodology
5. Observability Signal Taxonomy
6. Structured Logging Strategy Findings
7. Metrics Inventory and Label/Cardinality Findings
8. Trace/Span and Correlation Identifier Findings
9. Health, Liveness, Readiness, Dependency, and Degraded-Mode Check Findings
10. Administrative Diagnostics and Profile-Specific Observability Findings
11. API, Validation, Persistence, Ingestion, and Source-Trust Observability Findings
12. Dynamic-Data, Streaming/Event, Command/Control, Security/Policy, and Audit Observability Findings
13. DDIL, Synchronization, Migration, and Conformance Observability Findings
14. Redaction, Safe Disclosure, Debug-Mode, and Sensitive-Data Handling Findings
15. Rust Tooling and Optional Observability Stack Findings
16. Test, CI, Conformance, Performance, Security, and Interoperability Implications
17. Downstream Topic Handoff Matrix
18. Recommendations
19. Risks, Constraints, and Open Questions
20. Validation Against This Plan's Success Criteria
21. References

---

## 1. Executive Summary

Glaux should implement **one typed observability vocabulary with multiple explicitly different signals**. Structured logs diagnose discrete runtime events; metrics summarize bounded numeric behavior; traces explain causal execution; health reports whether a process or role can serve; system events record important operational transitions; domain events carry committed application facts; audit records provide authoritative accountability; administrative diagnostics expose protected current evidence; and conformance artifacts preserve test claims. Correlation connects these signals, but no signal substitutes for another. In particular, a log or sampled span never substitutes for the mandatory durable audit record or outbox/domain event. **[A/D/E/P]**

The first implementation should use Rust `tracing` spans/events with `tracing-subscriber`, custom Tower HTTP tracing, and JSON Lines on stdout/stderr in every shared environment; local development may select a human-readable formatter. The logging schema uses stable event names and typed fields, not prose parsing. It includes release/profile/configuration fingerprint, role/run, safe route template, outcome/error code, request/trace/span/workflow identifiers where applicable, and a classification/redaction marker. It excludes request/response bodies, raw URLs/query strings, credentials, headers, SensorML/SWE/Observation/Command payloads, policy labels/rules, source topology and database statements with values. **[A/D/E/P]**

Metrics should use the `metrics` facade and a single application-owned instrument registry, initially exported through a protected Prometheus/OpenMetrics endpoint using `metrics-exporter-prometheus`. Names, units, types and bounded labels are versioned as an API. Request/record/event/decision counts, error-class counts, latency histograms, in-flight work, pool/queue saturation, backlog size/age, last-success time and telemetry drops form the baseline. Resource IDs, actor/client/source/command/event/trace/request/sync IDs, raw paths, URLs, topics, policy labels, tenant names and error messages are prohibited labels. A cardinality budget and automated label-value stress test are release gates. **[D/E/P]**

OpenTelemetry is the portability model for traces and cross-signal resource/correlation fields. Glaux accepts bounded W3C `traceparent`/`tracestate`, strips unapproved baggage at trust boundaries, starts/links spans across HTTP, durable workers, publication, commands and synchronization, and can export OTLP through an optional Collector using `tracing-opentelemetry`. Export is disabled without configuration and never required for server correctness. Sampling affects diagnostic traces only: errors and selected low-volume safety workflows may receive higher sampling, but audit, security enforcement, domain state, delivery correctness and client responses are never conditional on sampling. **[D/E/P]**

Health has four separate views. `/health/live` is a constant-cost process/event-loop check with no external dependency. `/health/startup` represents completion of immutable configuration, local package and initialization gates. `/health/ready` is role- and profile-aware, using asynchronously maintained, freshness-bounded evidence for mandatory dependencies and schema compatibility; it returns `503` when normal traffic should stop but does not restart the process. Protected dependency diagnostics expose individual checks, freshness and reasons. Optional dependency loss produces the accepted degraded service posture rather than indiscriminately failing liveness or readiness. Public deployments need expose no health endpoint; if required by an external monitor, only a constant-shape shallow result is routed publicly. **[A/D/E/P]**

The server should have a private management listener or equivalent route boundary for metrics, readiness and diagnostics. Detailed dependency, source, trust, policy, command, queue, migration and synchronization state is authenticated, authorized, redacted and audited when accessed. Public CSAPI representations may communicate only already-authorized availability/freshness semantics defined by accepted API/DDIL reports. They do not reveal internal dependencies or hidden resources. Debug mode is a local/CI profile feature, not a log-level synonym, and is rejected in demo and operational-reference profiles. **[A/E/P]**

The base runtime requires no Collector, Prometheus, Grafana, Loki, Tempo or Jaeger. The optional Compose observability profile may add a pinned OpenTelemetry Collector, Prometheus and a development visualization backend. The Collector is a lossy/bounded telemetry gateway unless separately engineered; its queues and WAL do not transform telemetry into authoritative audit evidence. Backend retention, alert routing, dashboards, production topology, SLOs and incident procedures remain deployment/operations decisions. **[D/E/P/X]**

Acceptance of this report authorizes only IDR-SRV-049. It does not implement instrumentation, select production vendors, publish operational status, authorize physical commands or inbound Part 3, establish numeric SLOs/alerts/retention, or claim conformance/accreditation. **[P/X]**

---

## 2. Scope and Plan Alignment

### 2.1 Coverage

| Plan area | Coverage | Evidence location |
|---|---|---|
| signal taxonomy and prior traceability | Complete | Sections 3–5 |
| log, metric, trace and correlation model | Complete | Sections 6–8 |
| health/readiness/dependency/degraded checks | Complete | Section 9 |
| diagnostics and eleven profiles | Complete | Section 10 |
| all functional areas | Complete | Sections 11–13 |
| redaction/cardinality/debug/disclosure | Complete | Sections 7 and 14 |
| Rust and optional stack evaluation | Complete | Section 15 |
| test and downstream evidence | Complete | Sections 16–17 |

### 2.2 In scope

- application-emitted logs, metrics, traces and operational transitions;
- correlation across requests, transactions, durable work and external adapters;
- startup, liveness, readiness, dependency and degraded-state assessment;
- protected diagnostics and profile-specific exposure;
- signal schemas, sensitivity, redaction, cardinality and overload behavior;
- first-slice Rust instrumentation/export candidates; and
- verification obligations and later-topic contracts.

### 2.3 Out of scope

- production backend, SIEM, APM, pager, dashboard or observability-vendor selection;
- numeric availability/latency/error SLOs, alert thresholds, capacity or retention periods;
- enterprise monitoring topology, HA Collector design or organizational incident response;
- authoritative audit retention/custody already owned by IDR-SRV-041 and later lifecycle work;
- client analytics, user tracking, payload inspection or a public operational-status service;
- implementation, infrastructure provisioning, real command dispatch or new standards claims. **[X]**

### 2.4 Accepted invariants

Observability must preserve typed problem details, public-origin correctness, policy-authorized representations, source trust, validation evidence, one PostgreSQL/PostGIS write core, atomic outbox/audit references, durable streaming/replay, deny-by-default commands, E0–E5 audit behavior, orthogonal DDIL state, application synchronization, modular ports/adapters, eleven deployment profiles, explicit configuration fingerprints and secret/sensitive classification. **[A]**

---

## 3. Evidence Base and Authority Classification

### 3.1 Current primary evidence

| Source | State checked | Evidence used | Limitation |
|---|---|---|---|
| OpenTelemetry specification | 1.61.0; 2026-09-16 | signal/data/resource/context models, log correlation, OTLP, schema versioning | evolving semantic conventions require a pinned compatibility policy |
| W3C Trace Context | Recommendation; checked 2026-09-16 | `traceparent`/`tracestate` propagation and privacy/trust concerns | trace context is diagnostic, not authenticated identity |
| Prometheus practices | current 2026-09-16 | names, base units, online/offline work, error/latency/queue metrics, cardinality | backend-oriented guidance, not Glaux semantics |
| Kubernetes probes | current 2026-09-16 | distinct startup/liveness/readiness actions | comparison only; Compose/VM mappings differ |
| OWASP Logging/Error Handling | current 2026-09-16 | separate audit/security purposes, sanitization, exclusions and generic public errors | guidance, not accreditation or retention policy |
| PostgreSQL monitoring | current 2026-09-16 | activity/statistics sources and collection overhead | DB-wide visibility requires separate privilege/topology decisions |
| Rust crates | versions in report metadata | structured spans/events, layers, HTTP middleware, metrics facade/export and OTLP bridge | candidate versions, not implementation pins |
| OTel Collector docs | 0.160.0 deployment baseline/current docs | agent/gateway patterns, queues/retry/WAL and loss modes | not an authoritative event/audit store |

OpenTelemetry semantic conventions currently contain mixed stability in areas such as HTTP. Glaux must pin the emitted convention/schema version, test field/name changes and translate deliberately rather than inherit silent dependency upgrades. **[D/E/P]**

### 3.2 Accepted project evidence

IDR-SRV-001 through 024 supply API, link, error, media and standards-visible behavior. IDR-SRV-025 through 038 supply persistence, validation, ingestion, dynamic-data, publication and command workflows. IDR-SRV-039 through 043 supply security, policy, audit, DDIL and synchronization distinctions. IDR-SRV-044 through 047 supply Rust boundaries, deployment profiles, configuration identity, redaction and secret safety. Accepted project reports control Glaux meaning; telemetry products provide mechanisms. **[A/D]**

### 3.3 Non-normative implementation/community lessons

| Lesson | Evidence class | Glaux consequence |
|---|---|---|
| proxy/base-path and content-negotiation failures are hard to diagnose externally | OSH, SECD, client studies | record route template, negotiated representation and safe outcome; preserve external test evidence |
| mutable demos fail or drift without explaining dependency state | implementation/community studies | deterministic local gate plus bounded protected dependency diagnostics |
| OpenAPI/routes/conformance declarations drift | multiple server studies | emit capability/config fingerprint in test artifacts; instrument route/capability mismatch as startup failure |
| raw server errors and broad debug output disclose internals | server studies/OWASP | stable error code and correlation ID publicly; protected details internally |
| streaming/source workflows need backlog/lag, not only HTTP access logs | OSH/CS-Go lessons | queue age, delivery latency, last success and bounded failure classes are first-class instruments |
| another implementation's endpoint or log format is not normative | all studies | adopt failure classes, not product-specific schema or claims |

### 3.4 Evidence limits

- No Glaux workload exists from which to set histogram buckets, sampling rates, alert thresholds or overhead budgets.
- No Rust integration prototype proves cross-crate context propagation or graceful exporter shutdown.
- No production data classification, SIEM, backend or incident process is selected.
- Crate and semantic-convention versions are a dated evidence freeze and require implementation review.
- A health response proves only the evaluated checks at their recorded times; it is not a future-availability guarantee.

---

## 4. Observability Requirement Extraction Methodology

The research used seven passes:

1. identify every accepted workflow, invariant, effect boundary and degraded state;
2. identify the consumer and decision enabled by each proposed signal;
3. select the least revealing signal that answers that decision;
4. bind each emission to an operation/effect boundary and correlation context;
5. classify fields, cardinality, sampling, loss and access behavior;
6. map profile, startup, steady-state, failure, recovery and shutdown behavior; and
7. convert unknown numeric values into measured proof obligations rather than invented defaults.

### 4.1 Admission test for an instrument

An instrument is admitted only when it has a named question, owner/consumer, stable emission point, bounded dimensions, sensitivity class, expected lifecycle and test. “Useful someday” is insufficient. Duplicate instruments measuring the same fact at multiple layers are rejected unless they intentionally represent client/server or attempt/outcome views. **[E/P]**

### 4.2 Design criteria

| Criterion | Test |
|---|---|
| diagnostic value | can an operator/test identify which bounded stage failed? |
| semantic integrity | is the signal distinct from domain/audit truth and emitted at the correct boundary? |
| safe disclosure | can the intended audience learn it without inferring protected resources/actors/topology? |
| correlation | can it be joined without embedding payloads or unbounded IDs in metrics? |
| bounded cost | are event rate, attribute size, label space, buffering and export failure controlled? |
| portability | can stdout, Prometheus/OpenMetrics and OTLP consumers use it without backend-specific code? |
| testability | can schema/emission/redaction be asserted without comparing prose logs? |
| degradation | does telemetry failure leave correctness intact and report its own loss? |

---

## 5. Observability Signal Taxonomy

| Signal | Authoritative purpose | Durability/loss | Primary consumer | Must not become |
|---|---|---|---|---|
| application log event | diagnose a discrete runtime fact/error | bounded best effort; loss counted | developer/operator | audit record or domain truth |
| HTTP access event | one safe request outcome | bounded best effort | operator/test | request/response payload archive |
| metric | aggregate behavior, saturation, latency and state | cumulative/periodic; scrape/export gaps possible | monitor/performance test | per-resource ledger or audit trail |
| trace/span | causal/timing path through work | sampleable diagnostic | developer/operator/test | workflow state machine or proof of effect |
| health result | current bounded process/role eligibility | evaluated state with freshness | orchestrator/operator | deep topology disclosure or availability promise |
| system transition | notable runtime/config/dependency/posture change | structured log plus metric; audit when security-relevant | operator/security | mutable domain event |
| domain event/outbox item | committed business/resource fact | durable per accepted publication contract | application/subscriber | telemetry event |
| audit record | authoritative accountability evidence | mandatory durable E0–E5 semantics | auditor/security | debug/access log |
| admin diagnostic | protected current snapshot/explanation | on-demand, not historical truth | authorized operator | public API or secret dump |
| conformance evidence | reproducible test input/result/manifest | versioned test artifact | reviewer/harness | production monitoring stream |

### 5.1 Correlation without collapse

Signals share typed references such as `request_id`, `trace_id`, `workflow_id`, `audit_event_id` or `domain_event_id` only where classification and audience permit. They do not share the entire object. Audit stores its own correlation fields independent of trace sampling/export. Metrics use bounded operation/outcome classes and exemplars where supported, not identifier labels. **[A/D/E/P]**

### 5.2 Client-visible versus internal evidence

CSAPI resources, RFC 9457 problems, response headers and accepted DDIL representation assessments are API outputs—not observability endpoints. They may expose a safe request/correlation identifier and already-authorized freshness/availability semantics. Logs, traces, metrics and dependency details are not CSAPI resources and remain private. **[A/N/E/P]**

---

## 6. Structured Logging Strategy Findings

### 6.1 Format and schema

Shared environments emit one JSON object per line to stdout/stderr. Local development may render the same typed event through compact text; CI always tests JSON. Required common fields are:

- `timestamp`, `severity`, `event.name`, `event.version`, `message` (bounded static template) and `target`;
- `service.name`, `service.version`, `service.instance.id`, `service.role`, `deployment.profile` and `config.fingerprint`;
- `request.id`, `trace_id`, `span_id`, `workflow.id`, `correlation.id` and `causation.id` when safe/applicable;
- `operation`, normalized `http.route`, HTTP method/status class, duration and outcome;
- stable `error.code`/`reason.class`, never raw downstream error text by default; and
- `data.classification`, `redaction.applied` and `telemetry.schema`.

Missing optional context is omitted rather than emitted as misleading empty strings. Timestamps are UTC with wall-clock value plus monotonic duration measurement. Event names are dotted stable identifiers such as `glaux.http.request.completed` and `glaux.ingest.batch.rejected`; prose is not an API. **[D/E/P]**

### 6.2 Levels

| Level | Use | Examples | Rule |
|---|---|---|---|
| ERROR | current operation or mandatory component failed and needs action | transaction/audit write failure, exporter initialization fatal to selected profile | one event at ownership boundary; no repeated stack at every layer |
| WARN | degraded/retried/rejected condition worth attention | optional dependency unavailable, rate limit, invalid untrusted input aggregate | rate-limit repetitive events; stable reason class |
| INFO | lifecycle or material state transition | startup identity, first ready, readiness/posture change, shutdown summary | no per-record happy-path chatter |
| DEBUG | developer diagnostic for bounded operation | validation stage timing, adapter response class | local/CI only by default; same redaction rules |
| TRACE | very high-volume internal flow | per-stage detail | compiled/filtered and prohibited in demo/operational-reference without a reviewed temporary control |

Security/audit-required events cannot be disabled by changing the diagnostic log filter. Temporary filter changes are authenticated, audited, time-bounded and cannot enable payload/header/body fields that do not exist in the schema. **[A/D/E/P]**

### 6.3 Access and error events

HTTP access events record the route template after routing, not raw path or query; method; protocol; safe client-network class where approved; status code/class; response media-type family; bytes bucket if useful; duration; request/trace IDs; and stable problem type/code. Unmatched paths record a constant template such as `__unmatched__`. Authorization concealment produces only the externally appropriate outcome; internal policy/audit signals carry protected reason evidence. Panic/unknown failure returns a generic RFC 9457 response with request ID while protected logs record a sanitized error chain and source location only under policy. **[A/D/E/P]**

### 6.4 Backpressure and loss

Logging uses a bounded non-blocking writer for shared profiles. The queue policy must be explicit: low-severity diagnostic events may drop under overload; WARN/ERROR get a reserved or synchronous bounded path; mandatory audit uses its separate durable mechanism. Dropped log counts and queue saturation are metrics, with a rate-limited stderr fallback. Shutdown flush has a bounded deadline and records whether diagnostic telemetry was abandoned. `tracing-appender` is a candidate mechanism, but its loss counter, guard lifetime and shutdown behavior require proof. **[D/E/P]**

---

## 7. Metrics Inventory and Label/Cardinality Findings

### 7.1 Naming and types

Glaux-owned names use a `glaux_` Prometheus namespace after export, base units (`seconds`, `bytes`, ratios), `_total` for counters and explicit descriptions. Code owns logical instrument names/units and one registry; exporter translation is tested. Counters describe attempts/outcomes, gauges/up-down counters describe current bounded state, and histograms describe distributions. Summaries are avoided in the application baseline because server-side quantiles do not aggregate across instances; dashboards calculate quantiles from histograms. **[D/E/P]**

### 7.2 Baseline inventory

| Family | Instruments | Bounded dimensions | Purpose |
|---|---|---|---|
| HTTP | request count, duration, active requests, request/response bytes | route template, method, status class, capability | traffic, errors and latency |
| validation | attempts/failures/duration | stage, profile family, reason class | standards/semantic failure location |
| database | operation duration/failures, pool total/idle/in-use/waiters, transaction retries | operation class, outcome | latency and saturation without SQL values |
| ingestion | messages/batches/records received/accepted/rejected/duplicate/quarantined, duration, source-lag histogram, last success | adapter class, reason class, trust-state class | throughput, quality and stalls |
| dynamic data | observation/status writes and query duration | operation class, result class | data path behavior |
| outbox/events | backlog, oldest age, claimed/published/retried/failed, publication duration | transport class, event family, outcome | continuity and latency |
| streaming | active connections/subscriptions, delivered/filtered, replay attempts/failures, slow-consumer disconnects | protocol, reason class | load and backpressure |
| commands | submissions, feasibility/authorization/safety outcomes, transitions, dispatch attempts/outcomes/duration, unresolved count | simulated adapter, transition/outcome/reason class | safety workflow health without target/payload |
| security/policy | authn/authz/policy/rate-limit outcomes and decision duration | mechanism, operation class, outcome class | detection and evaluator behavior |
| audit | append attempts/failures/duration, spool backlog/oldest age, checkpoint status | event family, outcome | evidence-path health; not audit content |
| DDIL/sync | posture transitions/current state, dependency class availability, sync backlog/oldest age, gaps/conflicts/quarantine | state/dependency/conflict class | degraded/recovery behavior |
| process/runtime | build info, start time, CPU, memory, task/runtime saturation where proven | role/version/profile class | resource and runtime diagnosis |
| telemetry | log drops, span/export drops/failures, metric scrape/export failures, queue size/age | signal/exporter class, reason class | self-observability |

Exact histogram buckets and alert thresholds are measured in IDR-SRV-054/operations. The release still supplies bounded initial buckets and tests that observed values are not collapsed into a terminal overflow bucket. **[E/P]**

### 7.3 Cardinality and disclosure policy

Allowed labels come from closed enums or release-bounded registries: operation, route template, method, status class, result/reason class, adapter/transport class, role and coarse profile/capability. Prohibited labels include resource/FOI/system/datastream/observation/source/publisher/actor/client/tenant/command/event/request/trace/span/batch/sync/conflict IDs; raw status text; URL/path/query; IP; topic; issuer; database statement/table; policy/marking; filename; and exception message. **[D/E/P]**

The registry declares a maximum value set per label and a product budget for total active series. CI generates adversarial unique IDs/paths/errors and proves series count remains bounded. Metrics that could reveal protected population or denial rates are admin-only and may need aggregation, access audit or suppression under policy. Hashing an unbounded identifier does not fix cardinality or disclosure. **[E/P]**

### 7.4 Exposure and collection

The first slice exposes Prometheus/OpenMetrics text only on the private management listener with network/auth protection appropriate to the profile. Scrapes are time/size bounded and report collection errors. A Collector may scrape and translate/export. Public demo dashboards consume an explicitly curated aggregate set; the raw endpoint is not routed publicly. Metrics absence/export loss cannot affect authorization or domain behavior. **[D/E/P]**

---

## 8. Trace/Span and Correlation Identifier Findings

### 8.1 Context model

| Identifier | Scope | Origin/propagation | Exposure |
|---|---|---|---|
| `request_id` | one inbound HTTP attempt | server-generated; validated trusted caller value may be separately recorded | safe opaque value may be response header/problem field |
| `trace_id`/`span_id` | diagnostic causal graph | W3C trace context plus server spans | protected logs/traces; response exposure only by explicit policy |
| `correlation_id` | multi-request/business workflow | application-generated or accepted only from trusted integration | problem/audit/domain evidence as authorized |
| `causation_id` | immediate predecessor fact/work item | durable envelope/domain workflow | internal/audit; not a metric label |
| `workflow_id` | ingestion batch, command, sync or maintenance execution | application workflow | protected; client sees domain-specific ID only if API defines it |
| domain/audit IDs | authoritative entity/evidence | domain/audit stores | governed by resource authorization, not telemetry convenience |

`traceparent` and `tracestate` are validated and size-bounded but not treated as identity or authorization evidence. Unknown/untrusted baggage is discarded at the public boundary; an allowlist may propagate coarse non-sensitive values between trusted Glaux components. PII, policy labels, credentials and resource IDs never enter baggage or `tracestate`. **[D/E/P]**

### 8.2 Span model

Create spans at meaningful latency/effect boundaries:

- HTTP server/client operation using normalized route/peer service;
- authentication, authorization and policy evaluation with outcome class only;
- parse/structural/profile/semantic/stateful validation stages;
- application use case and transaction, with DB operation spans emitted by adapter instrumentation;
- ingestion receive/batch/normalize/commit;
- outbox claim/publication and replay;
- stream handshake and bounded session summary, not an unbounded event-body span;
- command submit/feasibility/approve/transition/dispatch/reconcile;
- audit append/checkpoint mechanics without audit content;
- dependency probe, migration stage and administrative operation; and
- synchronization session/exchange/classify/apply/conflict stage.

Span names are low-cardinality operations, not paths, IDs or statements. Attributes follow a pinned OpenTelemetry semantic-convention version where stable and a versioned `glaux.*` schema otherwise. SQL statement text, parameters and payloads are disabled. **[D/E/P]**

### 8.3 Async and durable boundaries

Within a process, instrumented futures/tasks carry the current context explicitly; detached task spawning without context is lint/review-sensitive. A durable outbox/work item records safe trace correlation plus application correlation/causation IDs. A later worker starts a new consumer span **linked** to the producer context rather than holding an hours-long parent span. Redelivery creates a new attempt span linked to the same durable work identity. Context failure never drops or duplicates domain work. **[D/E/P]**

### 8.4 Sampling

Local/CI/conformance may record all bounded test traces. Shared profiles use deterministic trace-ID ratio head sampling as the simple baseline; an optional Collector can tail-sample errors/latency under a measured topology. Low-volume command-simulation, migration and administrative traces may be retained at higher rates, but payloads remain absent. Sampling settings and effective rate are configuration evidence. Metrics, audit, security decisions and error responses are independent. The server records exporter/sampler drops where the SDK exposes them. **[D/E/P]**

---

## 9. Health, Liveness, Readiness, Dependency, and Degraded-Mode Check Findings

### 9.1 Endpoint/state contract

| Check | Question | Inputs | HTTP/action | Exposure |
|---|---|---|---|---|
| `/health/live` | can this process/event loop answer? | local constant-time sentinel only | 200; failure permits restart | private management; constant public proxy only if required |
| `/health/startup` | did immutable startup finish? | config/secrets/packages/schema compatibility/role initialization state | 200 complete, 503 pending; terminal invalid config exits | orchestrator/internal |
| `/health/ready` | may this role receive normal traffic now? | cached fresh mandatory-gate assessments and drain state | 200 ready, 503 not ready; no restart | orchestrator/private management |
| dependency diagnostic | what is each dependency's state and evidence age? | asynchronous probes, last success/failure, requirement class | 200 diagnostic envelope even if components fail; authorization governs | admin only |
| service posture | what operation is allowed/degraded under accepted DDIL semantics? | domain/config/dependency/authority evidence | authorized application assessment | authorized clients/admin, not raw health |

Response bodies use `HealthSummaryV1`: schema version, `status` (`live`, `starting`, `ready`, `not_ready`, `degraded`, `draining` as applicable), role, release/config fingerprint optionally by audience, observation time, and a coarse reason code. No hostname, DSN, source/peer ID, schema/table, policy bundle path, command target, queue content or error string appears publicly. `Cache-Control: no-store` applies. **[E/P]**

### 9.2 Readiness gates by role

API readiness requires validated effective configuration; compatible DB/migration state; required standards/profile packages; listener/router/capability consistency; and any profile-mandatory local security/policy/audit components. Worker readiness requires DB, migration compatibility, durable work access and its enabled adapter prerequisites. Admin/migration jobs have command-specific success/exit contracts rather than pretending to be serving processes. **[A/E/P]**

An optional broker, telemetry backend, remote schema source, IdP refresh endpoint, policy service or sync peer does not automatically make every role unready. The accepted profile and cached authority determine whether loss blocks new work, permits bounded local reads, or creates degraded posture. A process in graceful shutdown becomes not ready before draining; liveness continues until termination. **[A/E/P]**

### 9.3 Probe implementation

Dependency probes run asynchronously with bounded timeout, jitter and concurrency, store last-success/failure and observation timestamp, and feed a health registry. Endpoint requests read that registry; they do not fan out to every dependency. Each assessment has `required|optional|conditional`, `healthy|degraded|unavailable|unknown`, freshness/expiry and stable reason class. Hysteresis/failure thresholds live in deployment probe configuration, while server state transitions remain observable. **[D/E/P]**

Liveness excludes network/DB/export probes to prevent restart storms. Readiness is not a substitute for request-path timeouts/circuit behavior. Stale health evidence becomes `unknown`; it never remains green indefinitely. Probe traffic is tagged/suppressed from ordinary access metrics where necessary to avoid self-noise, while probe failures and state transitions remain counted. **[E/P]**

### 9.4 DDIL/degraded semantics

Dependency state and service posture are orthogonal. A disconnected node may be live and ready for its permitted local role while external identity/policy/schema/sync dependencies are unavailable. Readiness therefore evaluates the advertised role under the selected profile, not universal connectivity. Public clients receive accepted freshness/authority/problem semantics per operation; administrators see the dependency/posture explanation. Connectivity transition, evidence staleness and recovery create system events, metrics and audit only where security/authority decisions require it. **[A/E/P]**

---

## 10. Administrative Diagnostics and Profile-Specific Observability Findings

### 10.1 Protected diagnostics

The management surface may provide:

- redacted `EffectiveConfigV1` and fingerprint/source provenance from IDR-SRV-047;
- release, role, capabilities, standards packages and migration compatibility;
- dependency health evidence with last-observed age and coarse error class;
- schema/profile/vocabulary cache identity/freshness;
- DB pool, outbox, ingestion quarantine, command queue and audit-spool summaries;
- source-trust aggregates without raw source identity;
- DDIL posture inputs/result, sync backlog/gap/conflict aggregates; and
- telemetry pipeline queue/drop/export status.

These are bounded snapshots, not arbitrary SQL, stack dump, heap dump, environment dump, file browser or ad hoc introspection. Access requires admin authentication/authorization, is audited, rate/size limited and may be local-only. Public demo routes detailed diagnostics nowhere. **[A/E/P]**

### 10.2 Profile matrix

| Profile | Logs | Metrics | Traces | Health/diagnostics | Constraints |
|---|---|---|---|---|---|
| `dev-native` | text default; JSON selectable; DEBUG allowed | loopback scrape | console/OTLP optional, full sampling | all local protected/loopback | debug still redacts; Tokio console opt-in |
| `dev-compose` | JSON default | private scrape; optional stack | optional Collector | management network | no public raw endpoint |
| `ci-core` | JSON artifact | scrape/snapshot assertions | in-memory/test exporter, deterministic | readiness/dependency evidence captured | no external backend required |
| `conformance` | JSON stable schema | conformance-support metrics private | full bounded trace for failed cases | manifest/fingerprint plus safe health | telemetry not conformance oracle alone |
| `interop` | JSON | private | selected sampling, failure retention | external-origin and adapter diagnostics | mutable remote observations dated |
| `demo-public` | JSON INFO | private curated dashboard only | sampled/private | public constant shallow check at most; admin disabled externally | DEBUG/TRACE and bodies prohibited |
| `streaming-mqtt-exp` | JSON summaries | stream/broker/outbox families | linked publish/replay spans | broker optional/required per profile contract | no topic/resource labels |
| `command-sim` | JSON protected | aggregate command/safety metrics | high-retention simulated workflow traces | simulator/queue detail admin only | no parameters/targets/public command metrics |
| `ddil-single` | JSON local bounded sink/stdout | local scrape/snapshot | local sampling/export when connected | role-ready during permitted offline work | storage budgets and telemetry loss visible |
| `sync-two-node` | distinct node JSON | per-node private scrape | propagated/linked peer-test spans | per-node protected diagnostics | no peer/resource ID metric labels |
| `operational-reference` | JSON INFO+ with controlled temporary filter | protected scrape/approved export | sampled OTLP optional | private management; audited diagnostics | debug mode rejected; backend vendor unspecified |

### 10.3 Public status boundary

The standards landing page and conformance declaration communicate API capabilities, not internal health. If the public demo exposes a monitor target, the response is fixed-shape and no more detailed than `ok`/`unavailable`; degraded dependency and command/policy/source status remain private. Dashboards shown publicly use synthetic data and a curated metric allowlist resistant to resource-count inference. **[A/E/P]**

---

## 11. API, Validation, Persistence, Ingestion, and Source-Trust Observability Findings

### 11.1 Required observability matrix

| Signal type | Functional area | Purpose | Emission point | Fields/labels/span attributes | Correlation IDs | Sensitivity | Redaction | Profiles | Test/conformance use | Handoff | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|
| log + metric + span | HTTP/API | request outcome/latency | routed response completion | route template, method, status class, media family, duration | request/trace | internal | no raw path/query/header/body | all serve | contract/perf/interop | 050/054/056 | unmatched route constant |
| log + span | problem handling | diagnose safe client error mapping | problem construction/unknown-error boundary | stable problem/error code, status | request/trace/correlation | sensitive | no exception in response; sanitized internal chain | all | error corpus/security | 050/055/056 | RFC 9457 remains API contract |
| metric + span + validation artifact ref | validation | stage/failure/latency | each validation stage and terminal decision | stage, reason class, profile family | validation/workflow/trace | sensitive | no offending value/payload | relevant profiles | fixtures/conformance | 050/053/055 | artifact access separately governed |
| metric + span | query/persistence | latency, saturation, failure | repository/transaction adapter | operation class, outcome, pool state | trace/workflow | sensitive | no SQL/parameter/table/resource ID | DB profiles | perf/fault | 049/052/054 | DB native stats protected |
| log + metric + span | ingestion | throughput/quality/stall | receive, normalize, commit, quarantine | adapter class, counts, reason/trust class, lag | batch/workflow/trace | sensitive | source ID only in protected correlated detail; no payload | ingestion profiles | fixtures/perf/security | 053–055 | count attempts and outcomes |
| log + metric + audit ref | source trust | trust transitions/denials | trust evaluation/update boundary | trust-state/reason class, package version | workflow/audit | protected | no credential/topology/marking | write/DDIL | security/DDIL | 053/055 | decisions remain audit-authoritative |
| health + metric | DB/package dependencies | readiness and staleness | async probe/registry transition | dependency class, state, age bucket | run | sensitive | no endpoint/path/schema details | applicable | startup/fault | 049/052 | endpoint reads cached state |
| metric + span | dynamic data | write/query/latest behavior | application use case and commit | operation/result class, duration | workflow/trace/domain ref | sensitive | no observed value or resource ID | dynamic profiles | contract/perf | 053/054/056 | latest semantics remain domain-owned |
| log + metric + span | outbox/publication | continuity, backlog and delivery | commit, claim and transport outcome | event family, transport/outcome, backlog/age | event/workflow/trace | sensitive | no event body/topic/resource ID | streaming profiles | replay/fault/perf | 049/054/056 | attempt differs from acknowledgement |
| metric + span summary | streaming | connection, replay and backpressure | handshake, delivery summary and close | protocol/reason class, duration/count | subscription/trace protected | sensitive | no filter/token/topic/subscriber ID | live profiles | slow-consumer/load | 054/056 | avoid per-event high-volume spans |
| log + metric + span + audit ref | command | safety lifecycle and external effect attempt | each accepted transition/effect boundary | transition/outcome/reason class, simulated adapter | command/workflow/trace/audit | high sensitivity | no parameters, target, actor or gateway endpoint | command-sim | safety/fault | 055/056 | telemetry never declares physical outcome |
| log + metric + span + audit ref | authn/authz/policy | denial, latency and security transition | decision ownership boundary | mechanism, operation/outcome/reason class | request/trace/audit | protected | no subject/claims/scopes/policy labels/hidden resource | protected profiles | adversarial/security | 055 | concealment preserved by audience |
| metric + health + audit ref | audit subsystem | evidence-path availability and backlog | append/spool/checkpoint boundary | event family, outcome, backlog/age | audit/run | protected | no audit record content | write/effect profiles | failure/recovery | 049/055 | audit itself remains authoritative |
| log + metric + health | DDIL posture | dependency/posture transition and resource pressure | evidence update/posture evaluation | state/dependency/reason class, evidence age | run/workflow | sensitive | no topology/authority package detail | ddil-single | partition/recovery | 053–055 | connectivity differs from authority |
| log + metric + span + audit ref | synchronization | transfer/apply/gap/conflict behavior | session, classify and apply boundary | stage/outcome/conflict class, backlog/age | sync/trace/audit | high sensitivity | no peer/resource/revision/conflict ID label | sync-two-node | duplicate/gap/conflict | 054–056 | telemetry cannot resolve conflict |
| log + span + terminal evidence | migration/backup/restore | stage progress and final result | administrative stage boundary | stage/outcome, release/config/migration identity | run/workflow/trace | sensitive | no DSN, credential, object URL or content | admin profiles | lifecycle/recovery | 049 | exit/schema state is authoritative |
| manifest + selected signals | conformance/interop | reproducible test diagnosis | harness case/run boundary | release/profile/capability/fixture IDs, safe outcome | run/case/request/trace | controlled artifact | publish only redacted bounded bundle | test profiles | verdict support | 050/056 | telemetry is not test oracle |

### 11.2 API and query detail

Instrument negotiation, filter parse, authorized-query-plan construction, repository query and representation assembly as distinct spans. Metrics aggregate route/capability and outcome; they never encode filter text, result IDs, returned count for protected queries, bbox or time range. Logs may record bounded syntactic feature classes and safe page-size buckets, not client values. Pagination cursor failures use a reason class, never cursor content. **[A/E/P]**

### 11.3 Validation

`validation_id` links problem, protected log, validation artifact and audit when required. It is not a metric label. Counts distinguish structural, schema/profile, semantic/unit, stateful/current-invariant, source-trust and policy/safety stages using closed enums. Public errors retain accepted concealment; administrators with resource authority may retrieve a bounded artifact. Repeated hostile failures are aggregated/rate-limited in logs while counters continue. **[A/E/P]**

### 11.4 Persistence

Database instrumentation records logical operation names rather than SQL text. Pool wait, query and transaction duration are separate; retry/deadlock/serialization/idempotency/constraint failures use safe classes. Slow-query detection records operation class and threshold version, then relies on protected database-native tools for statement detail. `pg_stat_*` access uses a least-privilege monitoring role and is optional deployment telemetry, not application readiness truth by itself. **[D/E/P]**

### 11.5 Ingestion and source trust

Each receive attempt produces counters; each committed record and rejected/quarantined/duplicate outcome is counted at the ownership boundary to avoid double counting. Batch spans summarize counts, bytes and latency; per-record spans are disabled by default at volume. Source lag uses event/acquisition time only when semantically valid and reports unknown separately. Last-success timestamps distinguish quiet sources from stuck pipelines only when an expected cadence exists. Protected diagnostics may resolve a source-specific workflow ID after authorization. **[A/E/P]**

---

## 12. Dynamic-Data, Streaming/Event, Command/Control, Security/Policy, and Audit Observability Findings

### 12.1 Dynamic data and publication

Observation/status/current/latest metrics describe operations and latency, not observed values or resource identity. The transaction span links committed state, outbox and audit references. Outbox backlog has count and oldest-age gauges; publication records claim, attempt, acknowledgement, retry and terminal failure. Metrics distinguish transport/event family with bounded enums. Domain event IDs and sequence/watermark live in durable evidence and protected logs/traces, never labels. **[A/E/P]**

SSE/MQTT instrumentation records connection/subscription counts, handshake/replay/delivery latency, filtered event aggregate, backpressure and disconnect reason. A long-lived stream gets a connection span plus bounded summary/periodic events; per-event spans are sampled or linked only when diagnostically justified. Policy-filter counts are protected aggregates and cannot reveal a hidden resource to a subscriber. Topic strings, filters and last-event tokens are excluded. **[A/E/P]**

### 12.2 Commands and feasibility

Command observability distinguishes submission, validation, feasibility, authorization, safety, approval, accepted transition, dispatch ticket, gateway attempt/acknowledgement, status reconciliation, cancellation, timeout and unknown outcome. Client-visible problems/status remain the authoritative API; audit records the actor/decision/effect evidence; telemetry reports safe transition/outcome/reason classes. Command parameters, target identity/location, operator identity, authority/policy labels and gateway endpoint are forbidden in logs/traces/metrics. **[A/E/P]**

The current `command-sim` profile may retain all sanitized spans due to low volume. Public demo exposes no command metric that reveals capability or activity. Telemetry failure cannot permit, cancel or declare a command outcome. Unknown gateway outcome remains unknown even if a span timed out. **[A/E/P]**

### 12.3 Security and policy

Authentication successes/failures, authorization/policy outcomes, rate limiting, unsafe-profile attempts, admin operations, trust failures and redaction actions create appropriate logs/metrics and authoritative audit events where accepted. Metrics use mechanism/operation/outcome classes only. Logs do not record subject, token claims, scopes, groups, client IP, resource existence or policy labels unless an explicitly protected incident channel and classification policy later authorizes a pseudonymous reference. Denial volume and timing are themselves sensitive and remain private. **[A/D/E/P]**

Policy spans report evaluator kind, policy-version logical ID/digest where permitted, cache result, duration and allow/deny/indeterminate—not input attributes, rule text, markings or transformed content. Concealed `404` behavior remains concealed in access logs available to general operators; privileged audit retains the decision under its own access model. **[A/E/P]**

### 12.4 Audit separation and self-health

The audit subsystem exposes only operational metrics such as append attempts/failures/duration, spool depth/age, checkpoint state and export health. Audit record contents, actor/action/resource/decision fields and integrity chain are not duplicated into logs or metric labels. A mandatory audit failure follows accepted E0–E5 behavior even if telemetry is healthy; a telemetry failure never reports audit success. Access/export of audit evidence is itself audited. **[A/P]**

---

## 13. DDIL, Synchronization, Migration, and Conformance Observability Findings

### 13.1 DDIL

Observe connectivity/dependency/authority/data/time/synchronization/capacity dimensions independently. A bounded current-state gauge and transition counter use accepted enum states; logs/system events record transition, observation freshness and reason class; protected diagnostics show evidence. Client responses use `RepresentationAssessmentV1` and accepted problem semantics, not internal health. Telemetry buffers have explicit local storage/CPU budgets and lowest-priority drop policy; audit/domain/outbox data wins resource contention. **[A/E/P]**

During disconnection, stdout/local bounded metrics remain functional and OTLP export failure is summarized rather than retried without limit. On reconnect, exporter recovery does not flood the operational link: queues, batches, retry/backoff and maximum age/volume are bounded. Telemetry replay, if configured, remains distinguishable from current signals and never delays synchronization or command safety. **[D/E/P]**

### 13.2 Synchronization

Sync metrics include session/transfer/apply attempts/outcomes/duration, backlog/oldest age, gaps, duplicates, quarantines and conflict classes. Labels do not include peer, resource, revision or conflict IDs. A session trace links exchange, validation, policy/trust, apply transaction and audit, while redelivery is a new linked attempt. Protected diagnostics resolve specific `sync_id`/`conflict_id` through authorized application state. Telemetry never decides conflict resolution or authority. **[A/E/P]**

### 13.3 Migration/bootstrap/backup/restore

Same-image administrative commands emit JSON stage events, spans, duration, last-success/failure evidence and a terminal exit status. They include release/config fingerprint, migration range and backup/fixture package identity, but no DB endpoint/credential, object-store URL or record content. Migration readiness derives from authoritative schema compatibility, not a log scrape. IDR-SRV-049 owns lifecycle stages, backup/restore metrics, retention and evidence manifests. **[A/E/P]**

### 13.4 Conformance and interoperability evidence

Each run captures release/image, profile/config/capability/standards/fixture fingerprints; external origin; test/harness/client versions; start/readiness/dependency summaries; per-case request/correlation ID; RFC 9457 response; selected sanitized logs/traces/metric snapshots; and teardown outcome. Telemetry is supporting evidence, not the conformance verdict or oracle. External servers remain dated observations. A failing case bundle is bounded and redacted before publication. **[A/E/P]**

---

## 14. Redaction, Safe Disclosure, Debug-Mode, and Sensitive-Data Handling Findings

### 14.1 Field policy

The IDR-SRV-047 schema classification registry feeds observability policy. Each telemetry field declares allowed signals, audiences, value type/length, cardinality class and transformation. Serialization uses an allowlist; after-the-fact regex redaction is defense in depth. Secret wrappers cannot implement ordinary serialization/debug. Untrusted strings are length-bounded and structurally encoded to prevent CR/LF/delimiter/log injection. **[A/D/E/P]**

| Data | Logs/traces | Metrics | Health/diagnostics |
|---|---|---|---|
| credentials, tokens, cookies, private keys, DSNs | forbidden | forbidden | forbidden; resolution status only |
| request/response/SensorML/SWE/Observation/Command payload | forbidden | forbidden | counts/status only |
| raw path/query/filter/cursor/header | forbidden | forbidden | forbidden |
| actor/source/resource/command/event/sync IDs | protected reference only when necessary | forbidden labels | authorized lookup only |
| policy labels/rules/markings/hidden counts | forbidden/general; audit-owned | protected aggregate only if approved | authorized bounded explanation |
| internal endpoints/topology/files/SQL | summarized logical class | bounded class only | admin redacted logical IDs |
| route template/operation/status class/reason class | allowed | allowed bounded label | allowed by audience |
| release/profile/config fingerprint/schema version | allowed internally | bounded resource metadata | public only by explicit posture |

### 14.2 Debug controls

Debug mode and log level are separate. Debug mode may enable local developer diagnostics, Tokio console or additional safe spans, but never raw bodies/secrets. It is accepted only in `dev-native`, `dev-compose` and designated CI tests; startup rejects it elsewhere. Dynamic log filtering is the sole first-release reload candidate from IDR-SRV-047, remains capped by the profile, expires automatically when elevated, and creates an audit event. `RUST_BACKTRACE`, panic payloads and source locations are protected operator artifacts and disabled from public responses. **[A/E/P]**

### 14.3 Test and incident safeguards

Canary secrets, controlled metadata and unique IDs are injected through every path; tests assert absence from logs, OTLP exports, metric series, health, diagnostics, panic output and CI bundles. Negative tests cover encoded/newline/header/URI userinfo and downstream error leakage. Telemetry access, export and support-bundle creation are least-privilege and audited. Retention/disposal follows classification and legal policy; this report sets no duration. **[D/E/P]**

---

## 15. Rust Tooling and Optional Observability Stack Findings

### 15.1 Candidate stack

| Tool | Role | Decision | Proof required |
|---|---|---|---|
| `tracing` 0.1.44 | spans/events API | adopt candidate throughout adapters/application; domain remains telemetry-light | async parent/link correctness and overhead |
| `tracing-subscriber` 0.3.23 | layers, filters, JSON/text | adopt candidate; one composition-root registry | exact JSON schema, reload cap and sanitization |
| `tower-http` 0.7.1 TraceLayer | HTTP span lifecycle | adapt with custom make-span/on-response; do not accept raw URI defaults blindly | route-template timing, error and streaming closure |
| `tracing-appender` 0.2.5 | bounded non-blocking output | conditional candidate for stdout/file adapter | drop accounting, priority, flush/guard lifecycle |
| `metrics` 0.24.6 | recorder-neutral metric facade | adopt candidate with Glaux wrapper/registry | duplicate registration, cardinality and overhead |
| `metrics-exporter-prometheus` 0.18.3 | protected scrape | first-slice candidate | management-listener/auth, OpenMetrics mapping, scrape bounds |
| `opentelemetry` 0.32.0 + SDK/OTLP | portable trace/export model | optional exporter capability | version alignment, queue/drop/shutdown and DDIL behavior |
| `tracing-opentelemetry` 0.33.0 | tracing-to-OTel bridge | optional candidate | span fields/links/events and filtering semantics |
| Tokio console | task/runtime diagnosis | dev-only opt-in | overhead and exposure; never shared-profile default |

The application owns stable typed helper APIs and schema constants so crate swaps do not rename telemetry casually. There is one global tracing registry and one metrics recorder initialized at composition. Library crates emit but never install exporters. Build features may omit optional OTLP/console code; runtime profile/config controls installed capabilities. **[D/E/P]**

### 15.2 First implementation versus later scope

First implementation requires structured stdout JSON/text, HTTP/use-case/DB/validation spans, the baseline metrics families, private scrape, health registry/endpoints, redaction tests and in-memory test collectors. Optional first-slice OTLP trace export should follow only after propagation/shutdown proof. Direct OTel log export and OTel metrics export are deferred until they offer clear value over collecting JSON stdout and scraping metrics; dual emission that double-counts is prohibited. **[E/P]**

### 15.3 Optional deployment stack

The Compose `observability` profile may add one pinned Collector, Prometheus and Grafana plus either Tempo or Jaeger for traces and optionally Loki for logs. These are evaluation conveniences, not runtime dependencies or production selections. Prefer OTLP from app to Collector, scrape app/Collector metrics, and collect stdout through the deployment platform rather than writing application log files. Collector processors enforce memory limits, batching, attribute allowlists and secure export. **[D/E/P]**

Collector queues retry transient failure but can overflow, expire or lose memory-buffered data; persistent WAL also fails on disk exhaustion. Glaux monitors exporter/Collector self-telemetry and documents possible loss. Tail sampling and agent/gateway tiers are deferred until volume/topology justifies their state and complexity. Audit never travels only through this pipeline. **[D/E/P]**

### 15.4 Alternatives

Direct backend SDKs, a required sidecar, application-managed Loki/Jaeger formats and embedded dashboards are rejected because they couple Glaux to deployment choices. Direct OTel metrics instead of the `metrics` facade remains a viable later alternative; a prototype must compare semantic-convention alignment, Prometheus translation, crate maturity, test ergonomics and overhead before changing the baseline. **[E/P]**

---

## 16. Test, CI, Conformance, Performance, Security, and Interoperability Implications

### 16.1 Verification strategy

Tests inspect captured structured events/instruments/spans as typed records, not rendered prose. A `TestTelemetry` adapter provides bounded in-memory collection and query by event/instrument/span name. Contract tests assert required/forbidden fields, stable types, correlation, emission count and redaction. Snapshot tests are limited to versioned schemas/examples with normalized time/IDs; business tests do not couple to incidental DEBUG messages. **[E/P]**

| Suite | Evidence/assertions |
|---|---|
| unit/property | name/type/classification registries, label allowlist, state transitions, redaction transforms |
| API/integration | one request completion, route template, status/problem mapping, trace/request propagation |
| database/fault | pool saturation, timeout/rollback/retry classes, readiness transition without liveness failure |
| ingestion/stream | attempt/outcome counts, backlog/age, slow consumer and no per-ID series growth |
| command/security | each safety stage, concealment, audit separation, no parameter/identity leakage |
| DDIL/sync | role-aware readiness, evidence expiry, dependency/posture transitions, bounded reconnect export |
| conformance/interop | manifest/fingerprint, request correlation and sanitized failure bundle |
| performance | telemetry on/off/profile overhead, histograms, series count, queue/drop behavior |
| chaos/recovery | exporter/backend failure does not block correctness; loss/self-health visible |

### 16.2 CI artifacts

CI records the redacted effective manifest, JSON logs, health transition timeline, final metrics snapshot, selected failed-test traces, audit-evidence references, Compose resolution, test results and teardown. Successful high-volume lanes retain summaries; failed cases retain bounded contextual windows. Artifact access/retention follows classification. The CI lane fails on schema drift, forbidden fields/canaries, unbounded series, missing required events, duplicate counting or telemetry-caused behavior change. **[A/E/P]**

### 16.3 Performance and security

IDR-SRV-054 must measure CPU, allocation, latency and throughput with representative logging levels, metrics and trace sampling; “telemetry off” is a diagnostic comparison, not the operational benchmark. Stress fills log/export queues and cardinality inputs to confirm bounded memory. IDR-SRV-055 attempts log injection, secret/controlled-data exfiltration, metric-label DoS, trace-context abuse, diagnostic enumeration, health amplification and filter elevation. **[E/P]**

### 16.4 Interoperability

IDR-SRV-056 uses server-generated request IDs plus W3C trace context when a cooperating client supports it. Evidence distinguishes client observation, server access event, problem response and protected internal trace. The external API does not require a tracing vendor/header beyond standards-compatible optional propagation. Internal origins, tokens and filtered resource facts are removed before sharing test bundles. **[N/E/P]**

---

## 17. Downstream Topic Handoff Matrix

| Topic | Fixed input from IDR-SRV-048 | Still owned downstream |
|---|---|---|
| IDR-SRV-049 Migration/upgrade/backup | administrative stage events/spans, release/config/migration identity, last success/failure and readiness compatibility; no credentials/content | lifecycle algorithms, locking, rollback, backup/restore formats, RPO/RTO, retention and runbooks |
| IDR-SRV-050 Conformance harness | capture manifest, safe health, typed telemetry and per-case correlation as supporting evidence | test selection/mapping, verdicts, evidence packaging and official tooling |
| IDR-SRV-052 Rust tests | `TestTelemetry`, typed schema contracts and fault assertions | full test pyramid, harness architecture, property/fuzz strategy |
| IDR-SRV-053 Fixtures | canary/redaction, correlation and expected telemetry scenarios | corpus organization, provenance, generators and golden governance |
| IDR-SRV-054 Performance | metric/histogram families, cardinality budgets, on/off overhead and queue/drop tests | workloads, buckets/thresholds, resource budgets and acceptance criteria |
| IDR-SRV-055 Security/commands | disclosure matrix, trace/baggage trust, diagnostic access, command-safe telemetry and audit separation | adversarial matrices, authorization and command safety test implementation |
| IDR-SRV-056 Interoperability | request/trace/correlation evidence, external-origin diagnostics and safe bundles | client/server matrix, cases and result governance |
| deployment/operations | private management boundary, portable stdout/scrape/OTLP, role-aware probes and optional stack | production topology, backends, SLOs, alerts, retention and incident response |

### 17.1 Ordered implementation proofs

1. typed telemetry vocabulary/schema and forbidden-field registry;
2. custom Tower request span with route-template, problem and streaming lifecycle correctness;
3. JSON/text equivalence, log injection and canary-redaction tests;
4. metrics registry with bounded-label/cardinality adversarial test;
5. async context propagation and durable producer-consumer span links;
6. liveness/startup/readiness/dependency registry state-machine and no-fan-out proof;
7. role/profile readiness cases including DDIL local-ready and graceful drain;
8. bounded log/export queues, drop accounting and graceful shutdown;
9. optional OTLP trace export/Collector outage without business-path impact;
10. audit/domain/telemetry non-substitution and atomic-reference tests;
11. protected diagnostic authorization/redaction/rate/size tests; and
12. full CI evidence bundle plus telemetry-overhead/cardinality baseline.

---

## 18. Recommendations

1. Adopt the Section 5 signal taxonomy and prohibit logs/traces/metrics from substituting for domain events or audit. **[P]**
2. Use versioned typed event/instrument/span schemas and one composition-root telemetry registry. **[P]**
3. Adopt `tracing`/`tracing-subscriber` with custom Tower instrumentation and JSON stdout for shared profiles; allow equivalent text locally. **[D/E/P]**
4. Adopt the `metrics` facade plus protected Prometheus/OpenMetrics export as the minimal metrics path, with one bounded label registry. **[D/E/P]**
5. Use OpenTelemetry/W3C Trace Context for portable propagation and optional OTLP trace export, never for identity/authorization. **[D/N/P]**
6. Implement route-template, reason-class and operation-class instrumentation; prohibit raw paths, queries, payloads, SQL and unbounded IDs. **[P]**
7. Separate liveness, startup, role-aware readiness, protected dependency detail and client-visible service posture. **[D/A/P]**
8. Drive health endpoints from asynchronous freshness-bounded assessments; do not fan out or include external dependencies in liveness. **[P]**
9. Put metrics/readiness/diagnostics on a private management boundary and expose no detailed public health. **[P]**
10. Make telemetry queues/export bounded and self-observing; correctness, audit and safety remain independent. **[P]**
11. Keep Collector/Prometheus/Grafana/trace/log backends optional and vendor-neutral; do not claim their queues make evidence durable. **[D/P]**
12. Execute the twelve proofs in Section 17 before production-facing observability claims. **[P]**

---

## 19. Risks, Constraints, and Open Questions

### 19.1 Risk register

| Risk | Consequence | Control / proof |
|---|---|---|
| telemetry confused with audit/domain truth | missing accountability or state | separate types/stores/failure rules and non-substitution tests |
| raw values leak through attributes/errors | security/policy disclosure | allowlist schema, wrappers, injection/canary tests and output access controls |
| high-cardinality labels | memory/storage/availability failure | closed value registries, series budget and adversarial unique-ID tests |
| deep liveness probe | cascading restart storm | constant local liveness; cached async dependency evidence |
| over-broad readiness | unnecessary outage during optional/DDIL loss | role/profile mandatory-gate matrix and operation posture separation |
| under-broad readiness | traffic reaches incapable instance | schema/DB/package/capability consistency gates and expiry |
| trace context trusted as identity | spoofing/correlation disclosure | syntax/size validation, baggage allowlist and security-context separation |
| sampling hides rare safety issue | incomplete diagnosis | audit always durable; higher selected diagnostic sampling; metrics counters |
| telemetry outage blocks server | lost availability | non-blocking bounded export, drop accounting and fault test |
| telemetry overload steals DDIL resources | domain/audit loss | resource priority/budgets and bounded reconnect replay |
| schema/convention upgrades break dashboards/tests | silent observability drift | pin schema URL/versions and compatibility tests |
| public aggregate leaks hidden activity | inference side channel | private raw metrics, curated minimum public view, policy/security review |

### 19.2 Constraints

- No production SLO, alert threshold, bucket, sampling percentage or retention duration is set.
- Telemetry must function minimally with no external backend.
- Management endpoint authentication depends on accepted security adapters, not a special bypass.
- Data classification may require stricter deployment-specific suppression than this baseline.
- Profiling remains alpha in OpenTelemetry and is dev/performance tooling, not a first-slice signal.
- Inbound Part 3 and physical command effects remain outside the authorized server scope.

### 19.3 Open questions with owners

| Question | Default until resolved | Owner |
|---|---|---|
| exact histogram buckets and SLO/alerts? | conservative bounded buckets; no SLO claim | IDR-SRV-054/operations |
| direct OTLP metrics/logs versus scrape/stdout collection? | scrape metrics and stdout logs; optional OTLP traces only | implementation proof/deployment |
| Collector queue/WAL and retention sizes? | optional bounded development values only | deployment/operations |
| which dependency failure blocks each role? | accepted mandatory profile gates; all others degraded/diagnostic | IDR-SRV-049 and implementation profile tests |
| is a shallow public demo monitor endpoint needed? | no public health route unless deployment requires it | demo deployment owner |
| safe security/command aggregate dashboard? | none public; protected aggregate only | IDR-SRV-055 |
| trace sampling percentages and tail sampling? | full in bounded tests, measured deterministic head sampling shared | IDR-SRV-054/operations |
| production support-bundle contents/retention? | no automated bundle beyond redacted diagnostics | operations/security decision |

---

## 20. Validation Against This Plan's Success Criteria

| Success criterion | Result | Evidence |
|---|---|---|
| signal taxonomy with sources and traceability | Met | Sections 3–5 |
| log, metric, trace/span and correlation strategy | Met | Sections 6–8 |
| health/liveness/readiness/dependency/degraded/admin diagnostics | Met | Sections 9–10 |
| API, validation, persistence, ingestion, streaming, commands, security, policy, audit, DDIL and sync | Met | Sections 11–13 |
| redaction, disclosure, cardinality, debug and profile constraints | Met | Sections 7, 10 and 14 |
| Rust tooling and optional stack evaluated | Met | Section 15 |
| test, CI, conformance, performance, security and interoperability implications | Met | Section 16 |
| implementation/community lessons non-normative | Met | Section 3.3 |
| decision-usable and server-bounded recommendations | Met | Sections 2 and 18–19 |
| downstream handoffs explicit | Met | Section 17 |
| references explicit and reproducible | Met | Section 21 |

All planned phases and required report content are complete. Acceptance remains a project-lead action. **[P]**

---

## 21. References

### 21.1 Observability and health sources

- OpenTelemetry Specification 1.61.0: https://opentelemetry.io/docs/specs/otel/
- OpenTelemetry signals: https://opentelemetry.io/docs/concepts/signals/
- OpenTelemetry logs data model: https://opentelemetry.io/docs/specs/otel/logs/data-model/
- OpenTelemetry HTTP semantic conventions: https://opentelemetry.io/docs/specs/semconv/http/
- OpenTelemetry sampling: https://opentelemetry.io/docs/concepts/sampling/
- OpenTelemetry Collector deployment: https://opentelemetry.io/docs/collector/deploy/
- OpenTelemetry Collector resiliency: https://opentelemetry.io/docs/collector/resiliency/
- Prometheus metric/label naming: https://prometheus.io/docs/practices/naming/
- Prometheus instrumentation: https://prometheus.io/docs/practices/instrumentation/
- Kubernetes probes: https://kubernetes.io/docs/concepts/workloads/pods/probes/
- PostgreSQL monitoring: https://www.postgresql.org/docs/current/monitoring.html
- W3C Trace Context: https://www.w3.org/TR/trace-context/
- OWASP Logging Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html
- OWASP Error Handling Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html
- NIST SP 800-92: https://csrc.nist.gov/pubs/sp/800/92/final

### 21.2 Rust candidate sources

- tracing 0.1.44: https://docs.rs/tracing/0.1.44/tracing/
- tracing-subscriber 0.3.23: https://docs.rs/tracing-subscriber/0.3.23/tracing_subscriber/
- tracing-appender 0.2.5: https://docs.rs/tracing-appender/0.2.5/tracing_appender/
- tracing-opentelemetry 0.33.0: https://docs.rs/tracing-opentelemetry/0.33.0/tracing_opentelemetry/
- OpenTelemetry Rust 0.32.0: https://docs.rs/opentelemetry/0.32.0/opentelemetry/
- metrics 0.24.6: https://docs.rs/metrics/0.24.6/metrics/
- metrics-exporter-prometheus 0.18.3: https://docs.rs/metrics-exporter-prometheus/0.18.3/metrics_exporter_prometheus/
- tower-http 0.7.1 tracing middleware: https://docs.rs/tower-http/0.7.1/tower_http/trace/
- Tokio console: https://github.com/tokio-rs/console

### 21.3 Standards and project evidence

- OGC API - Connected Systems Part 1: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems Part 2: https://docs.ogc.org/is/23-002/23-002.html
- SensorML 3.0: https://docs.ogc.org/is/23-000/23-000.html
- SWE Common 3.0: https://docs.ogc.org/is/24-014/24-014.html
- RFC 9110: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9457: https://www.rfc-editor.org/rfc/rfc9457
- CloudEvents: https://cloudevents.io/
- Overall IDR Research Plan: [overall-idr-research-plan.md](../IDR%20Plans/overall-idr-research-plan.md)
- IDR-SRV-048 Research Plan: [idr-srv-048-observability-logs-metrics-and-health-check-strategy.md](../IDR%20Plans/idr-srv-048-observability-logs-metrics-and-health-check-strategy.md)
- Glaux Server Goal and Definition: [glaux-server-goal-and-definition.md](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- Accepted IDR-SRV-001 through IDR-SRV-047 reports: [IDR Reports](./)
- IDR-SRV-047 configuration report: [idr-srv-047-configuration-secrets-and-environment-strategy-report.md](idr-srv-047-configuration-secrets-and-environment-strategy-report.md)

### 21.4 Non-normative implementation/community evidence

- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- OGC API Connected Systems development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- Accepted IDR-SRV-014A through IDR-SRV-014G reports: [IDR Reports](./)

### 21.5 Reproducibility and evidence limits

Official specifications, product documentation and crate documentation were checked on September 16, 2026. Version numbers record the evidence freeze and must be revalidated through dependency, license, MSRV and security review. OpenTelemetry semantic-convention stability is recorded explicitly rather than assumed. Implementation/community studies remain informative. No Glaux telemetry implementation, workload, production backend, incident process or health failure drill existed; Section 17 makes those proof obligations explicit.

---

## Report Completion Checklist

- [x] All 21 required sections are present
- [x] Signal taxonomy and non-substitution rules are explicit
- [x] Log, metric, trace and correlation models are complete
- [x] Required 12-column observability matrix is complete
- [x] Health/readiness/dependency/degraded-state contracts are complete
- [x] All functional areas and eleven profiles are covered
- [x] Redaction, cardinality, debug and exposure controls are explicit
- [x] Rust/stack options, verification and downstream handoffs are complete
- [x] All 11 success criteria validate as Met
- [ ] Accepted by Glaux Project Lead

# Section 046: Reference Deployment Strategy - Research Report

**Topic ID:** IDR-SRV-046<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-046 Reference Deployment Strategy](../IDR%20Plans/idr-srv-046-reference-deployment-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All questions concerning deployment profiles, supporting services, images, Compose, local development, CI, conformance, demonstrations, DDIL simulation, proxy/TLS/origin, configuration/secrets, database bootstrap/migrations/backup/restore, observability/health, Glaux and external integrations, streaming, commands, synchronization, packaging, release evidence, operational caveats, and downstream handoffs<br>
**Methodology Used:** Accepted-requirement extraction; current primary-source runtime freeze; profile/capability and service-dependency matrixing; startup, trust, data and teardown-path analysis; implementation-lesson reconciliation; security/operability review; phased proof-gate synthesis<br>
**Research Time:** Approximately 34 hours of AI-assisted execution on September 15–16, 2026<br>
**Accepted Architecture Baseline:** IDR-SRV-045 Cargo-workspace modular monolith; one initial server artifact and authoritative PostgreSQL/PostGIS write core; logical API, worker and admin roles; ports/adapters at effect seams; profile-gated capabilities and measured extraction<br>
**Runtime Evidence Freeze:** Docker Compose 5.1.4; OCI Image Specification 1.1.1; PostgreSQL 18.6; PostGIS 3.6.4/current `18-3.6` image documentation; Caddy 2.11.3; OpenTelemetry Collector 0.160.0; SLSA 1.2; official documentation checked September 15, 2026<br>
**Standards Baseline:** OGC API - Connected Systems Parts 1 and 2 Version 1.0; SensorML 3.0; SWE Common 3.0; OGC API - Features; RFC 9110, 9457 and 7239; accepted Glaux IDR-SRV-001 through IDR-SRV-045<br>
**Document Purpose:** Define repeatable reference deployment shapes and runtime contracts without claiming accredited production hosting, selecting an enterprise platform, implementing infrastructure, or authorizing later Category H work<br>
**Author:** OpenAI Codex<br>
**Accepted By:** Glaux Project Lead<br>
**Acceptance Date:** September 16, 2026<br>
**Date:** September 16, 2026<br>
**Last Updated:** September 16, 2026

---

## Evidence and Decision Legend

- **[N] Normative:** approved external standard.
- **[A] Accepted project baseline:** accepted Glaux report or governing decision.
- **[D] Direct documentation:** current official product/specification documentation.
- **[I] Implementation evidence:** source, release, deployment or behavior from another implementation; informative only.
- **[T] Test/observation evidence:** reproducible test or captured operational observation; bounded to its conditions.
- **[E] Analysis:** reasoned synthesis from identified evidence.
- **[P] Project recommendation:** proposed Glaux decision pending acceptance of this report.
- **[X] Explicit boundary:** excluded claim, product selection or later-topic responsibility.

Architecture and deployment terms are functional, not accreditation claims. “Reference,” “operational-reference,” “secure,” “edge” and “DDIL simulation” do not mean production-approved, accredited, certified, cross-domain-capable or tactically qualified. **[X]**

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Deployment Requirement Extraction Methodology
5. Deployment Profile Taxonomy
6. Supporting Service Inventory
7. Containerization and Image Strategy Findings
8. Docker Compose / Local Development Findings
9. CI and Conformance Runtime Findings
10. Public Demo Runtime Findings
11. Tactical-Edge/DDIL Simulation Findings
12. Network, Proxy, TLS, CORS, and Generated URL/Link Findings
13. Configuration and Secret-Handling Implications
14. Database Bootstrap, Migration, Backup, Restore, Seed, and Fixture Findings
15. Observability, Logging, Metrics, Tracing, Health, and Readiness Findings
16. Publisher, Simulator, Webapp/Mobile, CSAPI Explorer, and External-Client Integration Findings
17. Streaming, Command/Control, and Federation/Synchronization Runtime Implications
18. Downstream Topic Handoff Matrix
19. Recommendations
20. Risks, Constraints, and Open Questions
21. Validation Against This Plan's Success Criteria
22. References

---

## 1. Executive Summary

Glaux should ship a **Docker Compose reference deployment centered on one immutable `glaux-server` OCI image and one PostgreSQL/PostGIS database**, with the same server image used for `serve`, `migrate`, validation/bootstrap and other bounded administrative commands. The base stack stays deliberately small. Static standards/schema/profile packages are versioned release inputs, not network dependencies. SSE uses the HTTP service and durable outbox/replay core, so the first live-data profile needs no broker. External identity, policy, object storage, MQTT broker, observability backends, reverse proxy, simulator, publisher, conformance harness, fault injector and second server node are named profile additions rather than hidden mandatory dependencies. **[A/D/E/P]**

The first implementation should support two equally real developer paths: native Rust server plus a containerized database for the shortest edit/test loop, and a fully containerized base stack for parity and onboarding. Compose profiles should add tools and scenarios without making optional services part of default startup. A one-shot migration job must wait for a healthy database and complete successfully before the server starts. Seed/fixture load is a separate explicit, versioned and idempotent action; PostgreSQL `/docker-entrypoint-initdb.d` behavior is unsuitable as the migration system because the official image runs initialization only against an empty data directory. **[D/E/P]**

CI should build once, identify the image by digest, start a uniquely named ephemeral stack, migrate an empty database, load the scenario manifest, wait on readiness, execute black-box tests, collect the resolved Compose configuration, version manifest, logs and results, and always destroy volumes. Unit tests remain container-free; database, conformance, security, interoperability and failure suites use purpose-built lanes. Mutable third-party demos remain observational and never gate a merge. **[A/I/E/P]**

The public demonstration profile should place an allowlisted TLS reverse proxy in front of the server, configure one explicit external base URL, expose only intended API and documentation routes, use synthetic public data, rate/size/time limits, protected ingestion/admin/diagnostics, and **disable physical command dispatch**. Read resources, dynamic data and authenticated or appropriately bounded SSE may be demonstrated; simulated publisher and simulated commands require isolated, visibly labeled profiles. Reset must rebuild from a versioned public fixture pack, never mutate an unexplained long-lived state. **[A/E/P]**

DDIL work begins as simulation, not field readiness. A self-contained node keeps its database, schema/profile/vocabulary package, policy and credential-verification material, audit journal, outbox/replay log and approved source buffers locally. A separate two-node test topology introduces controlled dependency loss, delay, duplication, reordering, partition and reconnect, then verifies the accepted synchronization/conflict contract. Database replication, broker clustering or a network fault tool does not decide Glaux authority or conflicts. **[A/E/P]**

Images use a multi-stage Rust build, a reviewed minimal runtime with CA certificates and required runtime data, a fixed non-root identity, read-only root filesystem compatibility, explicit writable mounts/tmpfs, dropped capabilities, graceful shutdown and OCI source/version/revision annotations. Release images and dependencies are pinned and deployed by digest; CI emits SBOM and provenance attestations. A distroless/scratch image is not assumed until TLS roots, diagnostics, health execution, native dependencies and incident needs are proven. The server image should target `linux/amd64` first. The current official PostGIS `18-3.6` image documents `amd64` support only, so `linux/arm64` is a gated target requiring a vetted database image/build and full spatial/backup/restore tests. **[D/E/P]**

Compose is the **reference-environment contract**, not the production orchestrator. The release must also define portable runtime invariants—commands, ports, public origin, health/readiness, graceful shutdown, secrets-as-files, volumes, migration lock/compatibility, telemetry and artifact identity—so an operator can map the deployment to Kubernetes, systemd, VMs or another approved platform later. Kubernetes manifests, high availability, enterprise PKI/IdP/policy/SIEM/secret management, regional topology, capacity numbers and accreditation are deferred. **[E/P/X]**

Acceptance of this report will authorize IDR-SRV-047 research only. It does not create deployment files, select production vendors, enable live commands or inbound experimental Part 3, approve a tactical node, authorize cross-domain transfer, or claim OGC conformance. **[P/X]**

---

## 2. Scope and Plan Alignment

### 2.1 Coverage

| Plan area | Coverage | Evidence location |
|---|---|---|
| profiles and first deployment | Complete | Sections 5, 8–11 |
| required/optional/deferred services | Complete | Section 6 |
| container, Compose, bootstrap and teardown | Complete | Sections 7–9, 14 |
| proxy/TLS/origin/CORS | Complete | Section 12 |
| configuration, secrets and security posture | Complete | Sections 10, 12–13 |
| database/migration/backup/restore | Complete | Section 14 |
| telemetry and health | Complete | Section 15 |
| project/external integrations | Complete | Sections 16–17 |
| packaging/release evidence | Complete | Sections 7, 19 |
| implementation lessons and handoffs | Complete | Sections 3.3, 18 |

### 2.2 In scope

- repeatable local, CI, conformance, public-demo, integration, streaming, command-simulation, DDIL/synchronization and operational-reference shapes;
- container image and Compose layout decisions;
- service classification and profile gates;
- runtime startup, readiness, reset, fixture and evidence contracts;
- security-visible distinctions between unsafe convenience, public demonstration and operational-reference expectations;
- portable contracts that later platforms must preserve; and
- explicit decisions and proof needs for IDR-SRV-047 through IDR-SRV-056.

### 2.3 Out of scope

- accredited production, enterprise, classified, cross-domain or tactical-network architecture;
- a Kubernetes platform, service mesh, database HA cluster, regional failover or cloud vendor design;
- final secrets, IdP, policy, telemetry, broker, object-store or proxy vendor selection;
- final migration/upgrade/retention/backup numbers and procedures owned by IDR-SRV-047 through 049;
- implementation code, Dockerfiles, Compose files, CI workflows or infrastructure provisioning;
- real device command authority or physical effect; and
- OGC or AEP conformance certification. **[X]**

### 2.4 Prior-decision invariants

Deployment must preserve: one authoritative PostgreSQL/PostGIS transactional core; exact artifacts separate from canonical state; native PostgreSQL time-series baseline; atomic inbox/outbox/audit references; SSE before broker; MQTT 5 and experimental Part 3 behind explicit gates; deny-by-default commands; application-layer authentication/authorization/policy/source-trust/audit; local DDIL evidence; and domain-aware synchronization distinct from database or broker replication. **[A]**

---

## 3. Evidence Base and Authority Classification

### 3.1 Current primary evidence

| Source | State checked | Evidence used | Limitation |
|---|---|---|---|
| Docker Compose documentation/releases | Compose 5.1.4; checked 2026-09-15 | profiles, dependency conditions, health, secrets, networks, project names, Watch, hardening fields | reference implementation behavior; not an enterprise platform guarantee |
| Docker Build documentation | current 2026-09-15 | multi-stage builds, digest pins, ephemeral images, multi-platform builds, SBOM/provenance | examples are guidance, not Glaux test evidence |
| OCI Image Specification | 1.1.1 | manifests/indexes, content digests, platform variants, source/version/revision/base annotations | image format does not secure build or runtime by itself |
| PostgreSQL documentation | 18.6/current | dump/restore semantics, consistent logical export, archive formats | not a complete production backup design |
| official PostgreSQL image docs | current 2026-09-15 | secret-file convention, first-volume initialization behavior, data mounts | entrypoint behavior is image-specific |
| PostGIS Docker repository | 3.6.4; `18-3.6`, dated 2026-06-19 | candidate database image, PG18 volume-path change, `amd64` support statement | mutable image tags; deployment must pin a tested digest |
| Caddy documentation/releases | 2.11.3; checked 2026-09-15 | optional demo TLS/proxy candidate, forwarded-header behavior | candidate only; no production proxy decision |
| OpenTelemetry documentation/releases | Collector 0.160.0; checked 2026-09-15 | optional collector, local/gateway patterns, queues/retries | telemetry backend/topology remains IDR-SRV-048 |
| Kubernetes documentation | current 2026-09-15 | liveness/readiness/startup semantic comparison | future-platform comparison only |
| SLSA | 1.2 | provenance meaning and build-track evidence | no SLSA level is claimed here |
| OGC/RFC sources | approved versions listed in metadata | API/link/HTTP behavior that deployment must preserve | do not prescribe Docker or topology |

### 3.2 Accepted project evidence

The report uses accepted IDR-SRV-025 through 030 for PostgreSQL/PostGIS, artifacts, time series, transaction/outbox and lifecycle; IDR-SRV-031 through 038 for ingestion, simulator, dynamic data, publication and commands; IDR-SRV-039 through 043 for security, policy, audit, DDIL and synchronization; and IDR-SRV-044/045 for Rust and service architecture. These are project authority for this topic, while current product documents supply mechanism evidence. **[A/D]**

### 3.3 Implementation and community lessons

| Lesson | Evidence | Glaux deployment consequence |
|---|---|---|
| public links can expose internal proxy origins | OSH/community report reproduced correction by setting public base URL | externally test landing, links, redirects, OpenAPI servers and auth scopes as one identity graph |
| public demos disappear, return 502, drift or expire certificates | OSH, SECD and client/interoperability studies | deterministic local stacks gate CI; live demos are dated observational lanes |
| default credentials/permissive CORS/proxy assumptions are easy prototype shortcuts | CS-Go and pygeoapi studies | insecure settings exist only in named loopback/CI profiles; demo is restricted and TLS-fronted |
| startup migration couples availability and schema change | CS-Go/pygeoapi studies | dedicated one-shot migration command and explicit schema compatibility gate |
| large Compose stacks can hide broken application/dependency setup | pygeoapi study | base stack contains only mandatory dependencies; optional tools use profiles |
| deployed OpenAPI can drift from routes/capabilities | OSH, CS-Go, pygeoapi and SECD studies | generate/verify deployment contract and conformance from the installed capability registry |
| fixture/example assets are useful but not normative | all implementation studies | every pack carries provenance, standard/profile pin, digest, expected validity and reset semantics |
| internal hostnames and mutable server data break clients | client/community studies | proxy/base-path and fixed discriminating datasets are mandatory interoperability cases |

These lessons identify failure classes; they do not make another project's image, credentials, proxy, migrations or topology authoritative for Glaux. **[I/T/E]**

### 3.4 Evidence limits

- No Glaux server image or deployment exists to benchmark.
- No throughput, recovery-point, recovery-time, resource-size, startup-time or edge-hardware number is claimed.
- Product versions are a dated research freeze, not automatic implementation pins.
- The PostGIS architecture constraint is based on its current official image documentation; another vetted image/build may change the ARM conclusion.
- Compose portability does not imply feature-identical behavior in Swarm, Kubernetes or third-party Compose implementations.
- Operational security and accreditation require environment-specific threat, authority and control evidence beyond this report.

---

## 4. Deployment Requirement Extraction Methodology

The analysis used six passes:

1. convert accepted server semantics into runtime invariants;
2. classify every supporting service as required, conditional, optional-tooling or deferred;
3. define profiles by purpose and prohibited effects, not by a loose list of containers;
4. trace startup, steady state, degradation, shutdown, reset, backup and restore paths;
5. test each proposal against security, DDIL, conformance and implementation lessons; and
6. allocate unresolved detail to the exact later topic and proof gate.

### 4.1 Evaluation criteria

| Criterion | Question |
|---|---|
| repeatability | can a pinned release and scenario reproduce the same topology and inputs? |
| semantic fidelity | are accepted authority, transaction, link, validation, command, audit and DDIL rules preserved? |
| minimum burden | does the base stack exclude services that do not enable a required first capability? |
| isolation | are tests, projects, volumes, networks, credentials and outbound effects bounded? |
| observable startup | can automation distinguish created, healthy, migrated, initialized and ready? |
| secure distinction | are unsafe local shortcuts impossible to mistake for demo/operational defaults? |
| evidence capture | are image/config/schema/fixture versions and results retained? |
| replaceability | are broker, IdP, policy, proxy, object and telemetry products adapters? |
| failure honesty | does dependency loss yield the accepted degraded/unavailable semantics? |
| teardown safety | is destructive reset explicit, scoped and recoverable where required? |

### 4.2 Core runtime invariants

- Only the database/artifact authority commits canonical state; container memory and broker state are not truth.
- One release image supplies compatible server and administrative commands.
- Capabilities are declared from effective runtime configuration and installed adapters.
- The server refuses incompatible database/schema/profile state rather than improvising startup mutation.
- Public URL construction uses an explicit validated origin/base path or trusted proxy evidence, never an arbitrary incoming header.
- Optional dependency failure cannot make mandatory readiness or data claims ambiguous.
- Physical command egress is absent unless a specifically authorized gateway/profile is installed.
- Reset, seed, migration, backup and restore are different operations with separate evidence.
- Audit remains authoritative evidence; logs/traces/metrics remain operational telemetry.
- Release identity is the image digest plus source revision, configuration/schema manifest and fixture/profile digest—not a mutable tag alone.

---

## 5. Deployment Profile Taxonomy

### 5.1 Required reference deployment matrix

| Deployment profile | Purpose | Enabled capabilities | Disabled/constrained capabilities | Required services | Optional services | Security posture | Configuration/secrets needs | Data/bootstrap needs | Observability needs | Test/conformance use | Interoperability use | DDIL/sync relevance | Downstream topic handoff | Notes / unresolved issues |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `dev-native` | fastest Rust edit/test loop | native API/workers, CRUD, queries, SSE | no broker; commands disabled; no public bind | native server; Compose PostgreSQL/PostGIS; migrate command | docs UI; simulator/publisher | loopback-only; explicit `unsafe-local`; synthetic identity | checked-in nonsecret dev config; generated local secret files | migrate; small deterministic dev pack | structured console logs; liveness/readiness | developer smoke and DB integration | local clients | none | 047/048/049/052/053 | first developer profile |
| `dev-compose` | onboarding and container parity | same core behavior in containers | optional stacks off | server, PostgreSQL/PostGIS, migrate | fixture tool, docs, Watch target | host bind loopback by default; non-root app | local secret files; public base URL | named volume; explicit seed/reset | logs plus health | image/Compose smoke | host clients/Webapp | none | 047–053 | first packaged profile |
| `ci-core` | deterministic merge gate | selected HTTP/write/read/SSE slices | no mutable external services; commands off | pinned server image, fresh DB, migrate, runner | fixture command | synthetic credentials; isolated project/network | ephemeral secrets; resolved config artifact | empty DB + exact scenario pack | captured logs/version/health | DB/API/contract tests | pinned local clients | none | 047–053 | one project name per job; destroy volumes |
| `conformance` | requirement/class evidence | only declared target classes | undeclared classes off; no accidental adapters | base stack, fixture pack, harness | proxy/TLS lane; external validators | test identities/policies; controlled data | class manifest and expected contract | class-specific immutable corpus | full evidence bundle | formal/internal conformance lanes | harness clients | no | 050/051/053 | conformance claim remains result-scoped |
| `interop` | external-client compatibility | approved read/write subsets per scenario | destructive actions isolated; no real targets | base stack, public-origin proxy as needed, fixtures | Webapp, Mobile, Explorer, OS4CSAPI clients | test auth; separate client principals | client/version matrix | discriminating canonical/adversarial packs | raw wire capture and server logs | black-box integration | primary profile purpose | optional one-node degraded cases | 053/056 | external live services never sole oracle |
| `demo-public` | stable public/semi-public demonstration | discovery, read, dynamic simulated data, bounded SSE | admin/metrics/raw diagnostics hidden; unrestricted ingest off; physical commands off | server, DB, migrate, TLS proxy, public fixture/reset control | simulator/publisher on private network; protected docs | HTTPS; allowlisted origins; auth/rate/size limits; least exposure | real deployment secrets; explicit external origin | synthetic public pack; scheduled controlled rebuild | protected ops telemetry and availability alerting | smoke/availability, not CI oracle | browsers/Explorer/public clients | none beyond visibly simulated state | 047/048/049/053/055/056 | no accreditation/conformance claim from uptime |
| `streaming-mqtt-exp` | broker and experimental Part 3 testing | durable core, SSE, optional MQTT 5 outbound | inbound Part 3/commands off unless separately authorized | base stack, broker adapter, MQTT broker | AsyncAPI viewer/test subscriber | authenticated TLS broker; scoped topics | broker trust/credentials as files | event/replay corpus; broker state disposable | delivery/replay metrics | reconnect, duplicate, slow consumer | MQTT clients | transport outage only, not sync | 047/048/050/054/055/056 | disabled by default and draft-labeled |
| `command-sim` | command lifecycle/safety testing | feasibility, admission, simulated dispatch/status/result | no route to real gateways/targets | base stack, isolated gateway simulator | approval test actor | deny by default; explicit grants; egress isolation; full audit | synthetic keys/policies/grants | command/control scenario pack | protected decision/effect telemetry plus audit | security/fault/state-machine tests | test clients only | disconnected/unknown-outcome cases | 047/048/053/055/056 | never public default |
| `ddil-single` | one-node degraded-mode simulation | local reads, authorized local writes, durable queues, assessment | unavailable central dependencies deliberately represented | self-contained server and local DB/cache/evidence | fault proxy; local simulator | locally verifiable bounded credentials/policy | versioned offline package and expiry controls | local fixtures, backlog and recovery checkpoints | local protected diagnostics/audit | service-posture and recovery tests | local clients | primary single-node DDIL profile | 047–055 | simulation label mandatory |
| `sync-two-node` | synchronization/conflict/reconnect verification | two authorities, manifests, exchange, inbox/outcome/conflict | no DB replication as conflict oracle; commands normally off | two isolated base nodes, sync adapter, fault control | independent proxies/collectors | peer identity/trust/policy; separate secrets | node/peer-specific immutable config | divergent fixture histories/tombstones/gaps | per-node correlation and audit | replay/conflict/fault/security tests | sync client/adapter | primary multi-node profile | 047–055 | dedicated stack; not first startup |
| `operational-reference` | document portable hardening contract | deployment-selected accepted capabilities | no convenience default; no claim of HA/accreditation | server image, DB and environment-selected required services | enterprise adapters | TLS, trusted identity, policy, secrets, audit and network controls required | no defaults; externalized validated secret/config sources | controlled migration/restore/readiness | protected telemetry and audit export | release/security/recovery qualification | approved clients | environment-defined | 047–049/054/055/057 | contract/checklist, not a ready production topology |

### 5.2 First implementation decision

`dev-native`, `dev-compose` and `ci-core` form the first implementation tranche. They share the same server/database/migration/fixture contracts. `demo-public` follows only after proxy-origin, security, reset and availability proofs. `conformance` and `interop` then specialize deterministic data and evidence. Broker, simulated command, DDIL and two-node sync profiles are added only when their owning application capability exists. **[P]**

Profile names are not configuration values that may silently weaken policy. The composition root loads a complete typed profile, reports its fingerprint and rejects contradictory settings—for example `demo-public` plus no authentication for write/admin routes, or command dispatch without an installed isolated gateway and grant policy. IDR-SRV-047 owns exact schema and override rules. **[P]**

---

## 6. Supporting Service Inventory

| Service/capability | Base requirement | Profile use | Core or external | Decision and gate |
|---|---|---|---|---|
| `glaux-server` | required | all | Glaux artifact | one image; logical API/worker/admin roles may use commands, not different builds |
| PostgreSQL 18/PostGIS 3.6 candidate | required full profile | all durable profiles | external authority adapter | pin tested digest; private network; ordinary PostgreSQL time-series baseline |
| migration command | required | all persistent profiles | same Glaux image | one-shot before readiness; lock and compatibility evidence |
| standards/schema/profile/vocabulary package | required | all | release artifact consumed by Glaux | local, immutable, digest-addressed; no startup Internet dependency |
| fixture/seed command | required tooling | dev/CI/demo/tests | same Glaux image or test package | explicit scenario manifest; never automatic production seed |
| API documentation UI | optional embedded static asset | dev/demo/interop | Glaux HTTP representation | no separate service required; contract must match enabled runtime |
| reverse proxy/TLS endpoint | not base; required public demo | demo/proxy/interop/operational | external | Caddy is first proof candidate; interface remains product-neutral |
| SSE | required first live slice | base/demo/conformance | Glaux HTTP | no broker; authorized replay cursor and outbox-backed publication |
| MQTT broker | optional/profile-gated | MQTT experimental | external | MQTT 5 adapter only after durable core; broker data rebuildable/disposable |
| NATS/Kafka | deferred | measured internal integration | external | no reference default without workload/integration proof |
| object storage | optional threshold | large-artifact/operational | external adapter | PostgreSQL/safe local artifact baseline first; atomicity and restore proof required |
| TimescaleDB | conditional accelerator | performance profile | database extension | absent from reference baseline until IDR-SRV-027 adoption gates are met |
| identity provider | optional adapter, operationally expected | OIDC/mTLS/reference integration | external | local signed test identity or static test credential only in named test profiles |
| policy engine | optional adapter | policy integration | external | embedded deterministic policy baseline; external failure semantics must be explicit |
| secret manager | deferred adapter | operational | external | Compose secret files for reference; enterprise selection in 047 |
| OTel Collector | optional | observability profile | external | direct console/metrics baseline; Collector gateway is an opt-in proof |
| Prometheus/Grafana | optional tools | diagnostics/demo/performance | external | never required for server correctness or audit; protected from public access |
| simulator/publisher | optional scenario services | demo/test/DDIL | separate Glaux projects/adapters | separate by default; join a named internal network when scenario requires |
| conformance harness/test clients | test-only | conformance/interop | external/test | one-shot runners with immutable version and result bundle |
| fault injection/network control | test-only | DDIL/sync/performance | external/test | isolated test environment; never packaged as operational dependency |
| second server node | test-only initially | sync-two-node | Glaux instance | separate authority/config/database; no shared DB shortcut |

The base Compose topology therefore contains `db`, `migrate` and `server`; `fixtures` is an explicitly targeted tool. It does not contain a broker, IdP, policy server, object store, dashboard stack, proxy, simulator or second node merely because the full server may eventually integrate them. **[E/P]**

---

## 7. Containerization and Image Strategy Findings

### 7.1 Server image

Use one Dockerfile with named multi-stage targets. The builder uses the accepted Rust toolchain and locked dependencies; test/diagnostic targets may carry development tooling; the release stage contains only the server binary, CA roots, timezone data only if proven necessary, static contract assets, migrations and immutable standards/profile packages. Build tools and Cargo caches do not enter the release image. **[D/P]**

The same binary/image exposes explicit commands such as:

- `serve --role api,worker` (initial combined default);
- `migrate status|apply`;
- `contract verify`;
- `fixtures validate|load` for non-operational profiles; and
- `version --json` for evidence capture.

This prevents migration or fixture helper images from drifting from the server schema. Role-specific containers may use the same image later; separate artifacts require a measured reason. **[A/E/P]**

### 7.2 Runtime hardening contract

- fixed non-root UID/GID with owned runtime directories;
- read-only root filesystem compatibility;
- explicit tmpfs for bounded transient files and volumes only for declared durable state;
- no Docker socket, privileged mode, host networking or broad host mounts;
- drop all Linux capabilities, adding a narrowly justified one only through a tested profile;
- `no-new-privileges`; bounded CPU/memory/file descriptors/processes where the runtime supports them;
- loopback or private-network binding by default; only proxy/server ports published;
- graceful SIGTERM shutdown with readiness withdrawal, request drain, worker lease release and bounded exit;
- secrets read from mounted files and never baked into layers, labels, build arguments or provenance; and
- safe version/profile/config fingerprints with no credential values. **[D/A/P]**

The runtime base should initially be a digest-pinned, supported minimal distribution rather than an unconditional `scratch`/distroless choice. A proof must cover dynamic libraries, CA roots, DNS, timezone behavior, allocator/native dependencies, health tooling or an internal probe subcommand, crash diagnostics, vulnerability updates and non-root file ownership. Distroless remains an optimization, not a security synonym. **[E/P]**

### 7.3 Reproducibility and release identity

Release builds shall:

1. use `Cargo.lock`, pinned Rust/MSRV policy and digest-pinned base images;
2. record OCI `source`, `version`, `revision`, `created`, `licenses`, `documentation`, `base.name` and `base.digest` annotations where applicable;
3. publish immutable SemVer/revision tags plus the deployment digest;
4. generate and retain SBOM and build-provenance attestations;
5. scan/review dependencies and image contents under IDR-SRV-044 policy;
6. verify startup, migration, contract and smoke behavior from the pushed digest; and
7. produce a release manifest binding server digest, database image digest, migration head, configuration schema, standards package, OpenAPI/AsyncAPI hashes and fixture-pack hashes. **[D/A/P]**

`latest` may be a convenience pointer but is never an evidence-grade deployment input. A tag does not identify immutable bytes; a digest without source/config/schema/fixture context does not identify the complete tested system. **[D/E/P]**

### 7.4 Platform targets

The initial verified target is `linux/amd64`. Glaux's Rust image should be built in a way that can add `linux/arm64`, but the release must not advertise a working multi-platform stack until every required dependency does. Current `postgis/postgis:18-3.6` documentation lists `amd64`; ARM readiness therefore requires a vetted PostGIS build/source, extension/version parity, migration, spatial query, performance, backup/restore and upgrade tests. QEMU build success alone is not runtime qualification. **[D/E/P]**

### 7.5 Release bundle and version relationship

Each release candidate should contain or identify:

- the server OCI digest, platform manifest, OCI annotations, SBOM and provenance;
- digest-pinned base Compose file and supported profile overlays;
- configuration schema, nonsecret sample profiles and secret-generation instructions;
- checked migration bundle and compatible schema range;
- standards/schema/profile/vocabulary packages and hashes;
- deployed OpenAPI plus capability-derived experimental AsyncAPI when enabled;
- separately versioned public/demo/test fixture manifests and hashes;
- smoke/verification commands or an equivalent test-runner artifact;
- upgrade/restore compatibility metadata; and
- a deployment README defining prerequisites, bootstrap, health, reset, evidence collection and non-production caveats.

The server release version, public API/standard profile version, database migration head, configuration-schema version and fixture version are related but not collapsed into one number. A Git tag selects a tested manifest; the manifest records every component identity. Patch releases may update image dependencies without changing the public API, while an API/profile change may require explicit compatibility/migration treatment. **[D/E/P]**

---

## 8. Docker Compose / Local Development Findings

### 8.1 Orchestration decision

| Option | Strength | Cost/risk | Disposition |
|---|---|---|---|
| native processes only | fastest debugging and fewest container assumptions | weak onboarding/CI parity; host dependency drift | retain as `dev-native`, not the distributable reference |
| Docker Compose | reproducible multi-service lifecycle, profiles, health/dependency conditions and broad developer/CI availability | single-host scope; implementation/version differences; not HA | select as first reference environment |
| Dev Container | consistent editor/toolchain onboarding | editor/runtime coupling; does not replace system deployment contract | optional convenience after base Compose works |
| Kubernetes/Helm | scheduling, rollout and ecosystem for larger operations | premature platform/HA/secret/network complexity and local burden | defer; use only to test portable contracts later |
| native service manager/VM/appliance | potentially suitable for constrained or controlled operations | environment-specific packaging/upgrade/identity design | future operational/edge mapping, not first reference |

Compose is selected because the reference goal is repeatable build/test/demo assembly on one host, not because Compose proves production suitability. No Compose-only behavior may become a hidden server semantic. **[D/E/P]**

### 8.2 Compose structure

Use a standards-compliant `compose.yaml` as the base and small, explicit overlays only when values genuinely differ by environment. Core services have no Compose profile so they start together; optional services use profiles such as `proxy`, `observability`, `mqtt`, `simulator`, `conformance`, `command-sim` and `ddil`. Compose documentation explicitly treats unprofiled services as core and supports targeted one-shot profiled services. **[D/P]**

The base dependency chain is:

`db healthy -> migrate completed successfully -> server started -> server ready`

Compose startup order alone is insufficient; official documentation states that a running container is not necessarily ready. Database health, the migration job and server readiness must be separate conditions. The application must also retry transient connections because Compose recreation retains service names but changes IPs and closes old connections. **[D/P]**

### 8.3 Networks and ports

- `data` network: server/migrate to database; database has no host port outside explicit developer overlays.
- `edge` network: proxy to server; server is not directly published in public profiles.
- `integration` networks: simulator/publisher/broker/client access only for the named scenario.
- service discovery uses Compose names, never container IPs.
- every CI job uses a unique Compose project name; no fixed `container_name` values.
- outbound access is absent or constrained for command simulations and deterministic tests.

Separate networks are containment, not authorization. Application identity, policy and source trust remain mandatory. **[D/A/P]**

### 8.4 Developer paths

`dev-native` starts the database (and optionally a proxy) in Compose, then runs `cargo run ... serve` on the host. This gives the shortest Rust rebuild/debug loop and exposes platform differences early. `dev-compose` builds/runs the release-like container. Compose Watch may be offered as a convenience, but Rust rebuild cost, writable-image requirements and cross-platform native artifacts must be measured before it becomes the recommended loop. **[D/E/P]**

Both paths use the same:

- typed configuration schema and sample values;
- migration head and database compatibility check;
- standards/profile artifact digest;
- seed/fixture manifest;
- smoke requests and expected semantic assertions; and
- public-origin/link tests.

Local no-auth is allowed only in an unmistakably named `unsafe-local` mode that refuses non-loopback binding, commands, unrestricted ingestion and admin exposure. The preferred default still uses synthetic signed identities or bounded test credentials so authorization paths are exercised. **[A/P]**

### 8.5 Bootstrap, reset and teardown

Normal start never destroys data, reruns seed blindly or rewrites a migration history. `docker compose down` stops the project while retaining named volumes. A destructive reset is a separate command requiring the project/profile name, displaying target volumes, rejecting non-development profiles and then recreating/migrating/loading the requested fixture pack. CI may always use `down --volumes` because its uniquely named project and disposable state are established in advance. **[E/P]**

Developer bootstrap should be one documented command or task runner target, but each internal phase remains visible and individually diagnosable. Hidden “magic” startup that conflates image build, database initialization, migration, seed and serve makes failures irreproducible. **[E/P]**

---

## 9. CI and Conformance Runtime Findings

### 9.1 CI lane model

| Lane | Runtime | State model | Gate/evidence |
|---|---|---|---|
| format/lint/unit | host runner; no services | no database | blocking; compiler/lint/unit results |
| architecture/contract | host plus generated artifacts | no durable services | blocking; dependency rules, OpenAPI/schema/route parity |
| database integration | PostgreSQL/PostGIS service | fresh database per job/suite | blocking; migration/query/transaction/concurrency evidence |
| image/base-stack smoke | pushed or locally exported image plus DB | unique disposable Compose project | blocking release candidate; digest/start/readiness/smoke logs |
| conformance | base stack plus exact harness/fixtures | immutable scenario pack | class-scoped result bundle; blocking only for claimed classes |
| interoperability | base/proxy stack plus pinned clients | isolated fixture state | scheduled/release or targeted merge gate |
| streaming/performance/security | named profile | purpose-specific reset | scheduled/release; thresholds owned by 054/055 |
| live ecosystem observation | external deployments/clients | uncontrolled | non-blocking dated evidence only |

Unit tests must not wait for containers. Tests that claim SQL, PostGIS, migrations, locks or concurrency behavior use a real supported database, not an in-memory substitute. Black-box HTTP tests connect through a real TCP listener; proxy-origin tests connect from outside the proxy network. **[A/P]**

### 9.2 Deterministic CI sequence

1. resolve and record source revision, dependency lock and tool versions;
2. build once and record the candidate image digest/SBOM/provenance;
3. create a unique Compose project and ephemeral secrets;
4. start the pinned database and wait for database health;
5. run the same-image migration job and verify the exact migration head;
6. validate then load the exact fixture/scenario manifest;
7. start the server with the target capability/profile manifest;
8. wait on startup/readiness with a bounded diagnostic timeout;
9. verify landing, conformance, API definition, schema, canonical links and profile identity before the suite;
10. execute semantic tests, preserving raw request/response evidence where required;
11. capture `docker compose config`, image digests, server version, migration/profile/fixture hashes, health snapshots, logs and test results; and
12. run cleanup in an unconditional finalizer and remove the uniquely scoped volumes/networks. **[D/A/E/P]**

Retries must address known infrastructure races, not mask test assertions. A failed migration, readiness timeout, fixture mismatch or semantic test is retained as evidence before teardown. CI never fetches mutable schemas, vocabularies or public demo data during a deterministic lane. **[A/P]**

### 9.3 Conformance separation

The conformance profile enables exactly the classes under test and derives landing/conformance/OpenAPI behavior from the effective capability registry. Each result binds standard/version, requirement class, server digest, configuration/profile digest, database/migration version, fixture pack, harness version, timestamps and raw evidence. A green container health check, route existence or third-party client success is not an OGC conformance result. **[N/A/P]**

Harnesses run as one-shot external clients, not as code linked into the server. They receive only their test principal and public endpoint. Destructive suites receive an isolated database/project; parallel cases use independent schemas/databases or proven cleanup boundaries. IDR-SRV-050 finalizes the official/internal harness allocation. **[P]**

---

## 10. Public Demo Runtime Findings

### 10.1 Recommended shape

`Internet/client -> TLS reverse proxy -> glaux-server -> PostgreSQL/PostGIS`

The simulator/publisher, when enabled, resides on a non-public integration network and authenticates as a narrowly scoped synthetic source. The database, migration/admin commands, metrics, tracing receiver, detailed health, management UI and broker administration are not publicly exposed. The server exposes only the accepted API, deliberately selected documentation, and shallow public health/availability behavior. **[A/E/P]**

Caddy 2.11.3 is the first reference proxy proof candidate because its official documentation supplies automatic HTTPS and conservative default handling of incoming `X-Forwarded-*` values. This is a convenience choice for the reference bundle, not an enterprise reverse-proxy selection. The server contract remains compatible with another proxy that supplies equivalent TLS, header, timeout, body-limit, streaming and trust behavior. **[D/E/P]**

### 10.2 Capability posture

- Enable landing, conformance, API definition, collections/resources, synthetic dynamic data and bounded SSE when stable.
- Enable writes only for specifically authenticated demonstration principals and reset-safe resource families; a read-mostly demo is the default.
- Keep source registration, policy administration, audit access, database/admin operations and detailed diagnostics private.
- Disable physical command dispatch. A command demonstration uses `command-sim`, an isolated simulator-only network, synthetic targets and prominent UI/API documentation.
- Keep experimental MQTT/Part 3 on a separate named endpoint/profile with its exact draft/version/deviation statement.
- Do not expose raw payloads, secrets, internal identifiers/topology, unbounded queries, unrestricted CORS, or high-cardinality metrics.

### 10.3 Public data and reset

The demo pack contains public, synthetic, license/provenance-reviewed data with stable identities and enough variation to exercise spatial, temporal, relationship, SensorML/SWE, paging, negotiation and dynamic behavior. A reset creates a new isolated state from the signed/digested pack, smoke-tests it, then switches or restarts the demo under a bounded maintenance procedure. It does not run `DELETE` against an unknown database or depend on an unversioned simulator stream to reconstruct truth. **[A/E/P]**

Availability monitoring must distinguish proxy/TLS, server liveness, readiness/database, fixture/version drift and external client traversal. Prior implementation studies showed that an advertised demo can be down or emit unusable internal links; therefore the demo has an owner, status notice, build/profile identity, synthetic smoke path and expiration/rebuild procedure. **[I/T/P]**

---

## 11. Tactical-Edge/DDIL Simulation Findings

### 11.1 Locality requirements

A disconnected-capable reference node needs locally available:

- the server image/binary and compatible database;
- approved standards, schema, profile and vocabulary packages;
- credential-verification and policy material with explicit issuer, scope, validity and offline authority;
- canonical data, source/admission state, audit, outbox/replay and synchronization evidence;
- bounded source buffers and durable worker state;
- configuration and time/evidence-quality status; and
- local operator diagnostics that do not require a central telemetry backend. **[A/P]**

Optional central IdP, policy service, broker, object store, telemetry gateway or peer loss must feed the accepted per-operation service-posture evaluation. Readiness must not collapse solely because an optional central dependency is unreachable; nor may a green process check imply that current authority, data or time evidence exists. **[A/P]**

### 11.2 Simulation stages

| Stage | Topology | Faults | Required assertions |
|---|---|---|---|
| local dependency loss | one server/DB plus controlled adapters | IdP/policy/broker/telemetry unavailable | accepted cached/offline authority, honest diagnostics, no invented freshness |
| source intermittency | one node plus simulator/fault control | delay, duplicate, reorder, disconnect, burst | inbox/idempotency, source health, backpressure, recovery |
| outbound backlog | one node | subscriber/broker/peer outage | canonical commits persist; outbox/replay resumes or exposes gap |
| restart/power-loss approximation | one node | forced process/container stop at durable boundaries | recovery from database state; no memory-only truth |
| two-node partition | distinct nodes/databases | partition, divergent authorized activity, reconnect | envelope validation, gap detection, conflict/quarantine/outcome evidence |
| stale authority/time | one/two nodes | expired credentials/policy, uncertain clock | safe per-operation posture; command restrictions; audit evidence |

Fault mechanisms may include a network proxy, container pause/stop, firewall rule or application test adapter, but the scenario must state exactly what was impaired. “Offline” is not one Boolean and simulated packet loss is not proof of a tactical radio environment. **[A/E/P/X]**

### 11.3 Edge packaging boundary

Compose remains suitable for repeatable lab simulation, not a conclusion that Docker/Compose is the operational edge supervisor. Native service managers, immutable appliances or another orchestrator may be more suitable later. They must preserve the same local data, health, graceful-shutdown, migration, secret, audit and evidence contracts. Hardware architectures, storage endurance, time sources, bandwidth, power, physical security and update custody require measured environment-specific work. **[E/P/X]**

---

## 12. Network, Proxy, TLS, CORS, and Generated URL/Link Findings

### 12.1 Reverse-proxy option comparison

| Option | Advantages | Costs/risks | Disposition |
|---|---|---|---|
| no proxy/application HTTP | smallest local/CI path | no public TLS boundary; direct forwarded-header exposure | local/CI only |
| Caddy | concise configuration, automatic HTTPS, conservative incoming forwarded-header defaults | product-specific certificate/storage behavior; limits/SSE/base path still require proof | first reference demo candidate |
| NGINX | mature, explicit and flexible proxy controls | more manual TLS/configuration; safe forwarded-header and streaming behavior must be authored | supported alternative, not baseline selection |
| Traefik | Compose/container discovery and dynamic routing | Docker provider/socket and label complexity can expand trust surface | conditional integration proof; no default Docker-socket grant |
| enterprise/managed proxy | may satisfy environment PKI/WAF/load-balancing controls | provider-specific behavior and ownership | operational mapping outside this report |

The server's explicit public-origin and trusted-proxy contract is authoritative across all options. The proxy never supplies application authority merely by adding headers. **[D/A/P]**

### 12.2 Profile network rules

| Profile class | Client transport | Backend transport | Public origin | Forwarded headers | CORS |
|---|---|---|---|---|---|
| local native/Compose | HTTP on loopback allowed | local/private HTTP | explicit localhost URL/port | ignored unless named local proxy lane | exact local Webapp origins or disabled |
| CI core | private HTTP | private HTTP | deterministic internal test origin | normally ignored | test-specific allowlist |
| proxy/interop | HTTPS at proxy | private HTTP or tested TLS | exact external test URL/base path | trusted only from proxy network/CIDR | exact client origins |
| public demo | HTTPS required | private network | exact deployed HTTPS origin | proxy allowlist; reject/directly ignore others | explicit public Webapp/Explorer origins; no wildcard credentials |
| operational reference | environment-approved TLS/mTLS | environment decision | validated configured origin | explicit trusted proxies/hops | least-privilege configured origins |

### 12.3 Origin and link contract

The server accepts a validated `public_base_url` including optional base path as the authoritative origin for canonical links, OpenAPI `servers`, redirects, SSE/replay links and authentication callback/audience scopes. If deployment policy permits deriving components from `Forwarded` or `X-Forwarded-*`, the direct peer must match an explicit trusted proxy list and the header chain must be parsed under a documented single/multi-hop rule. Untrusted forwarding headers are ignored or rejected, never reflected. **[D/A/P]**

The startup probe validates scheme, host, port, base path, allowed hosts and route-catalog compatibility. External smoke tests traverse from landing to conformance, API definition, collections/items, schemas and related resources and assert that no internal service name, container port or wrong scheme escapes. This makes the prior OSH/pygeoapi proxy failure a permanent regression scenario. **[I/T/P]**

### 12.4 Proxy behavior proof

The proxy test matrix covers:

- host and base-path preservation;
- standardized `Forwarded` plus supported `X-Forwarded-*` behavior;
- request ID/tracing headers without trusting caller-generated security context;
- maximum header/body sizes and request timeouts;
- SSE buffering disabled and idle/keepalive timeouts sufficient for the declared stream contract;
- uploads/large SensorML/SWE bounds and slow-client behavior;
- HTTP-to-HTTPS redirects and HSTS only where correct for the environment;
- client address interpretation without spoofed header trust; and
- graceful deployment/restart without emitting mixed origins. **[D/A/P]**

CORS is a browser permission mechanism, not authentication. Configuration enumerates origins, methods and headers by profile; it never uses a permissive wildcard with credentials and does not make ingestion/admin/command routes safe merely because a browser origin is blocked. Non-browser clients remain subject to normal authentication, authorization and policy. **[A/P]**

---

## 13. Configuration and Secret-Handling Implications

### 13.1 Configuration classes

| Class | Examples | Reference delivery | Rule |
|---|---|---|---|
| nonsecret immutable profile | capabilities, limits, roles, base URL, adapter selection | typed mounted file plus narrow env overrides | schema-validated and fingerprinted |
| secret | DB password, signing key, client secret, broker credential | mounted secret file | never committed, logged, labeled or embedded |
| trust material | CA roots, issuer keys, peer certificates, policy package signatures | read-only mounted files/package | version, issuer, validity and reload policy explicit |
| release artifact | schemas, vocabularies, OpenAPI, migrations | image/release bundle | digest-bound to release manifest |
| mutable operational metadata | registered sources, policy bindings, grants | authorized database/admin workflow | audited domain state, not env configuration |
| test scenario | fixture manifest, synthetic principals, fault plan | versioned test pack | forbidden in operational profile unless explicitly imported |

Environment variables are appropriate for simple nonsecret deployment coordinates and paths; Compose `.env` interpolation is not a secret store. Docker documentation warns about secret exposure in environment variables and supports per-service secret files. Glaux should implement a general `*_FILE` or typed secret-reference pattern, enforce mutual exclusivity with direct values, and prohibit printing resolved secrets. **[D/P]**

### 13.2 Profile safety

- No shipped default credential works in a public/non-loopback profile.
- Missing required secret/trust material causes startup failure before readiness.
- Example secret files contain obvious nonfunctional placeholders or are generated locally and ignored by Git.
- CI creates unique short-lived synthetic credentials and retains no value in artifacts.
- Secret rotation supports overlap/atomic reload where the adapter permits it; otherwise it is an explicit restart operation.
- Configuration precedence is deterministic and observable without exposing values.
- Unknown keys, invalid combinations and unsafe profile transitions fail closed.
- The effective security/capability profile is visible to protected diagnostics and release evidence.

IDR-SRV-047 must define the exact schema, precedence, validation, reload, redaction, secret-provider adapters and environment mapping. This report fixes the deployment inputs and safety invariants, not those field names. **[P]**

---

## 14. Database Bootstrap, Migration, Backup, Restore, Seed, and Fixture Findings

### 14.1 Separate lifecycle operations

| Operation | Input/state | Output/evidence | Must not do |
|---|---|---|---|
| initialize engine | empty Postgres data directory and admin secret | running database/cluster | pretend an existing volume received new init settings |
| migrate | compatible DB plus exact migration bundle | migration head/history and success receipt | seed examples or start serving early |
| bootstrap required reference data | migrated empty/compatible DB plus release packages | installed immutable package records/digests | overwrite changed authoritative data silently |
| seed/fixture load | explicit non-operational profile and manifest | scenario receipt, IDs, hashes | run automatically in operational profile |
| reset | verified scoped disposable target | clean migrated/seeded state | target an unknown/shared volume/database |
| backup | known release/schema plus data/artifact inventory | protected backup manifest and digest | treat volume persistence as a backup |
| restore | empty isolated target plus verified backup | validated restored state and report | overwrite the source or advertise readiness before checks |

The official PostgreSQL/PostGIS images apply initialization settings and `/docker-entrypoint-initdb.d` scripts only for an empty data directory. Glaux therefore may use the image entrypoint to create the engine/database/role, but all application schema evolution runs through versioned Glaux migrations. **[D/P]**

### 14.2 Migration strategy at deployment level

Run an explicit same-image migration job before server readiness. It acquires a migration lock, verifies database engine/extension and current schema compatibility, applies ordered checksummed migrations, records the result, and exits. Multiple server replicas never race startup migrations. The server checks the accepted schema range and refuses incompatible newer/older state. Roll-forward/rollback, expand/contract upgrades and irreversible changes remain IDR-SRV-049. **[A/E/P]**

### 14.3 Backup/restore reference proof

For reference environments, provide a logical PostgreSQL export/restore example using compatible PostgreSQL tools, plus an artifact/inventory manifest for any external exact-content store. PostgreSQL documents `pg_dump` as a consistent export and custom/directory formats as flexible `pg_restore` inputs, while warning that it is not automatically the right regular-production backup system. Therefore the reference proof demonstrates portability and restore verification; it does not define production RPO/RTO, WAL archiving, HA or legal retention. **[D/P/X]**

Every release candidate and migration change should restore a representative prior backup into an isolated target, migrate it forward, verify PostGIS/extensions, schema head, canonical counts/invariants, artifact digests, audit/outbox state and API smoke behavior. A backup is not accepted until restore is tested. Database named volumes improve persistence across container recreation but are not backup evidence. **[D/A/P]**

### 14.4 Fixtures and packages

Fixture manifests include pack/version/digest, source provenance/license, applicable standard/profile, required capabilities, deterministic clock/ID policy, load order, expected counts/invariants, intentional invalid cases and cleanup/reset class. Schema/profile/vocabulary packages use a separate immutable registry and cannot be replaced by fixtures. Demo, conformance, performance, security and sync packs remain different because their data and cleanup guarantees differ. **[A/P]**

---

## 15. Observability, Logging, Metrics, Tracing, Health, and Readiness Findings

### 15.1 Endpoint semantics

| Signal/endpoint | Meaning | Dependency scope | Exposure |
|---|---|---|---|
| liveness | process/event loop can answer; no deep network query | none or strictly local | shallow endpoint may be unauthenticated on management listener |
| startup | configuration/packages loaded and internal initialization completed | local initialization | orchestrator/internal only |
| readiness | instance may accept its advertised traffic role | mandatory dependencies such as DB/schema compatibility; role-specific | orchestrator/internal, bounded summary |
| dependency detail | current database, worker, broker, cache, source, sync, policy/identity evidence | all monitored dependencies with required/optional classification | protected admin only |
| service posture | accepted DDIL per-operation availability/freshness/authority assessment | domain evidence | authorized diagnostic/API projection only |
| metrics | operational counters/histograms/gauges | instrumentation | protected scrape/OTLP path |
| audit | authoritative accountability event | accepted E0–E5 paths | separate protected store/export, never health/log substitute |

Kubernetes documentation distinguishes startup, liveness and readiness because they drive different actions; Compose health can express a container's healthy state but does not create those semantics. Glaux defines all three independently and maps them to each platform. Liveness never performs a query whose transient failure creates a restart storm. Readiness excludes the node from normal traffic when a mandatory dependency or schema invariant fails, but optional broker/central service loss produces the accepted degraded posture instead of indiscriminate unready state. **[D/A/E/P]**

### 15.2 Default and optional telemetry

The base stack emits structured, redacted logs to stdout/stderr with version/profile, safe correlation IDs, event class and bounded outcome fields. It can expose protected application metrics and export OTLP when configured. It does not require Prometheus, Grafana or an OTel Collector to function. An `observability` Compose profile may add one pinned Collector and local backends/dashboards for development, demo diagnostics and performance tests. **[D/A/P]**

The Collector is an adapter/gateway, not a durable audit service. Official OpenTelemetry guidance notes that agent/gateway patterns add complexity and should be used when their processing, isolation or scale benefits are needed. A single optional local gateway is enough for the reference proof; enterprise agent/gateway/HA decisions remain IDR-SRV-048. **[D/P]**

### 15.3 Operational evidence

Startup logs and protected diagnostics record:

- server/source revision and image digest where supplied;
- active profile/capability/configuration fingerprints;
- migration and standards-package versions;
- external public origin and trusted-proxy mode without secrets;
- enabled adapters and mandatory/optional dependency classification;
- first-ready and readiness-transition reasons; and
- graceful-shutdown phase/outcome.

Metrics labels never contain raw resource IDs, UIDs, subjects, tokens, URLs with sensitive queries or payloads. Raw SensorML/SWE/Observation/Command/policy data is not logged. IDR-SRV-048 owns final instruments, cardinality, sampling, retention and backend protection. **[A/P]**

---

## 16. Publisher, Simulator, Webapp/Mobile, CSAPI Explorer, and External-Client Integration Findings

| Integration | Attachment model | Identity/trust | Data/reset | Required tests |
|---|---|---|---|---|
| Glaux Publisher | separate process/project; joins integration network or uses public endpoint | registered publisher/source credential; least privilege | replayable publisher scenario and admission receipts | duplicate/order/retry/backpressure/source revocation |
| Glaux Simulator | separate by default; simulator profile | unmistakable synthetic source/tenant | deterministic seed/clock/checkpoint; disposable | pause/resume/fault/restart/cleanup and provenance |
| Webapp | browser through public origin | OIDC/test principal; CORS allowlist | no privileged fixture shortcut | discovery/navigation/negotiation/paging/SSE/security |
| Mobile | external client path, including intermittent scenarios | device/client identity subject to same app policy | offline client data is not server truth | reconnect/cursor/resnapshot/token expiry/link traversal |
| CSAPI Explorer | external browser/client against proxy | public/read or test principal | canonical demo/conformance pack | landing-to-resource traversal and schema/media handling |
| OS4CSAPI clients | pinned test container/process | test principal per suite | immutable raw-wire fixture correlation | semantic assertions across supported workflows |
| other CSAPI servers | separate observational or sync stack | peer/source trust distinct from user auth | source-qualified records and sync fixtures | identity/link/representation/drift/conflict cases |

Publisher and simulator are not hidden child processes of the server. Keeping them separate proves the accepted source boundary, prevents test controls from leaking into production code and permits independent fault injection. A convenience umbrella Compose file may assemble repositories by pinned image digest; each project retains its own release and security identity. **[A/E/P]**

External clients test the same public route/proxy/link graph that users receive. Tests must not inject data directly into database tables and then claim publisher/API interoperability. Database fixtures may establish preconditions, but API/source workflows and expected evidence remain explicit. **[A/P]**

---

## 17. Streaming, Command/Control, and Federation/Synchronization Runtime Implications

### 17.1 Capability runtime matrix

| Area | First runtime | Optional/full-scope runtime | Hard boundary |
|---|---|---|---|
| live publication | HTTP query/change feed + authenticated SSE; DB outbox/replay | MQTT 5 experimental adapter/broker; later measured transports | broker state/ack is not canonical truth or replay proof |
| ingestion | HTTP/publisher adapter through common admission | MQTT/source adapters | transport authentication is not source authority |
| commands | lifecycle and feasibility may exist; dispatch disabled | isolated simulator gateway; later operational gateway | no physical egress without profile, grant, safety, fence and audit |
| experimental Part 3 | disabled | exact versioned outbound MQTT profile | no approved conformance claim; inbound commands/data separately unauthorized |
| federation/sync | one server | dedicated two-node sync test topology | no shared DB, arrival-wins or replication-as-conflict-policy shortcut |
| observability | local logs/metrics/optional OTLP | Collector/backends | telemetry loss does not rewrite domain/audit outcome |

### 17.2 Broker lifecycle

The MQTT profile treats broker subscriptions, retained messages and queues as transport state that may be reset and rebuilt from authorized canonical snapshot/replay where the declared contract allows. Broker reset never deletes the server outbox/log or creates proof that every consumer processed an event. Broker credentials, TLS, topic ACLs, quotas and storage are profile inputs; the adapter still performs application authorization and policy filtering. **[A/P]**

Slow consumers, disconnect, duplicate delivery, cursor expiry, backfill, policy change and broker loss are test scenarios. SSE and MQTT must converge on the same authorized event/replay semantics while retaining transport-specific envelopes and acknowledgements. **[A/P]**

### 17.3 Command isolation

`command-sim` uses a dedicated internal network with only the server and simulator gateway. It has no host route, credential or target configuration that can reach an operational system. Every target is synthetic and allowlisted; command authority grants, safety policies and dispatch tickets are generated for the scenario and expire. CI/public demo commands cannot be made real by changing only an endpoint string. **[A/E/P]**

The operational-reference profile intentionally provides no usable command default. Enabling a gateway requires an installed reviewed adapter, authenticated target identity, explicit egress allowlist, current command grant/policy/safety evidence, mandatory audit availability, reconciliation behavior and environment approval outside this report. **[A/P/X]**

### 17.4 Synchronization isolation

The two-node stack gives each node a separate database, node identity, public origin, policy package, audit stream and fixture history. The only exchange occurs through the sync adapter under controllable faults. Shared volumes, shared database tables or direct replication may be separate infrastructure experiments, but they cannot validate `SyncEnvelopeV1`, candidate classification, inbox/effect atomicity, conflict/quarantine, tombstone, gap or policy behavior. **[A/P]**

---

## 18. Downstream Topic Handoff Matrix

| Topic | Fixed input from IDR-SRV-046 | Topic-owned decisions/proofs |
|---|---|---|
| IDR-SRV-047 Configuration, Secrets, Environment | typed complete profiles; config/secret/trust/release/test classes; secret-file inputs; no public defaults; fail-closed combinations; effective fingerprint | exact schema, names, precedence, reload, secret providers, rotation and redaction |
| IDR-SRV-048 Observability, Logs, Metrics, Health | three distinct startup/liveness/readiness semantics; protected dependency detail; base structured logs; optional Collector/backends; audit separation | log/event schema, instruments, cardinality, sampling, exporters, retention, alerts, dashboards |
| IDR-SRV-049 Migration, Upgrade, Backup, Restore | same-image one-shot migration; schema compatibility gate; no init-script migration; isolated restore proof; release manifest | migration framework/process, expand-contract, rollback, RPO/RTO, WAL/physical/logical backup, disaster recovery |
| IDR-SRV-050 Conformance Harness | exact capability profile; external one-shot harness; immutable stack/fixture/result identity; no health-as-conformance shortcut | requirement traceability, ATS integration, verdict/evidence format and official claim process |
| IDR-SRV-051 Contract/Schema Verification | deployment-specific OpenAPI/schema/profile artifacts bound to route/capability manifest | generation/lint/bundle/diff/parity toolchain and release gates |
| IDR-SRV-052 Rust TDD Architecture | native and Compose developer paths; real DB/TCP/proxy lanes; same-image admin commands; architecture and image tests | test seams, crate-level suites, fakes, builders, CI implementation and coverage policy |
| IDR-SRV-053 Fixtures/Golden/Scenarios | distinct dev/demo/conformance/performance/security/sync manifests; provenance/digest/load/reset fields | complete corpus, storage layout, builders, licensing, golden-update governance |
| IDR-SRV-054 Performance/Load/Stress/Streaming | pinned profile/digest, resource limits, SSE/MQTT lanes, slow consumer/backlog/reconnect and amd64-first baseline | workloads, thresholds, capacity numbers, soak/fault methods and ARM qualification performance |
| IDR-SRV-055 Security/Auth/Command Tests | profile boundary tests; public demo controls; secrets-as-files; proxy spoofing; isolated simulated gateway; deny-default egress | complete abuse/adversarial matrix, credential/policy/command fixtures and security release criteria |
| IDR-SRV-056 External CSAPI Interoperability | public-proxy path; pinned external clients; deterministic local server plus separate live observation; raw-wire evidence | client/version matrix, semantic assertions, expected compatibility and publication format |
| IDR-SRV-057 Final Synthesis | Compose reference boundary, eleven profiles, service inventory, release evidence and explicit non-accreditation caveat | reconcile final roadmap, decisions, residual risk and implementation authorization |

### 18.1 Implementation proof backlog

1. **Base image proof:** multi-stage release image starts non-root with read-only rootfs, required tmpfs/mounts, dropped capabilities and graceful shutdown.
2. **Compose lifecycle proof:** healthy DB, completed migration, exact readiness, retained normal volume and explicitly scoped destructive reset.
3. **Origin proof:** externally traverse all canonical links/OpenAPI servers behind proxy and base path; spoof forwarded headers from an untrusted peer.
4. **Release-evidence proof:** rebuild/push by digest with OCI annotations, SBOM, provenance and complete version manifest; scan/test the pushed artifact.
5. **Restore proof:** export representative prior data/artifacts, restore in isolation, migrate, validate and smoke through the API.
6. **CI isolation proof:** run parallel uniquely named projects with no port, volume, credential or fixture collision and unconditional evidence-preserving teardown.
7. **Demo proof:** TLS, allowlisted origins, bounded queries/SSE, protected diagnostics, synthetic reset and no command/ingest/admin escape.
8. **Telemetry/health proof:** distinguish startup/liveness/readiness/optional dependency degradation and retain local diagnostics during Collector loss.
9. **DDIL proof:** single-node dependency/source/backlog/restart scenarios preserve accepted service-posture and audit behavior.
10. **Two-node proof:** partition/reconnect with distinct stores exercises gaps, duplicates, conflicts, quarantine, tombstones and policy outcomes.
11. **Architecture proof:** verify logical API/worker roles together first, then demonstrate that same-image role separation does not change transaction/work semantics.
12. **Platform proof:** qualify `linux/arm64` only with a vetted PostgreSQL/PostGIS runtime and full spatial/migration/restore/performance suite.

No proof result is claimed by this research. These are implementation gates and later-topic inputs. **[P]**

---

## 19. Recommendations

1. **Adopt Docker Compose as the first reference-environment format, not the production orchestrator.** Keep portable runtime contracts explicit. Priority: High.
2. **Ship one server OCI image and one required PostgreSQL/PostGIS dependency.** Use the same image for server and bounded administrative commands. Priority: High.
3. **Implement `dev-native`, `dev-compose` and `ci-core` first.** Add public demo only after proxy/security/reset proofs; add specialized profiles with their capabilities. Priority: High.
4. **Keep the base stack broker-, IdP-, external-policy-, object-store- and dashboard-free.** Admit each through a named profile and failure contract. Priority: High.
5. **Run migrations explicitly before readiness.** Never use PostgreSQL first-volume initialization as application migration or seed semantics. Priority: High.
6. **Treat public origin and proxy trust as API correctness.** Configure one external base, trust only named proxies and black-box traverse the deployed graph. Priority: High.
7. **Make public demo secure and non-actuating.** Require HTTPS, constrained routes/data and disable physical command dispatch; isolate any simulated commands. Priority: High.
8. **Use immutable deployment evidence.** Pin digests, emit OCI annotations/SBOM/provenance, and bind image/config/migration/standards/fixture versions in one manifest. Priority: High.
9. **Separate health semantics and optional degradation.** Liveness is shallow, readiness is role/mandatory-dependency aware, detailed status is protected, and DDIL posture remains domain evidence. Priority: High.
10. **Make CI disposable and diagnostic.** Unique project/state, empty DB, explicit migrate/load, bounded readiness, raw results/logs and unconditional scoped teardown. Priority: High.
11. **Treat DDIL and synchronization as dedicated simulations.** Preserve local authority/evidence and use truly separate node stores; do not infer operational readiness. Priority: Medium.
12. **Target `linux/amd64` first and gate ARM.** Resolve the documented PostGIS image architecture constraint before advertising an ARM stack. Priority: High.
13. **Prove restore, not just backup creation.** Keep reference logical export/restore bounded while IDR-SRV-049 designs production continuity. Priority: High.
14. **Keep implementation lessons as regression inputs.** Test demo availability, certificate/origin, schema/route parity, default credentials, CORS and fixture drift without copying another server's topology. Priority: Medium.

---

## 20. Risks, Constraints, and Open Questions

### 20.1 Risk register

| Risk | Consequence | Mitigation / gate | Owner |
|---|---|---|---|
| optional services leak into base | slow, fragile developer/CI startup | three-service base; profiles; dependency audit | implementation/052 |
| mutable tags or online schemas | irreproducible result/supply-chain drift | digest pins; local packages; release manifest | 044/049/051 |
| initialization mistaken for migration | existing volumes silently stale | same-image migration job and schema gate | 049 |
| generated links expose internal origin | unusable clients and security-scope errors | explicit public origin, proxy trust and external traversal | 047/056 |
| public demo convenience weakens security | exposed writes/admin/commands/data | dedicated demo profile, TLS/auth/limits, egress isolation | 047/055 |
| deep liveness causes restart loops | cascading outage/data interruption | shallow liveness; separate readiness/detail | 048 |
| optional central outage marks edge dead | loss of useful local service | per-operation posture and required/optional dependency model | 042/048 |
| named volume treated as backup | unrecoverable loss/corruption | tested isolated export/restore with manifest | 049 |
| telemetry mistaken for audit | missing accountability evidence | independent audit authority/store/export | 041/048 |
| broker state mistaken for replay | silent loss/duplicate/authorization error | DB outbox/log, opaque cursors, resnapshot contract | 035/054 |
| simulator can reach real target | unintended physical effect | isolated network, synthetic target allowlist, absent operational credentials | 038/055 |
| shared DB hides sync faults | false confidence in conflict handling | distinct node databases/identities and controlled exchange | 043/055 |
| Compose behavior drifts | profile/condition/secret incompatibility | version floor, `compose config`, release smoke on supported hosts | implementation/052 |
| PostGIS image lacks ARM | edge deployment promise fails | amd64 first; vetted ARM dependency and complete proof | 049/054 |
| minimal image blocks incident/debug needs | poor diagnosability or TLS/runtime failure | reviewed runtime base and diagnostic proof before distroless | 044/048 |
| fixture reset targets wrong state | destructive data loss | explicit profile/project/volume verification and operational refusal | 049/053 |

### 20.2 Constraints

- PostgreSQL/PostGIS is the required full-profile authority; replacing it is outside this topic.
- The modular monolith and one initial release artifact remain accepted.
- No network/service arrangement bypasses application authorization, policy, validation, command safety or audit.
- Reference configuration contains no real credentials, sensitive labels/data or operational authority.
- Compose must work on supported developer/CI hosts, but public/operational deployment qualification is Linux-container focused.
- Capability advertisement must match the running profile and installed adapters.

### 20.3 Open questions assigned downstream

- Exact Compose minimum version and support window after the implementation fields are known — implementation/IDR-SRV-052.
- Exact digest-pinned Rust runtime base and PostGIS image/build — implementation proof plus IDR-SRV-049.
- Whether Caddy remains the reference proxy after SSE/base-path/body-limit proof — implementation proof; alternative remains allowed.
- Configuration file format, precedence and reload semantics — IDR-SRV-047.
- Health payload shapes, metrics and alert thresholds — IDR-SRV-048.
- Migration rollback/expand-contract, RPO/RTO and production backup architecture — IDR-SRV-049.
- Exact demo write/auth/rate/data-reset policy — IDR-SRV-047/055 and deployment owner.
- Whether/when object storage or TimescaleDB crosses its adoption threshold — workload proofs/IDR-SRV-049/054.
- ARM database packaging and qualified edge hardware — implementation/IDR-SRV-054.
- Production orchestrator/HA/network-zone/PKI/secret/identity/policy/telemetry choices — future operational design, not this IDR.

None prevents the recommended base implementation. **[P]**

---

## 21. Validation Against This Plan's Success Criteria

| Success criterion | Evidence | Result |
|---|---|---|
| reference profiles with sources and traceability | Sections 3–5 | Met |
| services classified required/optional/profile-gated/deferred | Section 6 | Met |
| images, Compose, bootstrap, teardown, fixtures, packages and local development | Sections 7–8 and 14 | Met |
| CI, conformance, demo, interoperability, streaming, command, publisher/simulator and DDIL | Sections 9–11 and 16–17 | Met |
| security, configuration, secrets, telemetry, migration, backup/restore and health | Sections 10, 12–15 | Met |
| implementation/community lessons incorporated non-normatively | Section 3.3 and derived gates | Met |
| recommendations decision-usable and deployment-bounded | Sections 18–20 | Met |
| downstream handoffs explicit | Section 18 | Met |
| references explicit and reproducible | Section 22 | Met |

The report defines deployment strategy and proof gates without creating infrastructure or claiming that any profile has passed them. It is accepted as the planning baseline for IDR-SRV-047 and later deployment research. **[P]**

---

## 22. References

### 22.1 Current official runtime and packaging sources

1. Docker, [Using profiles with Compose](https://docs.docker.com/compose/how-tos/profiles/), checked 2026-09-15.
2. Docker, [Control startup and shutdown order in Compose](https://docs.docker.com/compose/how-tos/startup-order/), including health and successful-completion conditions, checked 2026-09-15.
3. Docker, [Compose services reference](https://docs.docker.com/reference/compose-file/services/), including healthcheck, read-only filesystem, capability drop, security options, secrets and restart behavior, checked 2026-09-15.
4. Docker, [Networking in Compose](https://docs.docker.com/compose/how-tos/networking/), including project networks, service-name discovery and changing container IPs, checked 2026-09-15.
5. Docker, [Manage secrets securely in Docker Compose](https://docs.docker.com/compose/how-tos/use-secrets/) and [environment-variable best practices](https://docs.docker.com/compose/how-tos/environment-variables/best-practices/), checked 2026-09-15.
6. Docker, [Use Compose Watch](https://docs.docker.com/compose/how-tos/file-watch/), checked 2026-09-15.
7. Docker, [Multi-stage builds](https://docs.docker.com/build/building/multi-stage/) and [building best practices](https://docs.docker.com/build/building/best-practices/), checked 2026-09-15.
8. Docker, [Multi-platform builds](https://docs.docker.com/build/building/multi-platform/), checked 2026-09-15.
9. Docker, [Build attestations](https://docs.docker.com/build/metadata/attestations/) and [SBOM attestations](https://docs.docker.com/build/metadata/attestations/sbom/), checked 2026-09-15.
10. Docker, [Volumes](https://docs.docker.com/engine/storage/volumes/), checked 2026-09-15.
11. Docker Compose, [release 5.1.4](https://github.com/docker/compose/releases/tag/v5.1.4), 2026-05-20.
12. Open Container Initiative, [Image Specification 1.1.1](https://github.com/opencontainers/image-spec/tree/v1.1.1), including [annotations](https://github.com/opencontainers/image-spec/blob/v1.1.1/annotations.md), checked 2026-09-15.
13. SLSA, [Specification 1.2](https://slsa.dev/spec/v1.2/) and [provenance](https://slsa.dev/spec/v1.2/provenance), checked 2026-09-15.
- Development Containers, [Specification](https://containers.dev/implementors/spec/), checked 2026-09-15.
- NGINX, [`ngx_http_proxy_module`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html), checked 2026-09-15.
- Traefik, [Docker setup](https://doc.traefik.io/traefik/setup/docker/) and [trusted forwarded-header configuration](https://doc.traefik.io/traefik/reference/install-configuration/entrypoints/), checked 2026-09-15.

### 22.2 Database, proxy, telemetry and health sources

14. Docker Official Images, [PostgreSQL image documentation](https://github.com/docker-library/docs/blob/master/postgres/README.md), including secret-file and empty-directory initialization behavior, checked 2026-09-15.
15. PostGIS, [Docker PostGIS](https://github.com/postgis/docker-postgis), documentation dated 2026-06-19, including `18-3.6`, PG18 volume path, extension initialization and documented `amd64` support.
16. PostgreSQL, [PostgreSQL 18 `pg_dump`](https://www.postgresql.org/docs/18/app-pgdump.html) and [Backup and Restore](https://www.postgresql.org/docs/18/backup.html), checked 2026-09-15.
17. Caddy, [Automatic HTTPS](https://caddyserver.com/docs/automatic-https), [reverse proxy](https://caddyserver.com/docs/caddyfile/directives/reverse_proxy), and [2.11.3 release](https://github.com/caddyserver/caddy/releases/tag/v2.11.3), checked 2026-09-15.
18. RFC Editor, [RFC 7239: Forwarded HTTP Extension](https://www.rfc-editor.org/rfc/rfc7239), June 2014.
19. OpenTelemetry, [Deploy the Collector](https://opentelemetry.io/docs/collector/deploy/) and [Collector 0.160.0 release](https://github.com/open-telemetry/opentelemetry-collector/releases/tag/v0.160.0), checked 2026-09-15.
20. Kubernetes, [Liveness, Readiness, and Startup Probes](https://kubernetes.io/docs/concepts/configuration/liveness-readiness-startup-probes/), used only for portable probe semantics, checked 2026-09-15.
21. Prometheus, [Securing API and UI endpoints](https://prometheus.io/docs/guides/basic-auth/), checked 2026-09-15.

### 22.3 Controlling standards and project evidence

22. OGC, [OGC API - Connected Systems - Part 1: Feature Resources](https://docs.ogc.org/is/23-001/23-001.html), Version 1.0.
23. OGC, [OGC API - Connected Systems - Part 2: Dynamic Data](https://docs.ogc.org/is/23-002/23-002.html), Version 1.0.
24. OGC, [SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html) and [SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html).
25. IETF, [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110) and [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457).
26. Glaux, [Database and Persistence Architecture Options](idr-srv-025-database-and-persistence-architecture-options-report.md).
27. Glaux, [Metadata and Document Storage Strategy](idr-srv-028-metadata-and-document-storage-strategy-report.md).
28. Glaux, [Transaction, Consistency, Idempotency, and Concurrency Strategy](idr-srv-029-transaction-consistency-idempotency-and-concurrency-strategy-report.md).
29. Glaux, [Server Write and Ingestion Model](idr-srv-031-server-write-and-ingestion-model-report.md).
30. Glaux, [Simulator-to-Server Contract Boundary](idr-srv-033-simulator-to-server-contract-boundary-report.md).
31. Glaux, [Streaming and Event Publication Strategy](idr-srv-035-streaming-and-event-publication-strategy-report.md).
32. Glaux, [Command Authorization, Safety, and Audit Strategy](idr-srv-038-command-authorization-safety-and-audit-strategy-report.md).
33. Glaux, [Authentication, Authorization, and API Security Threat Model](idr-srv-039-authentication-authorization-and-api-security-threat-model-report.md).
34. Glaux, [Zero Trust Architecture Alignment and Enforcement Model](idr-srv-039a-zero-trust-architecture-alignment-and-enforcement-model-report.md).
35. Glaux, [Policy, Releasability, and Cross-Boundary Access Constraints](idr-srv-040-policy-releasability-and-cross-boundary-access-constraints-report.md).
36. Glaux, [Audit Logging and Accountability Strategy](idr-srv-041-audit-logging-and-accountability-strategy-report.md).
37. Glaux, [DDIL-Informed Server Semantics](idr-srv-042-ddil-informed-server-semantics-report.md).
38. Glaux, [Server Synchronization and Conflict-Handling Boundary](idr-srv-043-server-synchronization-and-conflict-handling-boundary-report.md).
39. Glaux, [Rust Implementation Language and Framework Strategy](idr-srv-044-rust-implementation-language-and-framework-strategy-report.md).
40. Glaux, [Service Architecture and Modularization Strategy](idr-srv-045-service-architecture-and-modularization-strategy-report.md).

### 22.4 Non-normative implementation and community studies

41. Glaux, [OpenSensorHub CSAPI Server Implementation Study](idr-srv-014a-osh-csapi-server-implementation-study-report.md).
42. Glaux, [Connected Systems Go CSAPI Server Implementation Study](idr-srv-014b-connected-systems-go-csapi-server-implementation-study-report.md).
43. Glaux, [pygeoapi CSAPI Server Implementation Study](idr-srv-014c-pygeoapi-csapi-server-implementation-study-report.md).
44. Glaux, [SECD CSAPI Server Implementation Study](idr-srv-014d-secd-csapi-server-implementation-study-report.md).
45. Glaux, [OS4CSAPI Client Smoke-Test Findings Study](idr-srv-014e-os4csapi-client-smoke-test-findings-study-report.md).
46. Glaux, [SECD Interoperability Findings Study](idr-srv-014f-secd-interoperability-findings-study-report.md).
47. Glaux, [OS4CSAPI Discussions Lessons-Learned Study](idr-srv-014g-os4csapi-discussions-lessons-learned-study-report.md).
48. Glaux, [Draft CSAPI Part 3 Publish/Subscribe and Implementation Study](idr-srv-014h-draft-csapi-part-3-publish-subscribe-and-implementation-study-report.md).

### 22.5 Reproducibility and evidence limits

- Mutable official documentation and releases were checked on 2026-09-15; the exact implementation must record resolved image/tool digests and versions again.
- Product versions in the metadata are research comparison pins, not pre-approved production selections.
- The PostGIS architecture note quotes the current official image repository's documented support and is a release gate, not a claim that PostGIS itself cannot run on ARM.
- No image, Compose stack, proxy, migration, backup/restore, DDIL scenario, benchmark, security control or conformance harness was executed by this research.
- The shared upstream-history register remains Version 1.12; this deployment topic found no material published CSAPI Part 1/2 history change.

---

## Report Completion Checklist

- [x] All 22 required report sections are present.
- [x] The reference deployment matrix contains every required field.
- [x] Required, optional, profile-gated and deferred services are explicit.
- [x] Image, Compose, local, CI, conformance, demo, DDIL and synchronization shapes are defined.
- [x] Proxy/TLS/origin/CORS, configuration/secrets, migrations, restore, fixtures, telemetry and health are addressed.
- [x] Publisher, simulator, project clients, external clients, streaming and command boundaries are placed.
- [x] Current official runtime evidence and implementation lessons are authority-classified.
- [x] Twelve implementation proofs and all downstream handoffs are explicit.
- [x] IDR-SRV-047, deployment implementation and operational/accreditation claims remain unauthorized pending acceptance.

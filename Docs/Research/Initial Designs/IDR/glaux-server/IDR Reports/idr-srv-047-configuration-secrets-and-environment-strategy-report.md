# Section 047: Configuration, Secrets, and Environment Strategy - Research Report

**Topic ID:** IDR-SRV-047<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-047 Configuration, Secrets, and Environment Strategy](../IDR%20Plans/idr-srv-047-configuration-secrets-and-environment-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** Runtime profiles; configuration classes, schema, sources and precedence; startup validation; immutable and reloadable settings; effective-configuration evidence; secret inventory, loading, redaction, rotation and test safety; feature-specific configuration; unsafe combinations; samples, tooling and downstream handoffs<br>
**Methodology Used:** Accepted-requirement extraction; current primary-source review; profile/capability and configuration-item matrixing; startup and reload threat analysis; secret-lifecycle analysis; implementation-lesson reconciliation; downstream contract synthesis<br>
**Research Time:** Approximately 36 hours of AI-assisted execution on September 16, 2026<br>
**Accepted Deployment Baseline:** Eleven IDR-SRV-046 profiles; one immutable Glaux image; PostgreSQL/PostGIS; explicit same-image migration and fixture operations; profile-gated supporting services; secret-file inputs; explicit public origin and proxy trust; deterministic release/configuration/fixture identity
**Configuration Evidence Freeze:** Figment 0.10.19; config 0.15.25; clap 4.6.7; dotenvy 0.15.7; secrecy 0.10.3; zeroize 1.9.0; current Docker Compose, Kubernetes and OWASP documentation checked September 16, 2026
**Standards Baseline:** OGC API - Connected Systems Parts 1 and 2 Version 1.0; SensorML 3.0; SWE Common 3.0; accepted Glaux IDR-SRV-001 through IDR-SRV-046
**Document Purpose:** Define a safe, repeatable and testable server configuration contract without implementing it, selecting an enterprise secret manager, or authorizing later Category H work
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

“Operational-reference,” “secure,” “secret,” “DDIL” and “effective configuration” are design terms, not accreditation, production approval, hardware protection or tactical-readiness claims. **[X]**

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Configuration Requirement Extraction Methodology
5. Runtime Profile Taxonomy
6. Configuration Category Taxonomy
7. Configuration Sources and Precedence Findings
8. Startup Validation, Fail-Fast, Reload, and Effective-Configuration Findings
9. Secrets and Sensitive Configuration Inventory
10. Secret Loading, Redaction, Rotation, and Test-Safety Findings
11. Server/API and Database Configuration Findings
12. Schema/Profile, Ingestion/Source, Streaming/Event, and Command/Control Configuration Findings
13. Security, Authentication, Authorization, and Policy Configuration Findings
14. DDIL/Synchronization and Observability Configuration Findings
15. Test, CI, Conformance, Fixture, Public Demo, and Interoperability Configuration Findings
16. Sample Configuration and Developer Documentation Recommendations
17. Downstream Topic Handoff Matrix
18. Recommendations
19. Risks, Constraints, and Open Questions
20. Validation Against This Plan's Success Criteria
21. References

---

## 1. Executive Summary

Glaux Server should expose **one versioned, typed configuration contract** loaded only at the composition root. The recommended first implementation uses Serde domain types and Figment for ordered providers, profile selection and value provenance. This is a candidate library selection, not permission for configuration logic to leak into domain code. Application and domain modules receive validated capability-specific settings or constructed adapters; they do not read environment variables, parse files or resolve secrets. Unknown keys, unsupported schema versions, ambiguous profiles and unsafe combinations are fatal before the server binds a socket or performs migration, publication, ingestion or command effects. **[A/D/E/P]**

Profile selection is explicit and mandatory. The eleven accepted deployment profiles are contracts with allowed and prohibited capabilities, not shorthand for “development” or “production.” A profile supplies safe baselines and constraints; deployment files then supply environment-specific non-secret values. No setting may silently transform `demo-public`, `command-sim`, `ddil-single`, `sync-two-node` or `operational-reference` into another safety posture. Local no-auth behavior is confined to `dev-native` and `dev-compose`, requires the conspicuous `unsafe_local` gate, loopback binding, and disabled commands, admin exposure and unrestricted ingestion. **[A/E/P]**

The ordinary configuration precedence is deliberately small and inspectable: compiled safe defaults, one release profile file, ordered operator files, an allowlisted set of `GLAUX__...` scalar environment overrides, then a narrow non-secret CLI override set. Lists and structured objects do not accept environment overrides in the first implementation. `.env` is an explicitly enabled local-development convenience that populates process environment without overriding already-set values; it is never loaded in CI, demo, DDIL or operational-reference profiles and is never a secret store. Database-backed administrative state, policy data, registered sources, feature records and runtime-generated values are not startup configuration and never participate in this precedence chain. **[D/E/P]**

Secrets use a separate resolution plane. Ordinary configuration carries typed secret references, never secret values. Mounted files are the portable baseline; local/CI ephemeral files are acceptable, and a future external-provider adapter may resolve versioned references without changing consuming modules. Direct secret values in CLI arguments, TOML, Compose environment values, image layers, labels, fixtures or diagnostics are prohibited. Process-environment secret values are disabled outside an explicit local/CI compatibility gate and should be removed after bootstrap. Rust secret values should use a non-serializing, redacting wrapper such as `secrecy`, with explicit exposure and best-effort zeroization; that reduces accidental leakage but is not a memory-isolation guarantee. **[D/E/P]**

Startup follows a fixed pipeline: identify release and profile; merge ordinary sources while recording provenance; reject unknown fields; deserialize typed settings; perform field and cross-field validation; resolve only referenced secrets and trust artifacts; preflight mandatory dependencies; construct an immutable `EffectiveConfig`; emit a redacted manifest and fingerprint; then start listeners/workers. A failed stage exits non-zero with stable machine-readable diagnostics that name the key path, source and rule but never the secret value. Readiness remains false until all profile-required gates succeed. Migration and fixture commands run the same configuration pipeline with role-specific requirements. **[A/D/E/P]**

Configuration is immutable for the first server release except for a tightly allowlisted observability control such as log-filter level. Environment variables are never reread. Security, trust, policy, public origin, proxy trust, database, schemas, ingestion boundaries, streaming topology, command safety, DDIL authority and synchronization configuration require restart. Secret rotation occurs through versioned references, overlapping validity where the protocol allows it, atomic client/key-ring replacement and audit evidence—not through a general-purpose hot reload. A later subsystem may add authenticated, validate-then-swap reload only after concurrency, rollback and audit proofs. **[E/P]**

The effective-configuration record must bind release digest, configuration schema version, explicit profile, source identities/digests, enabled capability set, standards/profile/fixture package identities, and non-secret normalized values. Secret entries expose only logical name, provider, reference identifier, optional provider version and availability/expiry status. Do not print hashes of secret contents because low-entropy secrets can be guessed offline. The record's canonical redacted representation produces a configuration fingerprint used by health metadata, audit, CI evidence and incident correlation. **[E/P]**

This report establishes exact configuration ownership and handoffs but does not create files, select Vault/SOPS/Kubernetes, define telemetry cardinality, set backup retention, implement migrations, choose production credentials, enable real commands or claim conformance. Acceptance authorizes IDR-SRV-048 only. **[X/P]**

---

## 2. Scope and Plan Alignment

### 2.1 Coverage

| Plan area | Coverage | Evidence location |
|---|---|---|
| profiles and prior-topic traceability | Complete | Sections 3–5 |
| categories, sources and precedence | Complete | Sections 6–8 |
| secrets, redaction and rotation | Complete | Sections 9–10 |
| API/database and feature configuration | Complete | Sections 11–14 |
| unsafe combinations and fail-safe behavior | Complete | Sections 5, 8, 11–15 |
| tests, samples and developer experience | Complete | Sections 15–16 |
| implementation/community lessons | Complete, non-normative | Section 3.3 |
| downstream decisions | Complete | Section 17 |

### 2.2 In scope

- server-owned configuration schema and loading contract;
- accepted runtime-profile capability constraints;
- public, sensitive, secret, generated and administrative-state classification;
- deterministic source precedence and provenance;
- startup, restart, reload and secret-rotation semantics;
- feature-specific settings, safe defaults and invalid combinations;
- redacted diagnostics, configuration fingerprints and operator tooling; and
- bounded handoffs to observability, lifecycle, testing and interoperability research.

### 2.3 Out of scope

- enterprise secret-manager, orchestrator, PKI, IdP, policy engine or SIEM selection;
- environment-specific secret values, identity names, internal topology or policy labels;
- production capacity, timeout, retention, key lifetime or rotation-frequency numbers;
- implementation code, configuration artifacts or deployment infrastructure;
- database-backed domain/admin state schemas and user-facing configuration UI;
- live physical command authority, inbound experimental Part 3 publication, accreditation or conformance claims. **[X]**

### 2.4 Accepted invariants carried forward

Configuration must preserve one authoritative PostgreSQL/PostGIS write core; exact artifacts separate from canonical state; durable inbox/outbox/audit references; SSE before optional brokers; MQTT and experimental Part 3 behind gates; application-layer authentication, authorization, policy and source trust; deny-by-default commands; explicit public origin/proxy trust; local DDIL evidence; domain-aware synchronization; one immutable image; same-image administrative commands; and deterministic release/configuration/profile/fixture identity. **[A]**

---

## 3. Evidence Base and Authority Classification

### 3.1 Current primary evidence

| Source | State checked | Evidence used | Limitation |
|---|---|---|---|
| Figment documentation | 0.10.19; 2026-09-16 | typed extraction, profiles, provider merge/join and per-value provenance | library mechanism, not Glaux safety policy |
| config documentation | 0.15.25; 2026-09-16 | alternative hierarchical sources, prefixes and separators | evaluated alternative; provenance/profile fit is weaker for this design |
| clap documentation | 4.6.7; 2026-09-16 | typed CLI values and value-origin reporting | CLI syntax alone cannot enforce cross-field safety |
| dotenvy documentation | 0.15.7; 2026-09-16 | non-overriding local `.env` loading | local convenience only; parent-directory search is too implicit for Glaux defaults |
| secrecy / zeroize documentation | 0.10.3 / 1.9.0; 2026-09-16 | explicit exposure, non-serialization, redacted debug and drop-time zeroing | not hardware, swap, core-dump or copied-buffer protection |
| Docker Compose documentation | current 2026-09-16 | environment precedence, mounted secrets and per-service grants | Compose secrets are bind-mounted files, not an enterprise secret store |
| Kubernetes documentation | current 2026-09-16 | config/secret separation, least privilege, immutable objects and external-provider option | future-platform comparison only; Secrets may be unencrypted at rest by default |
| Twelve-Factor Config | current 2026-09-16 | separation of deploy-varying config from code | environment-only guidance is insufficient for structured Glaux profiles and secret references |
| OWASP Secrets Management | current 2026-09-16 | inventory, least privilege, rotation, revocation, expiry and never-log requirements | guidance, not a product or accreditation baseline |

Figment is recommended because it combines typed Serde extraction with explicit profiles and value-source metadata. Glaux should wrap it behind a small configuration module, pin the reviewed version through the Rust dependency process, prohibit `serde(flatten)` where it would obscure unknown-key or provenance errors, and retain schema-level validation independent of the library. **[D/E/P]**

### 3.2 Accepted project evidence

IDR-SRV-001 through 024 define standards, media, errors, links and compliance-visible behavior. IDR-SRV-025 through 030 establish persistence, artifacts, transactions and lifecycle. IDR-SRV-031 through 038 establish write, ingestion, source, dynamic-data, event and command boundaries. IDR-SRV-039 through 043 define security, policy, audit, DDIL and synchronization. IDR-SRV-044 through 046 establish the Rust stack, modular architecture, deployment profiles and runtime invariants. Those accepted reports control project intent; product documents establish mechanisms only. **[A/D]**

### 3.3 Non-normative implementation and community lessons

| Observed lesson | Source class | Glaux consequence |
|---|---|---|
| incorrect external links arise behind proxies | OSH/community implementation studies | require and validate one public origin plus trusted-proxy allowlist |
| permissive CORS, sample credentials and no-auth switches escape prototypes | CS-Go/pygeoapi studies | profile constraints override convenience; public and operational profiles reject them |
| startup migrations couple schema change to availability | CS-Go/pygeoapi studies | migration is an explicit same-image command with distinct credentials |
| optional infrastructure obscures minimum viability | deployment/community studies | each broker, IdP, policy engine and telemetry backend is capability-gated |
| OpenAPI and actual capabilities can drift | multiple implementation studies | one validated capability registry drives routes, conformance and published API metadata |
| mutable public demos and remote schemas disappear or drift | interop/client studies | pin local packages and fixtures; treat live systems as dated observations |
| deployment-specific shortcuts are not standards requirements | all studies | document adoption rationale and never cite another server as normative authority |

### 3.4 Evidence limits

- No Glaux server exists on which to test parsing, redaction, reload races or rotation.
- Candidate crate versions are a dated research freeze, not automatic release pins.
- No selected operational secret provider or deployment platform exists.
- Secret wrappers reduce accidental exposure; they cannot prove a value never existed in copied memory, swap, a core dump or a dependency.
- Exact timeouts, limits, pool sizes, key lifetimes and retention values require later measured or environment-specific decisions.

---

## 4. Configuration Requirement Extraction Methodology

The research used seven passes:

1. extract every deploy-varying value or capability gate from accepted reports;
2. distinguish configuration from normative behavior, administrative/domain state and generated state;
3. assign each item a type, owner, sensitivity, source, profile scope and failure behavior;
4. evaluate profiles as allow/deny constraints before considering defaults;
5. trace parse, merge, validation, secret resolution, startup, steady-state, reload, rotation and shutdown paths;
6. challenge combinations against security, DDIL, command and interoperability invariants; and
7. allocate later operational detail without leaving implementation ambiguity.

### 4.1 Decision tests

| Test | Required answer |
|---|---|
| correctness | can a setting alter a normative requirement that must never be optional? If yes, keep it in code, not config. |
| ownership | is this server boot configuration, administered domain state, deployment wiring or generated state? |
| sensitivity | would disclosure enable access, reveal protected topology/policy, or merely describe a public endpoint? |
| determinism | can two starts with the same release, inputs and profile derive the same effective contract? |
| safety | can an override cross a profile's prohibited-effect boundary? If yes, reject it. |
| operability | can the source and validation failure be explained without revealing a secret? |
| lifecycle | is change safe in-place, restart-only, or migration/re-bootstrap dependent? |
| verification | is there a unit, integration, negative or deployment test that proves the rule? |

### 4.2 Values that must not be configurable

Standards-mandated semantics, authorization enforcement points, audit-before-effect ordering, command deny-by-default, transaction atomicity, idempotency rules, tenant/scope isolation, secret redaction, validation-before-side-effect ordering and the prohibition on inbound experimental Part 3 are code and policy invariants. Configuration may select implemented conformance classes or adapters but cannot disable the correctness and safety rules associated with any advertised capability. **[A/E/P]**

---

## 5. Runtime Profile Taxonomy

### 5.1 Profile contract

`--profile <id>` is mandatory for every executable role, including `serve`, `migrate`, `fixture`, `config check` and maintenance commands. The selected identifier must be present in the release's profile registry. `GLAUX_PROFILE` is not accepted as an implicit selector; an operator must make profile choice visible in the invocation or deployment command. Custom profiles may eventually extend a registered base, but the first implementation supports only release-defined identifiers and operator value overlays. **[E/P]**

| Profile | Purpose | Mandatory posture | Prohibited posture |
|---|---|---|---|
| `dev-native` | native edit/test loop | loopback, explicit `unsafe_local` if no auth, local DB | public bind, real command dispatch, unrestricted remote ingestion |
| `dev-compose` | container-parity development | isolated Compose network, mounted dev secrets | public exposure or operational claims |
| `ci-core` | deterministic automated integration | ephemeral DB, fake clock/IDs where selected, clean teardown | live external services, persistent credentials, physical effects |
| `conformance` | standards harness execution | pinned capability set, fixture and standards manifest | undocumented extensions affecting expected behavior |
| `interop` | external-client testing | explicit public origin, test identities and evidence capture | reliance on mutable demo state as merge gate |
| `demo-public` | TLS-fronted public demonstration | authentication or bounded read-only exposure, limits, synthetic data | admin/diagnostics, unrestricted ingestion, physical commands, unsafe local mode |
| `streaming-mqtt-exp` | optional MQTT/Part 3 outbound experiment | broker trust, namespace, authorization, outbox continuity | inbound Part 3 or standards claim beyond accepted boundary |
| `command-sim` | isolated simulated command workflow | simulator-only target, approval/safety/audit, egress allowlist | route to physical devices or operational command gateway |
| `ddil-single` | one-node dependency-loss simulation | local schemas/trust/policy/data/outbox, explicit stale-state rules | required remote fetch, implicit authority elevation |
| `sync-two-node` | controlled synchronization testing | distinct node identity/config/DB, partition and recovery evidence | shared database, shared identity or replication-as-conflict-resolution |
| `operational-reference` | hardened portable contract | authn/authz/policy/audit/TLS expectations, no test flags, explicit integrations | no-auth, sample secrets, debug exposure, auto-bootstrap, simulated authority |

### 5.2 Constraint evaluation

Profile constraints are evaluated after merge and before secret resolution. They can reject an operator override but never silently rewrite it. The effective configuration records the selected profile and constraint-set version. A release test enumerates every profile and asserts its capability registry, mandatory fields, forbidden fields, bind exposure, secret references and startup dependencies. **[E/P]**

### 5.3 Capability gates

Use typed enumerations or structured modes rather than ambiguous booleans: `authentication.mode = local_unsafe | oidc | mtls | composite`; `events.transport = disabled | in_process | mqtt_outbound`; `commands.mode = disabled | simulated`; `schemas.fetch = disabled | allowlisted_online`; `sync.mode = disabled | peer_test`. There is no `commands.mode = real` or `part3.inbound = true` until a later accepted design explicitly adds it. Compile-time Cargo features may remove optional code/dependencies but never stand in for runtime authorization or profile safety. **[A/E/P]**

---

## 6. Configuration Category Taxonomy

| Category | Meaning | Examples | Storage/diagnostic rule |
|---|---|---|---|
| public configuration | intentionally client-visible behavior | public base URL, path prefix, advertised media types, API title | may appear in API/config manifest |
| internal non-sensitive configuration | operational but low disclosure impact | bind port, pool size, worker count, local cache path | visible to authorized diagnostics; normally not public |
| sensitive configuration | disclosure aids reconnaissance or reveals policy/topology | internal URLs, peer IDs, trust paths, policy bundle IDs, command target aliases | redact or summarize in public output; access-controlled internally |
| secret reference | locator/metadata used to obtain a secret | `file:/run/secrets/db_app`, provider key/version | reference may itself be sensitive; never replace with value in config record |
| secret value | possession grants access or cryptographic authority | password, API token, private key, client secret | non-serializing wrapper; never log, dump, fixture or persist accidentally |
| public trust material | authenticates others but need not be secret | CA certificate, JWKS, verification public key | integrity/version critical; publish only when intended |
| test-only configuration | deterministic or fake behavior | fixed clock, deterministic IDs, fake issuer, fault schedule | accepted only in test profiles; reject elsewhere |
| runtime-generated state | derived for this process | node run ID, listener address, config fingerprint | generated after validation; not an input source |
| administered/domain state | mutable records governed through application workflows | source registrations, grants, policies, subscriptions, commands | database/API ownership; never merged into startup config |
| release constant | code/artifact identity or invariant | build revision, supported schema versions, compiled capabilities | embedded and checked against manifest; not overridden |

Classification is schema metadata, not a naming convention. Every field declares its category, redaction, allowed source and reload policy. A key-name regular expression may be defense in depth but cannot be the primary redactor. **[E/P]**

---

## 7. Configuration Sources and Precedence Findings

### 7.1 Ordinary configuration precedence

From lowest to highest precedence:

1. **compiled safe defaults** for values safe in every applicable profile;
2. **release profile file**, immutable and digest-bound to the release;
3. **operator configuration files**, explicitly repeated in command order through `--config`, with later files overriding earlier scalar/map leaves;
4. **allowlisted scalar environment overrides** using `GLAUX__SECTION__FIELD`; and
5. **narrow non-secret CLI overrides** limited initially to config paths, profile, listener bind/port and log filter where the profile permits them.

The server prints source identities and per-field provenance through `config explain`, with sensitive values redacted. It rejects duplicate operator file paths, missing explicitly named files, unknown environment keys under `GLAUX__`, unknown TOML keys, arrays supplied through environment variables, and a field supplied both as direct value and secret reference. **[D/E/P]**

### 7.2 Source rules

| Source | First-release use | Rule |
|---|---|---|
| typed compiled defaults | universally safe primitives | never contain URLs, credentials, trust, public bind or enabled effects |
| release TOML profile | safe profile baseline/constraints | installed read-only; digest in release manifest |
| operator TOML | structured deployment values | explicit absolute/canonical path; owner/permission warnings where portable |
| mounted config file | container/operator delivery | same semantics as operator TOML; read once at startup |
| environment | allowlisted scalar overrides | double-underscore hierarchy; no secrets by default; provenance retained |
| `.env` | explicit local-only population | exact path, no parent search, no override; gitignored; disabled outside dev profiles |
| CLI | profile/path and small emergency scalar set | no secret-valued flags and no generic `--set key=value` |
| secret provider/file | second-stage secret value resolution | outside ordinary merge; one reference and provider per logical secret |
| database | administered/domain state only | not startup precedence; read through application repositories after readiness gates |
| Cargo feature | code inclusion | cannot authorize a capability or weaken runtime controls |

TOML is recommended as the canonical human-authored first-release format because the Rust ecosystem has mature Serde support and TOML avoids YAML's implicit typing complexity. JSON Schema should be generated or curated for editor validation, but the executable's typed parser and cross-field rules remain authoritative. JSON may be emitted for machine-readable effective manifests; accepting multiple authoring syntaxes is deferred to avoid semantic drift. **[E/P]**

### 7.3 Merge semantics

- maps merge recursively; a later scalar replaces an earlier scalar;
- arrays replace as a whole—never append implicitly;
- `null`/unset deletion is unsupported in first-release TOML; use explicit typed states;
- field aliases are prohibited after a documented deprecation window;
- numeric units live in names or typed strings (`request_timeout = "30s"`, `max_body_bytes`), never undocumented bare ambiguity;
- paths resolve relative to the file declaring them only when represented by an explicit file-relative path type; operational references should prefer absolute paths;
- profile constraints are evaluated after merge and are not overrideable; and
- ordinary configuration is fully materialized before any external connection or secret lookup. **[E/P]**

### 7.4 Why not environment-only

Environment variables remain useful for orthogonal scalar deployment values, but large allowlists, trust maps, peer definitions, media-type sets and nested DDIL/command policy are difficult to review and reproduce as strings. Docker Compose also has its own interpolation/container-environment precedence, which must not be mistaken for Glaux's in-process precedence. The release therefore captures `docker compose config` plus the Glaux redacted effective manifest as separate evidence. **[D/E/P]**

---

## 8. Startup Validation, Fail-Fast, Reload, and Effective-Configuration Findings

### 8.1 Startup pipeline

| Stage | Action | Failure behavior |
|---|---|---|
| 1 identity | read embedded release manifest; require explicit command/profile | exit configuration code before effects |
| 2 collect | read named ordinary sources and provenance | fail on missing/unreadable/duplicate source |
| 3 merge | apply fixed precedence and profile selection | fail on incompatible type or forbidden source |
| 4 decode | deny unknown fields; deserialize typed schema | report stable key path/source without value leakage |
| 5 validate | field, URI, CIDR, path, enum, bound and cross-field checks | aggregate safe errors; exit non-zero |
| 6 resolve | load referenced secrets and integrity-sensitive artifacts | fail closed when profile-required; never print contents |
| 7 preflight | verify database/schema/trust/dependency compatibility | readiness false; command exits or server fails according to profile requirement |
| 8 freeze | construct immutable `EffectiveConfig` and adapters | no module may retain raw merge tree or process-env reader |
| 9 attest | emit redacted manifest/fingerprint and startup audit event | failure is fatal if evidence sink required by profile |
| 10 activate | bind listeners and start workers | only after all required gates succeed |

Validation must distinguish **syntax/type**, **semantic field**, **cross-field**, **profile constraint**, **artifact integrity**, **secret availability**, **dependency compatibility** and **runtime degradation** failures. Only the last category can be a warning after successful startup, and only where the profile already defines degraded behavior. **[E/P]**

### 8.2 Invalid-combination register

The server rejects at least:

- non-loopback bind with `authentication.mode = local_unsafe`;
- `unsafe_local` in any profile except `dev-native` or `dev-compose`;
- TLS-off public origin, wildcard CORS with credentials, or trusted-proxy wildcard;
- proxy-forwarded headers without an explicit proxy CIDR/address allowlist;
- a public base URL whose scheme/path conflicts with TLS and path-prefix settings;
- runtime DB superuser/migration credentials or auto-migrate in `serve`;
- multiple replicas combined with an unlocked migration/bootstrap operation;
- unrestricted ingestion, admin/diagnostic exposure or raw payload logging in `demo-public`;
- command enablement without simulator identity, safety rules, approval mode, authorization, audit, egress allowlist and explicit simulated target;
- any physical target or inbound Part 3 setting in the current schema;
- broker mode without authenticated transport/trust, namespace, outbox and replay limits;
- advertised capability whose required adapter, schema package or route is disabled;
- remote schema/policy/trust dependency required for `ddil-single` local operation;
- synchronization peers sharing node identity or authoritative database;
- test clock, deterministic IDs, fake identity/policy/broker or fixture reset outside allowed test profiles;
- default/example/short secret identifiers in demo or operational-reference profiles;
- secret values in CLI, ordinary TOML, Compose labels or non-allowlisted environment keys; and
- unknown fields, unsupported `config_version`, unresolved deprecated aliases or mixed profile namespaces. **[A/E/P]**

### 8.3 Reload boundary

The first release treats configuration as restart-only. An optional authenticated control may adjust only a schema-allowlisted log filter, producing an audit event and bounded TTL if used for debugging. It does not reread files or environment variables. Metrics labels, exporter endpoints, sampling, redaction, health rules, security, trust, public origin, database, schemas, ingestion, events, commands, DDIL and sync are restart-only. A later general reload must parse a complete candidate, validate it, build replacement adapters, atomically swap an immutable snapshot, audit old/new fingerprints, and retain rollback; partial mutation and file-watch-triggered activation are prohibited. **[E/P]**

### 8.4 Effective configuration and fingerprint

`EffectiveConfigV1` contains:

- configuration schema and profile-constraint versions;
- release version/revision/image digest and executable role;
- profile ID and enabled capability IDs;
- normalized non-secret values plus their source class and source digest;
- standards/schema/profile/vocabulary, migration and fixture package identities;
- secret-reference metadata: logical name, provider kind, reference ID, version/lease status and resolution state only;
- dependency expectations and selected adapter kinds; and
- creation time, run ID and canonical redacted fingerprint.

The fingerprint is a digest of the canonical redacted manifest, not of raw configuration and never of secret contents. Two runs with different secret versions can be distinguished by provider version metadata without disclosing or guessably hashing the value. `config explain <path>` identifies the winning source and shadowed sources; `config fingerprint` and the admin-only diagnostics endpoint return bounded evidence. Public health endpoints expose at most release, profile class and fingerprint if approved by IDR-SRV-048. **[E/P]**

---

## 9. Secrets and Sensitive Configuration Inventory

| Secret/trust item | Classification | Consumers | Required handling / rotation class |
|---|---|---|---|
| application DB credential | secret | server/worker | dedicated least-privilege account; overlap or restart on rotation |
| migration DB credential | secret | `migrate` only | never mounted into `serve`; short-lived preferred |
| bootstrap/admin DB credential | secret | explicit bootstrap only | absent after bootstrap; never runtime default |
| OIDC client secret | secret | auth adapter if confidential client | versioned provider ref; overlap per issuer support |
| OIDC issuer/JWKS URI, audience | public/sensitive config | auth verifier | strict URI/issuer match; cached trust version |
| token/JWT signing private key | high-impact secret | issuer/admin utility if Glaux owns signing | key ring with active key ID and verification overlap |
| token verification keys/JWKS | public trust | auth verifier | integrity, issuer and freshness validation; not redacted as secret |
| TLS/mTLS private key | high-impact secret | proxy/server/client adapter | atomic certificate/key pair; file permissions; renew before expiry |
| TLS/mTLS certificates and CA anchors | public trust/sensitive | listeners/clients | pin/version; reject mismatched key pair or invalid chain |
| API/source/admin bootstrap keys | secret | auth/source bootstrap | unique per principal/environment; hash where server verifies; revoke/rotate |
| broker credentials/client key | secret | event adapter | scoped publish/subscribe rights; overlapping replacement |
| command-gateway credential/client key | high-impact secret | command adapter | command-sim only now; separate identity and egress target |
| object-store credential | secret | artifact adapter | bucket/prefix least privilege; provider rotation |
| policy-provider token | secret | policy adapter | read-only scope; cached policy validity independent of token |
| policy bundle signature/verification key | private secret/public trust | policy tooling/runtime | active signer separated from runtime verifier; key-ID overlap |
| audit/checkpoint signing key | high-impact secret | audit adapter | append-only key ID history; verification material retained |
| cursor/idempotency HMAC key | secret | API/application | versioned key ring; accept previous version during bounded transition |
| at-rest/application encryption key | high-impact secret | artifact/domain adapter if adopted | envelope/key-ID design; rotation may require migration/re-encryption |
| synchronization peer credential/key | high-impact secret | sync adapter | distinct node identity; peer-scoped; revoke independently |
| telemetry exporter credential | secret | telemetry adapter | least privilege; telemetry loss must not disclose credential |
| backup encryption/repository credential | high-impact secret | backup command only | never mounted to serve; rotation tested with restore |
| internal URLs, peer/source IDs, topology | sensitive config | adapters/operators | omit from public diagnostics; structured redaction |
| policy/trust/cache paths and bundle IDs | sensitive config | security/DDIL | show basename/version or logical ID, not full topology by default |

Secrets are classified by impact and lifecycle, not merely by format. A certificate may be public trust while its paired private key is a high-impact secret; a secret reference may reveal sensitive topology even though it does not grant access. **[D/E/P]**

---

## 10. Secret Loading, Redaction, Rotation, and Test-Safety Findings

### 10.1 Resolution model

Ordinary settings contain `SecretRef` values such as `{ provider = "file", id = "db_app_password" }`. The deployment maps the logical ID to a mounted path; application config does not hard-code `/run/secrets` as the only platform. Resolution occurs after profile validation through a `SecretProvider` port. The portable baseline supports mounted files. A local ephemeral-file provider and future external-provider adapter implement the same contract. There is exactly one winning reference per logical secret—secret values are never merged. **[D/E/P]**

File resolution must require a regular file, reject symlinks where the deployment contract can enforce that safely, bound size, preserve binary values where relevant, avoid lossy trimming, validate expected encoding/type, and close handles promptly. Whether one terminal newline is removed must be declared per secret type and tested; global whitespace trimming is prohibited. File owner/mode checks are warnings or fatal according to platform/profile capability, not a false portable guarantee. **[E/P]**

### 10.2 Provider posture by profile

| Profile family | Allowed provider posture | Rejected posture |
|---|---|---|
| dev | gitignored local file or OS credential integration; generated ephemeral value | checked-in sample value; accidental parent `.env` search |
| CI/test | job-scoped generated file/secret; fake cryptographic identities | organization production secret; fork-visible persistent secret |
| demo | deployment-mounted/provider secret unique to demo | default/sample/shared operational credential |
| DDIL simulation | sealed/provisioned local reference with validity metadata | mandatory online lookup during disconnected operation |
| operational-reference | mounted file or approved external-provider adapter | direct env/CLI value, image-baked value, repository plaintext |

Compose's secret-file mechanism is a delivery interface, not proof of encrypted storage or complete secret management. Kubernetes Secrets likewise require encryption-at-rest, RBAC and least-privilege configuration; Glaux remains provider-neutral. SOPS and Vault are viable future mechanisms to evaluate, not selections here. **[D/E/X/P]**

### 10.3 In-process handling and redaction

- represent resolved values with `SecretBox`/`SecretString` or an equivalent audited wrapper;
- require explicit `ExposeSecret` at the smallest adapter boundary;
- prohibit `Serialize`, ordinary `Debug`, `Display`, equality dumps and cloning by default;
- zero owned buffers on drop where supported and avoid intermediate `String` copies;
- register classification metadata for logs, tracing fields, problem details, panic hooks and diagnostics;
- render secret values as `[REDACTED]` and sensitive references as bounded logical identifiers;
- filter headers (`Authorization`, cookies, API keys), connection strings, URI userinfo and known payload credential fields before event creation;
- disable raw request/response/body logging globally, not merely in production profiles;
- prevent secrets from entering span names, metric labels, OpenAPI examples, support bundles, crash reports and CI command echo; and
- test redaction with canary values across startup errors, structured logs, traces, metrics, diagnostics and snapshots. **[D/E/P]**

Redaction does not make a diagnostic safe if field presence, length, path or provider metadata is itself sensitive. Public output uses a smaller allowlist than authenticated operator output. There is no `--show-secrets` command. **[E/P]**

### 10.4 Rotation state machine

| Secret class | Rotation approach | Failure rule |
|---|---|---|
| database/broker/API client credential | provision new; validate; atomically replace client/pool; drain old; revoke | retain old client only for bounded overlap; alert and remain on known-good if swap fails |
| signing/HMAC/audit key | versioned key ring: one active writer plus accepted prior readers | never discard verification key before retention/replay horizon is satisfied |
| TLS certificate/private key | load and validate matching pair/chain; atomic listener/client update or restart | mismatched/expired pair never replaces known-good; expiry readiness policy is profile-defined |
| external-provider lease | renew before expiry with jitter; record version/status | fail closed for new privileged effects; defined degradation for existing reads only |
| application encryption key | introduce new key ID and rewrap/re-encrypt through explicit migration | never silently reinterpret ciphertext or delete old key |
| bootstrap secret | use once; record completion; revoke/remove | startup rejects persistent bootstrap mode outside explicit operation |

Rotation events record logical secret ID, old/new provider version identifiers, actor/process, timestamps and outcome—not values. A general configuration fingerprint changes when version metadata changes. Emergency revocation and recovery are required alongside routine rotation. Exact lifetimes belong to environment security policy and later operational planning. **[D/E/P]**

### 10.5 Test safety

Tests generate ephemeral, visibly fake credentials inside isolated directories; never reuse production-shaped fixed secrets. Secret fixtures contain references or generator specifications, not reusable values. CI log masking is defense in depth, because Glaux must avoid emitting the value before the CI platform masks it. Forked/untrusted jobs receive no persistent secrets. Negative tests scan repository history/artifacts, image layers, Compose rendering, logs, JUnit output, snapshots and support bundles for canary secrets. **[D/E/P]**

---

## 11. Server/API and Database Configuration Findings

### 11.1 Required configuration matrix

| Configuration item | Category | Profiles | Source | Default | Req/opt | Classification | Validation | Redaction | Reload | Failure | Handoff | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `config_version` | schema | all | release/operator file | none | required | public | exactly supported version | show | restart | fatal | implementation | migrations require explicit tooling |
| profile ID | profile | all | CLI | none | required | public | registered profile only | show | immutable | fatal | all tests | not selected from env |
| `server.bind` / port | API | all | profile/file/env/CLI | loopback only where safe | required | internal | valid IP/port; profile exposure check | operator-only | restart | fatal | 048/054 | wildcard requires protected profile |
| `server.public_base_url` / path | API | proxy/public | file/env | none | conditional | public | absolute HTTPS where public; normalized path; no query/userinfo | show | restart | fatal | 048/056 | sole generated-link origin |
| trusted proxy allowlist | API/security | proxied | file | empty | conditional | sensitive | explicit IP/CIDR; no wildcard | summarize | restart | fatal | 055/056 | ignore forwarding headers otherwise |
| CORS origins/credentials | API/security | browser profiles | file | disabled | optional | public | exact origins; wildcard incompatible with credentials | show | restart | fatal | 055/056 | no origin reflection |
| API limits/timeouts/pagination | API | all | profile/file/env | conservative safe baseline | required | public/internal | bounded types and cross-field hierarchy | show safe values | restart | fatal | 048/054 | exact values later measured |
| media types/conformance/OpenAPI | API | all | release/profile | accepted capability set | required | public | capability registry consistency | show | restart | fatal | 050/056 | cannot advertise disabled behavior |
| app DB endpoint/name/user | database | all serving | file/env | none | required | sensitive | scheme/host/database, TLS posture, role identity | redact host/user by audience | restart | fatal | 049/052 | password separate ref |
| app DB credential ref | secret ref | all serving | file | none | required | secret/sensitive ref | provider allowed; resolvable | logical ID/status only | rotate/restart | fatal | 049/055 | no URL userinfo |
| migration DB credential ref/mode | lifecycle | migrate only | file | none | conditional | secret | role separation and lock/version checks | logical ID/status | immutable | fatal | 049 | rejected by `serve` |
| pool and statement limits | database | all serving | profile/file/env | bounded profile value | required | internal | nonzero; DB/server budget consistency | show operator | restart | fatal | 048/054 | no unbounded queue |
| database schema/extensions | database | all | release/file | pinned expected set | required | internal | identifier allowlist; PostGIS/version compatibility | show | restart/migrate | fatal | 049/050 | runtime does not install extensions |
| audit/outbox settings | persistence | write profiles | release/file | enabled where required | required | sensitive | cannot disable when effects/writes active | summarize | restart | fatal | 048/049/055 | retention later topic |
| schema/profile package refs | standards | all | release/file | release package | required | public/sensitive | digest, version and compatibility | show logical ID/digest | restart | fatal | 050/053/056 | no mutable implicit latest |
| remote schema fetch mode | standards/DDIL | selected online only | file | disabled | optional | sensitive | allowlist, TLS and cache rules | summarize | restart | fatal if prohibited | 050/055 | disabled in DDIL/conformance baselines |
| ingestion mode/limits/quarantine | ingestion | write profiles | profile/file | disabled unless profile enables | conditional | public/internal | auth/source trust, media/size/rate/retention checks | show safe | restart | fatal | 053–055 | demo unrestricted mode rejected |
| event transport/broker namespace | events | streaming | profile/file | in-process/SSE baseline | conditional | sensitive | adapter/capability/outbox/trust consistency | summarize | restart | fatal | 048/054–056 | MQTT outbound experimental only |
| broker credential ref | secret ref | broker profile | file | none | conditional | secret | provider and scoped identity | logical ID/status | rotate | fatal | 055 | never topic-embedded credential |
| command mode/simulator target | command | command-sim | profile/file | disabled | conditional | sensitive | simulator identity, egress and safety bundle | summarize | restart | fatal | 048/055/056 | no real mode in schema |
| authn/authz/policy modes | security | all exposed | profile/file | deny/no capability | required | public/sensitive | provider, issuer, audience, grants/policy compatibility | summarize | restart | fatal | 055 | local unsafe separately gated |
| DDIL mode/cache/validity | DDIL | ddil-single | profile/file | disabled | conditional | sensitive | all required local artifacts and stale rules | summarize | restart | fatal | 049/053–055 | no implicit fail-open |
| node identity/sync peer set | sync | sync-two-node | file | none/disabled | conditional | sensitive | unique identity/DB, peer trust and authority scopes | redact topology | restart | fatal | 054–056 | domain sync, not DB replication |
| telemetry/log/health settings | observability | all | profile/file/env | structured safe minimum | required | internal/sensitive | endpoint/auth/redaction/cardinality checks | bounded | log filter only | fatal/degrade by profile | 048 | no payload logging |
| deterministic test controls | test | CI/conformance only | fixture/profile | disabled | conditional | test-only | profile gate and seed/clock manifest | show in test evidence | immutable | fatal | 050/052/053 | rejected elsewhere |

### 11.2 API identity and proxy rules

The public base URL is an explicit deployment assertion, not reconstructed from arbitrary request headers. Direct deployments validate it against the listener; proxied deployments trust standardized forwarding data only from allowlisted peers and use the configured public origin for generated links, OpenAPI `servers`, redirects and auth callback/resource metadata. The path prefix is normalized once and shared by routing and link construction. CORS is a browser access policy, not authentication. **[A/E/P]**

### 11.3 Database separation

Use separate logical references and credentials for runtime application, migration, bootstrap, backup and restore roles. `serve` never auto-migrates. Configuration may require a schema compatibility range, but the database records the authoritative migration state. Missing extensions, newer/older incompatible schemas, identity mismatch or inability to acquire the explicit migration lock are fatal. Test reset and destructive fixture load require an ephemeral/test marker plus profile and database-name guards. **[A/E/P]**

---

## 12. Schema/Profile, Ingestion/Source, Streaming/Event, and Command/Control Configuration Findings

### 12.1 Schema, profile and vocabulary packages

Each standards resource is referenced through a manifest containing logical ID, standard/profile version, media type, content digest, source/provenance and local path. Resolution is local-first and deterministic. Online retrieval is disabled by default, allowlisted when enabled, size/time bounded, TLS-validated, cached by digest, and never silently substitutes a new version. DDIL and conformance profiles require complete local packages. Validation strictness is an enum tied to an accepted profile; it cannot suppress mandatory conformance checks. **[A/E/P]**

### 12.2 Ingestion and sources

Configuration selects installed ingestion adapters, global safety ceilings, accepted media types, quarantine storage, raw-artifact policy and source-registration mode. Individual source registrations, identities, grants, trust decisions and per-source state are administered domain records, not TOML arrays. Public demo accepts either no writes or explicitly authenticated, rate-limited, synthetic sources. Raw payload retention is off unless an accepted evidence purpose, classification, encryption/access and retention rule exist. Replay still passes idempotency, authorization, validation and audit. **[A/E/P]**

### 12.3 Streaming and events

SSE is part of the base HTTP capability and uses the durable outbox/replay contract. Broker configuration selects an adapter, authenticated endpoint, TLS trust, topic namespace template, delivery/retry limits and credential reference; it cannot redefine event identity, authorization or subscription semantics. Topic templates are validated against tenant/resource isolation and cannot interpolate untrusted raw identifiers. Slow-consumer, replay and backfill limits are explicit. MQTT/experimental Part 3 outbound publication remains profile-gated and cannot be advertised as implemented inbound publish/subscribe conformance. **[A/E/P]**

### 12.4 Commands

The schema currently permits only `disabled` and `simulated`. `simulated` requires a release-known simulator adapter, pinned safety-rule/policy bundle, authorization, operator-approval mode, audit availability, timeouts, cancellation rules, idempotency, target allowlist and network egress restriction. Discovery visibility is distinct from dispatch permission. Public demo and CI cannot reach a physical gateway even if a credential is accidentally mounted. Adding real dispatch requires a new schema version and accepted research/implementation gate; it cannot be enabled by an undocumented flag. **[A/E/P]**

---

## 13. Security, Authentication, Authorization, and Policy Configuration Findings

### 13.1 Authentication

Authentication modes are typed adapters with mode-specific required fields. OIDC validation requires exact issuer, expected audience, accepted algorithms, time-skew policy, HTTPS/JWKS trust, cache/refresh rules and explicit behavior during issuer unavailability. mTLS requires client-CA trust, identity mapping and revocation/freshness policy. API keys are test/bootstrap or explicitly accepted integrations, individually scoped and stored hashed when Glaux verifies them. Static bearer tokens are test-only. Authentication disabled is represented only as `local_unsafe`, never as a missing configuration. **[A/E/P]**

### 13.2 Authorization and policy

Authorization enforcement is always enabled for protected operations. Configuration chooses an implemented evaluator and policy bundle/reference, not whether an operation “needs” authorization. Required policy inputs and decision/audit schema versions bind to the release. External policy engines have authenticated endpoints, bounded timeouts, circuit behavior and a fail-closed rule; DDIL uses a locally verified, versioned policy with explicit validity/staleness semantics. A policy bundle update is restart-only in the first release and never activated until signature, schema, compatibility and negative tests pass. **[A/E/P]**

### 13.3 Trust and administrative exposure

Trust anchors are integrity-critical inputs even when public. Config distinguishes issuer trust, TLS roots, source trust bootstrap, policy verification, peer trust and package signatures instead of one global CA/key directory. Admin operations bind separately or remain route-disabled, require stronger authentication/authorization and never rely on obscurity. Public health, OpenAPI and conformance exposure are explicit per profile; detailed configuration, dependency and topology diagnostics are authenticated and redacted. **[E/P]**

### 13.4 Fail-closed rules

Expired/unavailable credentials, unverifiable policy, invalid trust, missing audit preconditions or unknown identity never enable privileged writes, ingestion, commands or synchronization. Read degradation is allowed only when an accepted profile defines local authority, cached-data semantics and response/audit labeling. Configuration cannot turn an `unknown` trust/policy state into `allow`. **[A/E/P]**

---

## 14. DDIL/Synchronization and Observability Configuration Findings

### 14.1 DDIL and degraded operation

`ddil-single` names local packages for schemas/vocabularies, identity verification, trust, policy, audit, outbox/replay and approved source buffers. Each cached authority artifact carries issuer, version, fetched/provisioned time, validity interval, integrity evidence and stale/expired behavior. Dependency reachability is runtime state, not a manually toggled “offline” truth. Operators may request a test fault state, but the server observes actual dependencies and reports connected, limited, intermittent, disconnected, local-only, recovering or unknown semantics from accepted IDR-SRV-042 rules. **[A/E/P]**

No runtime change may elevate authority during disconnection. Command mode remains separately controlled. Remote schema fetch is rejected. Time source/uncertainty, disk budget and retention ceilings are required simulation inputs; exact budgets remain later measured decisions. Reconnection does not hot-reload authority—it initiates verified refresh and synchronization workflows. **[A/E/P]**

### 14.2 Synchronization

`sync-two-node` uses separate complete config roots, node identities, database instances, secrets, public origins and audit/outbox state. Peer definitions supply logical peer ID, authenticated endpoint/trust, allowed scopes and protocol/capability version; conflict and authority policy remains accepted application logic. Gap limits, retry budgets and fixture fault schedules are test inputs, while peer watermarks/conflicts are runtime/domain state. Sharing a secret may be allowed only where the protocol requires a bounded pairwise credential; sharing node identity or DB is fatal. **[A/E/P]**

### 14.3 Observability handoff contract

Configuration must expose to IDR-SRV-048:

- structured log format and filter, service/node/run/release/profile/fingerprint resource attributes;
- request/correlation ID acceptance/generation rules;
- trace exporter endpoint, transport trust and credential reference;
- metrics listener/exposure/auth and bounded label policy;
- startup, liveness, readiness and dependency-check settings as distinct concepts;
- redaction classification registry and canary tests;
- authenticated support-bundle and effective-manifest controls; and
- debug/backtrace/panic settings forbidden from exposing payloads or secrets.

Telemetry export failure must not bypass business safety. Whether it blocks readiness depends on the profile's audit/operational requirement and will be fixed in IDR-SRV-048. Dynamic log filtering is the only first-release reload candidate; raw-body logging remains prohibited at every level. **[E/P]**

---

## 15. Test, CI, Conformance, Fixture, Public Demo, and Interoperability Configuration Findings

| Context | Required configuration evidence | Mandatory negative proof |
|---|---|---|
| unit/property tests | typed settings constructed in memory; schema/constraint version | domain/application code cannot read process environment or files |
| config test corpus | valid/invalid TOML, env allowlist, precedence and provenance snapshots | unknown/duplicate/ambiguous keys and every unsafe combination fail |
| CI integration | release/profile/source digests, redacted manifest, ephemeral secret refs, DB/migration/fixture IDs | no secret canary in logs/artifacts/image/Compose rendering |
| conformance | pinned capabilities, standards packages, fixture scenario, public origin | published conformance/OpenAPI cannot exceed active registry |
| fixtures/golden files | scenario manifest, deterministic clock/ID seed, provenance and expected validity | deterministic controls rejected outside test profiles |
| public demo | TLS origin, protected routes, limits, synthetic fixture/reset identity | commands/admin/unrestricted ingestion/local-unsafe all rejected |
| interoperability | external URL/path, media/capability set, auth test realm, client version | proxy links, redirects, OpenAPI servers and pagination never leak internal origin |
| DDIL/sync | per-node manifest, fault schedule, local package/trust versions | nodes cannot share identity/DB; authority does not elevate offline |
| performance | redacted manifest and workload/fixture identity | benchmark config cannot disable auth/validation/audit unless the benchmark explicitly measures that non-representative case |
| security/command | secret versions, trust/policy IDs and command simulator isolation evidence | physical egress target and real credentials absent |

CI invokes `config check`, records the canonical redacted manifest and fingerprint, and compares the expected profile contract before starting services. Configuration tests isolate process environment and run serially where mutation is unavoidable. Golden tests normalize paths and timestamps but do not erase semantically important provenance. Every example passes the executable validator in CI. **[D/E/P]**

Public examples use placeholders that cannot be mistaken for credentials. A sample may name `/run/secrets/db_app_password` but must not ship a value. Development bootstrap generates a new local value and records only its location. The repository includes secret-scanning hooks and CI, but prevention, review and rotation remain necessary because scanners are incomplete. **[E/P]**

---

## 16. Sample Configuration and Developer Documentation Recommendations

### 16.1 Repository artifacts

The implementation should provide:

- `config/schema/glaux-config-v1.schema.json` generated/verified against Rust types;
- read-only `config/profiles/<profile>.toml` for all eleven accepted profiles;
- `config/examples/operator.local.toml`, `operator.ci.toml`, `operator.demo-public.toml`, `operator.ddil-single.toml`, `operator.sync-node-a.toml` and `operator.operational-reference.toml`;
- `.env.example` containing only non-secret, local scalar examples and an explicit warning;
- `secrets/README.md` plus filename templates with no values;
- `docs/configuration-reference.md` generated from schema metadata;
- `docs/configuration-migrations.md` for schema-version changes; and
- Compose overlays that reference configuration and secret files without duplicating application defaults.

All examples must be validator-tested. A generated reference table lists key, type, class, profiles, default, source allowlist, constraints, reload policy, deprecation and related capability. **[E/P]**

### 16.2 Illustrative operator TOML

```toml
config_version = 1

[server]
public_base_url = "https://example.invalid/glaux"

[database]
host = "db"
name = "glaux"
user = "glaux_app"
password = { provider = "file", id = "db_app_password" }

[schemas]
package = { id = "ogc-baseline", digest = "sha256:<release-supplied-digest>" }
fetch = "disabled"

[commands]
mode = "disabled"
```

This is structural illustration, not a complete deployable file. The release profile supplies capability constraints; the operator file supplies deployment values. Secret contents and real infrastructure identifiers are absent. **[P/X]**

### 16.3 CLI contract

```text
glaux-server config check --profile <id> --config <path>...
glaux-server config explain --profile <id> --config <path>... <key>
glaux-server config render --redacted --format json ...
glaux-server config fingerprint ...
glaux-server config schema --version 1
```

`check` performs full static validation and optional explicitly requested dependency preflight. `explain` shows winning/shadowed source metadata without values for secret/sensitive fields. `render` is always redacted. Commands return stable exit categories suitable for CI. No generic key setter or secret-display mode exists. **[D/E/P]**

### 16.4 Change management

Every field change updates schema, documentation, examples, precedence/provenance tests and profile contracts. Breaking changes increment `config_version`; automatic migration creates a new file and preserves the original rather than silently rewriting it. Deprecated keys emit a bounded warning for one declared window but remain incompatible with their replacement appearing simultaneously. Release notes identify restart, migration, secret-rotation and capability implications. **[E/P]**

---

## 17. Downstream Topic Handoff Matrix

| Topic | Fixed input from IDR-SRV-047 | Still owned downstream |
|---|---|---|
| IDR-SRV-048 Observability | resource identity includes release/profile/fingerprint/run; schema-classified redaction; no bodies/secrets; distinct startup/liveness/readiness; log filter only reload candidate | signal schema, endpoints, SLOs, cardinality, exporters, retention and health dependency semantics |
| IDR-SRV-049 Migration/backup | separate app/migrate/bootstrap/backup secret refs and roles; no serve auto-migrate; schema compatibility gate; config versioning | migration locking/roll-forward/rollback, backup formats, schedules, RPO/RTO and restore runbooks |
| IDR-SRV-050 Conformance | explicit capability registry, pinned standards packages, deterministic profile/fixture manifest and external origin | harness selection, executable test mapping and evidence format |
| IDR-SRV-052 Rust testing | composition-root-only loading; typed immutable config; provider ports; invalid corpus; environment isolation | test pyramid, crate-level strategy, property/fuzz/contract mechanics |
| IDR-SRV-053 Fixtures | no secrets; versioned scenario manifests; deterministic clock/ID controls test-profile-only | corpus structure, provenance, generators, golden update governance |
| IDR-SRV-054 Performance | every result carries redacted config fingerprint; bounded queues/limits configured | workloads, budgets, thresholds, profiling and stress methodology |
| IDR-SRV-055 Security/commands | unsafe-combination register, secret/redaction/rotation contract, commands disabled/simulated only | attack cases, authorization matrices, command safety verification and secret-provider tests |
| IDR-SRV-056 Interoperability | public origin/path/capabilities/auth realm explicit; proxy and profile evidence | external client/server matrix, protocol cases and result governance |
| implementation/release | Figment-wrapped typed candidate, TOML v1, explicit profiles, fixed precedence, SecretProvider, effective manifest | code, dependency pin, schema generation, CLI and deployment artifacts |

### 17.1 Ordered implementation proofs

1. deny-unknown typed extraction and stable error paths;
2. exact precedence and per-value provenance across profile/file/env/CLI;
3. all eleven profile constraint snapshots and negative combinations;
4. mounted-file secret resolution without diagnostic leakage;
5. canary redaction across logs, errors, traces, metrics and support output;
6. deterministic canonical redacted manifest/fingerprint;
7. public-origin/proxy/CORS link identity tests;
8. distinct app/migration credentials and schema compatibility behavior;
9. capability registry consistency with routes/OpenAPI/conformance;
10. atomic credential/key-ring/certificate rotation prototypes;
11. DDIL local-package/validity/dependency-loss configuration cases; and
12. two-node configuration isolation and restart/recovery evidence.

No general hot reload, external secret manager or real command adapter should precede these proofs. **[E/P]**

---

## 18. Recommendations

1. Adopt a single versioned Serde configuration schema, TOML authoring format and composition-root loader. **[P]**
2. Use Figment 0.10.19 as the current candidate behind a Glaux-owned wrapper because profiles, provider ordering, typed extraction and provenance match the design; revalidate version/MSRV/license during implementation. **[D/E/P]**
3. Require explicit CLI profile selection and enforce the eleven accepted profiles as capability/safety constraints. **[A/P]**
4. Fix precedence to safe compiled defaults, release profile, ordered operator files, allowlisted scalar environment overrides and narrow non-secret CLI overrides. **[P]**
5. Reject unknown fields and environment keys, ambiguous array/object overrides, unsupported schema versions and every registered unsafe combination before effects. **[P]**
6. Keep database/admin/domain state and Cargo compile features outside startup-configuration precedence. **[P]**
7. Separate secret references from values; use mounted files first and a provider port for future integrations. **[D/E/P]**
8. Use redacting/non-serializing Rust secret wrappers with explicit exposure and zeroization, while documenting their limits. **[D/P]**
9. Make configuration restart-only except a bounded audited log-filter control; implement secret rotation as versioned adapter/key-ring replacement. **[P]**
10. Emit a canonical redacted effective manifest and fingerprint with provenance and secret-version metadata but no raw or content-hashed secrets. **[P]**
11. Generate and CI-validate schema, references, all profile/examples, precedence cases, unsafe combinations and redaction canaries. **[P]**
12. Carry the exact contracts in Section 17 into IDR-SRV-048 through 056, beginning only with IDR-SRV-048 after acceptance. **[P]**

---

## 19. Risks, Constraints, and Open Questions

### 19.1 Risk register

| Risk | Consequence | Control / proof |
|---|---|---|
| permissive local setting reaches exposed deployment | unauthorized access/effects | explicit profile, constraint matrix, loopback and negative deployment tests |
| hidden source precedence | unreproducible behavior | fixed source order, provenance, `explain`, redacted manifest |
| secret enters ordinary config/log | credential compromise | references only, typed classification, wrapper, canary scans and rotation drill |
| redaction relies on key names | missed leak or excessive hiding | schema metadata plus output-specific allowlists |
| reload creates mixed state | inconsistent authorization/routes/adapters | restart-only baseline; future validate-build-atomic-swap proof |
| content hash leaks low-entropy secret | offline guessing | record provider version, never secret digest |
| stale DDIL trust/policy silently remains authoritative | unauthorized operation | validity metadata, explicit stale rules and fail-closed effects |
| config permits capability unsupported by routes/artifacts | false conformance/interoperability | one capability registry and startup consistency test |
| operator file becomes shadow database | bypassed governance | keep sources/grants/policies/subscriptions in administered state |
| multiple syntax/providers expand attack surface | parser drift and ambiguity | TOML-only first release, allowlisted env scalars, one secret ref per value |
| zeroization overclaimed | false assurance | document copies/swap/core limits; minimize lifetime and privilege |
| secret rotation breaks old signatures/data | verification or availability loss | versioned key rings and retention-aware migration tests |

### 19.2 Constraints

- All sample identifiers and URLs are non-operational placeholders.
- No direct environment secret values are required for minimal operation; mounted files suffice.
- Profiles constrain but do not replace per-environment threat assessment and authorization.
- Cross-platform file permission/symlink guarantees vary and must not be overstated.
- Exact operational limits and lifetimes require measured and organizational policy inputs.
- This report cannot authorize inbound Part 3, physical commands or new conformance claims.

### 19.3 Open questions with owners

| Question | Default until resolved | Owner |
|---|---|---|
| should any telemetry control beyond log filter reload? | no; restart | IDR-SRV-048 |
| what startup dependency failures make readiness false versus terminate? | profile-required invariant failure terminates; transient post-start failures degrade | IDR-SRV-048 |
| how are config schema changes coordinated with DB migrations? | independently versioned and compatibility-checked | IDR-SRV-049 |
| which exact timeouts/limits/pool sizes ship? | conservative reviewed profile values, then measured | IDR-SRV-048/054 |
| which external secret provider is supported first? | none required; mounted-file port | implementation/environment decision |
| are provider lease renewals in-process or restart-triggering? | adapter-specific prototype required; fail closed at expiry | implementation/security testing |
| how long are prior signing/HMAC keys retained? | at least accepted verification/replay/data horizon | IDR-SRV-049/055 |
| should custom operator-defined profiles exist? | no in first release | future governance decision |

---

## 20. Validation Against This Plan's Success Criteria

| Success criterion | Result | Evidence |
|---|---|---|
| runtime profiles with anchors and traceability | Met | Sections 3 and 5 |
| categories, precedence, immutability/reload, validation and fail-safe behavior | Met | Sections 6–8 |
| secret/sensitive inventory with handling, redaction and rotation | Met | Sections 9–10 |
| API, DB, schema, ingestion, events, commands, security, DDIL, sync, observability and tests | Met | Sections 11–15 |
| unsafe profile combinations and insecure defaults | Met | Sections 5.2, 8.2 and 11–15 |
| sample configuration and documentation needs | Met | Section 16 |
| implementation/community lessons as non-normative evidence | Met | Section 3.3 |
| decision-usable and server-bounded recommendations | Met | Sections 2 and 18 |
| explicit downstream handoffs | Met | Section 17 |
| explicit reproducible references | Met | Section 21 |

All planned phases and required report content are complete. Acceptance remains a project-lead action. **[P]**

---

## 21. References

### 21.1 Current configuration and secret sources

- Figment 0.10.19 documentation: https://docs.rs/figment/0.10.19/figment/
- config 0.15.25 `Environment`: https://docs.rs/config/0.15.25/config/struct.Environment.html
- clap 4.6.7 documentation: https://docs.rs/clap/4.6.7/clap/
- dotenvy 0.15.7 documentation: https://docs.rs/dotenvy/0.15.7/dotenvy/
- secrecy 0.10.3 documentation: https://docs.rs/secrecy/0.10.3/secrecy/
- zeroize 1.9.0 documentation: https://docs.rs/zeroize/1.9.0/zeroize/
- Twelve-Factor App, Config: https://12factor.net/config
- Docker Compose environment precedence: https://docs.docker.com/compose/how-tos/environment-variables/envvars-precedence/
- Docker Compose secrets: https://docs.docker.com/compose/how-tos/use-secrets/
- Docker secrets: https://docs.docker.com/engine/swarm/secrets/
- Kubernetes ConfigMaps: https://kubernetes.io/docs/concepts/configuration/configmap/
- Kubernetes Secrets: https://kubernetes.io/docs/concepts/configuration/secret/
- Kubernetes secret good practices: https://kubernetes.io/docs/concepts/security/secrets-good-practices/
- OWASP Secrets Management Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html
- Mozilla SOPS documentation: https://github.com/getsops/sops
- HashiCorp Vault documentation: https://developer.hashicorp.com/vault/docs

### 21.2 Controlling standards and security guidance

- OGC API - Connected Systems: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems Part 1: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems Part 2: https://docs.ogc.org/is/23-002/23-002.html
- SensorML 3.0: https://docs.ogc.org/is/23-000/23-000.html
- SWE Common 3.0: https://docs.ogc.org/is/24-014/24-014.html
- OpenAPI Specification: https://spec.openapis.org/oas/latest.html
- RFC 9110, HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 7239, Forwarded HTTP Extension: https://www.rfc-editor.org/rfc/rfc7239
- NIST SP 800-53 Rev. 5: https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final
- NIST SP 800-92: https://csrc.nist.gov/pubs/sp/800/92/final

### 21.3 Accepted project evidence

- Glaux Server Overall IDR Research Plan: [overall-idr-research-plan.md](../IDR%20Plans/overall-idr-research-plan.md)
- IDR-SRV-047 Research Plan: [idr-srv-047-configuration-secrets-and-environment-strategy.md](../IDR%20Plans/idr-srv-047-configuration-secrets-and-environment-strategy.md)
- Glaux Server Goal and Definition: [glaux-server-goal-and-definition.md](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- Accepted IDR-SRV-001 through IDR-SRV-046 reports: [IDR Reports](./)
- IDR-SRV-046 Reference Deployment Strategy: [idr-srv-046-reference-deployment-strategy-report.md](idr-srv-046-reference-deployment-strategy-report.md)
- Research Report Template: [research-report-template.md](../../../../../Governance/research-report-template.md)

### 21.4 Non-normative implementation and community evidence

- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- OGC API Connected Systems development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- Accepted IDR-SRV-014A through IDR-SRV-014G reports: [IDR Reports](./)

### 21.5 Reproducibility and evidence limits

Official web and crate documentation was checked on September 16, 2026. Version numbers record the research freeze and must be revalidated through the dependency/license/security process at implementation. Product guidance was used for mechanism evidence only. Implementation and community studies remain informative. No Glaux configuration implementation, live secret-provider integration, reload stress test or rotation drill existed to inspect; those are explicit proof obligations in Section 17.

---

## Report Completion Checklist

- [x] Executive summary and scope complete
- [x] Evidence and authority classes explicit
- [x] Eleven runtime profiles constrained
- [x] Configuration categories, sources, precedence and merge rules defined
- [x] Startup, validation, effective manifest and reload boundary defined
- [x] Secret inventory, loading, redaction, rotation and test safety defined
- [x] Required configuration matrix complete
- [x] Feature-specific and unsafe-combination findings complete
- [x] Sample/documentation/CLI guidance complete
- [x] Downstream handoffs and implementation proofs complete
- [x] Plan success criteria validated
- [x] References and evidence limits explicit
- [ ] Accepted by Glaux Project Lead

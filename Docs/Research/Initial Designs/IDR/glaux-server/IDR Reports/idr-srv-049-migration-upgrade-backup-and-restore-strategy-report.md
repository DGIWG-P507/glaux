# Section 049: Migration, Upgrade, Backup, and Restore Strategy - Research Report

**Topic ID:** IDR-SRV-049<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-049 Migration, Upgrade, Backup, and Restore Strategy](../IDR%20Plans/idr-srv-049-migration-upgrade-backup-and-restore-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** Lifecycle responsibility; continuity inventory; schema/data migrations and backfills; compatibility/versioning; bootstrap/fixtures/reset; coherent backup; isolated restore/validation; audit, command, event, observation, trust, policy, DDIL and synchronization continuity; threats, telemetry, tests and downstream handoffs<br>
**Methodology Used:** Accepted-requirement extraction; current primary-source database/tool review; authority/rebuildability and lifecycle matrixing; failure/recovery and lineage analysis; high-risk effect fencing; implementation-lesson reconciliation; verification and handoff synthesis<br>
**Research Time:** Approximately 40 hours of AI-assisted execution on September 16, 2026<br>
**Accepted Runtime Baseline:** One PostgreSQL/PostGIS authoritative write core; exact artifacts separate from canonical state; durable inbox/outbox/audit references; eleven deployment profiles; explicit same-image administrative commands; typed configuration/release fingerprints; distinct health/readiness and observability signals
**Lifecycle Evidence Freeze:** PostgreSQL 18.6/current Version 18 backup, `pg_dump`, `pg_restore`, `pg_basebackup`, `pg_verifybackup`, `pg_upgrade`, checksum and monitoring documentation; PostGIS 3.6.x upgrade documentation; SQLx 0.9.0 migrations; Docker volume documentation; official sources checked September 16, 2026
**Standards Baseline:** OGC API - Connected Systems Parts 1 and 2 Version 1.0; SensorML 3.0; SWE Common 3.0; RFC 9110 and 9457; accepted Glaux IDR-SRV-001 through IDR-SRV-048
**Document Purpose:** Define bounded, testable Glaux lifecycle and continuity contracts without implementing them, selecting enterprise backup infrastructure, or claiming production disaster recovery
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

“Backup,” “restore,” “upgrade,” “recovery,” “consistent” and “operational-reference” are bounded design terms. They do not imply tested RPO/RTO, high availability, disaster-recovery accreditation, archive compliance or cross-domain transfer approval. **[X]**

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Continuity Requirement Extraction Methodology
5. Migration/Upgrade/Backup/Restore Scope Boundary
6. Data Continuity Inventory
7. Database Migration and Data Migration Findings
8. Upgrade Compatibility, Versioning, OpenAPI, and Conformance Artifact Findings
9. Bootstrap, Seed Data, Fixture Reset, and Demo Reset Findings
10. Backup Strategy Findings
11. Restore Strategy and Post-Restore Validation Findings
12. Audit, Command/Control, Event/Outbox, Observation/Status, and Latest-Value Continuity Findings
13. Source Trust, Policy/Releasability, DDIL, Synchronization, and Conflict Continuity Findings
14. Security and Threat-Mitigation Findings
15. Observability, CI, Conformance, Performance, Security Testing, and Interoperability Implications
16. Downstream Topic Handoff Matrix
17. Recommendations
18. Risks, Constraints, and Open Questions
19. Validation Against This Plan's Success Criteria
20. References

---

## 1. Executive Summary

Glaux should use **forward-only, immutable SQLx migrations executed by an explicit same-image `migrate` command**, never automatically by the serving process. Each release embeds a supported database-schema range and an immutable, checksummed migration set. One migration runner acquires a database-scoped advisory lock, validates the applied ledger and extension versions, creates a pre-change recovery point, executes reviewed migrations, runs structural and semantic verification, records a lifecycle evidence manifest, and only then permits the new server to become ready. `serve` fails closed on pending, dirty, missing, edited or incompatible migrations. Compose/CI may orchestrate the one-shot job automatically, but that is deployment sequencing rather than startup auto-migration. **[A/D/E/P]**

Migration files are never edited after release. Most schema changes are transactional; non-transactional index/extension operations are isolated, resumable and explicitly marked. Destructive down migrations are not the operational rollback strategy. Rollback means stop/fence the new release and restore the tested pre-upgrade backup with the compatible old image/config/artifacts, subject to a declared point of no return and reconciliation of any post-upgrade writes. For future rolling upgrades, expand/backfill/dual-read-or-write/contract sequences provide an explicit compatibility window; the first reference deployment may instead use a measured maintenance window. **[D/E/P]**

Continuity follows authority. Canonical resources, identities/aliases/lifecycle/tombstones, observations/status, source/trust/policy records, commands, audit, validation references, outbox/event log, idempotency/inbox, synchronization/conflict state and artifact metadata must be backed up coherently. Exact source documents/raw payloads held outside PostgreSQL are included by digest and storage reference. Latest-value/search/materialized projections and indexes are rebuildable only when their authoritative inputs and algorithms are preserved. Release-bundled schemas, profiles, vocabularies and generated OpenAPI are restored from the exact release manifest, not treated as mutable database truth. Ephemeral caches, telemetry and test data are disposable only in profiles that explicitly say so. **[A/E/P]**

The first portable backup proof uses a quiesced **PostgreSQL custom-format logical archive** plus a content-addressed artifact export and signed/encrypted `GlauxBackupManifestV1`. Quiescing rejects new writes, ingestion, command submissions/dispatch, publication claims and synchronization apply; drains in-flight transactions; clears/records leases; establishes outbox/audit/artifact high-water evidence; then captures a consistent database dump and required artifacts/config/release metadata. `pg_dump` provides a consistent single-database snapshot and portable newer-version restore path, but PostgreSQL explicitly cautions that it is generally not the right sole mechanism for regular production backup. Operational-reference design therefore requires infrastructure-owned physical base backups plus continuous WAL archiving/PITR, periodic logical portability exports, offsite/retention/encryption/key management and restore drills. This report selects no vendor and sets no RPO/RTO. **[D/E/P/X]**

Every restore occurs into a fresh, isolated target with external network/effects disabled. The process verifies signature/digest, provenance and PostgreSQL/PostGIS compatibility before executing archive content; PostgreSQL warns that restoring a dump executes source-superuser-chosen code. It restores with explicit ownership/privilege mapping, reacquires secrets separately, validates migration/config/release compatibility, checks structural and semantic invariants, rebuilds projections/indexes, reconciles outbox/commands/audit/sync, and runs smoke/conformance/security probes. Activation is a separate authorized cutover with fencing. A recovery replacement may retain logical node identity only after the old node is proven fenced; a clone/test/demo restore receives a new node identity, origin, credentials and lineage. **[D/E/P]**

Restored command rows are history, never instructions to redispatch. Completed/cancelled/expired states remain terminal; dispatched or ambiguous states become/reopen an explicit unknown-outcome reconciliation workflow; merely queued work requires policy/safety re-evaluation before any later effect. Publication is at-least-once around failure: stable event/delivery identities and consumer deduplication allow expired claims to retry, while a restore epoch prevents stale workers. A point-in-time restore creates an acknowledged lineage gap for events/audit/synchronization after the recovery point; Glaux never fabricates continuity or silently rewinds cursors. **[A/E/P]**

Reset, fixture and bootstrap tools are deliberately separate. `bootstrap` creates only required idempotent structural/system state. `fixture apply` consumes a versioned scenario manifest. `reset` works only against a database marked ephemeral/test/demo and a matching allowed profile, requires destructive confirmation, and is rejected for operational-reference. CI destroys named ephemeral volumes; the public demo rebuilds from a synthetic, immutable fixture pack and new lineage rather than issuing an unconstrained truncate. No backup, fixture or manifest contains secrets. **[A/E/P]**

This report closes Category H research execution but does not accept it, authorize Category I, create migrations/backups, select an enterprise platform, set retention/RPO/RTO, approve cross-domain movement, enable physical commands/inbound Part 3, or claim disaster recovery/conformance/accreditation. **[P/X]**

---

## 2. Scope and Plan Alignment

### 2.1 Coverage

| Plan area | Coverage | Evidence location |
|---|---|---|
| responsibility boundary and traceability | Complete | Sections 3–5 |
| authoritative/derived/cache/mutable/immutable inventory | Complete | Section 6 |
| schema/data/backfill/rebuild strategy | Complete | Section 7 |
| version/API/OpenAPI/conformance compatibility | Complete | Section 8 |
| bootstrap/fixture/reset | Complete | Section 9 |
| coherent backup and operational escalation | Complete | Section 10 |
| isolated restore and validation | Complete | Section 11 |
| high-risk audit/command/event/data/trust/DDIL/sync continuity | Complete | Sections 12–13 |
| threats, tests and downstream handoffs | Complete | Sections 14–16 |

### 2.2 In scope

- Glaux database and content migration contracts;
- compatibility gates among image, database, configuration and standards artifacts;
- idempotent bootstrap and profile-gated destructive reset;
- reference logical backup, multi-store coherence manifest and restore pipeline;
- application-level fencing/reconciliation after upgrade or restore;
- required infrastructure handoffs for physical/PITR/offsite continuity;
- lifecycle telemetry/audit and verification obligations; and
- Category I inputs.

### 2.3 Out of scope

- enterprise backup product, cloud provider, storage array, orchestrator or key manager;
- production schedules, retention, geographic copies, RPO/RTO, HA/failover topology and staffing;
- PostgreSQL cluster administration runbooks beyond server-visible contracts;
- cross-domain transfer, records-management/legal archive or accreditation;
- final fixture corpus, conformance harness or test implementation;
- actual migrations, destructive operations, live data, physical commands or inbound Part 3. **[X]**

### 2.4 Accepted invariants

Continuity must preserve stable identities/URIs, aliases, lifecycle/tombstones, provenance, policy-authorized views, temporal semantics, exact-artifact digests, source trust, command/audit history, outbox/idempotency, DDIL evidence and synchronization conflicts. It must respect one PostgreSQL/PostGIS authority, separate exact artifacts, explicit administrative roles/secrets, release/profile/configuration fingerprints, role-aware readiness, telemetry/audit separation and deterministic CI/deployment profiles. **[A]**

---

## 3. Evidence Base and Authority Classification

### 3.1 Current primary evidence

| Source | State checked | Evidence used | Limitation |
|---|---|---|---|
| PostgreSQL backup docs | Version 18/current; 2026-09-16 | logical, filesystem and continuous-archive approaches | mechanism; deployment still owns policy/topology |
| `pg_dump`/`pg_restore` | Version 18/current | consistent single-DB export, custom/directory archives, newer-version portability, parallelism, restore trust warning | not a coherent multi-store snapshot or sole production backup |
| `pg_basebackup`/WAL/PITR | Version 18/current | physical full/incremental bases, WAL continuity, timelines and point-in-time recovery | version/platform-specific and infrastructure-operated |
| `pg_verifybackup`/manifest/checksums | Version 18/current | file/WAL presence and checksum verification | cannot replace actual restore and semantic validation |
| `pg_upgrade` | Version 18/current | major-version checks and copy/clone/link/swap tradeoffs | extension compatibility and application semantics remain external |
| PostGIS documentation | 3.6.x/current | extension upgrade and dump/restore hard-upgrade paths | exact path depends on installed versions/features |
| SQLx migrations | 0.9.0 | embedded/directory migrations, immutable ledger validation, locking and run/undo mechanisms | Glaux policy chooses forward-only and external orchestration |
| Docker volumes | current 2026-09-16 | persistent volume mechanics and generic copy examples | raw live DB volume copy is not automatically crash-consistent |

PostgreSQL states that `pg_dump` creates a consistent export without blocking readers/writers and can generally load into newer servers, while also warning it is generally not the right sole regular-production backup. It also warns restores execute arbitrary source-superuser-selected code. Glaux therefore uses logical archives for portable reference proof and inspection, but requires trusted provenance and escalates operational continuity to tested physical/WAL infrastructure. **[D/E/P]**

### 3.2 Accepted project evidence

IDR-SRV-001 through 024 define standards, stable API behavior, aliases, lifecycles and compatibility. IDR-SRV-025 through 030 define PostgreSQL/PostGIS authority, documents, time series, atomic transactions, idempotency/outbox and lifecycle. IDR-SRV-031 through 043 define ingestion, events, commands, security, policy, audit, DDIL and synchronization. IDR-SRV-044 through 048 define Rust/SQLx, modular boundaries, deployment, configuration/secrets and lifecycle telemetry. Those accepted semantics govern what recovery must preserve. **[A]**

### 3.3 Non-normative implementation/community lessons

| Lesson | Evidence class | Glaux consequence |
|---|---|---|
| initialization hidden in server startup produces drift/failure coupling | CS-Go/pygeoapi studies | explicit migration/bootstrap jobs and readiness gate |
| container entrypoint initialization only handles an empty volume | deployment/official-image study | never use `/docker-entrypoint-initdb.d` as migration system |
| mutable demos disappear or lose reproducibility | OSH/SECD/client studies | immutable scenario pack plus destroy/recreate/reset evidence |
| OpenAPI/routes/conformance change independently from data | multiple implementations | release compatibility manifest and post-upgrade contract suite |
| sample credentials/data can escape into deployments | implementation/community studies | synthetic fixtures, secret exclusion and profile/database reset fences |
| undocumented schema shortcuts are implementation-specific | all studies | accept lessons as failure evidence, never normative requirements |

### 3.4 Evidence limits

- No Glaux schema, migration history, data volume or artifact store exists to benchmark.
- No backup has been restored; all RPO/RTO/duration/size claims remain open.
- PostgreSQL/PostGIS point versions are a dated evidence freeze, not automatic release pins.
- A cryptographic digest proves bytes, not trust, semantic completeness or recoverability.
- Logical backup cannot alone provide a consistent point across external artifact/config stores without an application protocol.

---

## 4. Continuity Requirement Extraction Methodology

The research used eight passes:

1. inventory every accepted persisted fact, artifact, projection, cache and release input;
2. assign authority, mutability, lineage, sensitivity and rebuildability;
3. trace create/update/delete/tombstone/outbox/audit transactions;
4. analyze empty-to-current and supported-prior-to-current migration paths;
5. trace quiesce, backup, corruption, restore, validation, reconciliation and cutover;
6. challenge command, audit, event and sync paths for accidental replay/rewind;
7. separate server contracts from deployment infrastructure and policy; and
8. turn unmeasured capacity/time assumptions into Category I proof obligations.

### 4.1 Continuity questions per item

| Dimension | Required answer |
|---|---|
| authority | is it canonical fact, exact evidence, derived projection, cache, release input or ephemeral state? |
| lineage | which node/release/backup epoch created it, and can a clone retain that identity? |
| consistency | which DB transaction/artifact commit/outbox/audit boundary must match? |
| recovery | restore bytes, rebuild deterministically, reacquire, reconcile or discard? |
| compatibility | which application/config/schema/extension/artifact versions can read/write it? |
| security | classification, secrets, policy/trust and restore-code/provenance threat? |
| verification | structural, referential, semantic, integrity, effect-fencing and external test? |
| profile | preserved, synthetic/resettable or forbidden? |

### 4.2 Recovery truth rule

Recovery never infers a stronger fact than the surviving evidence. Missing post-recovery observations remain missing; dispatched command outcome remains unknown until authoritative reconciliation; truncated audit/event/sync timelines are declared gaps; stale trust/policy stays stale; a rebuilt projection cites its inputs/algorithm. **[A/E/P]**

---

## 5. Migration/Upgrade/Backup/Restore Scope Boundary

### 5.1 Glaux Server responsibilities

- ship immutable ordered application migrations and compatibility metadata;
- provide explicit `migrate`, `bootstrap`, `fixture`, `backup manifest`, `restore validate`, `rebuild` and `reconcile` operations;
- acquire migration locks and expose/read authoritative schema version;
- fence effects and produce a consistent application backup epoch/high-water manifest;
- classify authoritative versus rebuildable state and enumerate referenced artifacts;
- validate restored semantics, links, projections, outbox, commands, audit and synchronization;
- expose role-aware readiness and protected lifecycle diagnostics;
- record lifecycle audit/telemetry without leaking data/secrets; and
- refuse unsafe profile, version, lineage or partial-restore combinations. **[A/E/P]**

### 5.2 Deployment/infrastructure responsibilities

- schedule, retain, encrypt, replicate and expire backup objects;
- operate PostgreSQL physical base backup/WAL/PITR, storage snapshots and major-version infrastructure;
- protect backup keys and repositories with least privilege/immutability where required;
- provision fresh databases, PostGIS binaries/extensions, object storage and secrets;
- define/test RPO/RTO, regional/offsite copies, capacity and disaster runbooks;
- fence old instances and authorize cutover/failback;
- manage legal records/archive/cross-domain transfer and incident response; and
- monitor backup execution/age and conduct periodic independent recovery exercises. **[D/E/P/X]**

### 5.3 Shared contract

Infrastructure receives a machine-readable lifecycle manifest and invokes versioned same-image commands. Glaux does not shell out to an enterprise backup product from request handlers. Infrastructure does not bypass application reconciliation by declaring a volume restored. Success requires both storage-level integrity and application-level validation. **[E/P]**

### 5.4 Lifecycle states

`normal -> draining -> quiesced -> backup|migrating|restoring -> validating -> reconciling -> ready` is the common control model. A terminal failure enters `recovery_required` and remains not-ready. Each transition requires an authorized operation, lease/fence token, persisted lifecycle record and audit event; process death does not erase the state. Only the owning administrative command advances it. **[E/P]**

---

## 6. Data Continuity Inventory

### 6.1 Required continuity matrix

| Data/resource category | Authority class | Persistence location | Migration requirement | Backup requirement | Restore requirement | Rebuildability | Validation check | Security/policy/audit implication | Profiles | Test/conformance implication | Handoff | Notes/open |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| resource IDs, URI aliases, relationships, lifecycle, tombstones | canonical mutable/history | PostgreSQL | additive schemas; never silently re-key | mandatory coherent DB | preserve IDs/links/versions/deletion semantics | not rebuildable | uniqueness, FK, URI/link traversal, lifecycle invariants | policy/existence concealment and provenance retained | all persistent | API/interop golden identity | 050/053/056 | ID translation requires explicit alias provenance |
| Systems/Deployments/Procedures/Sampling Features | canonical domain | PostgreSQL plus artifact refs | versioned model/backfill | mandatory | restore canonical state and refs | not rebuildable | domain/profile/relationship validation | markings/policy bindings preserved | all persistent | representative conformance set | 050/053 | no regeneration from documents alone |
| exact SensorML/SWE/source/raw artifacts | exact evidence | content-addressed object/file store plus DB metadata | metadata/schema changes; bytes immutable | mandatory when referenced/retained | verify digest/size/media/provenance before activation | not rebuildable unless trusted source exists | digest, reference closure, parser/profile validation | high sensitivity; policy/retention; no public fixture leakage | enabled profiles | artifact corruption/missing cases | 053/055 | multi-store coherence required |
| Datastream/ControlStream schemas and encodings | canonical versioned | PostgreSQL plus exact schema artifacts | compatibility/backfill explicit | mandatory | preserve version and active binding | not safely inferred | SWE/profile/unit/encoding compatibility | affects validation/commands | applicable | schema-evolution corpus | 050/053/056 | old observations/commands bind old schema |
| observations/status/system events | canonical time-series/history | PostgreSQL | partition/type/index/backfill paths | mandatory | preserve IDs, event/result time, provenance/order | not rebuildable generally | counts/ranges/duplicates/time/unit/link invariants | policy/source provenance and retention | dynamic profiles | temporal/perf/interop | 053/054/056 | PITR may create explicit tail gap |
| latest/current/materialized/search projections | derived | PostgreSQL materialized tables/views/indexes | rebuild algorithm/version | optional if cheaper; inputs mandatory | invalidate and rebuild before ready | yes, if complete authoritative inputs | compare sampled/full oracle, watermark and algorithm version | authorized-view rules still apply | serving profiles | rebuild determinism/performance | 052–054 | never treated as backup authority |
| source registration/trust/credential metadata | canonical security state | PostgreSQL; secret values external | versioned state/backfill | mandatory metadata; exclude secrets | preserve trust history; reacquire secret; revalidate freshness | not rebuildable | identity/version/status/expiry/authority rules | highly sensitive; audit access/change | ingestion/DDIL | adversarial stale/revoked cases | 053/055 | restored trust cannot become fresher |
| policy assertions/bindings/bundle metadata | canonical decision input | PostgreSQL plus signed package refs | no semantic downgrade; compatibility gate | mandatory metadata/packages | verify signatures/versions/validity | not rebuildable | binding closure, signature, policy-schema and stale-state | controlled-data and releasability | protected/DDIL | policy regression/security | 053/055 | decision cache is disposable |
| command/feasibility/status/result records | canonical safety/accountability | PostgreSQL plus artifact refs | lifecycle mapping explicit | mandatory | restore as history; fence/reconcile nonterminal | not rebuildable | transition graph, audit refs, dispatch fences, expiry | highest effect/replay risk | command-sim/current future | no-redispatch and unknown outcome | 055/056 | physical effects remain external |
| inbox/idempotency records | authoritative dedupe window | PostgreSQL | preserve key/version/expiry semantics | mandatory within accepted window | restore before accepting writes | partially expiry-governed, not recomputable safely | uniqueness/request-hash/outcome linkage | replay protection and privacy | write profiles | duplicate/retry tests | 052/055 | PITR rewinds remembered requests |
| outbox/durable event/replay state | authoritative publication intent/history | PostgreSQL | event schema/envelope migration | mandatory | stable IDs; expire leases; reconcile pending/acked | event may derive from fact only if contract says so | DB fact/outbox/audit atomicity, cursor/high-water | policy filter and disclosure history | streaming/sync | duplicate/gap/replay tests | 054/056 | at-least-once after restore |
| authoritative audit journal/checkpoints | append-only evidence | protected PostgreSQL/store plus external anchors | version-preserving reader/upcaster; no rewrite | mandatory plus checkpoints | restore verified prefix; declare truncation/gap; new recovery event | not rebuildable | chain/checkpoint/custody/sequence/reference checks | access/custody/retention; restore itself audited | protected profiles | integrity/rollback tests | 055 | telemetry is not backup |
| validation artifacts/quarantine metadata | evidence/history | PostgreSQL plus artifact refs | schema/version migration | per accepted retention and authority | restore refs/content or declare unavailable per policy | some can revalidate if exact inputs/version kept | digest, validator/profile/result consistency | may contain controlled invalid data | ingestion/test | invalid corpus/provenance | 053/055 | do not rebuild with newer validator silently |
| sync envelopes/watermarks/gaps/conflicts/quarantine | canonical continuity state | PostgreSQL plus exact artifacts | protocol/state-machine compatibility | mandatory | restore node lineage; mark peer divergence; resnapshot as needed | not safely rebuildable from DB replication | peer scope, sequence, dedupe, conflict/tombstone links | trust/authority/policy evidence | sync-two-node/DDIL | partition/PITR recovery | 053–056 | old node must be fenced |
| schemas/profiles/vocabularies/OpenAPI/conformance manifests | immutable release input/generated output | signed release/package store; optional cache index in DB | package/config compatibility, cache index migration | preserve exact release/package digest; cache optional | reinstall exact package; regenerate output and compare digest | yes from trusted release input | signature/digest, supported standard/profile/capability consistency | supply-chain/trust critical | all | contract/conformance | 050/056 | mutable remote latest prohibited |
| configuration files/effective manifest | deployment input/evidence | operator/release store, redacted fingerprint in DB/evidence | config-version tool separate from DB | operator backs files; backup manifest stores only safe identity | provide compatible config; reacquire secrets | values not inferred from DB | `config check`, profile/lineage/origin/secret ref status | secrets excluded; topology sensitive | all | profile/negative tests | 052/055 | no raw effective secret-bearing dump |
| telemetry, sessions, caches, temporary leases | ephemeral/derived | memory/tmp/telemetry backend | none except schema compatibility | no application backup | discard/recreate; expire leases | yes/not needed | startup/runtime checks | never substitute for audit/domain state | all | restart/fault | 052/054 | selected external telemetry retention separate |
| fixtures/demo synthetic state | scenario authority only | versioned fixture package and disposable DB | manifest/version migration | package in release; DB disposable | rebuild fresh with new demo/test lineage | yes from package | expected manifest/count/link/conformance checks | synthetic only; no reusable secrets | dev/CI/demo/test | golden/scenario corpus | 050/053 | never overlay operational DB |

### 6.2 Authority versus convenience

An index being rebuildable does not mean its inputs are disposable. A document being parseable does not make it sufficient to reconstruct accepted canonical identity, lifecycle, policy or transaction history. A remote source being theoretically reachable does not make a referenced artifact rebuildable under DDIL, retention or provenance requirements. Rebuildability is declared per release and proven from a backup fixture. **[A/E/P]**

### 6.3 Configuration and secrets

The backup manifest records config schema/profile/fingerprint and logical secret-reference/version metadata only. Operator configuration and release profile files are protected by their owning deployment repository/process. Secret values, `.env`, mounted secret files, tokens, private keys and DSNs are excluded. Restore receives newly provisioned secret references and validates them before cutover; cloning requires distinct credentials. **[A/P]**

---

## 7. Database Migration and Data Migration Findings

### 7.1 Tool and execution decision

Use SQLx 0.9.0 migrations as the current candidate because SQLx is the accepted persistence stack and its migrator validates applied migrations against the source. Migrations are embedded in the Glaux image/release, named by monotonically increasing version plus description, hashed in the release manifest and applied to a dedicated Glaux migration ledger. A Glaux wrapper adds lifecycle state, advisory lock, release compatibility, audit/telemetry and validation. Do not adopt a second migration framework absent a demonstrated gap. **[A/D/E/P]**

`glaux-server migrate plan|check|apply` uses migration-only credentials, never the runtime account. `check` is read-only. `apply` requires a backup/recovery-point reference where the profile demands it, an authorized operator, no incompatible serving writers, and one advisory lock. The migrator records start/completion, checksum, transaction mode, tool/release version and validation result. An applied file changing, missing or appearing out of order is fatal. **[E/P]**

### 7.2 Startup/readiness

`serve` never calls `Migrator::run`. It reads the ledger with a least-privilege query and becomes ready only when the schema is within its embedded `[minimum_readable, maximum_writable]` range and no lifecycle operation is active/failed. Local Compose and CI may define `migrate` as a dependency job; this preserves the explicit command and evidence while reducing user steps. Operational-reference uses a controlled maintenance/release workflow. **[A/E/P]**

### 7.3 Migration classes

| Class | Pattern | Transaction/recovery | Readiness |
|---|---|---|---|
| additive DDL | new nullable/default-safe column, table, constraint-not-valid | one transaction where PostgreSQL permits | old/new compatible during window |
| constraint/index | create index concurrently, add then validate constraint | explicit non-transaction stage, resumable progress | profile/operation-specific; verify before contract |
| semantic data backfill | populate normalized/versioned fields | chunked idempotent worker with checkpoint and invariant query | reads may use fallback; writes dual-populate if rolling |
| artifact/schema conversion | new exact representation/version | preserve old bytes and provenance; create new derived/versioned artifact | advertise only after validation |
| destructive contract | drop/rename/narrow/re-key | later release after usage/backfill proof and recovery point | incompatible old binary fenced |
| extension/platform | PostGIS/PostgreSQL major/extension update | infrastructure maintenance, backup, official check/tool path | server fully not-ready until compatibility proven |

Long backfills do not hold one giant transaction. Each chunk is deterministic and idempotent, records watermark/count/error, uses bounded load and can resume. The application distinguishes old/new representation during the compatibility window. A contract migration verifies zero old readers/writers and completed backfill before removal. **[E/P]**

### 7.4 Rollback policy

SQLx supports undo migrations, but Glaux does not rely on automated down migration in shared/operational profiles. Reversing a schema does not reconstruct discarded data or external effects. Before the point of no return, rollback may run the old binary against a still-compatible expanded schema. After it, recovery uses the declared backup/old release and reconciles intervening accepted writes, or proceeds forward with a corrective migration. Every release documents its rollback window and irreversible steps. **[D/E/P]**

### 7.5 PostgreSQL/PostGIS changes

Application schema migration is separate from PostgreSQL major and PostGIS extension upgrade. Minor compatible image updates still run full backup/restore and extension regression tests. Major upgrade options include fresh-cluster dump/restore (portable, slower) or infrastructure-owned `pg_upgrade` after `--check`; external modules/PostGIS compatibility must be proven. PostGIS extension updates use the documented extension mechanism or dump/restore hard-upgrade path. Never silently update to “latest.” **[D/E/P]**

---

## 8. Upgrade Compatibility, Versioning, OpenAPI, and Conformance Artifact Findings

### 8.1 Independent version dimensions

`ReleaseCompatibilityManifestV1` binds:

- server SemVer, source revision and image digest;
- Rust/toolchain/platform architecture and dependency lock/SBOM/provenance;
- database migration minimum/read/write/current targets;
- PostgreSQL/PostGIS required ranges and enabled extensions;
- config schema/profile-constraint versions;
- canonical/event/audit/sync/artifact schema versions;
- OGC/SensorML/SWE standards/profile/package digests;
- API capability/conformance declaration and OpenAPI digest;
- fixture/scenario and rebuild-algorithm versions; and
- supported upgrade-from releases plus rollback point.

These versions are related by an explicit compatibility matrix; they are not forced into one version number. Server SemVer communicates public compatibility policy, while a migration integer is an ordered storage history. **[D/E/P]**

### 8.2 API and identifier preservation

An upgrade cannot silently change resource IDs, public URIs, alias resolution, collection membership, link relations, pagination/cursor semantics, time interpretation, media types, conformance declarations or problem types. A necessary change follows IDR-SRV-010A deprecation/compatibility policy and preserves a versioned alias/mapping with provenance where feasible. Database re-keying never leaks into public identity without an explicit API migration. **[A/P]**

### 8.3 OpenAPI and conformance

OpenAPI and conformance responses derive from the installed capability registry and release artifacts, not restored stale documents. Pre-upgrade CI compares old/new routes, operations, schemas, media types and conformance declarations against the compatibility manifest. Post-upgrade smoke tests fetch the deployed landing/API/conformance/OpenAPI identity and compare expected digests/semantic diff. Historical conformance evidence retains the old release/profile/fixture manifest; it is never relabeled as evidence for the new release. **[A/E/P]**

### 8.4 Upgrade sequence

1. verify release provenance, compatibility and rollback artifact;
2. test migration and restore against a production-shaped sanitized clone;
3. take/verify a recovery point and record old manifest/high-water evidence;
4. disable command dispatch and fence/drain writes/workers as required;
5. apply extension/infrastructure and application migrations in declared order;
6. run structural, semantic, policy/audit and artifact validation;
7. rebuild projections and deploy the compatible server image/config/packages;
8. run protected smoke/conformance/security/interop canaries;
9. re-enable roles/effects incrementally after reconciliation; and
10. retain the old recovery set until the declared rollback window closes. **[E/P]**

---

## 9. Bootstrap, Seed Data, Fixture Reset, and Demo Reset Findings

### 9.1 Separate operations

| Operation | Allowed purpose | Content | Idempotency/safety |
|---|---|---|---|
| `migrate apply` | create/evolve structural schema | immutable release SQL | ledger/checksum/lock; no samples |
| `bootstrap apply` | required empty-system records/packages | system metadata, capability/config/package bindings | natural stable keys; create-or-verify, never overwrite drift |
| `fixture apply` | named test/demo scenario | synthetic resources/data/identities/policy and expected manifest | scenario ID/digest; targeted idempotency; profile gated |
| `rebuild <projection>` | derive cache/index/latest/search | authoritative input only | algorithm/watermark evidence; swap after validation |
| `reset` | destroy disposable environment | database/artifact/telemetry state named by scenario | multiple independent fences plus explicit destructive confirmation |

Bootstrap does not create default passwords, live sources, unrestricted publishers, operational policies or enabled commands. Required schemas/profile packages are release inputs with digests; cache index rows may be bootstrapped. Admin identities/credentials are deployment-provisioned through a separate one-time authorized process, not seed SQL. **[A/E/P]**

### 9.2 Destructive fences

`reset` requires all of: an allowed profile (`dev-*`, `ci-core`, `conformance`, `interop`, designated demo); a database-internal randomly generated environment marker matching configuration; an explicit exact target/lineage; no operational-preservation flag; no active serving/worker leases; and interactive confirmation or CI-specific noninteractive token. It refuses `operational-reference`, DDIL/sync data marked preserved, an unmarked database and any target outside the configured workspace. **[E/P]**

### 9.3 Profile behavior

- **dev:** explicit reset or targeted scenario replacement; developer data is not silently destroyed.
- **CI/conformance:** create a fresh uniquely named database/volume, migrate from empty, apply fixture, test, collect evidence and destroy.
- **interop:** restore/reseed a pinned scenario and record client/server manifests; remote mutable servers are observational only.
- **demo-public:** take out of service, destroy/recreate DB and artifact namespace from immutable synthetic pack, validate, issue a new demo lineage/cursor epoch and cut over.
- **command-sim:** reset simulator and command history together; never preserve a dispatchable nonterminal command.
- **ddil/sync:** scenario reset creates distinct node identities/databases; not a substitute for operational recovery.
- **operational-reference:** no reset path; use authorized lifecycle/retention/deletion and recovery procedures. **[A/E/P]**

---

## 10. Backup Strategy Findings

### 10.1 Coherent backup unit

`GlauxBackupManifestV1` records backup ID/lineage/epoch; start/end and database snapshot/high-water evidence; server/release/config/capability/migration/PostgreSQL/PostGIS/package versions; archive/artifact inventory with digest/size/classification; outbox/audit/sync/command watermarks; included/excluded derived sets; encryption/signature/key identifiers; tool versions/options; source node/profile; quiesce state; validation results; and parent/base relationship where relevant. It contains no secret values or sensitive endpoints. **[E/P]**

### 10.2 First portable reference backup

The initial proof uses:

1. authorize backup and persist epoch/fence;
2. become not-ready for write roles; stop new ingestion, command submission/dispatch, publication claims and sync apply;
3. drain in-flight transactions; expire/normalize worker leases; flush required audit/outbox state;
4. freeze the referenced exact-artifact set and record digests/high-water marks;
5. run a PostgreSQL 18-compatible `pg_dump -Fc` with reviewed explicit options and a least-privilege backup identity;
6. export referenced artifacts and exact release/config/package evidence;
7. create, sign and encrypt the manifest/package in a protected repository;
8. verify archive readability/digests and schedule an isolated restore test; and
9. audit completion and release the fence or remain recovery-required on ambiguous failure.

Custom format is portable and selectively inspectable with `pg_restore`; directory format/parallel dump is a measured option for larger datasets. The tool must be at least compatible with the server—`pg_dump` refuses newer servers—and warnings/errors are fatal. Cluster roles/passwords are not blindly dumped; infrastructure recreates least-privilege roles and Glaux remaps ownership/ACLs deliberately. **[D/E/P]**

### 10.3 Operational-reference escalation

The operational reference contract requires infrastructure to add physical base backup, continuous complete WAL archive and PITR/timeline management, with periodic logical portability export. PostgreSQL 18 supports full/incremental base backups, but an incremental chain must be combined and verified per official tooling. `pg_verifybackup` checks manifests/files/checksums/required WAL yet explicitly cannot replace test restore/semantic validation. File/data checksums are enabled/verified where supported. **[D/E/P]**

Raw tar of a live PostgreSQL Docker volume is prohibited as a database-consistent backup unless PostgreSQL is stopped cleanly or an approved coordinated snapshot/PITR mechanism makes it consistent across data, WAL and tablespaces. Docker's generic volume examples do not supply database semantics. Storage snapshots may supplement but not bypass PostgreSQL/application validation. **[D/E/P]**

### 10.4 Security and retention boundary

Backup data inherits the highest included classification. Encrypt in transit and at rest with keys outside the backup, authenticate/sign the manifest, use write-restricted backup identities and read-separated restore roles, log access, test revocation/key recovery and protect against deletion/tampering. Exact algorithms/providers, immutability, offsite topology and retention schedules belong to deployment security/records policy. A backup is not counted successful until verification completes; recovery confidence requires recurring restore drills. **[D/E/P/X]**

---

## 11. Restore Strategy and Post-Restore Validation Findings

### 11.1 Isolated restore pipeline

1. authorize a restore case and create a fresh isolated target/lineage;
2. verify manifest signature/digests, custody, classification and tool/version compatibility before use;
3. provision pinned PostgreSQL/PostGIS/extensions and empty database with effects/network egress denied;
4. restore archive with explicit ownership/ACL mapping and exit-on-error behavior;
5. install exact release packages/config metadata and retrieve distinct secret references;
6. validate migration ledger and apply only the declared forward path while still isolated;
7. verify structural/referential/domain/security/audit/artifact invariants;
8. rebuild and atomically activate derived projections/indexes/caches;
9. reconcile commands, outbox/inbox leases, audit lineage and synchronization gaps;
10. run protected smoke, conformance subset, security and performance sanity tests;
11. produce `RestoreValidationReportV1`; and
12. separately authorize fenced cutover, then enable read/write/effect roles in order. **[D/E/P]**

Archive content is active code: PostgreSQL warns a restore can execute arbitrary SQL chosen by source superusers, even with partial restore. Only trusted signed sources enter the restore environment; inspect archive lists/SQL as required, restrict network/OS/database privilege, and never restore an untrusted dump into a production-connected server. **[D/P]**

### 11.2 Validation gates

| Gate | Required proof |
|---|---|
| storage/tool | archive readable, all artifacts present/digests match, encryption/signature/custody valid |
| platform/schema | PostgreSQL/PostGIS/extensions compatible; migration ledger exact; no dirty/pending unexpected migration |
| relational | constraints/FKs/uniqueness, partition/index state, sequence/ID generators and row-family counts/ranges |
| domain | lifecycle/tombstones, links/aliases, temporal/provenance and schema bindings satisfy accepted invariants |
| exact artifacts | every required reference resolves to digest-matching bytes and declared parser/profile |
| security/policy/trust | identities/metadata/packages/signatures/validity restored; secrets reacquired; stale remains stale |
| audit | chain/checkpoint/prefix/custody and external-anchor comparison; recovery/gap event prepared |
| command/effects | all nonterminal rows fenced/classified; no automatic dispatch; external outcomes reconciled or unknown |
| events/idempotency | leases expired safely, pending/acked state consistent, stable IDs, replay/cursor lineage decision explicit |
| projections | latest/current/search/materialized results rebuilt at documented watermark and compared to oracle |
| DDIL/sync | node identity decision, watermarks/gaps/conflicts and peer divergence explicit |
| API/contract | landing/links/content negotiation/problems/OpenAPI/conformance/capability registry match release |

A row count and successful process start are insufficient. Readiness remains false until every profile-mandatory gate passes. Failure leaves the isolated target intact for protected diagnosis or securely disposes it; it never partially replaces the active system. **[A/E/P]**

### 11.3 Replacement versus clone

A replacement recovery may retain public resource IDs and logical node identity only when the prior writer is fenced, the recovery lineage/point is recorded, credentials are rotated/reacquired and clients/peers can detect any rewind/gap. A test/demo/forensic clone always gets a new node/service identity, origin, credentials, sync identity and effect fence; controlled data is sanitized/authorized. Copying a database does not authorize copying trust or operational command authority. **[A/E/P]**

### 11.4 Partial restore

Arbitrary table-level restore is prohibited for ordinary recovery because relationships, outbox, audit, policy and time-series consistency cross tables. Supported selective import must be a versioned application workflow that validates provenance, identity mapping, policy and conflicts and creates new audit evidence—not a `pg_restore -t` shortcut. Forensic extraction remains isolated/read-only. **[D/E/P]**

---

## 12. Audit, Command/Control, Event/Outbox, Observation/Status, and Latest-Value Continuity Findings

### 12.1 Audit

Audit is append-only authoritative evidence. Migrations add version-aware readers/upcasters or new fields; they do not rewrite event meaning, actor, time, ordering, chain or checkpoint. Backups include the complete required journal prefix and external checkpoints/keys or references under separate custody. A PITR to an earlier point is an observable fork/truncation relative to later checkpoints: the recovered node begins a new recovery epoch and emits an authorized recovery/gap event referencing the backup and last verified anchor. It never forges continuity. **[A/E/P]**

Backup, migration, restore validation, reconciliation and cutover are themselves audited with actor, authorization, source/destination lineage, manifest digest, stages/outcome and time evidence—never secret values. If audit capture required by the operation is unavailable, accepted E0–E5 failure semantics apply. **[A/P]**

### 12.2 Commands

Before lifecycle work, stop new submissions and dispatch claims, drain bounded in-flight adapter calls, persist dispatch fences and classify every nonterminal command. After restore:

- completed/rejected/cancelled/expired remain terminal history;
- accepted/queued but never dispatched remain fenced and require fresh authorization, policy, feasibility, safety, expiry and operator decision before any future dispatch;
- dispatch-intent/attempted/acknowledged-without-terminal-result become `unknown_outcome` until an authoritative gateway reconciliation proves more;
- restored approval does not automatically satisfy a new-time/current-state decision; and
- no migration/restore worker calls a command gateway.

External physical state cannot be rolled back by a database restore. Current scope remains simulated commands, but the model prevents later accidental replay. **[A/E/P]**

### 12.3 Events, outbox and cursors

Outbox insertion remains atomic with domain state and audit references. Backup captures row state and high-water. On restore, all stale claim leases expire under a new worker epoch. Unacknowledged rows may republish with the same stable event/delivery identity; downstream consumers require idempotency, yielding at-least-once recovery. Acknowledged rows are not blindly reset. If external acknowledgement cannot be proven, retry policy records ambiguity rather than claiming exactly once. **[A/E/P]**

Replay cursors bind lineage and durable sequence. Same-lineage replacement may continue only if the entire promised replay window survives. PITR or demo reset publishes a new epoch and returns accepted gap/resnapshot semantics for old cursors. Events after the recovery point are absent even if an external subscriber saw them; reconciliation/import requires an explicit trusted workflow. **[A/E/P]**

### 12.4 Observations, status and latest

Canonical observations/status preserve phenomenon/result time, arrival/order, source/provenance, schema version, ID and lifecycle. Restore verifies partition ranges, uniqueness/deduplication and spatial/time indexes. Latest/current is derived from accepted selection rules; invalidate and rebuild at a recorded input watermark, then compare against a direct authoritative query before swap. Late-arriving/replayed observations pass normal idempotency and policy/validation; restoring a backup does not make stale data current. **[A/E/P]**

---

## 13. Source Trust, Policy/Releasability, DDIL, Synchronization, and Conflict Continuity Findings

### 13.1 Source trust and policy

Restore preserves source registrations, trust decision history, issuer/key/package IDs, policy assertions/bindings and validity evidence. It does not restore secret values or extend validity. Certificates, cached credentials, policy and trust material are revalidated against restored observation time/current trusted time per the accepted DDIL model. Revocation information newer than the recovery point must be reacquired before privileged writes/effects; absence becomes unknown/denied, not trusted. **[A/E/P]**

Policy bundle bytes are verified against signed release/backup references. A new release may migrate metadata syntax but cannot broaden disclosure or erase markings as a convenience. Backups and restore environments inherit controlled-data rules; sanitization for test/demo creates a separately derived, provenance-recorded dataset, never a mislabeled operational clone. **[A/E/P]**

### 13.2 DDIL

DDIL backup includes local authoritative DB state, audit, outbox/replay, exact schemas/profiles/vocabularies, trust/policy/credential-verification metadata, approved source buffers and artifact inventory. Telemetry/cache is secondary. Restore works without mandatory network and records time uncertainty/evidence age. Storage pressure cannot evict authoritative/audit/command/conflict evidence ahead of declared disposable caches. Local backup media and keys require environment-specific custody beyond this report. **[A/E/P]**

### 13.3 Synchronization and conflicts

Restoring a synchronized node rewinds its local watermarks relative to peers. The node must retain/fence its identity as a true replacement or receive a new identity as a clone. On recovery it announces lineage/restore point, compares scoped inventories/watermarks and treats peer-ahead ranges as gaps. It never advances a watermark from backup metadata alone or lets database replication resolve application conflicts. Stable envelope/dedupe IDs prevent duplicate application; missing ranges resync or resnapshot under accepted authority/policy. **[A/E/P]**

Conflict, quarantine, tombstone and review records are canonical and restored with evidence. A conflict resolved after the recovery point may reappear locally; peer/current authoritative evidence must reconcile it without silently overwriting either history. Audit records both the recovery and the new resolution. **[A/E/P]**

### 13.4 Multi-node backup boundary

Each node backup is independently consistent and lineage-labeled. A simultaneous distributed snapshot is not assumed. Cross-node recovery uses application synchronization after local validation. Shared DB snapshots or synchronized clocks do not prove common authority. **[A/E/P]**

---

## 14. Security and Threat-Mitigation Findings

| Threat | Control | Verification |
|---|---|---|
| modified/reordered migration | immutable checksums, signed release, applied-ledger validation, review | mutate/missing/out-of-order negative corpus |
| concurrent migration/writer | advisory lifecycle lock, readiness/effect fence, single runner | race/failure-injection test |
| destructive reset on real data | profile + DB marker + lineage + target + lease + confirmation gates | wrong-profile/target/marker tests |
| malicious/untrusted dump code | signed custody, pre-inspection, isolated network/privilege, trusted source only | tampered/untrusted archive rejection |
| backup tamper/deletion | authenticated manifest, object digests, restricted/immutable repository, independent copy | bit flip/missing object/manifest replay tests |
| secret leakage | schema-driven exclusion, external keys, artifact scan | canary secret across archive/manifest/logs |
| policy/trust downgrade | version/validity/signature and current revocation checks; fail closed | old/revoked/expired package restore |
| audit rollback/deletion | external checkpoints, prefix/chain validation, recovery epoch/gap event | PITR before anchor and broken-chain tests |
| command replay | restore isolation, dispatch fence, state classification, fresh decision/reconciliation | every nonterminal state restore test |
| event duplication/gap | stable IDs, expired leases, dedupe, lineage-bound cursors, gap semantics | crash at each claim/ack boundary |
| fixture/cache poisoning | signed release/scenario manifests, digest/profile validation, synthetic-only rules | tampered package/cache tests |
| clone identity collision | new identity/origin/credentials by default; old writer fencing for replacement | simultaneous-old/new negative test |
| incomplete multi-store backup | quiesce/high-water/artifact closure manifest | missing/late artifact and concurrent-write test |
| overprivileged restore | distinct backup/restore/migrate/app roles and explicit ownership/ACL mapping | privilege-diff and access test |

Migration/backup/restore commands use separate credentials and administrative authorization. Outputs follow IDR-SRV-047 redaction and IDR-SRV-048 telemetry rules. Backup filenames/paths, internal topology, policy/source/command details and restore errors are protected; public demo reveals only stable availability. **[A/E/P]**

Encryption and signatures do not establish authorization to move controlled data. Any cross-boundary export requires a separately approved transfer/releasability process; normal Glaux backup is not a cross-domain solution. **[A/X/P]**

---

## 15. Observability, CI, Conformance, Performance, Security Testing, and Interoperability Implications

### 15.1 Lifecycle observability

Structured stage events/spans include operation/backup/restore ID, release/config/migration/package identity, lifecycle state, duration, counts, safe error code and outcome. Metrics include last successful backup/restore drill/migration time, duration, bytes/rows/artifacts processed, backfill/index progress, validation failures and backup age/availability as infrastructure-supplied evidence. Command/audit/outbox reconciliation counts are protected. No DSN, key, URL, payload, actor/source/command/resource ID or SQL value is emitted. **[A/P]**

Serving readiness reports incompatible/pending/running/failed/validating/reconciling states through coarse codes. Detailed lifecycle diagnostics are admin-only. Liveness remains shallow. Backup freshness is not server liveness and only affects readiness where an explicit deployment policy requires it. Lifecycle operations and access to their evidence are audited. **[A/P]**

### 15.2 CI matrix

Every migration PR must test:

- empty database through all migrations/bootstrap;
- each supported previous release fixture to target;
- applied-file checksum drift, missing/out-of-order and dirty/failure states;
- transactional rollback and non-transactional resume;
- backfill restart/idempotency and mixed-version compatibility window;
- PostGIS extension/schema assumptions;
- logical backup creation plus isolated restore with a non-owner mapping;
- complete semantic validation/rebuild/reconciliation;
- command no-redispatch, outbox duplicate/gap, audit anchor, trust/policy staleness and sync rewind;
- fixture/demo reset gates and secret canary absence; and
- old/new API/OpenAPI/conformance/interop compatibility diff.

Migration fixtures are immutable snapshots sanitized and versioned with generator/provenance—not ad hoc production dumps. A PR cannot delete a supported upgrade fixture without an explicit support-policy change. **[E/P]**

### 15.3 Performance and recovery exercises

IDR-SRV-054 measures migration/backfill/index/rebuild/dump/restore/validation duration, storage amplification, locks, write/read impact, WAL/archive volume and bounded parallelism on representative datasets. The result sets maintenance-window/capacity recommendations; this report invents none. Repeated recovery drills restore into isolated infrastructure and record achieved recovery point/time without converting observations into guaranteed RPO/RTO. **[D/E/P]**

### 15.4 Conformance and interoperability

After upgrade/restore, run landing/link/media/filter/pagination/problem/OpenAPI/conformance tests plus representative dynamic/event/command-disabled/security cases. Preserve old evidence under its original manifest. Clients receive declared deprecation/version behavior, stable identities and explicit cursor/replay reset semantics; they are not expected to infer a restore from broken links or unexplained 404s. IDR-SRV-050/051/056 own exact matrices and verdicts. **[A/E/P]**

---

## 16. Downstream Topic Handoff Matrix

| Topic | Fixed input from IDR-SRV-049 | Still owned downstream |
|---|---|---|
| IDR-SRV-050 Conformance harness | release/config/capability/standards/fixture/restore manifest and post-lifecycle contract suite | executable harness, official tests, verdict/evidence model |
| IDR-SRV-051 Traceability | migration/recovery requirements and proof gates trace to accepted decisions | requirement IDs, coverage representation and change impact process |
| IDR-SRV-052 Rust tests | SQLx wrapper boundaries, admin-command fakes, prior-version and fault fixtures | test architecture, crate lanes, property/fuzz patterns |
| IDR-SRV-053 Test data/fixtures | immutable previous-release DBs, synthetic scenario/reset packs, corruption/gap/rewind cases | corpus schema, generators, provenance and golden governance |
| IDR-SRV-054 Performance | lifecycle operations and measures listed in Section 15.3 | datasets, budgets, thresholds, tooling and acceptance criteria |
| IDR-SRV-055 Security/commands | threat matrix, privilege split, archive-code trust, secret exclusion and no-redispatch reconciliation | attack cases, authorization matrices and command/security test execution |
| IDR-SRV-056 Interoperability | stable IDs/links, compatibility manifest, cursor epoch/gap and post-upgrade evidence | external client/server matrix and result governance |
| final synthesis/implementation | forward-only SQLx, explicit jobs, coherent manifest, logical reference proof, physical/PITR handoff and validation gates | implementation sequencing, production policies and operational approval |

### 16.1 Ordered implementation proofs

1. immutable SQLx ledger/checksum and advisory single-runner behavior;
2. empty-to-current plus every supported-prior-to-current migration;
3. transactional failure rollback and resumable non-transactional/backfill stages;
4. server schema-range readiness and mixed old/new compatibility window;
5. reset/bootstrap/fixture profile, DB-marker and lineage fences;
6. coherent quiesce/high-water logical DB plus artifact manifest backup;
7. manifest/digest/signature/secret-exclusion and untrusted-archive rejection;
8. fresh isolated restore with explicit roles/ownership and full validation gates;
9. latest/index/materialized projection deterministic rebuild;
10. command non-redispatch and external-outcome reconciliation;
11. outbox/idempotency/cursor plus audit/checkpoint and sync-lineage recovery; and
12. production-shaped timed recovery drill and post-restore conformance/security/interop suite.

Category I should convert these proofs into executable traceability and test plans after acceptance. **[P]**

---

## 17. Recommendations

1. Adopt SQLx 0.9.0 migrations behind a Glaux lifecycle wrapper as the current candidate; keep released migrations immutable and forward-only. **[D/E/P]**
2. Run migrations only through an explicit same-image administrative command with dedicated credentials, lock, recovery point and validation; `serve` never auto-migrates. **[A/P]**
3. Embed schema compatibility ranges and fail readiness on dirty, pending or incompatible state. **[P]**
4. Use expand/backfill/contract for compatibility windows; use tested backup restore, not down migration, for destructive rollback. **[P]**
5. Adopt the Section 6 authority/rebuildability inventory as the backup and restore contract. **[P]**
6. Implement `GlauxBackupManifestV1` and a quiesced custom-format logical DB plus exact-artifact reference proof. **[D/E/P]**
7. Require operational-reference infrastructure to add physical base backup/WAL/PITR, offsite/retention/encryption and recurring restore drills; select no vendor here. **[D/P]**
8. Restore only into fresh isolated targets, verify provenance before SQL execution, reacquire secrets and activate through an independently authorized fenced cutover. **[D/P]**
9. Never redispatch restored commands; reconcile nonterminal effects and preserve unknown outcomes. **[A/P]**
10. Preserve stable event IDs with at-least-once recovery, lineage-bound cursors and explicit gap/resnapshot behavior. **[A/P]**
11. Keep bootstrap, fixtures, reset and rebuild separate; reject destructive reset outside explicitly disposable marked environments. **[P]**
12. Execute all twelve proofs in Section 16 before any continuity or recovery-readiness claim. **[P]**

---

## 18. Risks, Constraints, and Open Questions

### 18.1 Risk register

| Risk | Consequence | Control / proof |
|---|---|---|
| migration edits/drift | unreproducible or corrupt schema | immutable checksums/ledger/release signature and negative tests |
| auto/concurrent migrations | availability/partial change | explicit one-shot runner, lock and serving fence |
| down migration loses data | false rollback | forward corrective path or pre-change restore with point-of-no-return |
| DB-only backup misses artifacts | broken evidence/links | quiesced closure/high-water multi-store manifest |
| volume copy is inconsistent | unrecoverable PostgreSQL | logical or official physical/WAL mechanism; test restore |
| archive is malicious | code execution/privilege compromise | trusted signed custody and isolated least-privilege inspection/restore |
| backup leaks controlled data/secrets | compromise/cross-boundary violation | secret exclusion, classification, encryption, access/custody controls |
| PITR silently rewinds audit/events/sync | false continuity/duplicate effects | recovery epoch, external anchors, gaps, dedupe and reconciliation |
| restored command redispatches | duplicate physical effect | dispatch fence and nonterminal classification/fresh decision |
| derived projection accepted as truth | incorrect latest/search/result | rebuild from authority and oracle/watermark validation |
| clone shares identity/credentials | split brain/unauthorized sync | new identity/origin/secrets by default; replacement fencing proof |
| restore “succeeds” without semantics | latent API/policy corruption | full validation gates and conformance/security/interop canaries |

### 18.2 Constraints

- No production data, secrets or operational policy appear in examples/fixtures.
- No RPO/RTO, retention, schedule, media, region or capacity is selected.
- Reference logical backup is a portability/verification baseline, not the sole production mechanism.
- Physical/WAL backup, HA and storage-level snapshots require PostgreSQL/infrastructure expertise.
- Command scope remains simulated; recovery rules intentionally anticipate but do not authorize physical effects.
- A backup cannot recover external physical reality or post-recovery-point facts without external evidence.

### 18.3 Open questions with owners

| Question | Default until resolved | Owner |
|---|---|---|
| exact supported upgrade-from window? | previous accepted release plus empty DB, expanded only with fixtures/tests | release policy/IDR-SRV-051 |
| logical archive custom versus parallel directory at scale? | custom first; measure directory mode | IDR-SRV-054 |
| backup/PITR frequency, retention and RPO/RTO? | no production claim | deployment/operations/security |
| physical backup/vendor/object repository? | provider-neutral contract only | deployment decision |
| hot multi-store backup without quiesce? | not first implementation; require versioned object store/snapshot protocol proof | future implementation |
| selective tenant/resource restore? | prohibited; application import workflow required | future requirements/security |
| audit external checkpoint custody/keys? | separate protected authority | security/operations/IDR-SRV-055 |
| post-PITR client/sync resnapshot protocol detail? | accepted explicit gap/new lineage | IDR-SRV-056/implementation |

---

## 19. Validation Against This Plan's Success Criteria

| Success criterion | Result | Evidence |
|---|---|---|
| scope boundaries with sources and traceability | Met | Sections 3–5 |
| authoritative/derived/cache/mutable/immutable/append-only/disposable categories | Met | Section 6 |
| database/data/backfill/index/cache/OpenAPI/conformance implications | Met | Sections 7–8 |
| bootstrap/seed/reset/backup/restore/validation strategies | Met | Sections 9–11 |
| command/event/observation/latest/trust/policy/audit/DDIL/sync continuity | Met | Sections 12–13 |
| security, secret exclusion, tamper, audit and profile gates | Met | Sections 9, 10 and 14 |
| tests, CI, conformance, performance, observability, security and interop | Met | Section 15 |
| implementation/community lessons non-normative | Met | Section 3.3 |
| decision-usable and server-bounded recommendations | Met | Sections 2 and 17–18 |
| downstream handoffs explicit | Met | Section 16 |
| references explicit and reproducible | Met | Section 20 |

All planned phases and required report content are complete. Acceptance remains a project-lead action. **[P]**

---

## 20. References

### 20.1 Database, migration and storage sources

- PostgreSQL 18 Backup and Restore: https://www.postgresql.org/docs/18/backup.html
- PostgreSQL 18 `pg_dump`: https://www.postgresql.org/docs/18/app-pgdump.html
- PostgreSQL 18 `pg_restore`: https://www.postgresql.org/docs/18/app-pgrestore.html
- PostgreSQL 18 continuous archiving/PITR: https://www.postgresql.org/docs/18/continuous-archiving.html
- PostgreSQL 18 `pg_basebackup`: https://www.postgresql.org/docs/18/app-pgbasebackup.html
- PostgreSQL 18 `pg_verifybackup`: https://www.postgresql.org/docs/18/app-pgverifybackup.html
- PostgreSQL 18 backup manifest: https://www.postgresql.org/docs/18/backup-manifest-format.html
- PostgreSQL 18 `pg_upgrade`: https://www.postgresql.org/docs/18/pgupgrade.html
- PostgreSQL 18 data checksums: https://www.postgresql.org/docs/18/checksums.html
- PostgreSQL transactions: https://www.postgresql.org/docs/18/tutorial-transactions.html
- PostGIS upgrades: https://postgis.net/docs/postgis-en.html#upgrading
- SQLx 0.9.0 migrations: https://docs.rs/sqlx/0.9.0/sqlx/migrate/
- Docker volumes: https://docs.docker.com/engine/storage/volumes/
- Semantic Versioning 2.0.0: https://semver.org/
- OCI Image Specification: https://github.com/opencontainers/image-spec

### 20.2 Standards and security sources

- OGC API - Connected Systems Part 1: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems Part 2: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- SensorML 3.0: https://docs.ogc.org/is/23-000/23-000.html
- SWE Common 3.0: https://docs.ogc.org/is/24-014/24-014.html
- RFC 9110: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9457: https://www.rfc-editor.org/rfc/rfc9457
- NIST SP 800-53 Rev. 5: https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final
- NIST SP 800-92: https://csrc.nist.gov/pubs/sp/800/92/final
- OWASP Logging Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html

### 20.3 Accepted project evidence

- Overall IDR Research Plan: [overall-idr-research-plan.md](../IDR%20Plans/overall-idr-research-plan.md)
- IDR-SRV-049 Research Plan: [idr-srv-049-migration-upgrade-backup-and-restore-strategy.md](../IDR%20Plans/idr-srv-049-migration-upgrade-backup-and-restore-strategy.md)
- Glaux Server Goal and Definition: [glaux-server-goal-and-definition.md](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- Accepted IDR-SRV-001 through IDR-SRV-048 reports: [IDR Reports](./)
- IDR-SRV-048 observability report: [idr-srv-048-observability-logs-metrics-and-health-check-strategy-report.md](idr-srv-048-observability-logs-metrics-and-health-check-strategy-report.md)
- Research Report Template: [research-report-template.md](../../../../../Governance/research-report-template.md)

### 20.4 Non-normative implementation/community evidence

- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- OGC API Connected Systems development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- Accepted IDR-SRV-014A through IDR-SRV-014G reports: [IDR Reports](./)

### 20.5 Reproducibility and evidence limits

Official PostgreSQL, PostGIS, SQLx, Docker, standards and security documentation was checked on September 16, 2026. Version numbers record the evidence freeze and must be revalidated with the actual release image, extensions, dependency lock and platform. Implementation/community studies remain informative. No Glaux schema, migration, data volume, backup store or restore drill existed; the twelve proofs in Section 16 are therefore prerequisites to implementation claims.

---

## Report Completion Checklist

- [x] All 20 required sections are present
- [x] Scope/responsibility and lifecycle states are explicit
- [x] Required 13-column continuity matrix is complete
- [x] Migration, compatibility and rollback decisions are explicit
- [x] Bootstrap, fixture and destructive reset gates are explicit
- [x] Logical reference backup and operational physical/PITR handoff are explicit
- [x] Isolated restore, validation and lineage/cutover rules are complete
- [x] Audit, command, event, observation, trust, policy, DDIL and sync continuity is complete
- [x] Threats, tests, implementation proofs and downstream handoffs are complete
- [x] All 11 success criteria validate as Met
- [ ] Accepted by Glaux Project Lead

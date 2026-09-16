# Section 053: Test Data, Fixtures, Golden Files, and Scenario Corpus Strategy - Research Report

**Topic ID:** IDR-SRV-053<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-053 Test Data, Fixtures, Golden Files, and Scenario Corpus Strategy](../IDR%20Plans/idr-srv-053-test-data-fixtures-golden-files-and-scenario-corpus-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** corpus scope and taxonomy; source acquisition, licensing, provenance and sensitivity; manifest, identity, layout and lifecycle; CSAPI, SensorML, SWE Common, observation, status, query, error, ingestion, event, command, policy, DDIL and synchronization data; public demonstration; generated and large data; golden and semantic assertions; CI, review, drift and downstream handoffs<br>
**Methodology Used:** accepted-requirement extraction; official artifact and example inventory; authority, licensing and reproducibility analysis; positive/negative/boundary scenario partitioning; assertion-oracle and deterministic-generation modeling; repository/CI threat analysis; implementation/community lesson reconciliation; downstream synthesis<br>
**Research Time:** Approximately 44 hours of AI-assisted execution on September 16, 2026<br>
**Accepted Verification Baseline:** IDR-SRV-050 independent conformance harness, IDR-SRV-051 normalized traceability/evidence graph and IDR-SRV-052 obligation-first multi-layer Rust test architecture<br>
**Current Standards Artifact Pin:** OGC API - Connected Systems Parts 1 and 2 `v1.0.0`, tag commit `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`; official OGC schema directories checked September 16, 2026<br>
**Document Purpose:** Define a safe, reproducible, traceable fixture and scenario-corpus baseline without creating the complete corpus, implementing generators or tests, selecting quantitative performance/security gates, or asserting conformance/readiness<br>
**Author:** OpenAI Codex<br>
**Date:** September 16, 2026<br>
**Last Updated:** September 16, 2026

---

## Evidence and Decision Legend

- **[N] Normative:** approved external standard or normatively incorporated artifact.
- **[A] Accepted project baseline:** accepted Glaux report or governing project decision.
- **[D] Direct documentation:** official tool, schema-registry, security or platform documentation.
- **[I] Implementation evidence:** another implementation, client, example or community result; informative only.
- **[T] Test evidence:** reproducible observation with stated source, fixture, target and conditions.
- **[E] Analysis:** reasoned synthesis from identified evidence.
- **[P] Project recommendation:** proposed Glaux decision pending acceptance of this report.
- **[X] Explicit boundary:** excluded claim or later-topic responsibility.

An official example is not automatically normative, valid, redistributable, profile-correct or an oracle. A generated output is not test truth. A schema-valid document is not necessarily semantically or profile valid. **[N,D,E]**

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Fixture Requirement Extraction Methodology
5. Fixture Taxonomy
6. Fixture Metadata, Provenance, Sensitivity, and Traceability Model
7. Fixture Storage Layout, Naming, and Versioning Findings
8. Standards-Derived and CSAPI Resource Fixture Findings
9. SensorML, SWE Common, Observation/Status, Query/Filter, and Error Fixture Findings
10. Ingestion, Streaming/Event, Command/Control, Security/Policy, DDIL, and Synchronization Fixture Findings
11. Public Demo and Scenario Corpus Findings
12. Performance and Large/Generated-Data Set Findings
13. Golden-File, Semantic Assertion, Normalization, and Drift-Control Findings
14. CI Validation, Secret Scanning, Fixture Review, and Generation Workflow Findings
15. Downstream Topic Handoff Matrix
16. Recommendations
17. Risks, Constraints, and Open Questions
18. Validation Against This Plan's Success Criteria
19. References

---

## 1. Executive Summary

Glaux should maintain **one governed corpus registry with four non-interchangeable artifact roles**: curated source inputs, executable scenario recipes, human-authored expected facts/assertions, and generated run outputs/evidence. A test may connect all four by immutable identifiers and digests, but server output, live demo state, snapshots proposed by CI and generated performance data never become authoritative fixture truth merely because a run produced them. **[A,E,P]**

Every fixture or scenario has a stable, non-recycled identifier and a constrained YAML manifest validated against a pinned JSON Schema. CI canonicalizes that manifest to JSON for hashing and evidence linkage, following IDR-SRV-051. The manifest binds source locator and digest, retrieval and transformation history, license/redistribution terms, standards/profile pins, sensitivity and release class, exact files and hashes, setup/cleanup, deterministic clock/ID/seed controls, expected semantic facts, trace links, validation records, golden policy, public-demo eligibility and owners/reviewers. Source bytes and adapted copies are separate artifacts. **[A,D,P]**

The corpus is classified independently along origin, validity, scale, lifecycle, sensitivity and use. Origin distinguishes exact official artifacts, licensed adaptations, accepted project artifacts, hand-curated synthetic data, deterministic generated data, minimized regressions and externally held named-pair data. Validity distinguishes structurally valid, semantically invalid, profile-invalid and adversarial/resource-bound cases. These axes prevent labels such as “official,” “example,” or “synthetic” from conveying more authority than they possess. **[E,P]**

Small, reviewable, redistributable fixtures, manifests, licenses, generator sources, small seed corpora and minimized regressions belong in Git. Large datasets are described by versioned deterministic recipes and content digests, generated into ignored directories or hydrated from controlled content-addressed storage. Git LFS is conditional only for irreducible licensed binary source artifacts: pointer diffs reduce reviewability, and LFS is not a substitute for provenance, licensing, mirroring or size governance. Project-specific size budgets should be measured during implementation; repository checks should reject unexplained binary or rapid corpus growth from the start. **[D,E,P]**

The standards corpus starts with pinned CSAPI Part 1/2 OpenAPI/schema/example packages, SensorML 3.0, SWE Common 3.0, OGC API - Features, GeoJSON and problem-detail artifacts. Exact upstream copies remain immutable and are revalidated. Any repair or Glaux-profile adaptation receives a new identifier, transformation recipe and explicit “not official” status. Known defects become intentional negative cases rather than silently corrected positives. Required tests are offline and never depend on the current contents of a live OSH, Connected Systems Go or public demo endpoint. **[N,A,I,P]**

API fixture sets use a minimal/full/extension/omission/invalid/boundary pattern and preserve relationships across systems, deployments, procedures, sampling features, properties, datastreams, observations, status, controls, commands, events, trust, policy, audit and synchronization records. Query discriminator sets include hits, misses, ties, boundary instants/geometries and invalid inputs so pagination, sorting, counts, extents and policy filtering are observable. SensorML/SWE sets cover nested systems and scalar/record/vector/array/choice/nil/quality/UoM structures plus JSON, text and binary encodings. **[N,A,E,P]**

A scenario recipe declares an initial graph, synthetic actors and identities, policy/profile/configuration, logical clock, stable identifier namespace, ordered actions/events/faults, expected checkpoints, effect sinks and reset/cleanup. Ingestion, streaming/replay, command simulation, authorization/redaction, DDIL and two-node synchronization are separate scenario families that may compose but must resolve to a frozen run manifest. Command scenarios can target only an explicit simulator/recorder effect port; no corpus artifact contains a real endpoint or enables a physical adapter. **[A,P]**

The first public demonstration should use a clearly synthetic, geographically fictional environmental-sensor network. It exercises discovery, linked resources, observations, status changes and subscription/replay without implying operational authority. It is deterministic and resettable, carries conspicuous synthetic labeling, uses reserved/example identifiers and locations, and has a transitive dependency closure containing only public-release fixtures. Command behavior is disabled in the default public profile; a later explicitly named simulation may show feasibility/lifecycle while remaining unable to dispatch. Maritime, UGS, imagery, security, DDIL and conflict examples remain internal until separately reviewed. **[A,E,P]**

Goldens are selected per contract. Exact byte comparison is reserved for canonical encodings, digests/signatures, stable media artifacts and other deliberately lexical contracts. Typed/semantic JSON, XML or domain comparisons are the default for resources. Order-insensitive and partial comparisons require an explicit contract and negative assertions for facts that must not appear. Schema validation supplements rather than replaces semantic assertions. Normalization is path-specific, allowlisted, separately tested and recorded; blanket deletion of timestamps, IDs, links or URLs is prohibited. CI cannot accept or update goldens. **[A,D,P]**

Fixture CI performs manifest/schema, unique-ID, reference, digest, media/path, license/provenance, sensitivity, public-closure, deterministic-generation, source-drift and trace checks. It also scans for supported secrets, high-entropy/credential patterns, PII and controlled-data indicators and inspects supported archives. GitHub secret scanning is useful but cannot prove absence of unsupported secrets, PII or classified/controlled content, so automated checks are paired with named human sensitivity review. OWASP guidance reinforces excluding access tokens, passwords, keys, connection strings and sensitive PII from logs and artifacts. **[D,P]**

No unresolved issue blocks this strategy. Quantitative data volumes and performance budgets belong to IDR-SRV-054; adversarial/security depth and command-control verification to IDR-SRV-055; named client/server pairs and exchange packages to IDR-SRV-056. Acceptance does not authorize those topics, corpus implementation, use of controlled data, physical command effects, inbound Part 3, or a conformance/readiness claim. **[X]**

## 2. Scope and Plan Alignment

### 2.1 Completed Scope

This report completes all six authorized phases by:

- extracting corpus obligations from accepted IDR-SRV-001 through IDR-SRV-052 findings;
- inventorying current official CSAPI, SensorML, SWE Common and related artifacts;
- defining artifact roles, taxonomy, identity, manifests, provenance, licensing, sensitivity and trace integration;
- defining repository layout, versioning, acquisition, generation, golden and drift controls;
- mapping core resources plus ingestion, streaming, command, security, DDIL, synchronization, demo, performance and interoperability needs;
- defining CI, human review and generated-data reproducibility controls; and
- producing the required matrix, implementation proofs, recommendations and explicit later-topic handoffs.

### 2.2 Explicit Boundaries

This report does not:

- create every fixture, scenario, generator, schema bundle or test implementation;
- authorize copying an external artifact whose redistribution or modification terms have not been recorded;
- select performance volumes, rates, durations, service objectives or pass/fail budgets;
- define penetration-test depth, final authorization matrix, public command demo or physical gateway testing;
- select named external clients/servers or claim interoperability beyond a named run;
- use real operational, personal, credential, controlled, classified or command-endpoint data;
- authorize inbound Part 3 Publish/Subscribe, implementation work or topics after IDR-SRV-053; or
- establish conformance, certification, deployment, security or operational readiness.

### 2.3 Research Question Coverage

| Plan theme | Status | Evidence |
|---|---|---|
| fixture/scenario scope and taxonomy | Complete | Sections 4–5 |
| standards-derived and resource fixtures | Complete | Sections 8–9 |
| ingestion, event, command, policy, DDIL and synchronization | Complete | Section 10 |
| public demonstration and scenario design | Complete | Section 11 |
| large/generated/performance data | Complete, quantitative values deferred | Section 12 |
| metadata, provenance, licensing, sensitivity and traceability | Complete | Section 6 |
| layout, naming, versioning and lifecycle | Complete | Section 7 |
| exact/semantic/schema/partial/property assertion choices | Complete | Section 13 |
| generation, review, CI, secret scanning and drift | Complete | Sections 12–14 |
| implementation/community lessons | Complete and non-normative | Sections 3, 8, 11 |
| downstream performance/security/interoperability/final synthesis | Complete | Section 15 |

## 3. Evidence Base and Authority Classification

### 3.1 Primary Evidence

| Source | Pin/status checked | Authority | Use and limitation |
|---|---|---|---|
| OGC API - Connected Systems Part 1, OGC 23-001 | published 1.0 | normative | resource, representation and conformance fixture obligations; prose standard controls |
| OGC API - Connected Systems Part 2, OGC 23-002 | published 1.0 | normative | dynamic data, observation, status, control and command inputs/outputs |
| official Connected Systems repository | tag `v1.0.0`, commit `8e03b236…`; checked 2026-09-16 | official publication/source artifact | OpenAPI, schema and examples are candidate inputs; repository example status does not make each normative or defect-free |
| OGC schema registry Connected Systems Part 1/2 1.0 | checked 2026-09-16 | official schema distribution | offline schema bundle source; registry availability cannot be a required test dependency |
| SensorML 3.0 and official JSON schemas/examples | published/current registry checked 2026-09-16 | normative schema plus informative examples | structure/semantic fixture families; examples must be validated and licensed |
| SWE Common 3.0 | published 3.0 | normative | component, result and encoding fixture families |
| OGC API - Features Part 1, GeoJSON RFC 7946, HTTP RFC 9110, RFC 9457 | published editions | normative | collection/feature, geometry, HTTP and problem-detail assertions |
| IDR-SRV-001 through IDR-SRV-049 | accepted project reports | accepted baseline | obligations, models, profiles, safety, storage and lifecycle requirements converted into fixture needs |
| IDR-SRV-050 through IDR-SRV-052 | accepted project reports | accepted verification baseline | fixture manifest inputs, independent harness, trace graph, test layers, exact/semantic and seed controls |

The official registry currently exposes separate Part 1 and Part 2 1.0 schema packages; Part 1 includes OpenAPI example families for landing/conformance, collections, deployments, procedures, properties, sampling features and systems, while the schema tree contains corresponding resource and collection definitions. SensorML 3.0 publishes JSON schemas and an examples directory. This is sufficient to define acquisition families, not to pronounce every instance positive without validation. **[D,E]**

### 3.2 Tool, Platform, and Security Evidence

| Source | Checked | Evidence used |
|---|---|---|
| Cargo, nextest, SQLx, rstest, proptest, insta, assert-json-diff, jsonschema and schemars primary documentation | 2026-09-16 and IDR-SRV-052 pins | loaders, seeded regressions, structured comparison, validation, snapshot proposal and test isolation behavior |
| GitHub repository limits and large-file guidance | 2026-09-16 | 100 MiB object blocking, recommendation to keep individual objects small, generated/large data externalization |
| Git LFS documentation | 2026-09-16 | pointer-based storage and reduced ordinary PR diff visibility |
| GitHub secret scanning and push-protection documentation | 2026-09-16 | provider/pattern coverage and limitations; not a universal sensitivity detector |
| OWASP Secrets Management Cheat Sheet | 2026-09-16 | centralization, least privilege, lifecycle and never-log-secret controls |
| OWASP Logging Cheat Sheet | 2026-09-16 | exclude/mask tokens, passwords, connection strings, keys and sensitive PII |
| OGC legal/policy directives | 2026-09-16 | OGC schema copyright/rights language requires explicit license/usage treatment |

### 3.3 Informative Implementation and Community Evidence

- OSH and public demonstration endpoints show the usefulness of coherent, browsable sensor graphs but also show why mutable live data cannot be a regression oracle. **[I]**
- Connected Systems Go end-to-end examples show that deterministic seeded graphs and real HTTP workflows are practical; its outputs and internal model remain implementation evidence, not Glaux or standards truth. **[I]**
- CSAPI client, SECD and CSAPI Explorer findings show that small differences in links, representations, identifiers, schema/example interpretation and traversal can expose interoperability defects that broad snapshots miss. **[I]**
- pygeoapi and OGC API tooling reinforce using standards examples as acquisition candidates while keeping implementation-specific configuration and output separate. **[I]**

### 3.4 Authority and Conflict Rules

1. Published normative prose and normatively incorporated schemas control.
2. Accepted Glaux profile/architecture decisions control project behavior where they do not conflict with the standards.
3. Official examples and repository artifacts are reproducible evidence but do not silently override normative prose.
4. Implementation behavior and community discussions create hypotheses, regression cases and interoperability inputs, not obligations.
5. When schema, example, prose and implementation disagree, retain exact source bytes, record the discrepancy and create separate interpretation/adaptation artifacts. Do not “fix” the original in place.
6. A public URL supplies location, not redistribution permission, provenance completeness or a stability guarantee.

## 4. Fixture Requirement Extraction Methodology

The extraction unit is an assertion-level failure mode, not a file. For each accepted obligation, decision or risk the research followed:

`source/requirement → observable behavior → smallest discriminating facts → valid near-neighbor → negative/boundary mutations → test layer/target → source or generator → oracle type → setup/time/identity/effect controls → sensitivity/release class → evidence and downstream owner`

This prevents one “large realistic example” from becoming an opaque dependency for many unrelated claims. A fixture is retained only when it contributes a named fact, boundary, mutation or reusable setup graph. Scenarios compose those fixtures into stateful timelines. **[A,E,P]**

### 4.1 Extraction Partitions

Every relevant behavior is examined across these partitions:

- minimum valid and fully populated valid;
- optional field absent/present, extension known/unknown and profile-disabled;
- syntactically malformed, structurally invalid, semantically invalid and profile-invalid;
- just-inside, exact-boundary and just-outside values;
- hit, miss, tie, duplicate, stale, late, replayed, corrected and conflicting state;
- authorized-visible, authorized-redacted, policy-hidden and denied;
- connected, dependency-degraded, disconnected, replay/reconnect and recovered;
- small canonical, scenario-scale and generated performance-scale; and
- public-safe, internal, security-test-only and command-test-only.

An invalid case names its intended failing stage and a valid near-neighbor. A document that fails parsing cannot prove a semantic validation or authorization rule. **[A,P]**

### 4.2 Prior-Topic Trace Inputs

| Accepted source | Required corpus consequence |
|---|---|
| IDR-SRV-015–020 representation/validation | exact media types, profile/schema pins, canonicalization and negative-stage fixtures |
| IDR-SRV-021–024 resource/query/error | linked graph, discriminator query sets, pagination/order/extents and problem-detail cases |
| IDR-SRV-025–030 lifecycle/dynamic data | observation/status/control/command/event state sequences and effect-safe simulations |
| IDR-SRV-031–038 write/ingestion/persistence | idempotency, replay, quarantine, source trust, raw-reference and real-database state fixtures |
| IDR-SRV-039–043 streaming/Part 3 register | event/replay/gap/consumer scenarios; inbound Part 3 remains excluded unless separately authorized |
| IDR-SRV-044–049 implementation/operations | eleven profiles, typed configuration, redaction/telemetry, migration/restore and deterministic deployment fixtures |
| IDR-SRV-050 | exact scenario manifest, target-safety and independent oracle/evidence needs |
| IDR-SRV-051 | immutable IDs, typed edges, canonical digests, aliases/tombstones and evidence invalidation |
| IDR-SRV-052 | loader/support boundaries, deterministic clocks/IDs/effects, exact/semantic rules and CI tiers |

## 5. Fixture Taxonomy

### 5.1 Four Artifact Roles

| Role | Meaning | May be authoritative for | Must not become |
|---|---|---|---|
| source input | immutable bytes or facts fed to validator, server, adapter or generator | the input itself and recorded provenance | expected behavior merely because upstream called it an example |
| scenario recipe | declarative initial state, identities, clock, actions, faults and reset | how to construct/replay a scenario | a capture of whichever state a live server currently has |
| expected assertion/oracle | independently authored exact or semantic facts tied to requirements | the declared observable contract | production serializer/validator output copied back as expected truth |
| generated run output/evidence | resolved manifest, logs, responses, measurements, diffs and test results | what a named run observed | accepted fixture/golden without separate review and source change |

Generated candidate goldens belong to the fourth role until reviewed and promoted through an explicit source commit. **[A,P]**

### 5.2 Independent Classification Axes

| Axis | Values |
|---|---|
| origin/authority | `official-exact`, `official-adapted`, `accepted-project`, `synthetic-curated`, `deterministic-generated`, `regression-minimized`, `external-named-pair` |
| validity intent | `positive-valid`, `structurally-invalid`, `semantically-invalid`, `profile-invalid`, `adversarial-resource-bound` |
| artifact role | `source`, `scenario`, `oracle`, `generated-evidence` |
| scale | `micro`, `small`, `scenario`, `generated-scale` |
| sensitivity/release | `public`, `internal-development`, `security-test-only`, `command-test-only`, `restricted-reference-no-content` |
| lifecycle | `draft`, `approved`, `deprecated`, `superseded`, `tombstoned` |
| usage | unit/model, adapter contract, router/listener, database, conformance, performance, security, interoperability, demo |

“Synthetic” must describe provenance, not validity, sensitivity or realism. “Official” must identify the exact publisher/version/digest and still does not mean normative or positive. **[E,P]**

### 5.3 Core Corpus Families

1. bootstrap/metadata: landing page, conformance declaration, OpenAPI, collections and links;
2. Part 1 graph: systems, deployments, procedures, sampling features and properties;
3. SensorML/SWE: descriptions, components, values, records, arrays and encodings;
4. dynamic data: datastreams, observations, status and latest/materialized views;
5. query/error: filter discriminator sets, pagination/order and problem details;
6. ingestion/source trust: batches, identities, provenance, replay, quarantine and policy;
7. streaming/events: change events, subscription filters, replay, gap and reconnect;
8. controls/commands: definitions, feasibility, lifecycle and simulated effects;
9. security/policy/audit: fake principals, visibility, redaction and denied/hidden facts;
10. DDIL/synchronization: degraded state, two-node histories, collisions, conflict and reconciliation;
11. demo/interoperability: safe coherent stories and named-pair exchange packs;
12. performance/robustness: recipes, sentinel examples, fuzz seeds and minimized regressions.

### 5.4 Required Fixture Strategy Matrix

IDs below are reserved strategy identifiers; implementation registers their exact requirement/test edges in the IDR-SRV-051 catalog before use.

| Fixture/scenario ID | Category | Purpose | Requirement IDs | Test IDs | Source/provenance | Standards/profile version | Sensitivity classification | Generation method | Validation method | Golden/assertion strategy | CI applicability | Public-demo suitability | Downstream topic handoff | Notes / unresolved issues |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `FX-META-0001` | bootstrap minimal | landing/conformance/OAS traversal | CS1 metadata + accepted API | `TST-META-*` | curated from pinned official artifacts | CSAPI 1.0; profile pin | public | manual, immutable | schema + link/media semantics | semantic; exact media/header where contractual | PR/full conformance | yes | 056 | do not mirror current server output |
| `FX-GRAPH-0001` | Part 1 canonical | smallest complete linked system graph | CS1 resource requirements | `TST-GRAPH-*` | synthetic curated, official schemas | CSAPI 1.0; SensorML 3.0 | public | manual stable IDs | schema + referential/semantic checks | expected graph facts | PR/nightly | yes | 056 | base for other scenarios |
| `FX-GRAPH-0002` | Part 1 full/extension | optional fields, nested system and extensions | CS1/profile | `TST-GRAPH-FULL-*` | adapted/curated with transformation record | pinned profile | public | manual | schema + profile validator | semantic and explicit unknown-extension assertions | PR/nightly | conditional | 056 | keep adaptation distinct from exact source |
| `FX-SML-0001` | SensorML | minimal/full/nested descriptions | SML/CS1 | `TST-SML-*` | exact plus separately adapted OGC examples | SensorML 3.0 | public | acquired/manual adaptation | offline schema + semantic links | semantic XML/JSON facts; exact only canonical artifact | PR/nightly | yes | 056 | positive status requires independent validation |
| `FX-SWE-0001` | SWE Common | scalar/record/vector/array/choice/nil/quality/UoM | SWE/CS2 | `TST-SWE-*` | official definitions + synthetic values | SWE Common 3.0 | public | table-driven | schema + typed decode/round-trip | typed values; bytes exact only for encoding contract | PR/nightly | yes | 054,056 | binary separately claim-gated |
| `FX-OBS-0001` | observation/time | in/out-of-order, duplicate, late, corrected, spatial | CS2 observations | `TST-OBS-*` | synthetic logical timeline | CSAPI Part 2 1.0 | public | deterministic recipe | schema + temporal/domain invariants | stable IDs/facts and explicit order policy | PR/nightly | yes | 054,056 | not a load set |
| `FX-QUERY-0001` | discriminator set | filters, ties, pages, counts, extents, hidden rows | query/profile | `TST-QUERY-*` | synthetic curated graph | accepted query baseline | internal-development | manual + fixed builder | expected set math and boundary checks | semantic sets/order; exact links where required | PR/nightly | no default | 054,055,056 | include hit/miss/invalid neighbors |
| `FX-PROBLEM-0001` | errors | negotiation, validation, missing, denied, degraded | HTTP/RFC9457/profile | `TST-PROBLEM-*` | project-authored | RFC 9110/9457 + profile | public | manual | JSON Schema + status/header/privacy checks | exact type/status/code; semantic detail/instance | PR | yes subset | 055,056 | prose not exact unless public contract |
| `SCN-INGEST-0001` | ingestion | valid, duplicate, replay, quarantine, trust denied | accepted ingestion | `TST-INGEST-*` | synthetic publisher payloads | profile pin | internal-development | deterministic recipe | source/DB/event invariants | state transition + idempotency facts | PR/nightly | no | 055,056 | no real publisher identity |
| `SCN-STREAM-0001` | stream/event | subscribe, filter, replay, reconnect, gap, slow consumer | accepted event baseline | `TST-STREAM-*` | synthetic ordered events | enabled stream profile | internal-development | logical clock/fault schedule | listener plus event ledger | ordered checkpoints and permitted nondeterminism | nightly | no default | 054,055,056 | inbound Part 3 excluded unless authorized |
| `SCN-CMD-0001` | command simulation | feasibility, accept/reject/cancel/timeout/unknown | command lifecycle | `TST-CMD-*` | synthetic command definitions | command-test profile | command-test-only | effect-recorder recipe | state/effect ledger; adapter-absence gate | transition facts; zero physical effect | PR/nightly protected | no | 055 | only `sim://`/in-process non-network sink |
| `SCN-POLICY-0001` | security/policy | public, redacted, hidden, denied, revoked trust | policy/security baseline | `TST-POLICY-*` | fake identities and project policy | profile pin | security-test-only | manual deterministic | decision log + public/private oracle separation | semantic visible projection + explicit absence | protected nightly | no | 055 | hidden values never copied into public golden |
| `SCN-DDIL-0001` | DDIL | stale/last-known/dependency loss/recovery/command disabled | accepted DDIL baseline | `TST-DDIL-*` | synthetic graph/fault schedule | DDIL profile | internal-development | logical clock and proxy faults | health/state/event invariants | checkpoint semantics | nightly | no | 054,055 | quantitative duration deferred |
| `SCN-SYNC-0001` | synchronization | two-node duplicate, delayed, collision, conflict/reconcile | accepted sync baseline | `TST-SYNC-*` | synthetic two-node histories | sync profile | internal-development | deterministic histories | DB/audit/conflict graph | converged facts plus preserved lineage | nightly/RC | no | 054,055,056 | no arbitrary partial restore |
| `SCN-DEMO-0001` | public environmental demo | coherent discovery-to-observation/status story | demo/profile requirements | `TST-DEMO-*` | wholly synthetic fictional network | public demo profile | public | deterministic reseed | full public-closure, schema and smoke checks | semantic checkpoints | PR/demo deploy | yes, preferred | 056 | conspicuous synthetic labeling; commands disabled |
| `DATASET-PERF-0001` | generated scale | systems/datastream/time-series/rate/replay scales | later performance model | `TST-PERF-*` | project generator + recipe | build/profile pin | internal-development | seeded content-addressed generation | reproducibility/digest/statistics/sentinel | invariants and measurements, not huge golden | nightly/manual | no | 054 | sizes and budgets deferred |
| `FX-INTEROP-0001` | named-pair pack | portable exchange/traversal inputs | named pair | `TST-INTEROP-*` | public subset + partner-specific manifest | exact pair pins | public or controlled per pair | curated/exported | both-side validation and wire evidence | semantic pair expectations | manual/RC | conditional | 056 | never universal compatibility evidence |
| `FX-REG-0001` | minimized regression | permanent reproducer from property/fuzz/incident | linked defect/risk | `TST-REG-*` | minimized recorded failure | source/tool pins | reviewed classification | automated minimization then human promotion | target parser/model + corpus lint | exact minimal input + invariant | PR/nightly | usually no | 055 | raw crash artifact is not automatically committable |

## 6. Fixture Metadata, Provenance, Sensitivity, and Traceability Model

### 6.1 Manifest Contract

Use one `FixtureManifestV1`/`ScenarioManifestV1` family expressed as constrained YAML for review and canonicalized to JSON for hashing. Common required fields are:

- `id`, `title`, `artifact_role`, category/family, lifecycle state, manifest version, artifact version and `supersedes`/aliases;
- origin/authority, validity intent, scale and usage classes;
- source publisher, stable locator, source version/tag/commit, retrieval time, original digest, license/copyright/attribution, redistribution/modification decision and transformation recipe;
- exact standards, schema, profile, configuration and capability pins;
- sensitivity/release class, synthetic marking, public-demo eligibility and review evidence;
- each file's repository path, media type, encoding, byte size and cryptographic digest;
- generator ID/version/source digest, algorithm, seed, parameters, toolchain/environment and expected output statistics when generated;
- stable entity-ID namespace, logical clock/time zone, initial graph, timeline/fault schedule and permitted dynamic values;
- setup, isolation, reset, cleanup, effect sink and prohibited adapter/network conditions;
- expected facts/assertion-set IDs, exact/semantic/partial/order comparison policy and normalization rule IDs;
- requirement, test, risk, decision, deviation and downstream edges from the IDR-SRV-051 graph;
- validators/tools, versions, results, validation time and validation-evidence digests; and
- owner, sensitivity reviewer, technical reviewer, approval state, notes and unresolved discrepancies.

The canonical digest covers the manifest after stable path/reference resolution plus every declared source/oracle file digest. It does not pretend that a live URL's current bytes are immutable. **[A,P]**

### 6.2 Provenance and Licensing Rules

1. Every externally derived byte has a source locator, publisher, version/digest, retrieval date and license/terms record.
2. Public accessibility is not evidence of permission to redistribute or modify.
3. Exact source packages are retained only where permitted. Otherwise, retain metadata, expected digest and a controlled acquisition procedure; required tests still need a mirrored or generated offline-safe alternative whose rights permit use.
4. An adaptation never overwrites the exact source artifact. It records the input digest, deterministic transformation, changed facts and “not official” status.
5. Copied copyright/license notices remain adjacent under `licenses/`; a fixture manifest records any attribution needed in distributions or demos.
6. A source-term ambiguity blocks redistribution/public-demo approval, not necessarily internal analysis. Legal interpretation remains a project/legal responsibility.
7. Source drift creates a review proposal. It never silently updates a fixture, schema or oracle.

### 6.3 Sensitivity and Release Classes

| Class | Permitted content/use | Minimum control |
|---|---|---|
| `public` | wholly synthetic or licensed redistributable content safe for repo, CI artifacts and demo | automated scans plus technical/sensitivity review; transitive public dependency closure |
| `internal-development` | non-operational test internals with no sensitive source content | ordinary repository access, scan and reviewer |
| `security-test-only` | fake policies/identities, adversarial patterns and privileged expected facts | separated paths/artifacts, restricted CI access/retention, named security review |
| `command-test-only` | simulated command/control data and effect ledgers | profile gate, effect-recorder-only invariant, no physical adapter/endpoints |
| `restricted-reference-no-content` | metadata references a restricted source, but repository contains none of its text/data | provenance and review record; never reconstruct or imply validation of inaccessible source |

Real secrets, reusable credentials, personal data, operational locations/routes, controlled labels/metadata and real command targets are prohibited in all classes. Encryption is not a reason to commit such material as a fixture. **[A,D,P]**

### 6.4 Redaction Oracle Separation

A redaction test needs knowledge of what must be hidden without leaking that value into the public expected response. Store the privileged input/oracle only in a restricted test compartment, and store the public projection as independently declared visible facts plus explicit absent paths/categories. The public artifact must not contain the hidden value in comments, filenames, hashes intended for guessing, logs or candidate diffs. Fake values should be unmistakably synthetic and chosen to avoid resemblance to live credentials while still exercising the semantic field type. **[D,E,P]**

### 6.5 Trace Edges and Evidence Freshness

The fixture manifest owns facts about the artifact; the IDR-SRV-051 catalog owns normalized edges. Tests reference fixture/scenario/golden IDs and resolved digests, not fragile paths. A source, manifest, transformation, schema/profile, generator, seed/parameter, oracle or normalization-rule change invalidates dependent evidence until rerun. Renames preserve immutable IDs; replacements use `supersedes`; removed artifacts leave tombstones so historical evidence remains interpretable. **[A,P]**

## 7. Fixture Storage Layout, Naming, and Versioning Findings

### 7.1 Recommended Repository Layout

```text
fixtures/
├─ registry.yaml
├─ schemas/                    # manifest schemas and pinned offline standards schemas
├─ sources/                    # immutable exact upstream packages and acquisition manifests
├─ canonical/
│  ├─ metadata/
│  ├─ part1-graph/
│  ├─ sensorml/
│  ├─ swe/
│  ├─ observations/
│  ├─ query/
│  └─ problems/
├─ negative/                   # named failing-stage cases and valid near-neighbors
├─ scenarios/
│  ├─ ingestion/
│  ├─ streaming/
│  ├─ command/
│  ├─ policy/
│  ├─ ddil/
│  └─ synchronization/
├─ generators/                 # source, recipes, schemas and sentinel outputs
├─ goldens/                    # only approved deliberately stable oracle artifacts
├─ regressions/                # minimized property/fuzz/defect inputs
├─ demo/                       # public-only scenario manifests and presentation text
└─ licenses/

generated/fixtures/            # ignored; hydrated/generated outputs and resolved run manifests
target/test-evidence/           # ignored; run evidence and candidate diffs
```

Each leaf artifact directory contains a manifest and only the files that manifest declares. The registry is an index and validation root, not a second copy of metadata. Content-addressed storage/deduplication may be introduced behind stable manifest references if repetition becomes material; human-reviewable paths remain the interface. **[P]**

`docs/examples/` is for explanatory, public-safe documentation derived from approved fixture IDs; it is not a second fixture authority. Conformance, Rust tests, simulator and demo code resolve the same corpus IDs through read-only loaders rather than copying files into their own trees. External Glaux repositories receive a versioned release/export package with a manifest, not an unmanaged fork. **[A,P]**

### 7.2 Identifier and File Naming

- fixture: `FX-<FAMILY>-NNNN`;
- scenario: `SCN-<FAMILY>-NNNN`;
- generator: `GEN-<FAMILY>-NNNN`;
- approved golden: `GOLD-<FAMILY>-NNNN`;
- generated dataset descriptor: `DATASET-<FAMILY>-NNNN`.

IDs are uppercase ASCII, immutable, never recycled and unrelated to mutable file names. Slugs are lowercase kebab case for readability. Version/digest is explicit rather than embedded in the ID. Existing accepted IDs are preserved or registered as aliases; implementation must not mass-rename historical references merely to match this convention. **[A,P]**

### 7.3 Version and Lifecycle Semantics

- Manifest schema version changes when interpretation/required fields change.
- Artifact version changes when source bytes, recipe, expected facts or public meaning changes.
- A path-only move with identical canonical digest preserves artifact version and ID.
- Corrections that alter assertions or setup create a new version and stale dependent evidence.
- A semantically different scenario receives a new ID; compatible enrichment may version the same ID if old expectations remain reproducible.
- Deprecation points to a replacement; tombstones retain ID, last digest, reason and evidence-retention location.
- A corpus release freezes registry/manifests/digests/licenses/generator pins and produces an SBOM-like inventory. Git tag alone is insufficient without the resolved corpus manifest.

### 7.4 Git, LFS, and External Storage Decision

| Option | Benefits | Costs/risks | Decision |
|---|---|---|---|
| ordinary Git for small reviewable artifacts | atomic review, history, branch/PR diffs, simple offline use | repository growth if abused | default |
| Git LFS | permits irreducible large binary objects with Git pointers | ordinary diff shows pointer; client/storage/bandwidth and retention dependencies | conditional exception only |
| deterministic generation | tiny reviewed source; arbitrary scale; reproducible parameters | generator/tool drift, generation cost | default for scale datasets |
| content-addressed release/object storage | exact large output can be reused and verified | availability, mirroring, access and retention need ownership | optional for expensive generated/external artifacts |
| live external endpoint | current integration observation | mutable, unavailable, unsafe, non-reproducible | optional interoperability only; never required fixture source |

GitHub blocks files above its hard object limit and recommends keeping repositories and individual objects small; these are hosting constraints, not Glaux's design budget. During implementation, measure clone/checkout/CI-cache cost and set lower project thresholds. CI should reject undeclared binary formats, file-count explosions and unexplained aggregate growth even below platform limits. **[D,P]**

## 8. Standards-Derived and CSAPI Resource Fixture Findings

### 8.1 Acquisition Pipeline

For every official standard package:

1. pin published edition, repository tag/commit and registry path;
2. fetch once through a controlled acquisition task and compute digests;
3. preserve license/copyright and inventory every selected file;
4. validate exact bytes with independently pinned tools where applicable;
5. classify each example as positive, negative, ambiguous or not executable;
6. record prose/schema/example discrepancies;
7. create adapted profile artifacts only as separate derivations; and
8. publish an offline bundle used by required CI.

An online drift job may report new upstream bytes, tags or schemas. It cannot modify the accepted bundle or make ordinary CI depend on network availability. **[D,P]**

### 8.2 Resource Coverage Pattern

Each CSAPI resource family requires:

- minimal valid representation;
- fully populated representation;
- each meaningful optional relationship present/absent;
- known extension plus unknown extension behavior;
- alternate allowed representation where claimed;
- structurally invalid and semantically/profile invalid variants;
- missing, malformed, dangling, circular and policy-hidden links as applicable;
- create/update input variants for writable resources; and
- stable linked-graph expectations independent of insertion order.

Apply this pattern to landing page, conformance declaration, OpenAPI documents, collections, systems, deployments, procedures, sampling features, properties, datastreams, control streams, observations, status, system events, source registration/trust, policy metadata, commands, feasibility results, audit and synchronization/conflict records. The last four groups are project/profile records where the standard does not define the representation; their manifests must label project authority rather than cite CSAPI as the source. **[N,A,P]**

### 8.3 Exact Official Versus Adapted Assets

The immutable exact layer answers, “what did the pinned publisher distribute?” The adapted layer answers, “what input demonstrates the accepted Glaux profile?” Keeping both lets tests detect upstream defects and adaptation drift. A defect discovered in an official example is recorded with expected failing validator/stage and, when useful, retained as a negative fixture. A corrected positive has its own ID and transformation rationale. **[D,E,P]**

### 8.4 OpenAPI and Schema Fixtures

- Store the exact pinned OpenAPI/schema package and digest when licensing permits.
- Resolve references offline and test that the bundle is self-contained; remote resolution is a separate diagnostic.
- Validate OpenAPI syntax and project generation assumptions separately from endpoint conformance.
- Keep schema-valid examples distinct from semantic/profile-valid examples.
- Do not use server-generated OpenAPI as its own oracle. Compare generated output to independently curated contract facts and deliberately stable approved fragments.
- Treat schema dialect/format behavior and unresolved references as explicit validator configuration, not ambient defaults.

### 8.5 Implementation Lessons

OSH/live demos provide discovery stories but change over time. Connected Systems Go provides valuable example workflows but its choices cannot resolve a normative ambiguity for Glaux. CSAPI client and SECD work show that traversal and representation mismatches are often best isolated by tiny linked fixtures, not a giant copied dataset. Therefore implementations may inspire scenario families and regression cases only after provenance, license, profile and independent expected facts are recorded. **[I,E,P]**

## 9. SensorML, SWE Common, Observation/Status, Query/Filter, and Error Fixture Findings

### 9.1 SensorML Corpus

The minimum family includes simple sensor, procedure, platform, system-of-systems, deployment-linked system, output definitions, taskable description, minimal valid, fully populated, extension-bearing and intentionally invalid instances. Nested identity and link graphs must remain stable across JSON/XML or other claimed representations. Positive designation requires the appropriate official schema plus project semantic/profile checks; schema validation alone cannot prove resource linkage, identifier policy or taskability. **[N,A,P]**

Large SensorML documents should be represented by a small canonical example, a deterministic nesting/enrichment generator and a sentinel digest/statistics record. Commit a large exact source only when it is irreducible, licensed and reviewable through a defined process. **[E,P]**

### 9.2 SWE Common Corpus

Cover scalar numeric/text/boolean/time/category/quantity values; records; vectors; arrays; choices; nil reasons; quality; units; constraints; command parameter structures; observation result structures; and each claimed JSON/text/binary encoding. Every value fixture separates component definition from encoded values and records byte order, block/count/separator rules and nil/quality behavior where relevant. Typed decode/re-encode plus semantic value comparison is the default. Exact bytes are required only when the encoding's lexical form, signature or digest is the contract. **[N,P]**

### 9.3 Observation and Status Sets

Observation scenarios include single/multiple values, interval and instant time, exact boundary time, out-of-order arrival, duplicate/idempotent replay, correction/supersession, late arrival, spatial result/location, multi-field SWE result and invalid component/result mismatch. Expected facts distinguish event/phenomenon/result/ingestion times rather than normalizing them into one timestamp. **[A,P]**

Status sets cover available, unavailable, degraded, stale, last-known, unknown, delayed and source-unavailable states. They record whether state is directly observed, derived or cached, plus the logical time and staleness basis. Latest/materialized-view assertions name the selection/tie-break facts and lineage rather than snapshotting an entire database row or response. **[A,P]**

### 9.4 Query Discriminator Sets

A query set is useful only when expected results differ across the behavior being tested. It therefore includes:

- at least one match, one non-match and one invalid input per operator;
- exact lower/upper boundaries plus just inside/outside cases;
- equal sort keys and stable tie-break identifiers;
- enough rows for empty, partial, exact-full and next pages;
- geometries that are disjoint, intersecting, boundary-touching and antimeridian/CRS relevant where supported;
- nested/property values that distinguish missing, null, empty and present;
- public, redacted and hidden entities for policy-filtered count/extent/page behavior; and
- expected ID sets, order, totals/extents and link facts stated independently.

Do not use full response goldens as the only query oracle. Expected stable IDs and aggregate facts diagnose ordering, filtering and representation separately. **[A,P]**

### 9.5 Content Negotiation and Problem Details

Create tables for supported/unsupported `Accept`, request `Content-Type`, quality values, default representation, malformed body, validation finding, invalid query, missing resource, policy-hidden resource, unauthenticated/unauthorized request and degraded dependency. Exact assertions cover status, media type, required headers, stable problem type/code and privacy behavior. Human-readable title/detail, correlation/instance values and extension ordering are semantic unless explicitly made a public lexical contract. A 404 used to hide resource existence is compared against a public projection; the fixture must not leak the hidden identifier/value through detail, link, log or candidate golden. **[A,P]**

## 10. Ingestion, Streaming/Event, Command/Control, Security/Policy, DDIL, and Synchronization Fixture Findings

### 10.1 Scenario Recipe Model

Every stateful scenario resolves to:

```text
initial graph + profile/config + synthetic actors/identities/policy
+ logical clock/time zone + stable ID/entropy namespace
+ ordered actions/events + dependency/fault schedule
+ expected checkpoints/invariants + permitted nondeterminism
+ effect sink + isolation/reset/cleanup + safety/release class
```

Composition is permitted only if the resolver detects ID, clock, policy and dependency conflicts and emits a frozen resolved manifest/digest before execution. Hidden inheritance or “latest” scenario imports are prohibited. **[P]**

### 10.2 Ingestion and Publisher/Adapter Fixtures

The ingestion family covers valid submission, invalid source identity, structurally invalid payload, semantically invalid payload, duplicate batch, idempotent replay, modified replay, raw-payload reference, quarantine, trust denied/revoked, policy blocked and simulator-generated input. Inputs carry fake source identities and signed-test metadata only where the signature mechanics themselves are under test; reusable keys never enter the corpus. Expected assertions cover accepted canonical facts, provenance, idempotency key, quarantine reason, audit/event emission and absence of unintended writes. **[A,P]**

Cross-repository publisher/simulator integration uses exported corpus release IDs and digests. Server-only tests retain local source inputs; ecosystem tests pin the publisher/simulator build and transport envelope separately so a producer defect is not mislabeled a server failure. **[A,P]**

### 10.3 Streaming and Event Scenarios

Event types include system/resource change, observation, status, command lifecycle, trust/policy, DDIL transition and synchronization conflict. Scenarios cover initial subscribe, filtered subscribe, resume/replay, reconnect, duplicate delivery, event gap, slow consumer/backpressure, outbox replay, policy-hidden event and dependency interruption. **[A,P]**

Each expected event uses a stable logical sequence/checkpoint plus allowed delivery semantics from the accepted streaming design. Wall-clock arrival and thread scheduling are not oracles. When order is partial, the manifest declares a happens-before graph rather than sorting expected files after the fact. Part 3 Publish/Subscribe outbound work may reuse these fixtures only under its separately accepted boundary; inbound Part 3 remains excluded from this report. **[A,X]**

### 10.4 Command and Control Safety

The command family includes control-stream/definition, valid/invalid parameters, feasibility request/result, accepted/rejected/safety-denied/authorization-denied, cancel, timeout, unknown outcome and simulated gateway response. Required invariants are:

- the scenario profile cannot load a physical-effect adapter;
- the destination is an in-process recorder or a non-resolving reserved simulation URI, never a network-derived endpoint;
- a generated command carries explicit test-only identity and logical expiration;
- the effect ledger proves intended requests and also proves zero unauthorized/duplicate effects;
- reconnect/replay never repeats a physical effect; and
- cleanup cannot invoke cancellation against a real target.

The public demo profile disables command dispatch. A later public simulation requires its own review and conspicuous labeling; seeing a command lifecycle is not evidence of operational tasking authority. **[A,P]**

### 10.5 Security, Policy, and Redaction

Use fake unauthenticated, viewer, contributor, administrator and source identities with a project-authored policy bundle. Cover object/field/relationship/event visibility, redaction, existence hiding, revoked trust, stale policy, command denial and diagnostic suppression. Expected results include decision reason categories and public facts, not actual secret material. Security-test-only fixtures are never dependencies of demo/public fixtures. **[A,P]**

Adversarial payloads and credential-like strings require containment: document their inert purpose, minimize them, disable resolution/execution, classify the artifact and prevent accidental reuse. Tests that validate secret scanners can synthesize scanner-specific tokens inside ephemeral protected jobs rather than committing realistic credentials to history. **[D,P]**

### 10.6 DDIL Scenarios

Cover cached schema/profile availability, stale data, last-known value, delayed updates, dependency loss, local-only reads, degraded health/readiness, bounded queueing, recovery/reconciliation and command-disabled degraded mode. The manifest records the failure injection point, logical duration/order and expected user-visible provenance/staleness. IDR-SRV-054 will decide quantitative outage/delay/rate envelopes; IDR-SRV-055 will deepen security and command behavior. **[A,P]**

### 10.7 Synchronization and Conflict Scenarios

Construct deterministic two-node histories for duplicate replay, identifier collision, delayed observation batch, stale policy/trust conflict, command-status conflict, audit gap, quarantine and explicit conflict record. Each node has a stable namespace, source authority and logical/vector ordering as accepted by the synchronization design. Expected facts include preserved alternatives/lineage, deterministic resolution where defined, surfaced unresolved conflict and post-reconciliation convergence. Do not erase conflicts merely to make complete database snapshots equal. **[A,P]**

## 11. Public Demo and Scenario Corpus Findings

### 11.1 First Public Scenario

Select `SCN-DEMO-0001`, a fictional environmental monitoring network:

- several fixed stations and one generic mobile platform in explicitly fictional coordinates or a clearly non-operational demonstration area;
- systems, procedures, deployments, sampling features, properties and datastreams linked from discovery;
- temperature, pressure and generic air-quality observations with safe units/ranges;
- deterministic status change and delayed/late observation;
- read-only event subscription/replay if the selected public profile supports it; and
- command functionality disabled.

This family is understandable to non-specialists, exercises the central CSAPI graph and dynamic data, and avoids implying surveillance, maritime identity, military collection or operational tasking. All identifiers, organizations and measurements are labeled synthetic in machine metadata and presentation text. **[E,P]**

### 11.2 Deferred/Conditional Public Families

| Family | Decision | Reason/control |
|---|---|---|
| generic transportation | conditional | fictional routes/assets; avoid real schedules or identifiers |
| maritime/AIS-like | internal first | never use real MMSI, vessel, track or port operation; synthetic identity must be obvious |
| unattended ground sensor | internal first | sensitive implications; generic event semantics and communications review required |
| imagery/status | conditional | no real imagery, collection target, EXIF/location or controlled metadata |
| generic platform/sensor network | public candidate | retain fictional geography/authority |
| command-disabled tasking story | conditional later | display feasibility/lifecycle only with effect port impossible and explicit simulation label |
| DDIL/two-node conflict | internal first | useful engineering story but operational/synchronization implications need review |

### 11.3 Demo Packaging and Reset

The demo package contains a scenario manifest, public fixture dependency closure, licenses/attribution, narrative, synthetic-data notice, deterministic seed, logical start time, reset command and expected smoke checkpoints. Reset recreates the database from migrations plus the scenario recipe; it does not restore a hand-edited live snapshot. Periodic motion/observations derive from logical tick/seed so the story can appear dynamic while remaining replayable. Public deployment logs/evidence use only public fields. **[A,P]**

### 11.4 No Live Demo as Oracle

A public OSH, CS-Go or Glaux instance is useful for exploratory compatibility and presentation. Because its version, configuration, availability, data and time evolve, it cannot supply required expected results. An interoperability run captures its exact endpoint/build facts and observations as evidence, never imports current output into a golden automatically. **[I,P]**

## 12. Performance and Large/Generated-Data Set Findings

### 12.1 Generator Contract

A generator is versioned production-quality test infrastructure. Its manifest records source digest/build, algorithm version, deterministic pseudo-random implementation, seed, parameters, profile/schema pins, logical clock, ID namespace, environment/toolchain, output file count/size/digests and summary statistics. It validates generated instances and emits a resolved dataset descriptor. Running the same pinned generator/recipe/environment must yield the same declared semantic dataset; if compressed bytes or serialization are allowed to vary, that distinction is explicit. **[A,P]**

Generators themselves receive small example, property/invariant, boundary and determinism tests. A double-run check compares canonical digests/statistics. Changes to the algorithm or dependency lock stale generated evidence even when the seed text is unchanged. **[P]**

### 12.2 Dataset Families for IDR-SRV-054

- many systems/deployments/procedures and deep/wide linked graphs;
- many datastreams per system and many fields per stream;
- high-rate, long-range, out-of-order and late observation histories;
- dense/sparse geospatial distributions and difficult filter boundaries;
- pagination/sort ties and policy-hidden rows at scale;
- streaming fan-out, slow consumers, replay/backlog and reconnect waves;
- command-status churn against a recorder only; and
- DDIL queue/recovery and synchronization/conflict backlogs.

This report defines reproducibility and safety, not cardinalities, rates, durations, distributions, warm-up, resource envelopes or pass/fail thresholds. IDR-SRV-054 owns those choices. **[X]**

### 12.3 Commit, Generate, or Hydrate

- Commit manifests, generator source, small sentinels and compact minimized regressions.
- Generate ordinary large datasets on demand into ignored storage.
- Cache by recipe and output digest; verify before reuse.
- Hydrate exceptionally expensive outputs from controlled content-addressed storage with mirror/retention/availability ownership.
- Never commit a large SQL database dump as the canonical dataset; generate/load through public/domain inputs or deterministic seed tooling so schema migrations and semantics remain visible.
- Never require a live external data feed for a blocking run.

### 12.4 Property and Fuzz Corpora

Property strategies generate cases; their seeds/results are not individually curated fixtures. A failure is reproduced, minimized, classified for sensitivity and promoted as `FX-REG-*` with the failing invariant/tool/version and a stable expected outcome. Fuzz seed corpora remain small and structurally diverse. Raw crashes, memory dumps and arbitrary third-party payloads stay protected until reviewed; only minimal safe reproducers enter Git. **[A,D,P]**

## 13. Golden-File, Semantic Assertion, Normalization, and Drift-Control Findings

### 13.1 Assertion Selection

| Output/behavior | Primary assertion | Exact golden? |
|---|---|---|
| canonical encoding, signature input, digest or stable binary codec artifact | bytes plus media/encoding metadata | yes |
| OpenAPI/schema bundle or curated stable fragment | structural/schema checks plus exact approved artifact where release contract | selective |
| ordinary JSON/GeoJSON resource | typed semantic facts, links, numbers, geometry and absence rules | no full response by default |
| SensorML/SWE representation | schema plus typed semantic equivalence; canonical bytes only when claimed | selective |
| query result | expected ID set/order/count/extent/link facts | no |
| problem response | exact stable type/status/code/header; semantic/redaction-aware detail | selective fragment |
| event stream | event type/subject/data and declared ordering graph | no wall-clock/full transcript |
| command lifecycle | transition/effect ledger and prohibited-effect absence | no broad snapshot |
| human CLI/report | machine structure first; reviewed snapshot for intentionally stable presentation | selective |
| performance run | invariants plus measurements/environment | never a giant output golden |

Schema validation cannot prove business semantics; snapshots cannot prove schema/profile validity; partial assertions cannot omit required negative/absence facts. Use complementary assertions as the failure mode requires. **[A,P]**

### 13.2 Semantic Comparison Rules

- JSON objects compare by members and typed values; arrays remain ordered unless the specific contract declares set/multiset semantics.
- Numeric tolerances are field-specific, justified by the measurement/encoding contract and include boundary tests.
- Geometry comparison declares coordinate order, CRS, precision and topology/equality semantics; arbitrary rounding is prohibited.
- XML/SensorML comparison uses namespace-aware structure/canonicalization where lexical identity is not required.
- Links compare required relation, target-resolution semantics, type/title/hreflang fields as applicable; base-origin rewriting is a declared normalization, not deletion.
- Missing, null, empty and redacted are distinct states.
- Partial assertions enumerate required present facts and prohibited/absent facts so security or extension regressions cannot hide in ignored content.

### 13.3 Normalization Registry

Normalization is an allowlisted transformation identified by a stable rule ID, path/media applicability, rationale, before/after examples and tests. Permitted examples include replacing a recorded test origin with a placeholder or mapping a generated trace ID to a run-local symbolic ID when that exact value is non-contractual. Prohibited defaults include deleting every timestamp, ID, URL, link, ordering difference or unknown field. Security-sensitive fields are asserted absent/redacted, not normalized away. **[A,P]**

### 13.4 Golden Review Workflow

1. A failing test emits a candidate outside tracked fixture paths.
2. Tooling reports producer/tool/version, old/new digests, semantic diff, changed paths and affected fixture/assertion/requirement IDs.
3. The author states the source/decision change and why expected behavior changed.
4. CI validates but cannot write/accept the golden.
5. A qualified reviewer evaluates each logical change; “accept all” is not evidence.
6. Promotion is a source commit updating manifest/version/digest and staling dependent evidence.

For a normative conflict, update of the current server output is rejected until the requirement/interpretation is resolved. Candidate `.snap.new` or equivalent files are run artifacts and never implicitly accepted. **[A,D,P]**

### 13.5 Drift Detection

Drift checks cover source URI/tag/digest; schema/profile pin; exact source package; transformation recipe; generator/tool lock; seed/parameters; manifest/files; expected facts/goldens; normalization rules; requirement/test edges; sensitivity/public closure; and consumer compatibility. Upstream change is reported separately from local corruption or intentional project update. Evidence remains bound to the prior digest and becomes stale; historical artifacts are not rewritten. **[A,P]**

## 14. CI Validation, Secret Scanning, Fixture Review, and Generation Workflow Findings

### 14.1 Blocking PR Checks

1. validate manifest YAML against pinned schema and canonicalize deterministically;
2. enforce unique/non-recycled IDs, valid lifecycle/aliases and closed references;
3. verify path containment, declared media/encoding/size/digest and reject symlink/path traversal;
4. validate license/provenance/transformation and required attribution fields;
5. validate standards/profile/schema pins and run offline structural/semantic validators;
6. resolve trace edges and reject missing requirement/test/assertion/normalization references;
7. validate goldens/assertion descriptors; ensure update mode is disabled;
8. scan supported secrets, high-entropy/credential patterns, PII/sensitivity terms and prohibited operational/control indicators;
9. inspect permitted archives with entry/count/expanded-size/nesting/path limits; reject unknown binary/archive formats;
10. verify public/demo transitive closure contains only approved public artifacts/licenses;
11. run bounded generator determinism/sentinel checks and minimized regression tests; and
12. report per-file/count/aggregate corpus growth and require explicit review for threshold exceptions.

Secret scanning is defense in depth. GitHub's supported patterns/push protection cannot recognize every secret, organization-specific credential, PII or controlled value; false positives and timeouts also exist. A clean scanner result therefore never changes a fixture's sensitivity class or replaces named human review. **[D,P]**

### 14.2 Nightly and Release Checks

Nightly jobs expand schema/profile validation, generator double-runs, property/fuzz regression corpora, source-drift observation, full scenario replay and optional controlled external acquisition checks. Release-candidate jobs resolve the exact corpus release, regenerate/hydrate required datasets in a clean environment, verify all digests/licenses/public closure and bind the resolved manifest to conformance/performance/security/interoperability evidence. External-source unavailability is `error`/diagnostic, never target conformance failure or an excuse to consume mutable unverified bytes. **[A,P]**

### 14.3 Review Ownership

| Change | Required review |
|---|---|
| new/changed external source or license | corpus maintainer plus license/provenance review |
| positive/negative classification or standard adaptation | standards/profile reviewer |
| expected fact, golden or normalization | owning requirement/test reviewer; security reviewer if visibility affected |
| sensitivity or public-demo eligibility | named sensitivity/publication reviewer |
| command scenario/effect configuration | command-safety reviewer |
| generator/large-data recipe | corpus maintainer plus downstream performance/security owner as relevant |
| minimized adversarial/fuzz case | technical plus sensitivity/security review |

Authors cannot self-approve a change that both modifies behavior and rewrites its oracle. Repository branch protection should encode independent review where available. **[P]**

### 14.4 Safe Generation and Artifact Handling

Generators run with bounded CPU/memory/time/file counts, no ambient network, explicit output directory and least privilege. They cannot follow links outside the work area or unpack unbounded archives. Run logs apply accepted redaction rules and never print full sensitive inputs, environment variables, authorization headers or secret-provider material. CI artifacts inherit sensitivity, access and retention from their most restrictive input; ordinary public PR artifacts contain only public/internal-safe data. **[A,D,P]**

### 14.5 Implementation Proofs Required Before Corpus Expansion

1. manifest schema, canonical JSON digest and stable-ID/alias/tombstone validation;
2. exact-official plus adapted-copy provenance/license/transformation round trip;
3. cross-reference graph and evidence invalidation on fixture digest change;
4. official-example defect classified negative while corrected adaptation remains separate;
5. exact versus semantic comparison and path-scoped normalization with mutation tests;
6. generator double-run reproducibility with seed/parameters/tool lock and sentinel statistics;
7. content-addressed large-data hydration with corruption/offline/retention failure behavior;
8. sensitivity scan plus a demonstrated unsupported-secret/PII blind spot requiring human review;
9. command scenario proving the physical adapter cannot load and effect ledger remains simulated;
10. logical timeline replay for ingestion/stream/DDIL or synchronization checkpoints;
11. source/schema/profile drift staling dependent evidence without rewriting history; and
12. one shared fixture consumed by internal tests and independent harness while their oracle implementations remain independent.

These are implementation entry gates, not completed proofs in this research report. **[X]**

## 15. Downstream Topic Handoff Matrix

| Downstream owner | Inputs fixed by IDR-SRV-053 | Decisions explicitly retained downstream | Required non-substitution rule |
|---|---|---|---|
| IDR-SRV-054 performance/load/stress/streaming | generator/recipe/seed/digest contract; scale dataset families; logical clocks; event/DDIL/sync scenario structure; measurement outputs not goldens | workload model, distributions, cardinalities, rates, durations, concurrency, warm-up, resources, budgets and gates | reproducible data does not make a performance result representative or passing |
| IDR-SRV-055 security/authorization/command-control | sensitivity classes; fake identities/policy; public/private oracle separation; safe command effect recorder; adversarial/minimized corpus handling; scan limitations | threat/risk coverage, authorization matrix tests, abuse/resource cases, scanner/tool depth, credential lifecycle tests and command-control assurance gates | clean secret scan or synthetic data is not security assurance; simulated transition is not physical-effect authorization |
| IDR-SRV-056 external-client interoperability | public canonical graph, SensorML/SWE/query/problem packs; named-pair provenance/export manifest; live endpoint non-oracle rule | named clients/servers/versions, pair scenarios, negotiation/traversal expectations, compatibility findings and pair-specific packaging | success with one pair is neither universal compatibility nor conformance |
| IDR-SRV-057 final synthesis | accepted taxonomy, manifests, provenance/licensing, safety, demo, generation, golden, review and CI decisions | reconcile all accepted verification topics into final roadmap/design baseline | do not elevate an unimplemented strategy to completed evidence |
| server implementation roadmap | layout, loaders, registry, proof list and phased corpus priorities | implementation sequence, estimates, maintainers and actual thresholds | code/output cannot author its own oracle |
| publisher/simulator/web/mobile repositories | versioned public/export corpus package IDs and scenario contracts | repository-specific adapters and UI/demo behavior | do not fork or silently mutate fixture truth |

### 15.1 First-Implementation Corpus Slice

The smallest decision-usable slice is:

1. registry/manifest schemas, canonical digest and linter;
2. pinned offline CSAPI/OpenAPI/schema package plus licensing inventory;
3. `FX-META-0001`, `FX-GRAPH-0001`, `FX-SML-0001`, `FX-SWE-0001`, `FX-OBS-0001`, `FX-QUERY-0001` and `FX-PROBLEM-0001` in small positive/negative pairs;
4. one ingestion/replay scenario and one effect-recorder-only command lifecycle scenario;
5. the public environmental demo derived from only public artifacts;
6. exact/semantic/normalization helpers with update-disabled CI;
7. deterministic generator proof with a small sentinel and one generated scale descriptor; and
8. all twelve implementation proofs before broad corpus growth.

This slice supports TDD and early conformance work while avoiding premature performance volumes, broad security/adversarial data or named-client matrices. **[P]**

### 15.2 Required Governance Handoff

The next two governance actions are plan-owner acceptance of this completed report and authorization of exactly one next eligible topic, `IDR-SRV-054: Performance, Load, Stress, and Streaming Test Strategy`. The formal combined response is:

`accept IDR-SRV-053 and authorize IDR-SRV-054`

Under the established conversational shorthand, a subsequent bare `proceed` may express that combined action. Until the plan owner gives that instruction, this report remains in review and IDR-SRV-054 remains unauthorized. **[X]**

## 16. Recommendations

1. Adopt one governed corpus registry and the four-role separation of source, scenario, oracle and generated evidence. **Priority: high.**
2. Implement constrained YAML manifests validated by pinned JSON Schema and canonicalized JSON digests aligned with IDR-SRV-051. **Priority: high.**
3. Use immutable stable IDs, explicit versions/digests, aliases, supersession and tombstones; tests resolve IDs rather than paths. **Priority: high.**
4. Classify artifacts independently by origin, validity, role, scale, sensitivity, lifecycle and use. Do not let “official” or “synthetic” serve as a validity claim. **Priority: high.**
5. Acquire and pin official artifacts offline with exact provenance/license records; never silently modify an exact source, and preserve defective examples as explicit negative cases where useful. **Priority: high.**
6. Commit only small reviewable redistributable artifacts, recipes, sentinels and regressions; generate scale data and use content-addressed storage for expensive results. Make Git LFS an exception, not the baseline. **Priority: high.**
7. Apply minimal/full/extension/omission/invalid/boundary partitions and query discriminator sets across the CSAPI graph rather than relying on a few large realistic examples. **Priority: high.**
8. Model stateful scenarios declaratively with stable identities, logical time, ordered faults/checkpoints, explicit effects and deterministic reset. **Priority: high.**
9. Enforce command-test-only profiles with an in-process effect recorder and an invariant that physical adapters/endpoints cannot load. Keep command dispatch disabled in the default public demo. **Priority: high.**
10. Select the synthetic environmental-sensor network as the first public scenario and require a public-only transitive dependency closure, reset/reseed contract and conspicuous synthetic labeling. **Priority: high.**
11. Use exact goldens only for deliberate lexical/artifact contracts; default to typed semantic assertions, and govern every normalization by path, rationale, test and stable ID. **Priority: high.**
12. Prohibit CI snapshot/golden acceptance; require semantic diffs, source/decision rationale, affected trace IDs and qualified independent review. **Priority: high.**
13. Treat generators as versioned tested infrastructure; capture algorithm, toolchain, seed, parameters, clock/ID namespace, output digests and statistics. **Priority: high.**
14. Combine secret-pattern, entropy, PII/sensitivity, archive/binary and public-closure automation with named human sensitivity review; never infer safety from a clean GitHub scan. **Priority: high.**
15. Implement all twelve proofs before scaling the corpus, and carry quantitative/security/named-pair decisions unchanged to IDR-SRV-054 through IDR-SRV-056. **Priority: high.**

## 17. Risks, Constraints, and Open Questions

### 17.1 Risk Register

| Risk/constraint | Consequence | Control |
|---|---|---|
| fixture output copied from server | circular tests bless defects | independent expected facts and four-role separation |
| upstream/schema/example drift | stale or irreproducible evidence | exact pin/digest, offline bundles, drift proposal and evidence staleness |
| unclear external license | unlawful redistribution/demo packaging | per-artifact terms/attribution and block public release until resolved |
| giant realistic scenario | opaque coupling and poor failure diagnosis | micro fixtures plus composable declarative scenarios |
| broad snapshot acceptance | unintended API/security change hidden | exact/semantic policy, semantic diff and independent review |
| blanket normalization | contractual changes disappear | path-specific allowlist and mutation tests |
| generated data differs despite same seed | irreproducible performance/evidence | generator/tool/algorithm/environment pin and output digest/statistics |
| live endpoint dependency | nondeterministic/offline failure | optional named interoperability only |
| secret/PII/controlled data leak | permanent repository/artifact disclosure | prohibition, layered scans, human review and restricted evidence handling |
| security fixture leaks hidden value | redaction test defeats itself | separated restricted oracle and public absence assertions |
| command test reaches real gateway | unsafe physical effect | compile/profile adapter exclusion, non-network recorder and effect invariant |
| demo mistaken for operational truth | reputational/operational misunderstanding | fictional data, synthetic labels and no real identifiers/endpoints |
| Git/LFS corpus growth | clone/CI/review degradation | measured budgets, growth gates, generation and content-addressed storage |
| archives/path traversal/bombs | CI/workspace compromise | format allowlist, containment and bounded extraction |
| stale trace edges | false coverage/evidence claims | closed graph validation and digest-driven evidence invalidation |

### 17.2 Resolved Plan Questions

| Plan question | Resolution |
|---|---|
| sidecar YAML or embedded metadata? | one manifest sidecar per artifact/scenario, constrained YAML plus canonical JSON digest; payload remains standards-correct |
| copy/adapt or reference/generate standards examples? | exact pinned copy when licensed and useful; otherwise acquisition reference/cache; adaptations always separate with transformation |
| exact snapshot or semantic assertion? | exact only for deliberate lexical/artifact contracts; semantic is default |
| committed size threshold? | do not invent a universal number in research; start with small reviewable Git artifacts and enforce measured project budgets below host limits |
| first public scenario? | wholly synthetic fictional environmental sensor network |

### 17.3 Open Implementation Decisions

- Exact repository size/file-count review and failure thresholds require initial corpus and CI measurements.
- The content-addressed store/provider, mirror and retention owner for expensive datasets is an implementation/operations choice.
- Exact OGC artifact redistribution terms must be recorded per acquired file/package before committing or republishing it; this report does not provide legal advice.
- Concrete schema validators, canonical JSON implementation and XML semantic/canonicalization library require implementation proofs and dependency review.
- IDR-SRV-054 must define dataset scales/workloads; IDR-SRV-055 security corpus depth; IDR-SRV-056 named pairs.

None blocks acceptance of the strategic baseline. **[E]**

## 18. Validation Against This Plan's Success Criteria

| Topic Plan Success Criterion | Status | Evidence |
|---|---|---|
| fixture/scenario scope with source anchors and prior traceability | Met | Sections 3–5 |
| taxonomy, metadata, layout, naming, versioning, provenance and sensitivity | Met | Sections 5–7 |
| all named standards/resource/dynamic/security/DDIL/performance/demo/interop needs | Met | Sections 8–12 and required matrix |
| golden, semantic, schema, normalization, generated-data and drift strategies | Met | Sections 12–13 |
| CI, secret/sensitivity, review and reproducibility controls | Met | Section 14 |
| implementation/community lessons incorporated as non-normative | Met | Sections 3.3, 8.5 and 11.4 |
| recommendations decision-usable and bounded to server | Met | Sections 16–17 |
| downstream handoffs explicit | Met | Section 15 |
| references explicit and reproducible | Met | Section 19 |

### Report Completion Checklist

- [x] Topic ID and research-plan linkage match the overall index
- [x] All core questions are covered or explicitly handed to the owning later topic
- [x] Normative, accepted, direct, implementation, analysis and recommendation evidence are distinguished
- [x] Mutable sources include tags/commits or dated retrieval
- [x] Official examples are not treated as automatically valid/normative/licensed
- [x] The required 15-column strategy matrix is present
- [x] Source, scenario, oracle and generated evidence roles are separate
- [x] Provenance, licensing, sensitivity, public-demo and command-safety rules are explicit
- [x] Exact/semantic/schema/partial/property and normalization rules are explicit
- [x] CI, generation, golden review and drift controls are explicit
- [x] Twelve implementation proofs and later-topic handoffs are explicit
- [ ] Plan-owner acceptance and acceptance date recorded

## 19. References

### Governing and Accepted Project Sources

- Glaux Server Overall IDR Research Plan: `Docs/Research/Initial Designs/IDR/glaux-server/IDR Plans/overall-idr-research-plan.md`
- IDR-SRV-053 topic plan: `Docs/Research/Initial Designs/IDR/glaux-server/IDR Plans/idr-srv-053-test-data-fixtures-golden-files-and-scenario-corpus-strategy.md`
- Glaux Server Goal and Definition: `Docs/Plans/glaux-server/glaux-server-goal-and-definition.md`
- IDR-SRV-050 Conformance Harness Strategy: `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-050-conformance-harness-strategy-report.md`
- IDR-SRV-051 Requirement-to-Test Traceability Strategy: `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-051-requirement-to-test-traceability-strategy-report.md`
- IDR-SRV-052 Rust Test-Driven Architecture and Multi-Layer Test Strategy: `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-052-rust-test-driven-architecture-and-multi-layer-test-strategy-report.md`
- Accepted IDR-SRV-001 through IDR-SRV-049 reports: `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`

### Standards and Official Artifact Sources

- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems Part 1, OGC 23-001: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems Part 2, OGC 23-002: https://docs.ogc.org/is/23-002/23-002.html
- Official Connected Systems repository, tag `v1.0.0` at `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`: https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0
- OGC Connected Systems Part 1 schema registry 1.0: https://schemas.opengis.net/ogcapi/connected-systems/part1/1.0/
- OGC Connected Systems Part 2 schema registry 1.0: https://schemas.opengis.net/ogcapi/connected-systems/part2/1.0/
- OGC SensorML 3.0, OGC 23-000: https://docs.ogc.org/is/23-000/23-000.html
- OGC SensorML 3.0 schemas/examples: https://schemas.opengis.net/sensorml/3.0/
- OGC SWE Common 3.0, OGC 24-014: https://docs.ogc.org/is/24-014/24-014.html
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OpenAPI Specification: https://spec.openapis.org/oas/latest.html
- JSON Schema Draft 2020-12: https://json-schema.org/draft/2020-12
- GeoJSON, RFC 7946: https://www.rfc-editor.org/rfc/rfc7946
- HTTP Semantics, RFC 9110: https://www.rfc-editor.org/rfc/rfc9110
- Problem Details for HTTP APIs, RFC 9457: https://www.rfc-editor.org/rfc/rfc9457
- OGC policy directives/schema copyright: https://portal.ogc.org/public_ogc/directives/directives.php
- OGC legal information: https://www.ogc.org/legal/

### Tool, Repository, and Security Sources

- Cargo test: https://doc.rust-lang.org/cargo/commands/cargo-test.html
- cargo-nextest: https://nexte.st/
- insta snapshot documentation: https://insta.rs/docs/
- assert-json-diff: https://docs.rs/assert-json-diff/
- jsonschema crate: https://docs.rs/jsonschema/
- schemars crate: https://docs.rs/schemars/
- proptest: https://docs.rs/proptest/
- SQLx test fixtures: https://docs.rs/sqlx/latest/sqlx/attr.test.html
- GitHub repository limits: https://docs.github.com/en/repositories/creating-and-managing-repositories/repository-limits
- GitHub large-file guidance: https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github
- Git LFS: https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-git-large-file-storage
- GitHub secret scanning: https://docs.github.com/en/code-security/secret-scanning/introduction/about-secret-scanning
- GitHub push protection limitations: https://docs.github.com/en/code-security/secret-scanning/introduction/about-push-protection
- OWASP Secrets Management Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html
- OWASP Logging Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html
- NIST SP 800-53 Revision 5: https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final

### Informative Implementation and Community Sources

- OpenSensorHub: https://github.com/opensensorhub
- Connected Systems Go release baseline: https://github.com/SomethingCreativeStudios/connected-systems-go/tree/244f4dd586da685d4d9b75e43f73001028b5bd0e
- pygeoapi: https://github.com/geopython/pygeoapi
- OS4CSAPI client/testing corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

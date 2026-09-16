# Section 051: Requirement-to-Test Traceability Strategy - Research Report

**Topic ID:** IDR-SRV-051<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-051 Requirement-to-Test Traceability Strategy](../IDR%20Plans/idr-srv-051-requirement-to-test-traceability-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** Requirement sources; canonical IDs and aliases; requirement, test, fixture, evidence, implementation, profile and deviation records; coverage/disposition state; artifact formats and repository layout; source change and impact; CI gates and reports; PR/issue workflow; conformance, TDD, fixture, performance, security and interoperability integration; final synthesis handoffs<br>
**Methodology Used:** Accepted-trace inventory; current primary-source format, assessment and CI review; normalized entity/edge modeling; lifecycle and derived-state analysis; source-change/evidence-freshness analysis; workflow option comparison; implementation/community lesson reconciliation; downstream synthesis<br>
**Research Time:** Approximately 41 hours of AI-assisted execution on September 16, 2026<br>
**Standards Evidence Freeze:** OGC API - Connected Systems Parts 1 and 2 Version 1.0/tagged commit `8e03b236`; OGC API - Features Part 1; SensorML 3.0; SWE Common 3.0; OpenAPI 3.1.2 project baseline; JSON Schema Draft 2020-12; YAML 1.2.2; RFC 9110 and RFC 9457; checked September 16, 2026<br>
**Traceability Evidence Freeze:** Accepted Glaux IDR-SRV-001 through IDR-SRV-050; NIST SP 800-53A Rev. 5 Release 5.2.0 and OSCAL assessment-result patterns; OMG ReqIF overview; SARIF 2.1.0 stable-rule/result concepts; current GitHub Actions artifact, summary and attestation documentation; official sources checked September 16, 2026<br>
**Document Purpose:** Define one maintainable, machine-checkable requirement-to-verification graph without importing every record, implementing tooling, changing source obligations, or claiming conformance/readiness<br>
**Author:** OpenAI Codex<br>
**Date:** September 16, 2026<br>
**Last Updated:** September 16, 2026

---

## Evidence and Decision Legend

- **[N] Normative:** approved external standard or incorporated requirement/artifact.
- **[A] Accepted project baseline:** accepted Glaux report or governing project decision.
- **[D] Direct documentation:** official tool, format or platform documentation.
- **[I] Implementation evidence:** another implementation, client, discussion or project; informative only.
- **[T] Test evidence:** reproducible run/result with stated scope and conditions.
- **[E] Analysis:** reasoned synthesis from identified evidence.
- **[P] Project recommendation:** proposed Glaux decision pending acceptance of this report.
- **[X] Explicit boundary:** excluded claim or later-topic responsibility.

“Traced,” “implemented,” “tested,” “passing,” “covered,” “claimable,” “ready,” and “accepted deviation” are separate states. No percentage, issue label or passing test may silently substitute for another. **[X]**

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Traceability Source and Scope Extraction Methodology
5. Requirement Source Inventory
6. Requirement ID and Namespace Strategy
7. Requirement Record Model
8. Test Case Record Model
9. Fixture and Evidence Record Model
10. Coverage State, Disposition, Exception, and Deviation Model
11. Artifact Format and Repository Layout Findings
12. CI Validation and Generated Report Findings
13. Conformance Harness Integration Findings
14. TDD, Fixture, Performance, Security, and Interoperability Traceability Findings
15. PR Review, GitHub Issue, Workflow, and Final Synthesis Reporting Findings
16. Downstream Topic Handoff Matrix
17. Recommendations
18. Risks, Constraints, and Open Questions
19. Validation Against This Plan's Success Criteria
20. References

---

## 1. Executive Summary

Glaux should maintain one **normalized traceability graph** whose curated source is a constrained, human-authored YAML 1.2.2 subset validated by pinned JSON Schema Draft 2020-12 schemas. Requirements, decisions, profiles, implementation components, verification cases, fixtures, deviations and gates are versioned graph entities; typed edges express `derived-from`, `implements`, `verifies`, `uses-fixture`, `produces-evidence`, `applies-in`, `supersedes` and `gates`. Canonical normalized JSON is the machine/hash representation. Markdown, HTML, CSV, SQLite indexes, GitHub summaries, ReqIF, OSCAL and SARIF are generated views or adapters, never parallel sources of truth. **[P]**

The graph must avoid reciprocal manual maintenance. A test case declares which requirements or explicit risk/regression/exploration purpose it verifies. A fixture declares its provenance and expected facts. An evidence result identifies its test, target and immutable inputs. Reverse links—“tests for requirement,” “requirements affected by fixture,” and coverage totals—are generated. This single-edge rule prevents two lists from disagreeing and makes orphan detection meaningful. **[P]**

Published identifiers are preserved. Exact external requirement URIs remain source identifiers; accepted project IDs such as `CS1-REQ-005`, `V2-03`, `PKG-*`, `INT-*` and `IDR-SRV-*` remain stable aliases or canonical IDs where already collision-free. New internal IDs are opaque, immutable and allocation-ledger controlled. Titles, categories and module names do not belong in IDs because they change. Deleted or superseded IDs are tombstoned and never reused. **[A,P]**

A single “status” field is rejected. Each requirement has separate authority/lifecycle, applicability/disposition, implementation and verification facets. Coverage is a derived view calculated from the resolved profile, current graph and immutable evidence. `not-applicable`, `accepted-deviation`, `deferred`, `blocked`, `unresolved` and `superseded` require distinct semantics. A normative deviation may document risk acceptance but cannot make an unsatisfied conformance requirement pass or claimable. **[P]**

Every source pin, entity and evidence input has a semantic digest. When a source, interpretation, requirement, case, fixture, profile, implementation artifact or tool changes, the impact engine traverses typed dependency edges and marks dependent evidence stale; it never edits historical run results. Source changes enter `needs-review` until a responsible reviewer records equivalence, migration, supersession or re-verification. This supplies defensible change impact without rerunning everything blindly or hiding changed obligations. **[P]**

CI blocks invalid schemas, duplicate IDs/aliases, broken references, illegal graph cycles, missing source pins, unauthorized disposition transitions, expired deviations, orphan non-exploratory tests, missing release-required tests, invalid skips, stale release evidence and unreproducible generated outputs. PR summaries show the trace delta and impact set; release gates evaluate the complete profile graph. GitHub issues and PRs remain workflow references rather than authoritative requirement state because they are mutable and may disappear. **[D,P]**

This strategy integrates directly with the accepted IDR-SRV-050 `ConformanceCaseV1`, `CaseResultV1` and `ConformanceRunV1` semantics. It does not duplicate the harness or prescribe every Rust annotation, fixture file, performance threshold, security assessment or client matrix. Those details pass to IDR-SRV-052 through IDR-SRV-056. **[A,X]**

## 2. Scope and Plan Alignment

### 2.1 Completed Scope

This report completes all six authorized phases:

- inventories normative, AEP, inherited, project, security, profile, verification and risk sources;
- defines canonical/alias namespaces and stable identity rules;
- defines requirement, test, fixture, evidence, component, profile, gate and deviation records;
- separates lifecycle, disposition, implementation, verification, evidence and derived coverage states;
- selects constrained YAML plus JSON Schema and canonical-JSON generation;
- defines repository, CI, PR, issue, reporting, source-delta and evidence-freshness behavior; and
- maps the model to the harness and later verification topics.

### 2.2 Explicit Boundaries

This report does not:

- import every requirement from IDR-SRV-001 through IDR-SRV-050 into a new registry;
- renumber accepted identifiers, reproduce restricted STANAG/AEP content or publish protected locators;
- implement schemas, generators, CI workflows, GitHub automation, databases or dashboards;
- redefine OGC applicability, conformance, AEP obligations or accepted Glaux decisions;
- choose exact Rust test macros, fixture storage, performance budgets, security tools or client versions;
- authorize IDR-SRV-052 or any later topic; or
- treat trace completeness as conformance or implementation readiness.

### 2.3 Research Question Coverage

| Plan theme | Status | Evidence |
|---|---|---|
| source scope and authority | Complete | Sections 3–5 |
| identifier/namespace stability | Complete | Section 6 |
| requirement/test/fixture/evidence records | Complete | Sections 7–9 |
| states, exceptions and deviations | Complete | Section 10 |
| format, repository and generation | Complete | Section 11 |
| CI checks and reports | Complete | Section 12 |
| conformance harness integration | Complete | Section 13 |
| TDD, specialized verification and profiles | Complete | Section 14 |
| PR, issue and synthesis workflow | Complete | Section 15 |
| implementation lessons and downstream ownership | Complete | Sections 3.4 and 16 |

## 3. Evidence Base and Authority Classification

### 3.1 Controlling Project Evidence

| Source group | Accepted contribution to traceability |
|---|---|
| IDR-SRV-001–005 | AEP/STANAG authority boundary, report-local obligations, controlled-source handling and two-layer AEP-to-OGC trace chain |
| IDR-SRV-006–008 | exact 25-class, 233-requirement, five-recommendation, 240-ATS inventory; conditions, prerequisites, class profile and claim gate |
| IDR-SRV-009–014 | discovery, route, query, representation, error, OpenAPI, contract/runtime parity and interpretation records |
| IDR-SRV-014A–014G | regressions showing static claims, status-only tests, mutable demos, tool normalization and client gaps are weak evidence |
| IDR-SRV-015–024 | canonical model, identity, lifecycle, time, provenance, SensorML/SWE, validation and semantic/profile rule IDs |
| IDR-SRV-025–038 | persistence, transaction, ingestion, dynamic-data, streaming, command, feasibility and safety obligations |
| IDR-SRV-039–043 | security, ZTA, policy, audit, DDIL and synchronization requirements and evidence boundaries |
| IDR-SRV-044–049 | implementation stack, modular components, profiles, configuration, observability and continuity inputs |
| IDR-SRV-050 | case/run/result semantics, six outcomes, fixture/target/evidence model and closed-graph claim evaluation |

### 3.2 External Primary Evidence

| Source | Version/state | Use | Limitation |
|---|---|---|---|
| CSAPI Parts 1/2 and tagged artifacts | 1.0, `8e03b236` | exact requirements, recommendations, ATS and artifacts | defects/conditions require accepted interpretation records |
| SensorML/SWE Common | 3.0 | model, schema, mapping and encoding obligations | schema validity is not whole semantic conformance |
| YAML | 1.2.2 | human-authored serialization candidate | aliases, tags, implicit types and key equality require constrained processing |
| JSON Schema | Draft 2020-12 | structural schema and validation vocabulary | semantic graph and authorization rules need separate checks |
| NIST SP 800-53A | Rev. 5, Release 5.2.0 | control-to-assessment objective/evidence pattern | security/privacy assessment model, not generic CSAPI authority |
| OSCAL Assessment Results | current reference reviewed; 1.2.x family | optional security assessment/result interchange | system/control-centric and too heavy as Glaux's universal source |
| OMG ReqIF | public overview/current specification family | future RM-tool interchange option | XML exchange model, not optimal Git-native authoring source |
| SARIF | 2.1.0 plus Errata 01 | stable rule IDs, result fingerprints and static/security finding export | result format, not requirements graph authority |
| GitHub Actions documentation | current 2026-09-16 | artifacts, job summaries, retention and attestations | CI artifacts can expire; provenance does not prove correctness |

ISO/IEC/IEEE 29148 was not available through the workspace or a public authoritative full text during this execution. It is not used to support a normative claim. Its future licensed review can be mapped without changing this source-authority hierarchy. **[X]**

### 3.3 Authority Rules

1. A trace record points to authority; it does not become the authority.
2. Source text is summarized only when handling rules permit; exact source edition, clause/URI and digest remain separately identifiable.
3. Accepted IDR decisions govern Glaux project behavior unless explicitly superseded through governance.
4. Requirements derived from risks, defects or implementation choices are labeled project requirements and cite their derivation.
5. Test and implementation evidence can support or refute a behavior claim but cannot create a standards requirement.
6. Generated reverse links and coverage states are reproducible views, never curated facts.

### 3.4 Implementation and Community Lessons

- OSH and CS-Go demonstrate that rich tests can coexist with fixed or inaccurate conformance declarations; traceability must connect exact capabilities and release evidence.
- pygeoapi/52°North demonstrates how routes, OpenAPI, representations and tests drift when maintained separately.
- SECD shows that status-only smoke tests miss ignored filters and semantic failures; requirement links must identify assertion intent and discriminating fixtures.
- OS4CSAPI work shows that client parser gaps, mutable demos and tool coercion can be misclassified as server findings; evidence provenance, raw capture and disposition history matter.
- Long Markdown matrices were effective research artifacts but are unsuitable as the implementation source of truth; machine schemas and generated views are required.

These findings are non-normative regression and workflow evidence. **[I]**

## 4. Traceability Source and Scope Extraction Methodology

### 4.1 Normalized Graph

The traceability graph uses these core entity types:

- `SourceRecord`: immutable edition/artifact authority and handling metadata;
- `RequirementRecord`: one independently dispositionable obligation, recommendation, profile rule or risk-derived expectation;
- `DecisionRecord`: accepted interpretation, design decision or source-conflict resolution;
- `ProfileRecord`: capability/profile applicability and release selection;
- `ImplementationUnitRecord`: stable logical component, public operation, schema/rule or migration identity;
- `VerificationCaseRecord`: IDR-SRV-050 case metadata or specialized test intent;
- `FixtureRecord`: deterministic inputs, provenance and expected facts;
- `EvidenceRecord`: immutable run/case/tool result and artifact manifest;
- `GateRecord`: CI/release condition over derived graph states; and
- `DeviationRecord`: approved bounded departure, not-applicable rationale or risk acceptance.

Typed edges are authored once in the direction of knowledge ownership. For example, a test owns `verifies` and `uses-fixtures`; a component owns `implements`; an evidence result owns `result-of`. Reverse indexes are generated. **[P]**

### 4.2 Extraction Chain

`source pin → atomic obligation → source identifier/condition → accepted interpretation/decision → profile applicability → implementation owner → verification intent → fixture → immutable result/evidence → gate → generated readiness/claim view`

An obligation is atomic when it can receive one applicability/disposition and can be independently implemented or verified. Lettered sub-obligations that fail independently become child requirements linked to their parent formal requirement; this preserves the official ID without hiding partial coverage. **[P]**

### 4.3 Source Delta and Impact

Every source package records edition, stable locator, retrieved bytes/digest where permitted, access date, authority and handling. A change creates a new source revision; it never mutates old evidence. Dependency traversal produces an impact set across requirements, decisions, tests, fixtures, components, profiles and evidence. A reviewer then records one of:

- `equivalent`: no semantic obligation change; rationale required;
- `changed`: revise/supersede affected records and rerun evidence;
- `added` or `removed`: allocate/tombstone requirements explicitly;
- `unresolved`: withhold affected readiness/claim; or
- `source-unavailable`: retain last pin but mark revalidation blocked.

Digest change is an alert, not proof of semantic change. **[P]**

## 5. Requirement Source Inventory

### 5.1 Source Classes

| Source class | Examples | Authority | Required trace treatment |
|---|---|---|---|
| NATO adoption/mission | STANAG 4789, AEP-4789 Volumes I/II | controlled adoption/operational authority | opaque source pin, permitted clause/page anchor, handling, releasable accepted summary |
| direct OGC normative | CSAPI Parts 1/2 | technical normative | exact edition, class, requirement/recommendation URI, condition, ATS |
| inherited OGC normative | Features/Common and selected dependencies | normative when inherited/selected | exact inherited class/requirement and activation edge |
| representation/encoding | SensorML, SWE Common, schemas | normative/supporting artifact | exact class/schema/rule, wrapper/mapping and profile applicability |
| web/API standards | HTTP, Problem Details, OpenAPI, JSON Schema | normative or project-selected contract | exact edition/clause and owning API behavior |
| accepted project decision | IDR reports/governance | project-controlling | stable decision ID, report/section, acceptance and supersession |
| profile/deployment | eleven profiles, configuration, release policy | project requirement | explicit profile/capability condition and safe disabled behavior |
| security/policy/audit | accepted IDR plus selected NIST/project controls | mixed authority | source-classified controls and separate assessment methods |
| risk/defect/regression | threat, issue, implementation observation | project verification input | derivation, severity/owner and regression/exploratory purpose |
| implementation constraint | MSRV, module boundary, migration/tool policy | project implementation | component/gate owner and release applicability |
| external tool/client | official ETS, Schemathesis, named clients | test/interoperability evidence | exact tool/version/target; never normative unless source says so |

### 5.2 Representative Traceability Matrix

This matrix demonstrates the required record/view fields across source families. It is not the future exhaustive registry. `*` denotes a namespace family and “candidate” denotes IDs allocated only when implementation imports the accepted source. **[P]**

| Requirement ID | Source | Source reference | Requirement summary | Normative/profile/design classification | Implementation area | Verification method | Test IDs | Fixture IDs | Evidence artifacts | Coverage state | Disposition/rationale | Downstream topic handoff | Notes / unresolved issues |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `V2-03` / candidate canonical alias | AEP-4789 Volume II | accepted IDR-SRV-003 controlled anchor | fixes adopted OGC editions/package relationship | adoption/profile | release/profile registry | document pin and generated profile audit | `TEST-PROFILE-*` | `FX-PROFILE-*` | source/package manifest | baselined; test planned | in scope; controlled summary only | 052,057 | do not publish protected text/locator |
| `CS1-REQ-005` | OGC 23-001 | exact `/req/...` URI and Annex mapping | stable Part 1 API obligation | normative | route/domain/representation registry | normative ATS plus supplemental negative/semantic | `TEST-CS1-*` | `FX-CS1-GRAPH-*` | `CaseResultV1`/wire | derived per build | in scope where class applies | 052,053 | preserve exact external URI |
| `CS1-REC-001` | OGC 23-001 | exact `/rec/...` URI | Part 1 recommendation | normative recommendation/advisory | relevant capability | Annex recommendation test | `TEST-CS1-REC-*` | case-specific | warning/advisory result | derived warning/pass | not mandatory unless profile strengthens | 052 | no silent promotion |
| `CS1-NP-*` candidate | OGC 23-001 model/table | clause/table/row | non-numbered incorporated/model constraint | normative-derived atomic child | domain/representation | semantic/schema/graph assertion | `TEST-CS1-NP-*` | canonical/negative | case evidence | import pending | in scope when source obligation applies | 052,053 | never counterfeit `/req` URI |
| `CS2-REQ-017` | OGC 23-002 | exact requirement/ATS URI | ControlStream/Command obligation family | normative/conditional | tasking API/domain/persistence | ATS, lifecycle, negative, safety-profile | `TEST-CS2-*` | `FX-CMD-SIM-*` | case/effect/audit transcript | derived per profile | command-sim only until safe capability enabled | 052,053,055 | exact condition required |
| `CS2-NP-*` candidate | OGC 23-002 model/table | clause/table/row | non-numbered lifecycle/model constraint | normative-derived atomic child | dynamic/tasking model | state-machine/schema assertion | `TEST-CS2-NP-*` | dynamic/command corpus | case evidence | import pending | in scope when resource selected | 052,053 | source summary cannot replace table |
| `OAF1-REQ-*` | OGC API - Features Part 1 | exact inherited class/requirement | inherited API behavior | normative inherited | HTTP/query/GeoJSON | official ETS plus Glaux adapter | `EXT-ETS-OAF1-*` | ETS-compatible | native ETS plus wrapper manifest | derived separate lane | in scope when inherited claim selected | 052 | OAS 3.0/3.1 seam retained |
| `SML3-REQ-*` | SensorML 3.0 | exact class/schema/clause | selected SensorML representation obligation | normative/profile | representation/validation | schema, mapping, semantic round trip | `TEST-SML3-*` | `FX-SML-*` | validation/round-trip record | planned by enabled class | selected applicable subset | 052,053 | tool limitation != target fail |
| `SWE3-REQ-*` | SWE Common 3.0 | exact class/schema/clause | component/encoding obligation | normative/profile | SWE contract/codecs | schema, mapping, codec/property tests | `TEST-SWE3-*` | `FX-SWE-*` | decoded-value evidence | planned by encoding | JSON/Text/Binary separate | 052–054 | binary claim remains gated |
| `GLX-SRV-REQ-*` candidate | accepted IDR decision | report ID, section and acceptance commit | Glaux-specific design/profile behavior | project design/profile | named logical component | unit/integration/black-box as appropriate | `TEST-GLX-*` | scenario-specific | layer result/run | import pending | in scope after registry import/review | 052,053 | ID opaque; category is metadata |
| `GLX-SEC-REQ-*` candidate | accepted security/ZTA/policy IDR | report/decision/control anchor | server security or policy control | project/security | enforcement point/policy/audit | control assessment, negative, integration | `TEST-SEC-*` | synthetic identities/policy | evidence plus optional OSCAL export | planned | in scope by profile | 052,053,055 | deep assessment later |
| `GLX-DDIL-REQ-*` candidate | IDR-SRV-042 | accepted decision anchor | degraded-operation semantic | project/profile | operation/service posture | fault-schedule black-box case | `TEST-DDIL-*` | `FX-DDIL-*` | fault/run transcript | planned | DDIL profiles only | 052–055 | not direct OGC class |
| `GLX-SYNC-REQ-*` candidate | IDR-SRV-043 | accepted decision anchor | duplicate/gap/conflict behavior | project/profile | sync pipeline | isolated two-node scenario | `TEST-SYNC-*` | `FX-SYNC-*` | envelope/state/audit evidence | planned | sync profile | 052–055 | broker/DB replication insufficient |
| `GLX-DEPLOY-REQ-*` candidate | IDR-SRV-046–049 | accepted profile/config/lifecycle anchors | profile safety and reproducibility | project/deployment | composition/config/admin | manifest/static/integration/restore tests | `TEST-DEPLOY-*` | deployment manifests | CI/package/restore evidence | planned | per profile | 052–055 | operational values deferred |
| `GLX-HARNESS-REQ-*` candidate | accepted IDR-SRV-050 | recommendation/proof anchor | harness integrity/claim rule | project/verification | conformance harness | meta-tests and twelve proofs | `TEST-HARNESS-*` | synthetic graph/evidence | self-test package | planned | required before claim use | 052 | harness proves mechanism only |
| `RISK-*` candidate | threat/defect/lesson | issue/report finding pin | regression or exploratory risk | verification-derived | affected component | regression, fuzz, exploratory or manual | `TEST-REG-*` | minimal reproducer | finding/case result | open/mitigated derived | cannot be orphaned; closure rationale | 052–056 | issue URL is reference, not identity |

## 6. Requirement ID and Namespace Strategy

### 6.1 Identity Principles

1. IDs are immutable, case-sensitive ASCII strings and globally unique within the Glaux trace catalog.
2. IDs are opaque enough to survive title, module, owner, severity, profile and directory changes.
3. Exact external IDs/URIs are stored separately as `source_identifier`; they are never rewritten into a counterfeit project URI.
4. Already published accepted IDs remain canonical when unambiguous or become permanent aliases when a global canonical ID is needed.
5. An alias resolves to exactly one canonical ID and is never reassigned.
6. Superseded/deleted IDs become tombstones with replacement/rationale; no reuse.
7. Numeric allocation is monotonic per namespace through a reviewed ledger; gaps are valid.
8. IDs identify records, not versions. Record revision and semantic digest identify content versions.

### 6.2 Namespace Families

| Entity | Recommended form | Rule |
|---|---|---|
| source | `SRC-<AUTHORITY>-<DOCUMENT>-<EDITION>` | edition-specific immutable source record |
| formal CSAPI requirement | preserve `CS1-REQ-NNN` / `CS2-REQ-NNN` | map exact official URI and ATS separately |
| recommendation/ATS | `CS1-REC-NNN`, `CS1-ATS-NNN`, `CS2-ATS-NNN` | distinct entity/edge; no requirement conflation |
| inherited/formal standard | `<STANDARD>-REQ-NNNN` plus exact source URI | allocate only when imported and reviewed |
| Glaux project requirement | `GLX-SRV-REQ-NNNN` | category/owner not encoded in immutable ID |
| decision/interpretation | `GLX-DEC-NNNN` / `GLX-INT-NNNN` | accepted `INT-*` aliases preserved |
| verification case | `TEST-<LANE>-NNNN` | stable metadata ID; test function name may change |
| fixture/scenario | `FX-NNNN` / `SCN-NNNN` | separate stable scenario from concrete fixture version |
| implementation unit | `COMP-NNNN` or stable public operation/rule ID | logical unit, not source path alone |
| gate | `GATE-NNNN` | expression over graph/evidence states |
| deviation | `DEV-NNNN` | approval/review entity, never embedded status text |
| evidence/run | UUID plus content digest | immutable occurrence; not monotonic human ID |

Existing `PKG-*`, `V1-*`, `V2-*`, `INT-*`, `STREAM-*`, `IDR-SRV-*` and similar accepted references enter the alias registry with their source scope. A migration report must resolve collisions before any alias becomes globally searchable. **[A,P]**

### 6.3 Versioning and Supersession

- Editing a summary without changing semantics creates a new record revision with the same ID and digest change classified `editorial`.
- Changing obligation semantics normally supersedes the old requirement with a new ID and a typed `supersedes` edge.
- A new source edition creates new `SourceRecord`; unchanged requirements may be mapped as semantically equivalent after review while retaining project IDs.
- Splitting a compound requirement tombstones or retains the parent as an aggregate and creates child IDs; evidence must identify which granularity it covers.
- Merging requirements retains all old tombstones/aliases and identifies the replacement; historical evidence stays attached to historical IDs.

## 7. Requirement Record Model

### 7.1 Required Curated Fields

| Field | Purpose |
|---|---|
| `schema_version`, `id`, `revision` | parse and identity control |
| `title`, `summary` | releasable human understanding; not authoritative source text |
| `source_id`, `source_identifier`, `source_anchor` | exact authority/edition/clause/URI linkage |
| `authority`, `strength`, `kind` | normative, recommendation, inherited, profile, design, risk-derived, etc. |
| `parent_ids`, `derived_from`, `decision_ids` | decomposition and derivation graph |
| `conditions`, `prerequisites` | exact activation/applicability inputs |
| `profiles`, `capabilities` | candidate applicability; resolved per build/profile |
| `obligation_owner`, `implementation_area_ids` | accountable logical ownership |
| `verification_expectation` | required layers/methods without duplicating tests |
| `disposition`, `disposition_ref` | explicit scope/deviation/supersession entity |
| `handling`, `sensitivity` | controlled-source/output rules |
| `status_review`, `approvals` | baseline/review governance |
| `aliases`, `supersedes`, `notes` | stable history and bounded unresolved context |

Requirement summaries are concise project-authored interpretations. Restricted or copyrighted source text is not copied unless project handling and quotation authority explicitly permit it. **[P]**

### 7.2 Generated Fields

Test IDs, fixture IDs, evidence IDs, latest results, coverage state, claim/readiness state, downstream impact and source freshness are generated reverse views. They must not be hand-edited into requirement records. Cached values, if materialized for performance, carry generator version and graph digest and fail validation when stale. **[P]**

### 7.3 Requirement Granularity

- One formal numbered requirement retains one parent record even when it has several lettered obligations.
- Independently failing sub-obligations receive child IDs with `part-of` edges.
- Incorporated schemas/tables receive child or artifact-rule records only when independently testable/dispositionable.
- Conditions are structured expressions over capability/profile facts, not prose-only notes.
- A broad accepted design recommendation becomes multiple implementation requirements only when implementation planning can assign and verify them independently.

## 8. Test Case Record Model

IDR-SRV-050 `ConformanceCaseV1` supplies the core. Every verification record must include:

- `id`, schema/revision, title, purpose and authority lane;
- `verifies` requirement IDs, or exactly one explicit `risk`, `regression`, `exploratory` or `tool-self-test` purpose;
- verification layer/category, target types, profiles/capabilities and applicability expression;
- fixture/scenario IDs and version constraints;
- setup/action/assertion/cleanup intent with case-owned assertion IDs;
- comparison/normalization and evidence requirements;
- automation state, CI tier/gate role, owner and implementation locator;
- source/tool versions, issue/PR references and supersession; and
- sensitivity, destructive/effect classification and authorization constraints.

### 8.1 Relationship Rules

- A multi-requirement test lists each requirement and assertion mapping; a broad test cannot mark all linked requirements covered unless each has an explicit passing assertion.
- One requirement may map to several layers. Coverage rules declare which layers are required and which are supplemental.
- Negative/regression cases identify the source rule they protect plus risk/finding where relevant.
- Exploratory tests require owner, hypothesis and expiry/review; their findings become requirements/regressions or are closed with rationale.
- Test names and source paths may change; the stable test ID remains metadata. Encoding IDs only in function names is insufficient.
- Disabled/quarantined cases remain in the graph and affect readiness according to applicability; deletion is not a disposition.

## 9. Fixture and Evidence Record Model

### 9.1 `FixtureRecordV1`

Each fixture or scenario record requires:

| Field group | Required content |
|---|---|
| identity | fixture/scenario ID, schema version, revision, semantic/content digest |
| purpose | requirement/risk/test relationships, data category and scenario intent |
| provenance | source, license/handling, generator identity/version/seed, acquisition/creation date |
| applicability | profiles, target types, prerequisite capabilities and incompatible combinations |
| material | repository/artifact locator, exact file hashes, media/schema/profile versions |
| expected facts | stable identities, graph/state/time/result invariants and intentional deviations |
| lifecycle | setup/bootstrap command, namespace/lease, cleanup and final-state proof |
| security | synthetic/operational classification, policy markings, secret prohibition and redaction rules |
| golden relations | exact/normalized/semantic comparison target IDs and approved update workflow |
| governance | owner, review, supersession, related issues and downstream topic |

Fixtures do not manually list all tests that use them; each test owns `uses-fixtures`, and the reverse list is generated. A fixture change invalidates evidence through its digest and dependency edges. **[P]**

### 9.2 Evidence Model

IDR-SRV-050 `CaseResultV1` and `ConformanceRunV1` are the canonical functional-run forms. The trace graph indexes rather than copies them. An `EvidenceRecord` includes:

- immutable evidence ID, manifest/content digest, schema and producer version;
- result namespace/lane: conformance, unit/integration, security, performance, interoperability, manual or operational validation;
- run/case/tool identifiers and native artifact manifest;
- exact requirement/test/fixture/profile/component/build/config/source digests;
- target identity, environment class, start/end times and actor/tool origin;
- terminal result and assertion/finding IDs without flattening child outcomes;
- sensitivity/handling, redaction/truncation facts and retention class;
- storage locator and verification state; and
- superseding evidence links, never mutable “latest” replacement.

The latest usable evidence for a requirement is a computed selection by gate policy, not a pointer stored in the requirement. Historical failures remain queryable after a later pass. **[P]**

### 9.3 Evidence Authority and Freshness

Evidence is current only when every gate-selected dependency matches:

`requirement semantic digest + decision/interpretation digest + test digest + fixture digest + profile/capability digest + implementation/release digest + tool/config digest + target class + permitted time/retention policy`

A gate may declare which tool/runtime changes are non-semantic, but that equivalence decision is itself versioned and reviewed. Expiry, artifact deletion, failed hash verification, missing restricted component or source supersession makes evidence stale/incomplete/invalid; it does not mutate its recorded outcome. **[P]**

### 9.4 External Evidence Adapters

- Official OGC suite output remains native evidence linked through a wrapper manifest.
- JUnit is a presentation/exchange view and cannot carry the whole trace graph.
- SARIF is appropriate for stable static/security rules and findings, with native rule IDs/fingerprints preserved.
- OSCAL Assessment Results may export security-control scope, observations, findings and evidence links when IDR-SRV-055 requires it.
- ReqIF may exchange requirements with external RM tools after an explicit field/loss mapping.
- GitHub artifact attestations may bind release evidence packages to workflows and commits, but provenance does not prove requirement satisfaction.

## 10. Coverage State, Disposition, Exception, and Deviation Model

### 10.1 Orthogonal State Facets

A requirement never has one overloaded status. The graph calculates and displays these facets separately:

| Facet | States | Meaning |
|---|---|---|
| record lifecycle | `identified`, `analyzed`, `baselined`, `needs-review`, `superseded`, `retired` | authority/record review state |
| applicability | `applicable`, `not-applicable`, `conditional-unresolved`, `unknown` | resolved against exact profile/build |
| disposition | `in-scope`, `deferred`, `blocked`, `unresolved`, `accepted-deviation`, `superseded` | project treatment, not evidence outcome |
| implementation | `not-planned`, `planned`, `partial`, `implemented`, `removed`, `unknown` | implementation assertion with component evidence |
| test design | `unplanned`, `planned`, `implemented`, `manual`, `disabled`, `quarantined`, `deprecated` | verification asset state |
| execution | `not-run`, `pass`, `fail`, `skip`, `warning`, `error`, `inconclusive`, plus `flaky` | exact run/case outcome from IDR-SRV-050 |
| evidence | `absent`, `generated`, `valid-current`, `stale`, `incomplete`, `invalid`, `superseded` | artifact integrity/freshness |
| derived coverage | `unmapped`, `planned`, `implemented-unverified`, `verified-current`, `failing`, `stale`, `excepted`, `blocked`, `unresolved`, `not-applicable`, `superseded` | computed consumer view |

Percentages report each denominator and state count. `not-applicable`, `deferred`, `accepted-deviation` and `superseded` are never added to “passed.” **[P]**

### 10.2 Allowed Transition Principles

- `identified → analyzed → baselined`; source/semantic change sends a record to `needs-review`, then back to `baselined` or to `superseded`.
- implementation progresses `not-planned → planned → partial → implemented`; regressions/removal may move backward with rationale and impact.
- test design progresses `unplanned → planned → implemented`; `disabled`, `quarantined` and `deprecated` require owner/reason and do not erase the case.
- evidence is append-only: `generated → valid-current`; later invalidation yields a new validation/disposition fact and the derived view becomes `stale` or `invalid`.
- `not-applicable` and `accepted-deviation` are approval records with scope, rationale and review conditions, not convenient state edits.
- supersession is an explicit edge and tombstone; history remains immutable.

Illegal transitions—such as `identified → verified-current`, `fail → pass` by editing evidence, or `applicable → not-applicable` without a changed profile/decision—fail CI. **[P]**

### 10.3 Not Applicable

A not-applicable record contains exact profile/build scope, the structured condition evaluation, source/applicability basis, reviewer, approval date and re-evaluation trigger. It cannot be used merely because a capability is unfinished. A requirement selected by a claimed class cannot be made not applicable contrary to the source condition. **[N,P]**

### 10.4 Accepted Deviation

`DeviationRecordV1` contains:

- deviation ID, affected requirement/profile/release IDs and exact scope;
- proposed behavior versus required/accepted behavior;
- authority classification and whether a conformance claim is affected;
- rationale, risk, compensating controls and user/interoperability impact;
- owner, approver role/identity, approval evidence and date;
- expiration/review date or event, remediation/exit criteria and issue/plan link;
- verification/evidence required during the deviation; and
- supersession/closure disposition.

Only the Glaux Project Lead or a formally delegated governance role may accept a project deviation. A deviation from a normative OGC requirement may authorize a knowingly nonconforming release decision, but it cannot turn the requirement into a pass or preserve the affected class claim. Security/policy deviations also require the designated security authority. **[P]**

### 10.5 Blocked, Deferred and Unresolved

- `blocked`: an identified external dependency or decision prevents progress; blocker and unblock condition required.
- `deferred`: planned for a named future milestone/profile; owner and target required; remains in full-scope denominator.
- `unresolved`: source meaning, conflict or design decision is not settled; cannot be implemented/claimed as if resolved.

These states remain visible in every readiness report. A date may be omitted when governance has not authorized a schedule; false dates are worse than explicit absence. **[P]**

### 10.6 Final Readiness Rules

For a release/profile requirement to count `verified-current`:

1. record is baselined and source/decision pins are current;
2. applicability resolves true and disposition is in scope;
3. implementation units for every atomic obligation are present in the candidate release;
4. every gate-required verification layer is implemented and has valid current evidence;
5. no applicable required case is failed, errored, inconclusive, invalidly skipped, flaky, disabled or quarantined; and
6. fixture, tool, profile, build and evidence manifests validate.

Project readiness may explicitly accept a bounded deviation according to governance; conformance claimability remains governed by IDR-SRV-008/050 and does not accept a normative deviation as pass. **[A,P]**

## 11. Artifact Format and Repository Layout Findings

### 11.1 Format Comparison

| Format | Benefits | Risks/costs | Role |
|---|---|---|---|
| Markdown tables | readable in review; existing corpus | weak types, escaping, merge and machine-integrity problems | generated reports only |
| JSON | unambiguous types; JSON Schema; canonicalization | verbose for curation; no comments | schema, canonical machine form and release snapshot |
| YAML 1.2.2 | readable/editable; comments; JSON-compatible model possible | implicit typing, aliases/tags, duplicate-key/tool divergence | constrained curated source |
| TOML | clear scalars/configuration | awkward deeply nested graph records and repeated polymorphic edges | reject as primary |
| CSV | universal flat analysis | loses nesting, types and relationships | generated export only |
| SQLite | fast joins/dashboard queries | binary/non-diffable and migration/state authority risk | disposable generated index/cache |
| ReqIF | requirements-tool interchange | XML complexity and field-mapping loss | optional future import/export adapter |
| OSCAL | strong security control/assessment structures | control/system-specific and heavyweight for all domains | optional security adapter |
| SARIF | mature rule/finding exchange | result-oriented, not obligation graph | static/security finding adapter |

### 11.2 Selected YAML Profile

Curated source uses YAML 1.2.2 restricted to the JSON data model:

- only mappings with unique string keys, sequences, strings, booleans, null and JSON-compatible finite numbers;
- no anchors, aliases, merge keys, explicit/custom tags, directives, complex keys, duplicate keys, NaN or infinities;
- IDs, digests, versions, dates, durations and strings resembling booleans/numbers are quoted;
- timestamps use RFC 3339 with timezone and validate as strings;
- parser limits cover bytes, nesting, aliases (zero), scalar length, collection size and document count;
- parse must reject duplicate keys before JSON Schema validation;
- parsed instances validate against pinned Draft 2020-12 schemas with unknown fields rejected by default; and
- generator emits deterministic canonical JSON with defined key ordering/number/string rules for hashing.

YAML comments are reviewer guidance and never semantic evidence. Mapping order is not semantic. **[D,P]**

### 11.3 Recommended Repository Layout

```text
traceability/
  README.md
  schemas/
    source-record.schema.json
    requirement-record.schema.json
    decision-record.schema.json
    profile-record.schema.json
    implementation-unit.schema.json
    verification-case.schema.json
    fixture-record.schema.json
    deviation-record.schema.json
    gate-record.schema.json
  sources/
  requirements/<namespace>/
  decisions/
  profiles/
  components/
  tests/
  fixtures/
  deviations/
  gates/
  aliases.yaml
  allocation-ledgers/
generated/traceability/
  graph.json
  matrix.md
  coverage.json
  coverage.md
  reports/
```

One logical record per file is the default for requirements, decisions and deviations because it minimizes merge conflicts and preserves blame. High-volume mechanically imported formal requirements may use deterministic source packages only if tools support stable record-level diffs and edits. Evidence packages live in CI/release artifact storage; the repository stores schemas, manifests, small approved fixtures and immutable release pointers/digests, not every run payload. **[P]**

### 11.4 Generated Artifacts

- `graph.json`: reference-closed normalized graph and semantic digests;
- `matrix.md`: human review table with plan-required columns;
- `coverage.json/md`: complete counts and denominators by source/class/profile/layer/state;
- `impact.json/md`: changed nodes and transitive dependents;
- `readiness.json/md`: gate outcomes and blocking reasons;
- optional HTML/GitHub Pages: navigable graph/report, never required for CI truth;
- optional CSV/ReqIF/OSCAL/SARIF: explicitly versioned lossy/lossless adapter reports.

Generated files are rebuilt in CI and compared byte-for-byte or semantic-digest-wise. Whether selected views are committed is a later workflow choice; release snapshots are immutable and content-addressed. **[P]**

## 12. CI Validation and Generated Report Findings

### 12.1 Pull-Request Blocking Checks

1. strict YAML parse and JSON Schema validation;
2. global ID and alias uniqueness; allocation-ledger and tombstone rules;
3. all references resolve to the permitted entity type and revision constraints;
4. source pins, anchors, authority, handling and required digests are present;
5. typed-edge cardinality and legal cycle rules pass;
6. profile/capability prerequisite and applicability expressions resolve;
7. no illegal lifecycle/disposition transition or approval bypass;
8. deviation/not-applicable/blocked/deferred records have required owners, rationale and review conditions;
9. every non-exploratory test maps to a requirement/risk/regression/tool purpose; exploratory expiry is valid;
10. every release-required applicable requirement has a verification plan or approved explicit disposition;
11. referenced test/component/fixture/artifact locators exist and their declared digests match where applicable;
12. generated graph/reports reproduce with no uncommitted source change;
13. changed records include the required trace/impact review and CODEOWNER approvals; and
14. public outputs contain no restricted locators/text, secrets or prohibited sensitive data.

### 12.2 Execution and Evidence Checks

- Validate `ConformanceRunV1`, specialized run records and native external manifests.
- Reject unknown test/fixture/requirement/profile/build IDs or digest mismatches.
- Preserve all six IDR-SRV-050 outcomes and invalid skip/flake semantics.
- Calculate evidence freshness without rewriting historical results.
- Fail current release gates for missing, invalid, stale or incomplete required evidence.
- Verify cleanup/effect-safety evidence for destructive, command and external-target cases.
- Ensure official/internal/security/performance/interoperability lanes remain separately labeled.

### 12.3 Cadence

| Cadence | Checks |
|---|---|
| every PR | parse/schema/graph/ID/reference/transition/generator checks; change impact; changed requirement/test coverage; fast evidence subset |
| merge/protected branch | complete current-claim graph, orphan scan, generated report and profile closure |
| nightly/manual | external source health/delta watch, extended tests, stale evidence, expiring deviations/explorations and observational lanes |
| release candidate | full selected profile graph, all required current evidence, reproducibility, source/fixture/tool pins and immutable manifest/attestation |

External URL availability, issue state and future-standard movement produce warnings/review tasks unless an accepted gate explicitly makes them release inputs. Third-party outage never becomes a Glaux conformance fail. **[P]**

### 12.4 Reports

CI generates:

- full traceability matrix;
- coverage by source, conformance class, profile/capability, implementation area and verification layer;
- unplanned/unimplemented/untested/failing/flaky/disabled/quarantined/stale requirements;
- deferred, blocked, unresolved, not-applicable, superseded and deviated lists;
- orphan tests/fixtures/components/evidence and expired exploratory cases;
- source-delta and transitive-impact report;
- deviation/approval/expiry report; and
- exact release readiness/claim gate summary.

GitHub job summaries contain a compact, safe overview and artifact links because step summaries have size/display limits and are presentation artifacts. Complete JSON/Markdown evidence is uploaded separately under explicit retention. Attestation binds provenance when enabled but remains separate from correctness. **[D,P]**

### 12.5 Coverage Mathematics

Every ratio states:

- selected source/profile/build and graph digest;
- numerator state and required verification layers;
- denominator including counts for applicable, conditional-unresolved and each excluded disposition;
- evidence freshness cutoff and gate policy; and
- generated timestamp/tool version.

“95% covered” without these facts is prohibited. A dashboard must permit drilling from any count to exact IDs and blocking reasons. **[P]**

## 13. Conformance Harness Integration Findings

### 13.1 Authority Direction

The trace catalog selects applicable verification cases from source/profile facts. The harness consumes a read-only resolved catalog snapshot and emits immutable results. The trace generator ingests those results to compute evidence and claim views. The harness never writes “passing” into curated requirement files, and the target server's `/conformance` output never selects its own obligations. **[A,P]**

### 13.2 Contract Mapping

| IDR-SRV-050 concept | Traceability integration |
|---|---|
| `ConformanceCaseV1` | verification record or generated executor view; `verifies` edges and exact case digest |
| `CaseResultV1` | immutable evidence child linked to assertion/requirement IDs |
| `ConformanceRunV1` | evidence/run manifest binding build, profile, sources, fixtures and tools |
| six terminal outcomes | preserved without flattening; applicability/skip separately validated |
| `flaky` | orthogonal attempt fact; blocks stable release evidence |
| claim evaluator | consumes resolved trace graph and current evidence; does not change results |
| official/external lanes | native output plus adapter manifest and explicit authority lane |

### 13.3 Assertion-Level Coverage

Case-to-requirement association alone is insufficient for broad scenarios. Each applicable requirement maps to one or more assertion IDs or an explicit manual assessment objective. A case pass covers only assertions executed and passed under the required profile/fixture. Setup failure, early abort or filtered assertion produces incomplete/error evidence for all unexecuted mappings. **[P]**

### 13.4 Claim and Declaration Parity

For each build/profile the graph generates:

- target end-state classes;
- implemented-but-evidence-incomplete classes;
- claimable classes with closed evidence;
- actually declared classes; and
- mismatches/blocking reasons.

CI requires the deployed `/conformance` set to equal the approved declared set and prohibits a class from being declared when the graph is not claimable. Internal project/profile requirements remain visible without being emitted as OGC class URIs. **[A,P]**

## 14. TDD, Fixture, Performance, Security, and Interoperability Traceability Findings

### 14.1 Multi-Layer TDD

Requirements declare required verification methods/layers, not a blanket end-to-end rule. Typical mapping is:

| Requirement character | Minimum useful layers |
|---|---|
| pure parser/domain invariant | unit plus property/negative where relevant |
| transaction/persistence invariant | unit/service plus database-backed concurrency/integration |
| public HTTP/media/link behavior | contract/in-process plus independent black-box |
| normative conformance class | black-box ATS adaptation plus required supplemental/inherited lanes |
| lifecycle/async workflow | state-machine unit plus database/worker and black-box scenario |
| security/policy | decision-unit plus enforcement integration and negative black-box/security suite |
| performance/scalability | functional correctness plus dedicated benchmark/load evidence |
| external-client compatibility | standards-correct server evidence plus named-pair interoperability run |

Source-code tags are optional secondary navigation. Stable component/rule IDs in manifests and test metadata are preferred over scattering requirement comments across every function. IDR-SRV-052 owns the Rust mechanism. **[P]**

### 14.2 Fixtures and Goldens

Tests link stable scenario/fixture IDs with version/digest constraints. Golden files link to the specific assertion and comparison mode. An approved golden update requires a semantic diff, affected requirement list, reviewer and reason; regenerating and approving current server output blindly is prohibited. IDR-SRV-053 owns corpus layout and update workflow. **[P]**

### 14.3 Performance and Streaming

Non-functional requirements record metric definition, workload/scenario, dataset, target profile, environment, threshold/budget authority, statistical method, warm-up/duration and regression policy. Evidence references raw result and analysis tool versions. A functional conformance pass and a performance pass are separate edges/gates. IDR-SRV-054 owns actual thresholds and workload design. **[P]**

### 14.4 Security, Authorization, and Command Safety

Security records retain control/threat/source authority, enforcement points, profile, assessment objective/method and evidence. OSCAL export is optional for selected control assessments; Glaux IDs remain canonical. Command-safety requirements map to simulated-effect and prohibited-physical-effect evidence. Accepted deviations require security authority as well as project governance. IDR-SRV-055 owns depth and tooling. **[P]**

### 14.5 Interoperability

An interoperability record binds exact server build/profile, external client/server name/version/commit, scenario/fixture, transport/configuration, expected semantic result, result and raw evidence. “Works with client X” never covers all versions or proves conformance. Named-pair findings may create regressions without weakening canonical server behavior. IDR-SRV-056 owns the matrix. **[P]**

### 14.6 Profile and Full-Scope Preservation

The all-class Glaux target remains visible independently from a release/profile selection. A profile overlay resolves each requirement to applicable, not applicable with proof, or conditional-unresolved. Disabled behavior has requirements too: absent routes/claims, safe denial, configuration refusal and no effect. Deferred end-state requirements remain in full-scope reports even when excluded from an early release denominator. **[A,P]**

## 15. PR Review, GitHub Issue, Workflow, and Final Synthesis Reporting Findings

### 15.1 Pull Request Contract

Every behavior-changing PR should include or generate:

- changed requirements/decisions/components/tests/fixtures/profiles/deviations;
- direct and transitive impact set;
- required evidence reruns and resulting state;
- new/changed source anchors and semantic-change classification;
- generated trace/report diff;
- profile/conformance/declaration impact; and
- unresolved/blocking/deviation review needs.

Docs-only/editorial changes may prove zero semantic impact through digest classification. “No tests needed” requires a traceable reason, not an empty checklist. **[P]**

### 15.2 Review Ownership

CODEOWNERS or equivalent review rules should route:

- normative/import changes to standards/profile maintainers;
- security/policy/control changes to security/policy owners;
- deviation/not-applicable approvals to authorized governance roles;
- fixture/golden changes to domain plus test-data owners; and
- generator/schema/gate changes to traceability/harness owners.

The author cannot self-approve a new deviation that suppresses their failing requirement. **[P]**

### 15.3 GitHub Issues and Pull Requests

Issues and PRs are referenced by immutable repository/number plus observed state/time, but they are not the requirement ID or disposition authority. Closing an issue does not implement or verify a requirement; merging a PR does not automatically make evidence current. Automation may post generated summaries/links but must not infer approval from labels alone. **[D,P]**

### 15.4 Final IDR and Implementation Readiness

IDR-SRV-057 should consume a signed/content-addressed trace snapshot containing:

- accepted source and decision inventory;
- complete Category I requirement/test/fixture/evidence/gate relationships;
- full-scope versus first-release/profile coverage;
- unresolved, blocked, deferred and deviated items;
- current conformance claimability and official/external evidence lanes;
- source/tool/profile/fixture/build pins; and
- limitations preventing stronger readiness claims.

The final synthesis may summarize but cannot improve underlying states. A missing or stale edge remains missing or stale in the final report. **[P]**

## 16. Downstream Topic Handoff Matrix

| Consumer | Receives from IDR-SRV-051 | Must decide or prove later | Must not reinterpret |
|---|---|---|---|
| IDR-SRV-052, TDD and implementation workflow | stable IDs, assertion-level mappings, component allocation, PR impact set, state/evidence contract | Rust attributes/manifests, test organization, local/CI developer workflow and enforcement rollout | a test name or merged PR is not current evidence |
| IDR-SRV-053, fixtures and test data | fixture/scenario/golden identities, provenance/digest fields, restricted-data projection and approval rules | corpus layout, generators, licenses, update workflow and representative datasets | regenerated output is not automatically an approved golden |
| IDR-SRV-054, performance and scalability | non-functional requirement links, workload/environment/result fields and separate gate semantics | metrics, thresholds, budgets, statistical method, environments and regression policy | functional coverage cannot substitute for performance evidence |
| IDR-SRV-055, security validation | control/threat/source links, assessment-objective/evidence structure, sensitive-evidence handling and deviation approval | tool suite, control depth, abuse cases, policy matrix and accreditation handoff | a documented deviation cannot produce a normative conformance pass |
| IDR-SRV-056, interoperability | named-pair run identity, exact version/configuration/fixture pins and result/evidence relationships | client/server matrix, scenarios, cadence and gating pairs | one named-pair result cannot imply universal interoperability or conformance |
| IDR-SRV-057, final synthesis | content-addressed trace snapshot, full-scope/profile views, current/stale evidence, gaps, deviations and claims | final readiness aggregation, unresolved-decision disposition and implementation sequencing | summaries cannot improve the underlying graph state |
| IDR-SRV-050 harness implementation | verification-case/evidence/gate contracts, assertion mapping and selection-input boundary | concrete import/export adapter and runner integration | the system under test cannot choose or grade its own conformance cases |
| server implementation and architecture records | requirement/decision/component graph, source pins, impact queries and alias migration | actual module ownership and code-level annotations | traceability records do not amend normative sources |
| release and operations governance | release-gate closure, immutable evidence pointers, retention classification and deviation expiry | artifact store, retention/signing service, release roles and operational values | artifact attestation proves provenance, not correctness or security |
| OGC liaison and standards maintenance | exact source anchors, conformance-class and ATS links, draft/official authority classification | upstream-change monitoring and official-suite adoption | draft or community material cannot be relabeled normative |

### 16.1 Required Implementation Proofs

Before the strategy is treated as operational, implementation should demonstrate these twelve bounded proofs:

1. **Schema and normalization round trip:** valid constrained YAML parses, validates and produces byte-stable canonical JSON; forbidden YAML constructs and duplicate keys fail before schema validation.
2. **Existing-ID migration:** representative `CS1-*`, `CS2-*`, `V2-*`, `PKG-*`, `INT-*`, `STREAM-*` and `IDR-SRV-*` references import without renumbering; alias collisions and reuse fail.
3. **Source-delta impact:** changing one pinned source fragment deterministically identifies direct and transitive requirements, tests, fixtures, components, gates and evidence that require review.
4. **Assertion-level coverage:** a multi-assertion case can pass one requirement and fail another without case-level overstatement.
5. **State-transition enforcement:** illegal lifecycle, approval, execution and evidence transitions fail; authorized transitions preserve actor, time and rationale.
6. **Deviation and applicability safety:** missing authority, expiry, compensating control, profile scope or source anchor prevents an accepted deviation/not-applicable disposition and cannot yield a conformance pass.
7. **Evidence freshness:** source, requirement, test, fixture, tool, profile or build digest changes make only the correct dependent evidence stale while retaining the immutable prior run.
8. **Restricted-source projection:** an authorized internal record and a public projection retain stable correlation without leaking controlled text or locators.
9. **Orphan and blind-spot detection:** the validator reports unallocated requirements, unlinked tests, unused fixtures, missing verification plans and contradictory generated links.
10. **Profile-claim closure:** a claim passes only when its selected applicable requirement graph has allowed current terminal evidence and no unresolved blockers.
11. **Deterministic generation:** two clean runs from the same input create identical Markdown, CSV, JSON and index outputs with no repository mutation outside generated paths.
12. **Report drill-down:** every aggregate number resolves to the exact denominator, requirement records, test assertions, fixture versions and evidence artifacts that produced it.

These are implementation gates, not claims that tooling already exists. **[P]**

### 16.2 Adoption Sequence

Adopt incrementally: freeze schemas and identifier policy; import sources and accepted decisions; migrate existing IDs/aliases; add test and fixture manifests; connect the IDR-SRV-050 runner; enable advisory reports; repair gaps; then make narrowly defined checks blocking. Bulk manual completion should not be required before developers receive useful impact and gap reports. **[P]**

## 17. Recommendations

1. Make a normalized typed graph the conceptual model, with constrained YAML as the reviewable source and canonical JSON as the validation/hash boundary.
2. Validate against JSON Schema Draft 2020-12 after a strict parse that rejects duplicate keys, aliases, anchors, merge keys, tags, directives, complex keys and non-JSON scalar values.
3. Preserve exact external and already-accepted project identifiers; issue opaque immutable Glaux IDs only where no stable identifier exists, and retain permanent one-to-one aliases and tombstones.
4. Record the source artifact, version/digest, anchor, authority and classification separately from the requirement summary so editorial text cannot silently replace an obligation.
5. Author each relationship once in its ownership direction and generate reverse navigation; do not maintain duplicate test/fixture/evidence lists on requirement records.
6. Keep lifecycle, applicability, disposition, implementation, test-design, execution, evidence and derived-coverage states orthogonal.
7. Treat `not applicable`, `deferred`, `blocked`, `unresolved`, `superseded` and `accepted deviation` as distinct states with explicit authority, rationale, scope and review/expiry rules.
8. Never convert a normative deviation or waived failure into a conformance pass; show it separately in readiness and claim reports.
9. Make evidence immutable and content-addressed, and calculate freshness from pinned source, requirement, test, fixture, tool, profile, configuration and build semantics.
10. Use assertion-level mappings for multi-requirement cases and require each applicable release requirement to have an approved verification method and planned case before claim gating.
11. Generate full, profile/release, change-impact, gap, stale-evidence, deviation, orphan and readiness views from the same graph; every percentage must publish its denominator and dispositions.
12. Let the independent IDR-SRV-050 harness consume a trace snapshot and return immutable results; do not permit the service under test or GitHub labels to self-select, grade or approve coverage.
13. Put a generated trace delta, transitive impact set, evidence-rerun list and claim/profile impact in behavior-changing PRs, with role-based review for normative, security, fixture and deviation changes.
14. Keep ReqIF, OSCAL, SARIF, Markdown, CSV, SQLite, GitHub summaries and dashboards as generated adapters or views, never parallel sources of truth.
15. Introduce blocking CI only after the import and advisory-report period, but block immediately on parse/schema failures, duplicate/reused IDs, dangling or illegal edges, unauthorized dispositions and false release claims.
16. Carry the twelve proofs and explicit topic handoffs into IDR-SRV-052 through IDR-SRV-057 without prematurely selecting exact Rust mechanisms, datasets, thresholds, security tools or interoperability partners here.

Together these recommendations yield defensible traceability without turning the repository into a second requirements-management product. **[D,P]**

## 18. Risks, Constraints, and Open Questions

| Risk or constraint | Consequence | Mitigation/owner |
|---|---|---|
| source anchors drift across standards revisions | links remain syntactically valid but semantically wrong | pin source digest/version, store fragment digest, classify changes; standards maintenance |
| YAML ambiguity or parser variance | different graph/hashes across tools | constrained JSON-compatible subset, duplicate-key rejection, canonical JSON and proof 1; IDR-SRV-052 |
| imported identifier collision or renumbering | broken history and ambiguous evidence | namespace registry, permanent aliases/tombstones and migration proof; traceability owner |
| duplicate authored relationships | contradictory forward/reverse coverage | one ownership direction, generated reverse indexes and CI equality checks |
| coverage percentages hide exclusions | false readiness or conformance confidence | publish denominator, applicability/disposition counts and drill-down |
| stale evidence is treated as current | changed behavior ships on obsolete proof | semantic digest dependency graph, impact calculation and release closure gate |
| deviations become permanent waivers | normative or safety debt disappears from review | authorized approver, scope, rationale, compensating control, expiry/review and separate reporting |
| manual upkeep overwhelms development | trace graph decays or work bypasses it | generated links/views, PR deltas, staged enforcement and developer-local validation |
| restricted AEP/STANAG detail leaks | disclosure or licensing breach | minimal public projection, opaque source locator/digest, sensitivity labels and access-controlled evidence |
| GitHub state is mistaken for engineering state | issue closure or merge falsely signals verification | store only external references; require graph state and immutable evidence transitions |
| tool or adapter export loses semantics | competing or incomplete truth sets | canonical graph owns semantics; round-trip/export tests and declared lossiness |
| early implementation choices pre-empt later studies | weak fixture, performance, security or interoperability design | preserve explicit IDR-SRV-052–056 ownership boundaries |

Open implementation questions remain deliberately bounded:

- Which Rust crates and code annotations best implement the manifest/schema contract? IDR-SRV-052.
- What corpus partitioning, generator and licensing rules implement fixture identities? IDR-SRV-053.
- Which workloads, environments, thresholds and statistical rules are release gates? IDR-SRV-054.
- Which security tools, control sources and evidence-retention classifications are required? IDR-SRV-055 and security governance.
- Which exact client/server versions and scenarios gate interoperability? IDR-SRV-056.
- Which artifact store, signing identity, retention periods and release roles operate the evidence system? Release/operations governance.
- Whether ReqIF, OSCAL or SARIF export is required by a stakeholder remains an integration decision; none is required to begin the native graph.

None prevents IDR-SRV-052 research. **[P]**

## 19. Validation Against This Plan's Success Criteria

| Success criterion | Result | Evidence |
|---|---|---|
| requirement sources and scope identified | Met | Sections 2–5 |
| requirement, test, fixture, evidence and disposition models documented | Met | Sections 7–10 |
| ID namespace, stability, supersession, exception and deviation rules documented | Met | Sections 6 and 10 |
| coverage, test and evidence states and transitions documented | Met | Sections 9–10 |
| artifact format, layout, reports and CI checks documented | Met | Sections 11–12 |
| harness, TDD, fixture, performance, security, interoperability and synthesis integration documented | Met | Sections 13–16 |
| PR review, GitHub issue linkage and reporting documented | Met | Sections 12 and 15 |
| implementation and community lessons incorporated non-normatively | Met | Sections 3.4, 11–15 |
| recommendations decision-usable and server-bounded | Met | Sections 2, 17 and 18 |
| downstream handoffs explicit | Met | Section 16 |
| references explicit and reproducible | Met | Section 20 |

All planned phases and required report content are complete. Acceptance remains a project-lead action. **[P]**

## 20. References

### 20.1 Controlling Standards and Official Artifacts

- OGC API - Connected Systems Part 1: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems Part 2: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems publication repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems v1.0.0 API artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common 3.0: https://docs.ogc.org/is/24-014/24-014.html
- OGC schema registry: https://schemas.opengis.net/
- OpenAPI Specification: https://spec.openapis.org/oas/latest.html
- RFC 9110, HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9457, Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457

### 20.2 Traceability, Validation, Assessment, and Interchange

- YAML 1.2.2 specification: https://yaml.org/spec/1.2.2/
- JSON Schema Draft 2020-12 specification: https://json-schema.org/draft/2020-12
- JSON Canonicalization Scheme, RFC 8785: https://www.rfc-editor.org/rfc/rfc8785
- Date and time format, RFC 3339: https://www.rfc-editor.org/rfc/rfc3339
- Uniform Resource Identifier syntax, RFC 3986: https://www.rfc-editor.org/rfc/rfc3986
- NIST SP 800-53A Revision 5, Assessing Security and Privacy Controls: https://csrc.nist.gov/pubs/sp/800/53/a/r5/final
- NIST OSCAL Assessment Results model: https://pages.nist.gov/OSCAL/learn/tutorials/assessment/assessment-results/
- OMG Requirements Interchange Format (ReqIF): https://www.omg.org/reqif/
- OASIS SARIF 2.1.0 Errata 01: https://docs.oasis-open.org/sarif/sarif/v2.1.0/errata01/os/sarif-v2.1.0-errata01-os-complete.html
- ISO/IEC/IEEE 29148 overview/catalog record: https://www.iso.org/standard/72089.html

The ISO catalog record was used only to identify the standard's scope; the paywalled full text was not available and no normative claim in this report depends on it. **[E]**

### 20.3 CI, Evidence, and Conformance Tooling

- GitHub Actions artifacts: https://docs.github.com/actions/using-workflows/storing-workflow-data-as-artifacts
- GitHub Actions job summaries: https://docs.github.com/actions/using-workflows/workflow-commands-for-github-actions#adding-a-job-summary
- GitHub artifact attestations: https://docs.github.com/actions/security-for-github-actions/using-artifact-attestations
- Cargo test: https://doc.rust-lang.org/cargo/commands/cargo-test.html
- cargo-nextest: https://nexte.st/
- OGC Validator and test-suite inventory: https://cite.ogc.org/teamengine/
- OGC compliance program: https://www.ogc.org/compliance/
- TEAM Engine repository: https://github.com/opengeospatial/teamengine

### 20.4 Accepted Project Evidence

- Overall IDR Research Plan: [overall-idr-research-plan.md](../IDR%20Plans/overall-idr-research-plan.md)
- IDR-SRV-051 Research Plan: [idr-srv-051-requirement-to-test-traceability-strategy.md](../IDR%20Plans/idr-srv-051-requirement-to-test-traceability-strategy.md)
- Glaux Server Goal and Definition: [glaux-server-goal-and-definition.md](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md)
- Accepted IDR-SRV-001 through IDR-SRV-050 reports: [IDR Reports](./)
- IDR-SRV-006 Part 1 baseline: [idr-srv-006-csapi-part-1-requirement-baseline-report.md](idr-srv-006-csapi-part-1-requirement-baseline-report.md)
- IDR-SRV-007 Part 2 baseline: [idr-srv-007-csapi-part-2-requirement-baseline-report.md](idr-srv-007-csapi-part-2-requirement-baseline-report.md)
- IDR-SRV-008 conformance mapping: [idr-srv-008-conformance-class-and-requirement-mapping-report.md](idr-srv-008-conformance-class-and-requirement-mapping-report.md)
- IDR-SRV-014 OpenAPI strategy: [idr-srv-014-openapi-description-and-api-documentation-strategy-report.md](idr-srv-014-openapi-description-and-api-documentation-strategy-report.md)
- IDR-SRV-031 write and ingestion model: [idr-srv-031-server-write-and-ingestion-model-report.md](idr-srv-031-server-write-and-ingestion-model-report.md)
- IDR-SRV-035 streaming and draft Part 3 strategy: [idr-srv-035-streaming-and-event-publication-strategy-report.md](idr-srv-035-streaming-and-event-publication-strategy-report.md)
- IDR-SRV-049 lifecycle strategy: [idr-srv-049-migration-upgrade-backup-and-restore-strategy-report.md](idr-srv-049-migration-upgrade-backup-and-restore-strategy-report.md)
- IDR-SRV-050 conformance harness strategy: [idr-srv-050-conformance-harness-strategy-report.md](idr-srv-050-conformance-harness-strategy-report.md)
- Research Report Template: [research-report-template.md](../../../../../Governance/research-report-template.md)

### 20.5 Non-Normative Implementation and Community Evidence

- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client/testing corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OGC CSAPI developer site: https://csapi.developer.ogc.org/

### 20.6 Evidence Limits

Official specifications and documentation were checked September 16, 2026. Mutable web pages, schemas, tool versions and upstream branches must be pinned and revalidated during implementation. No Glaux traceability tooling or server implementation existed to execute, so the state machine, layouts, gates and twelve proofs are design recommendations rather than observed tool behavior. ReqIF, OSCAL and SARIF were assessed as possible bounded adapters, not adopted requirements. No controlled AEP/STANAG text is reproduced, and public records must retain only authorized metadata/projections. **[E,P]**

---

## Report Completion Checklist

- [x] All 20 required sections are present
- [x] Required 14-column traceability matrix is complete
- [x] Stable ID, alias, supersession and tombstone rules are explicit
- [x] Requirement, test, fixture, evidence, disposition and deviation models are complete
- [x] Coverage, lifecycle, execution and evidence states and transitions are explicit
- [x] Artifact format, layout, reports and CI validation are explicit
- [x] Harness, TDD, fixture, performance, security, interoperability and synthesis handoffs are explicit
- [x] PR review and GitHub linkage rules are explicit
- [x] Twelve implementation proofs and downstream handoffs are explicit
- [x] All 11 success criteria validate as Met
- [ ] Accepted by Glaux Project Lead

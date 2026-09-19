# Section 062: CS-GO Engineering Practices and Development History Study - Research Plan

**Topic ID:** IDR-SRV-062<br>
**Status:** Research complete; report in review; project-lead acceptance pending<br>
**Last Updated:** September 18, 2026<br>
**Estimated Research Time:** Not yet calibrated; four bounded phases below, with additional research iterations only if needed for the stated coverage.<br>
**Actual Research Time:** Approximately 30 minutes elapsed through research/report review, 00:03–00:33 UTC September 19, 2026 (September 18 EDT), in one AI-assisted iteration; not a human-hours estimate and excluding subsequent publication. No upstream tests executed.<br>
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-062-cs-go-engineering-practices-and-development-history-study-report.md`

---

## Usage Instructions

Use the existing [Research Plan Template](../../../../../Governance/research-plan-template.md) and, for the later report, the [Research Report Template](../../../../../Governance/research-report-template.md), preserving their section order. The pinned OS4CSAPI exemplars inform question-led investigation, concrete examples and practical recommendations; their client-specific scope, metrics and estimates are not Glaux requirements.

The project lead's initial September 18, 2026 `proceed` authorized this plan and its minimal registration in the [overall index](overall-idr-research-plan.md), published in `409afdd2ac946b6e9a6455c22fe709a39c59d7e4`. The next `proceed` accepted the plan for execution and authorized the research/report iteration, now complete with the [report in review](../IDR%20Reports/idr-srv-062-cs-go-engineering-practices-and-development-history-study-report.md). Report acceptance, the synthesis addendum and discussion of planning changes follow separately. No special acceptance phrase is required. Research completion does not accept the report, publish implementation issues or change the Goal, Guide, Roadmap or final synthesis.

---

## 1. Research Objective

Determine what Glaux can learn from **how CS-GO was designed, developed, tested and maintained**, beyond the implementation and standards-behavior findings already captured in IDR-SRV-014B and later research. Examine engineering decisions, their evolution, the author's documented reasoning, test craftsmanship, tools, comments, review practices, and choices about what to include or leave out.

Produce an evidence-backed engineering case study with concrete lessons for a well-written, standards-faithful Rust reference server. Explain successful practices and useful simplifications as well as limitations. Each material lesson must identify whether the current Glaux plans already cover it, a bounded change is worth discussing, it is not applicable, or evidence is insufficient.

### Why This Topic Order

The original 67-topic IDR and supplements 058–061 are complete and accepted. [IDR-SRV-014B](../IDR%20Reports/idr-srv-014b-connected-systems-go-csapi-server-implementation-study-report.md) already studied architecture, persistence, API behavior, tests, documentation and audit-derived corrections. Later reports examined selected feature paths. They do not establish a comprehensive account of development practice or test-first sequencing.

The [Goal v1.7](../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md), [Guide v1.0](../../../../../Plans/glaux-server/glaux-server-implementation-guide.md) and [Roadmap v1.1](../../../../../Plans/glaux-server/glaux-server-roadmap.md) are the comparison baseline. The project lead requested this follow-up before publication of the complete implementation issue set. Its purpose is to improve the existing design and issue-sized tasks where justified, without reopening accepted research wholesale.

### Critical Constraints

- Study the author's upstream repository, `SomethingCreativeStudios/connected-systems-go`. Distinguish its engineering from contributed findings and documentation in the OS4CSAPI audit fork; attribute authorship and contributions accurately.
- CS-GO is informative implementation evidence. Its behavior, comments, tests and omissions do not replace the standards or change Glaux's approved scope. Preserve the selected draft experiments, filtering scope and Part 5 deferral.
- Separate observed code/history, the author's stated rationale, analyst inference and proposed Glaux action. An absent public artifact is not proof that the author never used a practice privately.
- A substantial test suite, test-related commit message or combined implementation/test commit does not establish test-driven development (TDD). Seek evidence of a failing test before the implementation; squashed or unpublished work may leave sequencing unknowable.
- Inventory the public engineering surfaces broadly, then investigate representative decisions deeply. Do not promise knowledge of every private decision, inspect every inherited dependency, or expand into a new whole-server conformance audit.
- No implicit software installation on the project lead's company laptop. Any execution must use an already available, permitted, isolated environment. Do not contact the maintainer, post upstream findings, operate external services or reuse source/fixtures without the applicable authorization and licensing checks.

---

## 2. Research Questions

### Core Questions

1. **Q1 — Architecture and evolution:** How is CS-GO organized, how did important boundaries and mechanisms evolve, and what reasons or tradeoffs are supported by the record?
2. **Q2 — Development process and tools:** What can commits, PRs, discussions, executable commands and release records tell us about implementation, review, debugging, automation and maintenance?
3. **Q3 — Testing and TDD:** How are tests and fixtures designed, what behaviors do they genuinely establish, and what evidence supports or limits a claim of test-first development?
4. **Q4 — Comments, documentation and coding guidance:** How does the repository preserve design knowledge and guide contributors or coding assistants, and how well do those instructions agree with the code?
5. **Q5 — Scope choices and omissions:** What was included, simplified, deferred, delegated or left unresolved, and which choices have an evidenced rationale rather than an inferred one?
6. **Q6 — Transfer to Glaux:** Which practices should reinforce or improve the existing Rust design, implementation workflow and issue-sized tasks, with what adaptation and verification?

### Detailed Questions

**Architecture and scope (Q1, Q5)**

- Trace startup/composition, handlers, domain models, representations, validation and repositories. Examine where reuse is effective, where resource-specific behavior belongs, and how relationships, errors and transactions cross those boundaries.
- Examine the engineering of querying/pagination, spatial and temporal handling, observation storage, command/status/results and publication. Compare selected earlier and current implementations; do not repeat every endpoint's normative analysis.
- Investigate documented alternatives, deliberate simplifications, deployment assumptions and experimental boundaries. Distinguish missing implementation, unsupported behavior, external responsibility and unknown intent. Identify complexity the design avoids as well as complexity it introduces.

**Development process and tools (Q2)**

- Follow selected changes from motivation through code, tests, review/discussion, resolution and release. Distinguish author, contributor, reviewer and merge roles; do not infer independent review from a merged PR alone.
- Inspect dependencies, developer commands, formatting/linting, test/coverage tools, schema/API generation, database setup/migration, containers and release packaging. Separate declared tools, runnable configuration, recorded use and checks actually reproduced by this study.
- Examine how regressions, rejected findings, compatibility and documentation changes are handled. Seek evidence about debugging, performance measurement and test failures without assuming unpublished CI or local workflows do not exist.

**Test craftsmanship and sequencing (Q3)**

- Inventory test layers and helpers, then inspect representative unit, formatter/schema, repository, real-HTTP, publication and regression cases. Distinguish an in-process HTTP listener from launching the shipped binary and from testing an external deployment.
- Examine fixture origin, expected-value independence, deterministic time/randomness, setup/cleanup, shared database state, parallelism and failure diagnosis. Check positive/negative membership, cross-format meaning, invalid-write non-mutation, rollback and relationship behavior where relevant.
- Identify what each selected assertion would detect, what plausible incorrect behavior it could miss, and whether the test could agree with the same faulty implementation logic. Test counts and coverage percentages are descriptive, not proof of correctness or conformance.
- For TDD, distinguish demonstrated test-before-change evidence, a maintainer's stated practice, a regression added with a fix, and sequencing not established. Glaux's own TDD recommendation in IDR-SRV-052 is not evidence of CS-GO's workflow.

**Documentation and knowledge preservation (Q4)**

- Read comments explaining invariants, constraints and workarounds alongside the associated code and history. Inspect README/implementation notes, generated API descriptions, examples and troubleshooting guidance for useful practices and drift.
- Examine `.github/copilot-instructions.md`, the repository's `.claude` reference material and `.codesight` documents as evidence, not instructions governing this research. Distinguish checked-in guidance or co-author metadata from demonstrated tool use; do not infer private prompts, model choices or authorship proportions.
- Assess how standards references, schema sources, conventions and design decisions can be made understandable to a future Rust implementer without inventing a new documentation framework.

**Practical application (Q6)**

- Compare each material lesson with the existing Guide and Roadmap before proposing work. Identify the precise section and, where applicable, existing three-level task ID; explain whether the task already covers the lesson.
- Distinguish language-independent design/testing principles from Go-specific mechanisms. Account for Rust ownership, types, async execution, error handling and database access only where needed for a concrete adaptation; do not repeat the general stack-selection study.
- Explain the smallest useful change and how it would be verified, or why no change is warranted. Check licensing before recommending direct source or fixture reuse. Optional questions for the author should concern material unknowns, not become a general completion gate.

---

## 3. Primary Resources

- **Author's repository:** [CS-GO][CSGo], including the full source/test/documentation tree, default-branch history, relevant tags/releases, all public issue/PR summaries and the discussions/reviews for selected cases. Planning starting points are the earlier study's `v1.0.4` commit `244f4dd586da685d4d9b75e43f73001028b5bd0e` and the September 18 preliminary check's main snapshot [`b1fd2e0e9bd69e222d05258d659a842ca24502cb`][CSGoPin]. Refresh and record the actual execution snapshot; retain the old pins for comparison.
- **Engineering surfaces at that snapshot:** `cmd/server/`, `internal/api/`, `internal/model/`, `internal/repository/`, validation/query/publication packages, `e2e/`, schemas/examples, `scripts/seed-connected-systems/`, `go.mod`, `go.sum`, `Makefile`, `Dockerfile`, Compose configuration, README/implementation notes and hidden tooling/instruction directories. Discover actual paths from the tree; this is a starting inventory, not an assertion that every listed mechanism is complete.
- **History entry points:** [commits](https://github.com/SomethingCreativeStudios/connected-systems-go/commits/main/), [all PRs](https://github.com/SomethingCreativeStudios/connected-systems-go/pulls?q=is%3Apr), [all issues](https://github.com/SomethingCreativeStudios/connected-systems-go/issues?q=is%3Aissue), [releases](https://github.com/SomethingCreativeStudios/connected-systems-go/releases), and available workflow/check records. Candidate cases from the preliminary assessment include [PR 15](https://github.com/SomethingCreativeStudios/connected-systems-go/pull/15), [PR 16](https://github.com/SomethingCreativeStudios/connected-systems-go/pull/16), [spatial correction `ddc9b84`](https://github.com/SomethingCreativeStudios/connected-systems-go/commit/ddc9b84b2b859731b7e6c16c79f7785036ce2dad), [Part 3 refinement `913c112`](https://github.com/SomethingCreativeStudios/connected-systems-go/commit/913c112f07a425d33fe8508e52ce1cf162e9afb1), and the current-snapshot storage/validation/seeder change. Recheck them before making findings; add early architecture/refactoring and release cases so the sample is not limited to recent fixes.
- **Contribution context:** [OS4CSAPI audit fork](https://github.com/OS4CSAPI/connected-systems-go), limited to evidence that explains a selected upstream change or contribution. Pin any inspected material separately; do not attribute the fork's research process to the upstream author.
- **Controlling standards where a lesson depends on correctness:** [CSAPI Part 1](https://docs.ogc.org/is/23-001/23-001.html), [Part 2](https://docs.ogc.org/is/23-002/23-002.html), [SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html), [SWE Common 3.0](https://docs.ogc.org/is/24-014/24-014.html), and the [existing standards-history register](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md). Consult/date-check only entries implicated by selected cases and follow their official resolution links; refresh material changes during research. Do not put ordinary CS-GO engineering history in this standards register.
- **Tool behavior when needed:** official [Go testing](https://pkg.go.dev/testing), [Testcontainers for Go](https://golang.testcontainers.org/), [Testify](https://github.com/stretchr/testify), [GORM](https://gorm.io/docs/), [Cargo testing](https://doc.rust-lang.org/cargo/guide/tests.html) and [Rust test organization](https://doc.rust-lang.org/book/ch11-03-test-organization.html). Use the version actually implicated by a finding and consult other official dependency documentation only when necessary to explain it.

---

## 4. Supporting Resources

- [IDR-SRV-014B report](../IDR%20Reports/idr-srv-014b-connected-systems-go-csapi-server-implementation-study-report.md), especially §§4, 8, 10–14 and its execution limitations; its [plan](idr-srv-014b-connected-systems-go-csapi-server-implementation-study.md) defines the original coverage. Build an explicit already-covered/newly-examined comparison.
- [014E client smoke-test findings](../IDR%20Reports/idr-srv-014e-os4csapi-client-smoke-test-findings-study-report.md), [014G community lessons](../IDR%20Reports/idr-srv-014g-os4csapi-discussions-lessons-learned-study-report.md) and [014H draft Part 3 study](../IDR%20Reports/idr-srv-014h-draft-csapi-part-3-publish-subscribe-and-implementation-study-report.md): contribution attribution, adjudication and experimental context.
- [044 Rust strategy](../IDR%20Reports/idr-srv-044-rust-implementation-language-and-framework-strategy-report.md), [045 modularization](../IDR%20Reports/idr-srv-045-service-architecture-and-modularization-strategy-report.md), [049 migration/recovery](../IDR%20Reports/idr-srv-049-migration-upgrade-backup-and-restore-strategy-report.md), [050 conformance harness](../IDR%20Reports/idr-srv-050-conformance-harness-strategy-report.md), [051 test traceability](../IDR%20Reports/idr-srv-051-requirement-to-test-traceability-strategy-report.md), [052 Rust TDD](../IDR%20Reports/idr-srv-052-rust-test-driven-architecture-and-multi-layer-test-strategy-report.md) and [053 fixtures](../IDR%20Reports/idr-srv-053-test-data-fixtures-golden-files-and-scenario-corpus-strategy-report.md): existing recommendations to reconcile, not automatic additional obligations.
- Reports [054 performance](../IDR%20Reports/idr-srv-054-performance-load-stress-and-streaming-test-strategy-report.md), [055 security tests](../IDR%20Reports/idr-srv-055-security-authorization-and-command-control-test-strategy-report.md), [056 interoperability](../IDR%20Reports/idr-srv-056-interoperability-test-matrix-for-external-csapi-clients-report.md), and supplements [058](../IDR%20Reports/idr-srv-058-draft-csapi-part-4-sampling-features-study-report.md), [059](../IDR%20Reports/idr-srv-059-enhanced-csapi-querying-and-spatial-observation-retrieval-study-report.md), [060](../IDR%20Reports/idr-srv-060-csapi-part-5-protobuf-first-implementation-study-report.md), [061](../IDR%20Reports/idr-srv-061-provenance-and-related-metadata-interoperability-study-report.md): consult relevant CS-GO passages and unresolved limitations, without re-executing those studies.
- [Final synthesis and existing addenda](../IDR%20Reports/final-idr-research-report.md), Goal v1.7, Guide v1.0 (particularly architecture, verification, tests, development and interpretation sections) and Roadmap v1.1 (capability/subtask outline and issue workflow): downstream comparison targets, unchanged by the study itself.

---

## 5. Research Methodology

### Phase 1: Establish coverage and select cases

**Objective:** Identify what is new to this study and prevent selective or duplicated investigation.

**Tasks:**

1. Review the earlier study and relevant later passages; map Q1–Q6 to existing answers, dated findings and remaining questions.
2. Pin upstream identity, default branch, release baseline and retrieval date. Inventory the full repository tree, including hidden tooling, tests and examples. Screen the available default-branch commit history, releases and public issue/PR inventories with pagination; record totals, access limits and branches included rather than silently using only a first page.
3. Group the engineering surfaces by responsibility. Select representative histories covering architecture/refactoring, a successful feature, a defect/regression, querying/persistence, experimental scope and build/documentation/release practice. Cases may overlap; justify their selection and list areas receiving only inventory-level review. Follow other branches only when relevant to a selected case.

**Expected Output:** A compact coverage inventory and case list within the developing report, with reproducible source pins. No separate tracking platform or research topic is needed.

### Phase 2: Trace engineering decisions and development practice

**Objective:** Answer Q1, Q2, Q4 and Q5 through concrete source/history walkthroughs.

**Tasks:**

1. For each case, read relevant before/after code, comments, tests, commit/PR descriptions and available discussions/reviews. Record motivation, change, verification, disposition and supported rationale; identify what cannot be reconstructed.
2. Trace selected resource paths across boundaries and examine the actual build/test/generation/release commands and contributor/assistant guidance. Check guidance against current code, and distinguish configured automation from execution records.
3. Attribute contributed audit findings separately from upstream implementation. Examine beneficial choices and simplifications alongside defects, rejected proposals and unimplemented work. Consult the standards/register only where needed to avoid learning incorrect behavior.

**Expected Output:** Evidence-backed case narratives and findings, with explicit rationale and public-history limitations.

### Phase 3: Examine tests and assess Rust transfer

**Objective:** Answer Q3 and develop practical Q6 recommendations.

**Tasks:**

1. Inspect representative test bodies and their helpers/fixtures at each identified layer. Explain the setup, input, independently expected result, assertion, cleanup and class of failure detected. Include a successful workflow, rejection/non-mutation, filtered inclusion/exclusion and a regression-history example where available.
2. Investigate TDD sequencing using the selected histories and any explicit maintainer account. State exactly what is demonstrated, merely stated, consistent with the evidence, or not established. Do not turn missing history into a positive or negative verdict about the author.
3. Check local tool/dependency availability without installing anything. If a permitted isolated environment exists, run selected upstream tests that resolve a material uncertainty; record exact revision, command, dependencies, result and exclusions. Read commands first for side effects. Do not run release/deployment targets, external writes or load tests. If execution is unavailable, distinguish source inspection from verified runtime behavior and identify the remaining proof needed; do not manufacture pass counts, coverage or benchmarks.
4. Compare lessons with existing Guide sections and Roadmap leaf tasks. Describe a Rust adaptation and observable verification only where useful. Recommend already covered/no change, a bounded improvement, not applicable, or unresolved; explain incremental complexity without inventing hours or implementation commitments.

**Expected Output:** Concrete test-pattern analysis, a qualified TDD answer and a small, actionable planning-impact comparison.

### Phase 4: Synthesis

**Objective:** Produce one readable, decision-usable report in the existing template.

**Tasks:**

1. Answer Q1–Q6, reconcile any changed conclusions with dated earlier evidence, and retain explicit unknowns. Keep source-backed findings, inference and recommendations distinguishable.
2. Present the most useful lessons first, supported by case details and source anchors. Include optional maintainer questions only when public evidence leaves a consequential gap; no outreach is part of this study.
3. Check coverage, references, attribution, success criteria and proposed Guide/Roadmap touchpoints. Prepare the acceptance and separate synthesis-addendum handoff without editing those downstream artifacts.

**Expected Output:** One research report for project-lead review. If the research requires more than one iteration, keep that report explicitly in progress, record completed phases and resume only on the next `proceed`; do not treat partial coverage as completion.

---

## 6. Success Criteria

This topic research is complete when:

- [x] Q1–Q6 have evidence-backed answers or explicit limitations with their decision consequences.
- [x] Repository/history coverage, source pins, sampling rationale and unexamined areas are visible; author and audit-fork contributions are distinguished.
- [x] Representative cases explain engineering choices and their evolution, including useful practices/simplifications as well as limitations.
- [x] Test examples explain meaningful assertions, fixture/isolation behavior, expected-value independence and the failures they detect; static inspection and actual execution are separate.
- [x] TDD is assessed using sequencing evidence or a qualified statement of uncertainty, not inferred from test presence or Glaux's own recommendations.
- [x] Tooling, comments, documentation and coding-assistant guidance are compared with executable behavior and available history; private practice is not invented.
- [x] Omissions are classified by evidence, and any consequential standards-history checks are authority-qualified and reproducible.
- [x] Each material lesson has a justified Glaux disposition and applicable existing Guide/three-level Roadmap task references; no-change is an acceptable outcome.
- [x] The report follows the report template, validates these criteria and identifies optional author questions and any remaining implementation proof without creating new project scope.

Completion does not require a maintainer interview, access to private workflow records, every historical diff to be read, a whole-product certification run or all upstream tests to pass. Such limits must narrow the relevant conclusion rather than disappear from the report.

---

## 7. Deliverable

**Deliverable Name:** CS-GO Engineering Practices and Development History Study - Research Report<br>
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-062-cs-go-engineering-practices-and-development-history-study-report.md`

Use the existing report template, including its executive summary, question coverage, evidence base, findings, decision analysis, recommendations, implementation implications, risks/unknowns, success-criteria validation and handoff. Put the engineering inventory, case histories and test examples within the corresponding sections or optional appendices.

For each material lesson capture **CS-GO evidence → supported rationale or explicit inference → Glaux applicability → existing coverage or proposed bounded change → verification**. Keep the plain-language summary independently readable. Preserve IDR-SRV-014B as the earlier dated assessment and explain updates by source revision; do not overwrite it.

The report supplies evidence for a later addendum to the existing final synthesis and discussion of Guide/Roadmap changes. It is not a new implementation guide, requirements document, test framework or mandate to copy the Go implementation.

---

## 8. Dependencies

### Must Complete Before Starting

**Internal project prerequisites (completion gates):**

- The original IDR, including 014B and the relevant supporting reports, and supplements 058–061 are complete and accepted in the controlling overall plan. Goal v1.7, Guide v1.0 and Roadmap v1.1 provide the current planning baseline. No prerequisite exception is proposed.
- Publish this plan and its matching index entry before the next `proceed` authorizes execution. The preceding conversational assessment is a source-selection aid, not a substitute for this study.

**External evidence prerequisites:**

- Access to the author's public source, tests and history for the selected cases. For inaccessible sources, record identity, attempted access, affected questions and resulting limits; do not infer contents.
- A permitted local Go/database/container environment is needed only for the tests actually executed, not to claim source-level observations. Missing tools require an honest execution limitation, not implicit installation.
- Private records and an author response are not general completion gates. Further contact or environment changes require separate direction.

### Blocks (What This Topic Unlocks)

- A separately authorized accepted-findings addendum to the final synthesis, followed by discussion of specific Guide/Roadmap implications.
- Resumption of complete implementation-issue publication after that discussion and any agreed planning updates. Preserve the user's complete-issue-set-before-coding and one-ready-issue-per-iteration workflow.
- No invalidation of the original research, earlier supplements or approved scope. No server implementation or GitHub issue creation occurs during this study.

---

## 9. Research Status Checklist

- [x] Phase 1 complete
- [x] Phase 2 complete
- [x] Phase 3 complete
- [x] Phase 4 synthesis complete
- [x] Deliverable draft complete
- [x] Deliverable reviewed (source/plan-alignment checks; project-lead acceptance remains pending)
- [ ] Deliverable accepted

**Actual Research Time:** Approximately 30 minutes elapsed through research/report review, September 18, 2026 EDT; source analysis only, not human-hours and excluding subsequent publication.<br>
**Completion Date:** September 18, 2026 (research/report preparation; acceptance pending).

The report inventories 68 default-branch commits, 11 ordinary issues, five PRs, three releases and 76 test files at `b1fd2e0e9bd69e222d05258d659a842ca24502cb`. It distinguishes selected deep review from inventory coverage, does not establish test-first sequencing, and records the unavailable local runtime prerequisites without installing anything. Most lessons reinforce existing planning; two small harness/documentation clarifications are proposed for later discussion. The official Part 3 recheck found no material register change. No downstream planning artifact or implementation issue was changed.

---

## 10. Notes and Open Questions

- The project lead knows the developer and regards CS-GO as a valuable engineering exemplar. This motivates a careful learning-oriented study; it is not evidence for a particular method, design rationale or conformance result.
- Preliminary observations identify promising source paths, not settled report conclusions. Recheck the execution baseline and retain historical comparisons without silently replacing older evidence.
- Co-committed or squashed tests may not reveal writing order. A maintainer account could clarify intended practice while still not proving every change followed it.
- A smaller server or different deployment assumption may justify a choice that Glaux should adapt differently. Do not equate every difference with a defect or a new Glaux requirement.
- Stop when the defined questions and cases are covered with usable lessons and explicit unknowns. Broader investigation requires a demonstrated decision need, not a claim that every possible lesson has been exhausted.

---

## References

- [Research Planning Approach](../../../../../Governance/research-planning-approach.md), [Initial Planning Guidance](../../../../../Governance/initial-planning-guidance.md), [Research Plan Template](../../../../../Governance/research-plan-template.md), [Research Report Template](../../../../../Governance/research-report-template.md) and [Overall Research Report Template](../../../../../Governance/overall-research-report-template.md).
- [Controlling overall IDR plan](overall-idr-research-plan.md), with this topic registered separately from the accepted original IDR and earlier supplements.
- [Pinned OS4CSAPI research-plan exemplar corpus](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans), particularly plans 01, 15 and 38 for direct-source walkthroughs, fixture analysis and useful synthesis.
- Specific primary sources and accepted research inputs are listed in Sections 3–4. Project-lead discussions establish research intent and sequencing, not facts about the developer's private working methods.

[CSGo]: https://github.com/SomethingCreativeStudios/connected-systems-go
[CSGoPin]: https://github.com/SomethingCreativeStudios/connected-systems-go/tree/b1fd2e0e9bd69e222d05258d659a842ca24502cb

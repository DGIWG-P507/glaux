# Section 014H: Draft CSAPI Part 3 Publish/Subscribe and Implementation Study - Research Plan

**Topic ID:** IDR-SRV-014H<br>
**Status:** Complete<br>
**Last Updated:** August 31, 2026<br>
**Estimated Research Time:** 15-19 hours<br>
**Actual Research Time:** Approximately 5 hours of AI-assisted execution<br>
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-014h-draft-csapi-part-3-publish-subscribe-and-implementation-study-report.md`

---

## Usage Instructions

Before executing this plan, review the full exemplar corpus and align the future report to the proven standards for content nature, type, level, and detail:

- Full research-plans corpus (audited snapshot `754411897173c2ec4debaa9bcf4ed9e0f8a9e230`):
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans
- Early exemplar (blueprint-first depth):
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans/01-pr114-blueprint-analysis.md
- Mid-stream exemplar (inventory and sourcing rigor):
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans/15-fixture-sourcing-organization.md
- End-state synthesis exemplar:
  - https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans/38-testing-playbook-synthesis.md

The commit-pinned snapshot controls exemplar structure and style. The future report must follow the audience, authority, source-limitation, and acceptance rules in the Research Report Template.

---

## 1. Research Objective

This topic research must establish an early, evidence-qualified baseline for deciding whether **draft OGC API - Connected Systems - Part 3: Publish/Subscribe** is mature and valuable enough to become an explicit Glaux Server implementation target, and what design constraints must be carried into the intervening resource-model, schema, persistence, ingestion, security, and test topics before `IDR-SRV-035` makes the final streaming architecture and implementation-profile decision.

The research must answer:

- What the exact pinned Part 3 working draft defines for Resource Events, Batch Resource Events, Resource Data Messages, content negotiation, discovery, protocol bindings, conformance, and Abstract Test Suite coverage.
- Which statements are draft requirements, incomplete stubs, unresolved proposals, informative examples, or dependencies on another draft or approved specification.
- What Connected Systems Go (CS-Go) and the OpenSensorHub `osh-addons` Part 3 work actually implement, how each maps to the pinned draft, where they differ from one another, and which behaviors are implementation choices rather than draft obligations.
- Which Part 3 concerns can materially affect Glaux decisions before `IDR-SRV-035`, including identifiers, canonical URLs, resource lifecycle events, transaction boundaries, message schemas, content negotiation, publisher restrictions, authorization, batching, durability, ordering, replay, deployment, and verification.
- Whether the current evidence supports carrying Part 3 forward as an experimental implementation candidate, monitoring it without implementation commitment, or deferring it; and what questions must remain open for `IDR-SRV-035`.

The output must be a draft-Part-3 and implementation-study baseline with reproducible source pins, an authority/status ledger, draft requirement and completeness matrices, CS-Go and OSH implementation maps, a cross-implementation divergence matrix, interoperability and security implications, downstream handoffs, and an explicit adoption-readiness recommendation. It must not select the final Glaux streaming architecture, broker, Rust library, or production conformance claim.

### Why This Topic Order

This topic follows the accepted CSAPI behavior, implementation, client, interoperability, and community-evidence topics `IDR-SRV-006` through `IDR-SRV-014G`. It is inserted before `IDR-SRV-015` because Part 3 has changed from remote future-alignment evidence into an active implementation candidate supported by substantial CS-Go code and an OSH draft pull request.

Researching the candidate now allows later topics to account for event identity, canonical resource references, lifecycle triggers, serialization, transaction/outbox boundaries, policy enforcement, and test fixtures without prematurely adopting a mutable protocol binding. The final architecture decision remains in `IDR-SRV-035`, after `IDR-SRV-015` through `IDR-SRV-034` establish the Glaux resource, temporal, event, schema, persistence, transaction, ingestion, and dynamic-data semantics that a publish/subscribe interface must expose.

### Critical Constraints

- OGC API - Connected Systems Parts 1 and 2 are approved standards and remain controlling for their scope. The Part 3 `part3-working-draft` branch is mutable development evidence, not an approved Glaux obligation.
- Begin from the planning snapshot at official commit `c95c1d6003359d0883c4dc759d7a148ab115fdb1`, but refresh the branch, issue, pull-request, and dependency state at research execution and record any delta before relying on it.
- The planning snapshot identifies three message-type classes but explicitly leaves the MQTT clause unwired, the Abstract Test Suite absent, and other material items incomplete. Do not fill missing requirements from implementation behavior or analyst preference.
- Treat the Part 2 AsyncAPI 2.6 artifacts, draft OGC API Publish-Subscribe Workflow material, CloudEvents, MQTT, and AsyncAPI according to their own exact version and authority. Do not assume that mention or packaging makes an artifact normative for Part 3.
- Treat CS-Go and OSH code, tests, configuration, documentation, issues, and pull requests as non-normative implementation evidence. Record exact release, commit, branch, pull-request, build, and test status.
- Separate the Part 3 protocol-agnostic message model from any MQTT-specific topic hierarchy, QoS, retain, session, broker-policy, or authorization choice.
- Do not claim Part 3 conformance while the researched draft lacks a complete protocol binding and ATS. The report may recommend conditions for a future experimental compatibility profile.
- Do not copy an implementation's topic layout or event envelope as the Glaux design merely because it runs. Compare independent implementations and trace every adopted candidate behavior to its evidence class.
- Do not finalize durable event storage, outbox/inbox design, ordering, replay, backpressure, broker selection, or transport selection here. Identify constraints and hand those decisions to `IDR-SRV-025` through `IDR-SRV-035` and later security/deployment/testing topics.
- Keep the research bounded to Glaux Server behavior and server-side contracts. Publisher, simulator, web, mobile, and operator behavior is included only where it changes the server boundary.

---

## 2. Research Questions

### Core Questions

1. What is the exact authority, maturity, completeness, and dependency status of the current draft CSAPI Part 3 material?
2. What publish/subscribe message classes, semantics, discovery mechanisms, bindings, requirements, and conformance expectations does the pinned draft define, and what remains missing or unresolved?
3. What do CS-Go and the OSH Part 3 branch implement, how faithfully does each map to the draft, and where do their externally observable contracts diverge?
4. Which Part 3 and implementation findings must influence Glaux topics before the final streaming strategy is selected in `IDR-SRV-035`?
5. Should Glaux carry Part 3 forward as an experimental implementation candidate, monitor it without commitment, or defer it, and under what safeguards and decision gates?

### Detailed Questions

#### Authority, Version, and Dependency Baseline

- What commit, document number, status marker, branch, generated artifact, and retrieval date define the research snapshot?
- What changed between the earlier `a1f1f03b71f5f645486b23ec8b5fae1f9ba334bc` snapshot and the research snapshot?
- Which Part 3 clauses depend on approved Parts 1 and 2, draft OGC API Publish-Subscribe Workflow material, CloudEvents, MQTT, AsyncAPI, SWE Common, SensorML, or other sources?
- Which dependencies are approved, draft, informative, missing, version-unspecified, or unresolved?
- Which official issues and pull requests materially affect the snapshot, and what are their current state, target branch, merge commit, publication inclusion, and authority class?
- Does the branch contain an AsyncAPI document, schema package, examples, generated HTML/PDF, conformance declarations, or ATS, and are those artifacts mutually consistent?

#### Message and Event Model

- What distinguishes Resource Events, Batch Resource Events, and Resource Data Messages in trigger, envelope, payload, authority, direction, and intended use?
- Which CSAPI resource types and operations are covered, and which mappings remain TODO, ambiguous, or implementation-defined?
- What CloudEvents version, mode, media type, context attributes, uniqueness rule, source, subject, type vocabulary, time behavior, data content type, data payload, parent identifier, and retransmission semantics are specified?
- How are create, update, delete, enable/disable, status, result, system-event, observation, command, and command-status behaviors represented or omitted?
- What aggregation window, grouping key, count, time range, partial-window, shutdown, and high-volume semantics apply to Batch Resource Events?
- Which Resource Data Messages may be published server-to-client or client-to-server, and what registration, update, validation, and delete semantics are defined or absent?
- How are native CSAPI JSON, SensorML, SWE Common JSON/Text/Binary, and other encodings selected and identified?

#### Protocol Binding, Topics, Discovery, and Conformance

- What does the draft actually define for MQTT 3.1.1 and MQTT 5, and what remains an unwired stub?
- Are topic namespace, canonical resource path, collection path, message-class suffix, content-type suffix, wildcard, hierarchy, and node/server prefix rules specified or left to implementations?
- What publish and subscribe directions are allowed per message/resource class, and how are client publication attempts on server-controlled event channels rejected?
- What, if anything, is specified for QoS, retained messages, sessions, reconnect, correlation data, MQTT 5 properties, flow control, duplicate delivery, ordering, and backpressure?
- How are supported event types, channels, message formats, broker locations, security schemes, and operations discovered?
- Is AsyncAPI required, recommended, proposed, or merely one candidate, and which AsyncAPI version fits the draft and implementation evidence?
- Which requirements classes, conformance classes, and tests exist, and which conformance claims are currently impossible to substantiate?
- What relationship should Part 3 have to SSE, WebSocket, NATS, Kafka, AMQP, DDS, or other transports, without treating candidates as adopted bindings?

#### CS-Go Implementation Evidence

- What Part 3 behavior exists at the pinned CS-Go commit, and which commits or pull requests introduced and revised it?
- How are MQTT connectivity, feature flags, publication, subscription, ingestion, HTTP-triggered events, batching, CloudEvents, topic generation/parsing, AsyncAPI, and shutdown handled?
- Which resources and operations are supported, partially supported, rejected, or undocumented?
- What durability, error handling, retry, echo suppression, QoS, retain, reconnect, authorization, publisher restriction, and broker-policy behavior exists?
- What tests exercise the implementation, what can be reproduced locally, and what external broker or tool dependencies limit the evidence?
- Which behaviors does CS-Go explicitly identify as provisional or non-conformant because the draft is incomplete?

#### OpenSensorHub Implementation Evidence

- What Part 3 behavior exists in `opensensorhub/osh-addons` PR #194 at the pinned head, and what remains absent from the merged default branch or released artifacts?
- How are node/endpoint prefixes, nested resource paths, wildcards, content-type topic suffixes, Resource Data Messages, Resource Events, batch observation events, publisher restrictions, permissions, and OSH event-bus integration handled?
- Which resources, directions, encodings, event types, wildcard patterns, and batch cases are supported?
- What tests exist, what can be reproduced, and what build/runtime dependencies or branch state constrain confidence?
- Does OSH expose AsyncAPI or another discovery artifact for this work?
- Which behaviors are inherited from earlier OSH MQTT support rather than introduced for the current Part 3 draft?

#### Cross-Implementation and Interoperability Analysis

- Where do CS-Go and OSH agree on message semantics, event shape, resource direction, publisher authority, content negotiation, and batching?
- Where do their topic namespaces, hierarchy, suffixes, identifiers, type tokens, payloads, discovery mechanisms, security enforcement, timing, and failure behavior differ?
- Which differences follow an explicit draft choice, an older draft, an incomplete clause, a host architecture constraint, or an undocumented implementation decision?
- Can either implementation consume the other's messages without adapters? If not, what is the minimal reproducible incompatibility?
- Which implementation patterns are safe candidates for Glaux, which should be avoided, and which require a later prototype or standards decision?
- What cross-server fixtures and interoperability scenarios should be reserved for `IDR-SRV-053`, `IDR-SRV-054`, `IDR-SRV-055`, and `IDR-SRV-056`?

#### Reliability, Security, DDIL, and Operational Implications

- What delivery assumptions are implied by the draft and observed implementations, and where are durability, replay, loss, duplication, or ordering undefined?
- How should an HTTP mutation and its published event remain transactionally consistent, and which questions belong to `IDR-SRV-029` and `IDR-SRV-035`?
- What authorization must be enforced by the API process, MQTT service, broker ACL, or shared policy decision point?
- How can resource-level access control, policy/releasability, concealed existence, and command authority propagate to subscriptions and publications?
- What event identity, correlation, causation, provenance, audit, and observability fields are needed without overloading the draft envelope?
- What reconnect, retained-state, snapshot, replay, backfill, batching, bandwidth, and freshness questions matter for DDIL-informed operation?
- What security risks arise from wildcard subscriptions, topic guessing, cross-tenant leakage, unauthorized publishing, broker compromise, oversized messages, retained sensitive data, or ambiguous public origins?

#### Glaux Adoption Readiness and Downstream Handoffs

- Which draft concepts are stable enough to shape an internal Glaux event model without committing to one external topic binding?
- Which choices must remain feature-gated, version-pinned, explicitly experimental, disabled by default, or isolated behind an adapter?
- What exact profile/version identifier, capability declaration, documentation warning, and migration rule would an experimental implementation need?
- What evidence threshold must be met before Glaux can claim draft compatibility or later approved Part 3 conformance?
- Which findings must be handed to `IDR-SRV-015`, `IDR-SRV-016`, `IDR-SRV-018`, `IDR-SRV-020`, `IDR-SRV-023`, `IDR-SRV-025`, `IDR-SRV-029`, `IDR-SRV-031` through `IDR-SRV-035`, `IDR-SRV-039` through `IDR-SRV-043`, and verification/deployment topics?
- What must `IDR-SRV-035` decide rather than this topic?
- What upstream issues warrant monitoring or an evidence-backed SWG contribution, and what can Glaux decide locally without implying a standards change?

---

## 3. Primary Resources

The future research report must analyze these sources directly and refresh mutable state at execution.

### Project and Governance Sources

- Glaux Server Overall IDR Research Plan: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Creation Prompt: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-creation-prompt.md
- Research Plan Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md
- Shared OGC API - Connected Systems upstream-history register: `Docs/Research/Initial Designs/IDR/glaux-server/IDR Evidence/ogc-connected-systems-upstream-history-register.md`

### Controlling and Draft Standards Sources

- OGC API - Connected Systems - Part 1: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: https://docs.ogc.org/is/23-002/23-002.html
- Official Connected Systems repository: https://github.com/opengeospatial/ogcapi-connected-systems
- Part 3 planning snapshot at commit `c95c1d6003359d0883c4dc759d7a148ab115fdb1`: https://github.com/opengeospatial/ogcapi-connected-systems/tree/c95c1d6003359d0883c4dc759d7a148ab115fdb1/api/part3
- Part 3 source entry showing draft wiring and omissions: https://github.com/opengeospatial/ogcapi-connected-systems/blob/c95c1d6003359d0883c4dc759d7a148ab115fdb1/api/part3/standard/23-003r0.adoc
- Packaged Part 2 AsyncAPI artifacts at `v1.0.0`: https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0/api/part2/asyncapi
- OGC API - Publish-Subscribe Workflow - Part 1: Core, draft OGC 25-030: https://docs.ogc.org/DRAFTS/25-030.html
- OGC API - Environmental Data Retrieval - Part 2: Publish-Subscribe Workflow: https://docs.ogc.org/is/23-057r1/23-057r1.html
- OGC Discussion Paper 23-013, Publish-Subscribe workflow in OGC APIs: https://docs.ogc.org/dp/23-013.html
- CloudEvents Specification 1.0.2: https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/spec.md
- CloudEvents JSON Event Format 1.0.2: https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/formats/json-format.md
- MQTT 3.1.1: https://docs.oasis-open.org/mqtt/mqtt/v3.1.1/mqtt-v3.1.1.html
- MQTT 5.0: https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html
- AsyncAPI specifications: https://www.asyncapi.com/docs/reference/specification/latest

### Official Part 3 Maintenance Evidence

Consult the register first, then refresh only the Part 3 cluster and materially linked artifacts:

- Issue #14, legacy AsyncAPI scope: https://github.com/opengeospatial/ogcapi-connected-systems/issues/14
- Issue #68, common OGC Pub/Sub alignment: https://github.com/opengeospatial/ogcapi-connected-systems/issues/68
- Issues #187 through #195, current Part 3 message, binding, encoding, identity, batch, parent, and payload questions: https://github.com/opengeospatial/ogcapi-connected-systems/issues?q=is%3Aissue%20state%3Aopen%20label%3A%22target%3A%20api-part-3%20v1.0.0%22
- PR #198 and merge commit `c95c1d6003359d0883c4dc759d7a148ab115fdb1`, CloudEvent identity update: https://github.com/opengeospatial/ogcapi-connected-systems/pull/198
- Part 3 branch commit history: https://github.com/opengeospatial/ogcapi-connected-systems/commits/part3-working-draft/api/part3

### Implementation Sources

- Connected Systems Go repository, planning snapshot `4a00aa6f7bb1a519ffd56a9f5526d0b8d1c98952`: https://github.com/SomethingCreativeStudios/connected-systems-go/tree/4a00aa6f7bb1a519ffd56a9f5526d0b8d1c98952
- CS-Go experimental Part 3 merge commit `7643bb3`: https://github.com/SomethingCreativeStudios/connected-systems-go/commit/7643bb3
- CS-Go draft-alignment update `913c112`: https://github.com/SomethingCreativeStudios/connected-systems-go/commit/913c112
- CS-Go MQTT/Part 3 documentation at the planning snapshot: https://github.com/SomethingCreativeStudios/connected-systems-go/blob/4a00aa6f7bb1a519ffd56a9f5526d0b8d1c98952/README.md#mqtt-publishsubscribe
- OpenSensorHub `osh-addons`: https://github.com/opensensorhub/osh-addons
- OSH draft PR #194, `[CSAPI] MQTT Module update to CS API Part 3 Draft`: https://github.com/opensensorhub/osh-addons/pull/194
- OSH PR #194 planning head `50774ec7e9c98f6ab8da827171e5c5abb9923a49`: https://github.com/opensensorhub/osh-addons/tree/50774ec7e9c98f6ab8da827171e5c5abb9923a49/services/sensorhub-service-consys-mqtt
- OpenSensorHub core repository: https://github.com/opensensorhub/osh-core

### AEP-4789 and Project-Controlled Sources

Use source material available to the Glaux project team and record exact title, volume, version, date, status, storage location, and handling constraints:

- STANAG 4789
- AEP-4789 Volume I
- AEP-4789 Volume II
- Any approved or draft AEP-4789 streaming, Pub/Sub, event, status, command, DDIL, security, or implementation-profile material relevant to the server

---

## 4. Supporting Resources

- Accepted Glaux reports `IDR-SRV-001` through `IDR-SRV-014G`: `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`
- `IDR-SRV-014A` OSH CSAPI Server Implementation Study, including its exact implementation and evidence boundaries
- `IDR-SRV-014B` Connected Systems Go CSAPI Server Implementation Study, including the earlier `v1.0.4` baseline and any delta required for later Part 3 commits
- `IDR-SRV-014E` OS4CSAPI Client Smoke Test Findings Study
- `IDR-SRV-014F` SECD Interoperability Findings Study
- `IDR-SRV-014G` OS4CSAPI Discussions Lessons-Learned Study
- Existing `IDR-SRV-035` Streaming and Event Publication Strategy plan
- OS4CSAPI organization and implementation coordination: https://github.com/OS4CSAPI
- Eclipse Paho MQTT Go client documentation used by CS-Go: https://pkg.go.dev/github.com/eclipse/paho.mqtt.golang
- Eclipse Mosquitto documentation for reproducible broker probes: https://mosquitto.org/documentation/
- Rust Tokio, axum, tracing, MQTT, CloudEvents, and AsyncAPI ecosystem sources only when needed to identify downstream implementation questions; final Rust/platform selection belongs to later topics.

---

## 5. Research Methodology

### Phase 1: Source Inventory, Version Pins, and Authority Baseline (2-2.5 hours)

**Objective:** Establish a reproducible, authority-classified Part 3 and implementation evidence set.

**Tasks:**

1. Refresh the official `part3-working-draft` head, document metadata, generated artifacts, branch history, and diff from the planning snapshot; select and record the controlling research commit.
2. Consult upstream-history register entries #14, #68, #104, and #187 through #195; refresh their issue, PR, branch, commit, and publication state without mining unrelated repository history.
3. Inventory every Part 3 source clause, requirement file, generated artifact, schema, example, AsyncAPI file, ATS file, TODO, broken reference, commented include, and missing dependency at the pinned snapshot.
4. Pin the CS-Go release/commit and the OSH PR head used for study; record repository, branch, commit, date, license, build/runtime requirements, and whether the code is merged or released.
5. Establish the authority hierarchy and evidence labels for approved standards, draft standards, official maintenance history, implementation code/tests, documentation, reproduced behavior, and analyst recommendation.
6. Define the draft-requirement, implementation-mapping, divergence, and downstream-handoff matrix fields.

**Expected Output:** Reproducible source inventory, version ledger, authority classification, and completeness inventory.

### Phase 2: Part 3 Draft Requirement and Completeness Analysis (3-4 hours)

**Objective:** Determine exactly what the draft defines and what it does not define.

**Tasks:**

1. Extract each message-class, content-negotiation, discovery, protocol-binding, security, and conformance statement with source anchor, keyword strength, dependency, and completeness status.
2. Map Resource Events, Batch Resource Events, and Resource Data Messages across triggers, resources, operations, directions, envelopes, identifiers, payloads, encodings, authority, and intended use.
3. Analyze the CloudEvents profile against CloudEvents 1.0.2, including source/id uniqueness, subject, type, time, content type, data, retransmission, and batch shape.
4. Analyze MQTT and other transport material, explicitly recording unwired includes, stubs, TODOs, missing topic/operation/QoS/session/retain/flow-control rules, and missing tests.
5. Analyze discovery and AsyncAPI evidence, distinguishing the packaged Part 2 artifacts, Part 3 candidates, implementation-generated documents, and requirements actually present in the draft.
6. Reconcile contradictions among source AsciiDoc, generated HTML/PDF, requirement files, examples, issues, and linked dependencies without selecting an implementation-preferred answer.

**Expected Output:** Part 3 requirement, message-model, dependency, and normative-gap matrices.

### Phase 3: CS-Go and OSH Implementation Mapping (3.5-4.5 hours)

**Objective:** Establish what the two active implementations actually do and how their evidence quality differs.

**Tasks:**

1. Inspect the pinned CS-Go Part 3 implementation path-by-path, including configuration, MQTT manager, topics, parsing, ingestion, HTTP mutation events, batching, AsyncAPI, startup/shutdown, error handling, and tests.
2. Inspect the pinned OSH PR #194 changes path-by-path, including service configuration, connector, topic validator, resource-event publisher, batch handling, content negotiation, permissions, event-bus integration, and tests.
3. Build and run relevant unit/integration tests when tooling and dependencies permit; record exact commands, versions, results, skipped tests, and environmental limitations. Do not infer pass status from the presence of test files.
4. Where practical, run bounded broker-assisted probes for publication, subscription, invalid topic, unauthorized event publication, content negotiation, reconnect, and batch behavior; otherwise derive only code-supported observations and label them accordingly.
5. Map each observed implementation behavior to an exact draft statement, unresolved issue, legacy artifact, host-architecture constraint, or implementation-specific choice.
6. Record completeness, security, reliability, and conformance-claim limitations for each implementation.

**Expected Output:** Evidence-qualified CS-Go and OSH implementation maps with reproducible test/probe records.

### Phase 4: Cross-Implementation Interoperability and Risk Analysis (2.5-3.5 hours)

**Objective:** Determine where independent implementations can interoperate and where the incomplete draft permits incompatible contracts.

**Tasks:**

1. Compare topic prefixes, nesting, collection/resource paths, suffixes, wildcards, format selection, type tokens, CloudEvents fields, payloads, event triggers, batching, discovery, and publish/subscribe directions.
2. Identify the minimal message/topic examples that demonstrate compatibility or divergence, and attribute each divergence to evidence rather than preference.
3. Evaluate client-to-server Resource Data ingestion, server-generated Resource Events, publisher restrictions, permissions, broker ACL assumptions, and failure behavior.
4. Evaluate durability, transaction consistency, loss, retry, duplicate, echo, ordering, reconnect, retain, replay, backpressure, and graceful-shutdown behavior without prematurely designing Glaux's solution.
5. Identify security, policy, DDIL, observability, performance, and deployment implications and the owning downstream topics.
6. Define cross-server fixtures and future interoperability experiments without creating engineering tickets or executing later-topic design work.

**Expected Output:** Cross-implementation divergence, interoperability, security, and operational-risk matrices.

### Phase 5: Glaux Adoption Readiness and Downstream Impact (2-2.5 hours)

**Objective:** Convert the evidence into an early Glaux planning decision and explicit downstream constraints.

**Tasks:**

1. Compare three bounded paths: carry forward as an experimental implementation candidate, monitor without commitment, or defer.
2. Evaluate safeguards for any experimental candidate: exact draft pin, explicit profile identifier, feature gate, disabled-by-default configuration, transport adapter boundary, generated contract, no unsupported conformance claim, and migration/change policy.
3. Identify draft-resilient internal concepts that may inform event, transaction, schema, persistence, policy, and fixture design without freezing an external MQTT contract.
4. Map each material finding to the owning downstream topic, especially `IDR-SRV-015`, `016`, `018`, `020`, `023`, `025`, `029`, `031` through `035`, security/DDIL, deployment, conformance, performance, and interoperability.
5. Define the questions and evidence gates reserved for `IDR-SRV-035`, including final adopt/defer/reject, transport choice, durable event architecture, discovery contract, compatibility profile, and implementation sequencing.
6. Identify any evidence-backed upstream clarification or monitoring action, while keeping proposed standards contributions separate from Glaux requirements.

**Expected Output:** Adoption-readiness decision matrix and downstream handoff baseline.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert the research into a decision-usable report without implementing or finalizing Part 3 support.

**Tasks:**

1. Consolidate the authority ledger, draft requirement extraction, completeness gaps, implementation maps, divergence findings, risks, and handoffs.
2. Reconcile the findings with accepted `IDR-SRV-014A`, `014B`, and `014G` evidence, explicitly recording post-report implementation deltas rather than rewriting accepted reports.
3. State an explicit, bounded adoption-readiness recommendation and the conditions that could change it.
4. Update only the Part 3-related upstream-history register entries whose state materially changed during execution.
5. Validate every core and detailed question, matrix field, and success criterion.
6. Prepare the report at the target deliverable path for plan-owner review.

**Expected Output:** Completed IDR-SRV-014H research report in review-ready form.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] The exact Part 3 research snapshot, branch, commit, status, generated artifacts, dependencies, and retrieval date are recorded.
- [ ] Every in-scope Part 3 requirements class and material statement is authority-classified and mapped, with TODOs, stubs, missing includes, missing AsyncAPI/ATS material, and unresolved dependencies explicit.
- [ ] Resource Events, Batch Resource Events, Resource Data Messages, content negotiation, discovery, protocol bindings, security, and conformance are analyzed separately.
- [ ] Part 3-related official issues and PRs are refreshed through the shared register with merge, branch, commit, publication, and authority status preserved.
- [ ] CS-Go and OSH are pinned and analyzed at exact commits, with code, documentation, tests, configuration, merge/release state, and reproducibility limitations explicit.
- [ ] The two implementations are compared field-by-field and behavior-by-behavior, and every material agreement or divergence has a reproducible source anchor.
- [ ] Tests or broker probes are executed where practical, and every unexecuted or environment-limited claim is labelled rather than inferred.
- [ ] Draft requirements, implementation precedent, observed behavior, analyst inference, and Glaux recommendation remain visibly distinct.
- [ ] Security, policy, durability, transaction, DDIL, observability, deployment, performance, conformance, and interoperability implications are bounded and handed to their owning topics.
- [ ] An explicit `experimental candidate / monitor / defer` recommendation is supported by a decision matrix, safeguards, evidence gates, and change conditions.
- [ ] `IDR-SRV-035` receives a precise decision backlog without having its final architecture decision preempted.
- [ ] References, commands, versions, commits, issue/PR states, and evidence limitations are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Draft CSAPI Part 3 Publish/Subscribe and Implementation Study Research Report<br>
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-014h-draft-csapi-part-3-publish-subscribe-and-implementation-study-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base, authority hierarchy, and version ledger
4. Part 3 document, dependency, artifact, and completeness inventory
5. Resource Events findings
6. Batch Resource Events findings
7. Resource Data Messages and content-negotiation findings
8. Protocol binding, topic, discovery, AsyncAPI, and conformance findings
9. CS-Go Part 3 implementation findings
10. OpenSensorHub Part 3 implementation findings
11. Cross-implementation comparison and interoperability findings
12. Reliability, transaction, durability, ordering, replay, and DDIL implications
13. Security, authorization, publisher-restriction, and policy implications
14. Glaux adoption-readiness decision analysis
15. Downstream topic handoff matrix, including the `IDR-SRV-035` decision backlog
16. Recommendations
17. Risks, constraints, open questions, and monitoring triggers
18. Validation against this plan's success criteria
19. References
20. Reproducible commands, fixtures, and supplemental matrices as appendices

The draft-requirement and completeness matrix should include, at minimum:

- Requirement or construct identifier
- Exact source anchor
- Snapshot/version/status
- Authority class
- Message or binding class
- Normative keyword or informative status
- Dependency
- Defined behavior
- Completeness or unresolved gap
- Linked issue/PR and current disposition
- CS-Go support
- OSH support
- Glaux implication
- Owning downstream topic
- Notes / confidence

The cross-implementation matrix should include, at minimum:

- Behavior or contract dimension
- Draft anchor/status
- CS-Go behavior and evidence
- OSH behavior and evidence
- Agreement/divergence
- Interoperability effect
- Security/reliability effect
- Candidate Glaux treatment
- Later decision owner
- Reproduction/test status

---

## 8. Dependencies

### Must Complete Before Starting

**Internal project prerequisites (completion gates):**

- The Overall Glaux Server IDR Research Plan and this topic plan must be accepted and current.
- `IDR-SRV-001` through `IDR-SRV-014G` research reports must be complete and accepted.
- The Glaux Server Goal and Definition, Research Report Template, and shared upstream-history register must be available.

**External evidence prerequisites:**

- The official Part 3 branch, relevant official issues/PRs, approved Parts 1 and 2, and directly referenced standards must be reachable or their access limitation recorded.
- The pinned CS-Go repository and OSH PR branch must be reachable. If build/runtime dependencies prevent execution, record the exact limitation and narrow behavioral claims to code/document evidence.
- Any project-controlled AEP-4789 material used must be available under its applicable handling rules; absence must be recorded without invented content.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-015: Canonical Glaux Server Resource Model`
- `IDR-SRV-016: Identifier, URI, and Resource Lifecycle Strategy`
- `IDR-SRV-018: Temporal, Validity, and Freshness Model`
- `IDR-SRV-020: Status, Availability, and System Event Model`
- `IDR-SRV-023: Schema and Encoding Validation Strategy`
- `IDR-SRV-025: Database and Persistence Architecture Options`
- `IDR-SRV-029: Transaction, Consistency, Idempotency, and Concurrency Strategy`
- `IDR-SRV-031: Server Write and Ingestion Model`
- `IDR-SRV-032: Publisher-to-Server Contract Boundary`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`
- `IDR-SRV-035: Streaming and Event Publication Strategy`
- `IDR-SRV-039` through `IDR-SRV-043` security, policy, DDIL, and synchronization topics
- `IDR-SRV-045` through `IDR-SRV-048` architecture, deployment, configuration, and observability topics
- `IDR-SRV-050` through `IDR-SRV-056` conformance, traceability, test, performance, security, and interoperability topics
- Final overall IDR synthesis report

---

## 9. Research Status Checklist

Update this section as work progresses.

- [x] Phase 1 complete
- [x] Phase 2 complete
- [x] Phase 3 complete
- [x] Phase 4 complete
- [x] Phase 5 complete
- [x] Phase 6 synthesis complete
- [x] Deliverable draft complete
- [x] Deliverable reviewed
- [x] Deliverable accepted

**Actual Research Time:** Approximately 5 hours of AI-assisted execution<br>
**Completion Date:** August 31, 2026

---

## 10. Notes and Open Questions

- Planning snapshot: official Part 3 commit `c95c1d6003359d0883c4dc759d7a148ab115fdb1`; refresh at execution.
- Planning implementation snapshots: CS-Go `4a00aa6f7bb1a519ffd56a9f5526d0b8d1c98952`; OSH PR #194 head `50774ec7e9c98f6ab8da827171e5c5abb9923a49`; refresh at execution.
- Known planning concern: the Part 3 source entry does not wire its MQTT stub or an ATS, so implementation topic layouts cannot currently be promoted into normative requirements.
- Known planning concern: CS-Go and OSH appear to use materially different resource-event topic layouts; the report must verify the exact difference and its cause.
- Open question: Is the protocol-agnostic message model stable enough for Glaux to use internally even if the MQTT binding changes?
- Open question: Which experimental capability/profile identifier can state exact draft compatibility without implying OGC conformance?
- Open question: Can broker-side publisher and subscription restrictions be verified and documented as part of one server conformance posture?
- Open question: Which Part 3 changes are likely before `IDR-SRV-035`, and what refresh trigger should reopen the adoption decision?
- Risk: A large implementation may create false confidence where independent interoperability has not been demonstrated.
- Risk: Designing the internal event model around one draft topic hierarchy could impose avoidable migration cost.
- Risk: Deferring all Part 3 awareness until `IDR-SRV-035` could force rework in identifiers, event schemas, transactions, authorization, and fixtures.
- Assumption: No Part 3 implementation work begins solely from this research plan; implementation authorization belongs to later planning after the research conclusions are accepted.

---

## References

- Glaux Server Overall IDR Research Plan: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md
- Research Plan Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Shared upstream-history register: `Docs/Research/Initial Designs/IDR/glaux-server/IDR Evidence/ogc-connected-systems-upstream-history-register.md`
- OGC API - Connected Systems Parts 1 and 2: https://ogcapi.ogc.org/connectedsystems/
- Official Part 3 planning snapshot: https://github.com/opengeospatial/ogcapi-connected-systems/tree/c95c1d6003359d0883c4dc759d7a148ab115fdb1/api/part3
- Official Part 3 issue cluster: https://github.com/opengeospatial/ogcapi-connected-systems/issues?q=is%3Aissue%20state%3Aopen%20label%3A%22target%3A%20api-part-3%20v1.0.0%22
- CloudEvents 1.0.2: https://github.com/cloudevents/spec/tree/v1.0.2
- MQTT 3.1.1 and 5.0: https://docs.oasis-open.org/mqtt/
- AsyncAPI Specification: https://www.asyncapi.com/docs/reference/specification/latest
- Connected Systems Go planning snapshot: https://github.com/SomethingCreativeStudios/connected-systems-go/tree/4a00aa6f7bb1a519ffd56a9f5526d0b8d1c98952
- OSH Part 3 draft PR #194: https://github.com/opensensorhub/osh-addons/pull/194
- Testing exemplar corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230/docs/research/testing/research-plans

# Section 035: Streaming and Event Publication Strategy - Research Plan

**Status:** Planned  
**Last Updated:** June 11, 2026  
**Estimated Research Time:** 14-18 hours  
**Actual Research Time:** TBD until complete  
**Deliverable Target:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-035-streaming-and-event-publication-strategy-report.md`

---

## Usage Instructions

Before executing this plan, review the exemplar corpus and align the report to the proven standards for content nature, type, level, and detail:

- Full research-plans corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans
- Blueprint-first exemplar: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/01-pr114-blueprint-analysis.md
- Inventory/sourcing rigor exemplar: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/15-fixture-sourcing-organization.md
- End-state synthesis exemplar: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/blob/phase-9/docs/research/testing/research-plans/38-testing-playbook-synthesis.md

This plan is limited to research planning. It does not execute the research and does not produce the research report.

---

## 1. Research Objective

This topic research must define the Glaux Server planning baseline for **streaming and event publication strategy** across OGC API - Connected Systems Part 2 dynamic data, datastream live updates, observation streaming, status update streaming, system event publication, source-health events, command-status events, feasibility and command lifecycle events, event replay, subscription recovery, backfill, broker integration, WebSocket/SSE/MQTT-style publication, message ordering, delivery guarantees, DDIL reconnect behavior, policy filtering, auditability, observability, conformance testing, performance testing, and external-client interoperability.

The research must answer:

- What dynamic records and resource changes should Glaux Server publish as live streams or events?
- What is the semantic difference between persisted observations, status updates, system events, command-status records, operational diagnostics, audit records, and externally published event messages?
- Which streaming and publication behaviors are required or implied by CSAPI Part 2, AEP-4789 server responsibilities, prior Glaux IDR findings, and practical interoperability needs?
- What publication patterns should Glaux Server support or prepare for:
  - server-sent events,
  - WebSocket streams,
  - MQTT topics,
  - message-broker publication,
  - internal outbox workers,
  - durable event logs,
  - replay/backfill APIs,
  - client polling fallbacks?
- How should Glaux Server handle durable persistence before publication, event ordering, delivery guarantees, duplicate prevention, subscription resume, replay, backpressure, fanout, filtering, authorization, and policy/releasability?
- How should streaming interact with ingestion normalization, time-series storage, latest-value views, system events, command/control lifecycle, DDIL caching/synchronization, source trust, observability, performance testing, conformance testing, and external clients?

The output must be a streaming and event publication strategy baseline with source anchors, event taxonomy, publication-pattern evaluation, durable-event/outbox guidance, subscription and replay semantics, delivery and ordering guidance, policy/security implications, downstream handoffs, and recommendations for Glaux Server.

### Why This Topic Order

This topic follows:

- `IDR-SRV-033: Dynamic Data Ingestion and Normalization Pipeline`
- `IDR-SRV-034: Datastream, Observation, and Status Update Semantics`

`IDR-SRV-033` defines how dynamic records are received, validated, normalized, and persisted. `IDR-SRV-034` defines what datastreams, observations, status updates, latest values, and dynamic records mean after ingestion. This topic defines when and how those persisted or state-changing records should be published as live events or streams. It must precede command lifecycle, feasibility, command authorization/safety/audit, DDIL synchronization, observability, performance testing, and interoperability testing because all of those depend on clear event and publication semantics.

### Critical Constraints

- Treat CSAPI Part 2, CSAPI Part 1, SensorML 3.0, SWE Common 3.0, ingestion findings, datastream/observation/status semantics, transaction/idempotency findings, time-series storage findings, source trust, AEP-4789 server responsibilities, and prior IDR findings as controlling.
- Do not treat streaming as a substitute for durable persistence. Define durable state, event records, publication messages, and transient notifications separately.
- Do not assume one publication protocol satisfies all use cases. Evaluate patterns and identify first-implementation and full-scope candidates.
- Do not claim exactly-once delivery unless evidence supports a practical implementation path. Distinguish durable persistence, at-least-once delivery, idempotent clients, replay, and duplicate handling.
- Do not finalize command lifecycle semantics here. Identify publication needs and hand detailed command behavior to `IDR-SRV-036` through `IDR-SRV-038`.
- Do not finalize DDIL behavior here. Identify replay, backfill, cache, and reconnect requirements and hand detailed DDIL work to `IDR-SRV-041`.
- Do not finalize security/policy architecture here. Identify streaming-specific security and releasability implications and hand detailed policy work to Category G.
- Keep the research bounded to Glaux Server behavior and server-side publication contracts.

---

## 2. Research Questions

### Core Questions

1. What records, resource changes, state changes, and operational events should Glaux Server publish?
2. What event taxonomy should distinguish observations, status updates, system events, source-health events, command-status events, validation events, audit events, and operational diagnostics?
3. Which publication patterns and protocols should Glaux Server support or prepare for?
4. How should durable event records, outbox processing, subscription delivery, replay, backfill, ordering, deduplication, filtering, and authorization work?
5. What downstream implications follow for command/control, DDIL, security, observability, fixtures, performance, conformance, and interoperability?

### Detailed Questions

#### Standards and Publication Baseline

- What streaming, event, dynamic-data, subscription, pub/sub, or live-update behavior is defined or implied by CSAPI Part 2?
- What CSAPI Part 1 resources must be referenced in event messages or stream topics?
- What SensorML and SWE Common structures affect streamed observation, status, or command-status payloads?
- What AEP-4789 server responsibilities imply streaming, event publication, status dissemination, operational alerts, command-status dissemination, or DDIL replay?
- Which publication behaviors are standards-driven, profile-driven, implementation-defined, or interoperability-driven?
- Which draft/development OGC CSAPI pub/sub work should be tracked as a source for future alignment?

#### Event and Stream Taxonomy

- What event/stream categories should Glaux Server distinguish:
  - observation value events,
  - datastream sample events,
  - latest-value update events,
  - status update events,
  - dynamic property events,
  - system events,
  - resource lifecycle events,
  - source-health events,
  - ingestion validation events,
  - command submission events,
  - command accepted/rejected events,
  - command progress events,
  - command completed/failed/cancelled events,
  - feasibility evaluation events,
  - synchronization/DDIL events,
  - policy/audit events,
  - operational diagnostics?
- Which categories are client-visible CSAPI events?
- Which categories are administrator-only, audit-only, or internal operational diagnostics?
- Which categories must be persisted before publication?
- Which categories may be transient notifications?

#### Publication Trigger Semantics

- Which actions should trigger publication:
  - new observation accepted,
  - observation corrected/superseded,
  - latest value changed,
  - status changed,
  - resource created/updated/retired,
  - source health changed,
  - ingestion failed/quarantined,
  - command state changed,
  - feasibility result created,
  - synchronization completed/conflicted,
  - policy-relevant state changed?
- Which triggers should be configurable?
- Which triggers should be generated by ingestion, transaction commit, background processing, command workflows, or admin actions?
- How should publication avoid duplicate triggers during retries, replay, or idempotent submissions?

#### Durable Event and Outbox Strategy

- Should Glaux Server use a durable event table, outbox pattern, event log, broker persistence, or hybrid pattern?
- What information must be stored in durable event records:
  - event identifier,
  - event type,
  - subject/resource,
  - source,
  - time fields,
  - payload reference,
  - sequence/cursor,
  - authorization/policy metadata,
  - delivery state,
  - replay eligibility,
  - correlation identifiers,
  - causation identifiers?
- How should event records link to observations, status records, command records, validation artifacts, and audit records?
- Which events should be immutable?
- Which events should be retained, archived, compacted, or pruned?
- How should durable events support replay, backfill, DDIL reconnect, and conformance evidence?

#### Publication Protocol and Pattern Evaluation

- Which publication patterns should be evaluated:
  - REST polling,
  - long polling,
  - Server-Sent Events,
  - WebSocket,
  - MQTT,
  - NATS,
  - Kafka,
  - CloudEvents-compatible messages,
  - internal queue/outbox worker,
  - federated pull from event resources,
  - static/batch export for DDIL?
- What are the tradeoffs for each pattern across:
  - CSAPI alignment,
  - browser-client support,
  - tactical/mobile support,
  - broker interoperability,
  - local development,
  - CI/conformance testing,
  - DDIL operation,
  - authentication/authorization,
  - backpressure,
  - ordering,
  - replay,
  - operational complexity?
- Which patterns are first-implementation candidates versus full-scope readiness candidates?
- Which patterns belong in core Glaux Server versus optional deployment adapters or broker integrations?

#### Topic, Channel, and Subscription Model

- How should event channels or topics be organized:
  - by datastream,
  - by system,
  - by deployment,
  - by collection/resource family,
  - by source,
  - by event type,
  - by command/control stream,
  - by policy scope,
  - by tenant/mission profile?
- What subscription filters should be supported:
  - resource id,
  - collection id,
  - datastream id,
  - event type,
  - time range,
  - spatial filter,
  - observed property,
  - source,
  - status/freshness,
  - command id,
  - policy scope?
- Which filtering should occur server-side before publication?
- Which filtering can occur client-side?
- How should topic naming avoid leaking sensitive resource identities, source identities, or command authority?

#### Ordering, Cursor, Replay, and Backfill Semantics

- What ordering guarantees are needed:
  - per datastream,
  - per system,
  - per source,
  - per command,
  - global approximate ordering,
  - no global ordering?
- What cursor/resume model should be evaluated:
  - event id,
  - sequence number,
  - source offset,
  - time-based cursor,
  - broker offset,
  - opaque resume token?
- How should replay and backfill behave after disconnect?
- How should late-arriving data, out-of-order records, corrected records, and replayed records appear in event streams?
- How should clients detect duplicates?
- How should DDIL reconnect differ from normal client reconnect?

#### Delivery Guarantees, Backpressure, and Failure Handling

- What delivery guarantees are appropriate for each event category:
  - best effort,
  - at-most-once,
  - at-least-once,
  - effectively-once through idempotent consumers,
  - durable replayable?
- How should Glaux Server handle slow consumers, disconnected clients, broker failures, publication worker failures, retry storms, dropped messages, and backpressure?
- Which failures should affect source health, server health, or event backlog diagnostics?
- How should publication failures be separated from ingestion/persistence success?
- Which events must never be dropped?

#### Payload Shape and Content Negotiation

- What should event payloads contain:
  - full resource,
  - compact event envelope,
  - links to resource,
  - observation result only,
  - latest-value summary,
  - command-status summary,
  - CloudEvents-style envelope,
  - CSAPI-native representation,
  - policy-filtered payload?
- How should content negotiation and media types apply to streamed payloads?
- How should payload shape vary by client, protocol, event type, policy, or profile?
- How should payloads preserve SensorML/SWE Common result semantics, units, nil values, quality, and semantic bindings?
- How should payload size and bandwidth limits be handled?

#### Security, Authorization, and Policy Filtering

- How should streaming subscriptions be authenticated and authorized?
- How should policy/releasability affect topic visibility, subscription acceptance, event payloads, event metadata, replay, backfill, counts, and latest-value publication?
- How should Glaux Server prevent leakage through event timing, topic names, resource identifiers, counts, cursor gaps, or error messages?
- How should command-related events be restricted?
- Which events are administrator-only or audit-only?
- How should subscription authorization be reevaluated over long-lived connections?
- Which findings should be handed to Category G and security-test topics?

#### DDIL and Federation Implications

- How should streaming behave under disconnected, degraded, intermittent, and limited-bandwidth conditions?
- How should Glaux Server support local buffering, replay, catch-up, summaries, latest-state snapshots, and bandwidth-reduced streams?
- How should federated servers exchange or expose events?
- How should stale streams, delayed events, reconnection, and conflict events be represented?
- Which findings should be handed to `IDR-SRV-041` and federation/interoperability topics?

#### Command, Control Stream, and Feasibility Publication

- Which command lifecycle transitions should be published?
- How should command status streams differ from observation/status streams?
- How should feasibility results be published or made available?
- How should command/event publication support audit and safety?
- How should command events be correlated with command requests, control streams, target systems, and operators?
- Which findings should be handed to `IDR-SRV-036`, `IDR-SRV-037`, and `IDR-SRV-038`?

#### Observability and Operational Diagnostics

- What operational metrics are needed:
  - connected subscribers,
  - subscriptions by type,
  - event backlog,
  - publication latency,
  - delivery failures,
  - reconnect count,
  - dropped messages,
  - replay requests,
  - backpressure events,
  - broker connection health,
  - authorization failures?
- Which metrics are safe to expose publicly, administrator-only, or internal-only?
- How should streaming diagnostics link to source health, ingestion health, and command health?
- Which findings should be handed to `IDR-SRV-049`?

#### Fixtures, Conformance, Performance, and Interoperability

- What streaming fixtures are needed:
  - observation stream,
  - status stream,
  - event stream,
  - command-status stream,
  - late data,
  - duplicate data,
  - replay,
  - backfill,
  - disconnected/reconnect client,
  - policy-filtered stream,
  - slow consumer,
  - broker failure?
- What conformance tests should verify event and streaming behavior?
- What performance/load/stress tests are needed for fanout, high-rate observation streams, reconnect, replay, backpressure, broker failure, and policy-filtered subscriptions?
- What interoperability tests should involve CSAPI Explorer, OS4CSAPI clients, webapp/mobile clients, MQTT/broker clients, and external CSAPI clients?

#### Implementation and Interoperability Lessons

- What streaming/event publication lessons were identified in OSH, Connected Systems Go, pygeoapi, and SECD?
- What smoke-test or interoperability findings indicate streaming, event, status-update, command-status, replay, or publication issues?
- What OS4CSAPI discussion lessons affect streaming and event publication strategy?
- Which implementation patterns should Glaux Server adopt, avoid, or investigate further?
- Which findings must be treated as implementation-specific rather than standards-derived?

---

## 3. Primary Resources

The future research report must analyze these sources directly.

### Project and Governance Sources

- Glaux Server Overall IDR Research Plan: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Creation Prompt: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-creation-prompt.md
- Research Plan Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Overall Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/overall-research-report-template.md
- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md

### Prior IDR Topic Sources

- `IDR-SRV-001` through `IDR-SRV-034` research reports, once complete:
  - `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/`

### Controlling Standards and API Sources

- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- OGC schemas: https://schemas.opengis.net/

### Streaming, Event, Messaging, and Reliability Candidate Sources

Use current official documentation and primary-source technical material when executing the research. Candidate sources include, but are not limited to:

- MQTT v5 specification: https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html
- Eclipse Mosquitto documentation: https://mosquitto.org/documentation/
- NATS documentation: https://docs.nats.io/
- Apache Kafka documentation: https://kafka.apache.org/documentation/
- CloudEvents specification: https://cloudevents.io/
- Server-Sent Events specification: https://html.spec.whatwg.org/multipage/server-sent-events.html
- WebSocket RFC 6455: https://www.rfc-editor.org/rfc/rfc6455
- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9457 - Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457
- OpenTelemetry documentation: https://opentelemetry.io/docs/
- Rust Tokio documentation: https://docs.rs/tokio/
- Rust axum documentation, if evaluated: https://docs.rs/axum/
- Rust tracing documentation: https://docs.rs/tracing/

These are candidates for evaluation, not assumed selections.

### AEP-4789 and STANAG 4789 Sources

Use source material available to the Glaux project team and record exact title, volume, version, date, status, storage location, and classification/handling constraints:

- STANAG 4789
- AEP-4789 Volume I
- AEP-4789 Volume II
- Any approved or draft AEP-4789 schemas, profiles, implementation guidance, dynamic-data guidance, pub/sub or streaming guidance, event guidance, command/status guidance, DDIL guidance, or standards-package annexes relevant to streaming, event publication, and NATO JISR sensor integration

### Implementation and Lessons-Learned Sources

Use implementation-study outputs and source repositories as non-normative evidence:

- OSH / OpenSensorHub sources and `IDR-SRV-014A`
- Connected Systems Go sources and `IDR-SRV-014B`
- pygeoapi sources and `IDR-SRV-014C`
- SECD sources and `IDR-SRV-014D`
- OS4CSAPI Client Smoke Test Findings and `IDR-SRV-014E`
- SECD Interoperability Findings and `IDR-SRV-014F`
- OS4CSAPI Discussions Lessons-Learned and `IDR-SRV-014G`
- Category C findings from `IDR-SRV-015` through `IDR-SRV-020`
- Category D findings from `IDR-SRV-021` through `IDR-SRV-024`
- Category E findings from `IDR-SRV-025` through `IDR-SRV-030`
- Category F findings from `IDR-SRV-031` through `IDR-SRV-034`
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/

---

## 4. Supporting Resources

Use these sources to interpret project context, downstream dependencies, expected research depth, and report style.

- DGIWG P5.07 Glaux main repository: https://github.com/DGIWG-P507/glaux
- Glaux project website: https://dgiwg-p507.github.io/glaux/
- DGIWG P5.07 GitHub organization: https://github.com/DGIWG-P507
- Glaux Server repository, if available or created: https://github.com/DGIWG-P507/glaux-server
- Glaux Webapp repository, if available or created: https://github.com/DGIWG-P507/glaux-webapp
- Glaux Mobile repository, if available or created: https://github.com/DGIWG-P507/glaux-mobile
- Glaux Publisher repository, if available or created: https://github.com/DGIWG-P507/glaux-publisher
- Glaux Simulator repository, if available or created: https://github.com/DGIWG-P507/glaux-simulator
- OS4CSAPI testing research-plan corpus: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9/docs/research/testing/research-plans
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

---

## 5. Research Methodology

### Phase 1: Source Collection and Streaming Framework Setup (1.5-2 hours)

**Objective:** Establish the evidence base and extraction framework for streaming and event publication research.

**Tasks:**

1. Gather prior IDR reports, standards sources, implementation studies, messaging/streaming documentation, dynamic-data semantics findings, and project architecture sources.
2. Extract streaming/event requirements from prior topics and classify them by event type, trigger, publication audience, persistence need, delivery guarantee, policy sensitivity, and downstream dependency.
3. Define inventory fields:
   - event/stream category,
   - trigger,
   - source record/resource,
   - durable/transient classification,
   - publication pattern,
   - ordering/cursor need,
   - replay/backfill need,
   - authorization/policy need,
   - payload shape,
   - downstream handoff.
4. Define evaluation criteria:
   - standards alignment,
   - client interoperability,
   - browser/mobile support,
   - broker compatibility,
   - durable replay,
   - delivery robustness,
   - DDIL suitability,
   - security/policy suitability,
   - testability,
   - operational complexity.
5. Establish evidence hierarchy for standards, AEP material, prior reports, implementation evidence, and technology documentation.

**Expected Output:** Streaming/event publication extraction framework and evaluation rubric.

### Phase 2: Event Taxonomy and Publication Trigger Inventory (3-4 hours)

**Objective:** Determine what Glaux Server should publish and why.

**Tasks:**

1. Inventory dynamic record categories and state changes from `IDR-SRV-033` and `IDR-SRV-034`.
2. Classify publication candidates as observations, status updates, latest-value changes, system events, source-health events, command-status events, feasibility events, validation events, DDIL events, operational diagnostics, or audit-only records.
3. Identify which candidates require durable event records.
4. Identify which candidates are client-visible, administrator-only, internal-only, or audit-only.
5. Build an event taxonomy and trigger matrix.

**Expected Output:** Event/stream taxonomy and publication trigger matrix.

### Phase 3: Durable Event, Outbox, Ordering, Replay, and Payload Analysis (3-4 hours)

**Objective:** Define durable publication semantics and event-message behavior.

**Tasks:**

1. Analyze durable event records, outbox/inbox patterns, and publication workers.
2. Analyze event ordering, cursors, resume tokens, replay, backfill, late data, duplicate data, corrected data, and client deduplication.
3. Analyze payload shapes, envelopes, content negotiation, links, full-resource payloads, compact messages, and CloudEvents-style patterns.
4. Analyze retention, pruning, replay eligibility, and audit/conformance evidence.
5. Identify unresolved questions requiring prototype validation or performance testing.

**Expected Output:** Durable event and replay strategy matrix.

### Phase 4: Publication Protocol, Subscription, Security, and DDIL Analysis (3-4 hours)

**Objective:** Evaluate protocols and high-risk subscription behavior.

**Tasks:**

1. Evaluate REST polling, SSE, WebSocket, MQTT, NATS/Kafka-style broker publication, and hybrid patterns.
2. Analyze subscription topics/channels, server-side filtering, policy-filtered delivery, authorization, long-lived connection behavior, and safe error diagnostics.
3. Analyze delivery guarantees, slow consumers, backpressure, failures, broker outages, and retry behavior.
4. Analyze DDIL, reconnect, bandwidth reduction, latest-state snapshots, and federated publication.
5. Map findings to security, DDIL, deployment, and performance topics.

**Expected Output:** Publication protocol and subscription strategy matrix.

### Phase 5: Command, Observability, Fixtures, Performance, Conformance, and Interoperability Analysis (3-4 hours)

**Objective:** Prepare streaming findings for downstream implementation and verification.

**Tasks:**

1. Analyze command lifecycle and feasibility publication requirements.
2. Identify observability metrics and diagnostics for streaming/event publication.
3. Identify streaming fixtures and simulated client/source behaviors.
4. Identify conformance tests for event publication, replay, subscription filtering, payloads, and error behavior.
5. Identify performance/load/stress tests for high-rate streaming, fanout, backpressure, reconnect, replay, and policy-filtered subscriptions.
6. Identify interoperability tests with CSAPI Explorer, OS4CSAPI clients, webapp/mobile clients, broker clients, and external CSAPI clients.

**Expected Output:** Streaming downstream verification and operations matrix.

### Phase 6: Synthesis (1.5-2 hours)

**Objective:** Convert streaming/event publication research into a decision-usable baseline.

**Tasks:**

1. Consolidate event taxonomy, trigger rules, durable event/outbox guidance, protocol evaluation, subscription guidance, replay/backfill guidance, security/policy findings, and downstream implications.
2. Produce recommended streaming and event publication strategy with rationale and unresolved questions.
3. Identify sequencing for command, DDIL, security, deployment, observability, fixture, conformance, and performance topics.
4. Produce downstream handoff matrix.
5. Prepare the deliverable for review using the research-report template.

**Expected Output:** Completed topic research report at the target deliverable path.

---

## 6. Success Criteria

This topic research is complete when:

- [ ] Event and stream categories are identified and distinguished with source anchors.
- [ ] Publication triggers, durable event records, outbox/inbox implications, payload shapes, ordering, cursor, replay, and backfill semantics are documented.
- [ ] Publication protocols and patterns are evaluated against explicit criteria.
- [ ] Delivery guarantees, duplicate handling, slow consumers, backpressure, broker failure, reconnect, and DDIL implications are documented.
- [ ] Security, authorization, policy/releasability, command-event, observability, fixture, conformance, performance, and interoperability implications are documented.
- [ ] Implementation-study and community-lesson findings are incorporated as non-normative evidence.
- [ ] Recommendations are decision-usable and bounded to Glaux Server.
- [ ] Downstream handoffs are explicit.
- [ ] References are explicit and reproducible.

---

## 7. Deliverable

**Deliverable Name:** Streaming and Event Publication Strategy Research Report  
**Deliverable File:** `Docs/Research/Initial Designs/IDR/glaux-server/IDR Reports/idr-srv-035-streaming-and-event-publication-strategy-report.md`

**Required Content:**

1. Executive summary
2. Scope and plan alignment
3. Evidence base and authority classification
4. Streaming/event requirement extraction methodology
5. Event and stream taxonomy
6. Publication trigger findings
7. Durable event, event-log, and outbox/inbox findings
8. Ordering, cursor, replay, backfill, retention, and duplicate-handling findings
9. Payload shape, content negotiation, and message-envelope findings
10. Publication protocol and pattern evaluation
11. Topic/channel, subscription filtering, delivery guarantee, and backpressure findings
12. Security, authorization, policy, and releasability implications
13. DDIL, reconnect, latest-state snapshot, and federation implications
14. Command, control stream, feasibility, and command-status publication implications
15. Observability, fixture, conformance, performance, and interoperability test implications
16. Downstream topic handoff matrix
17. Recommendations
18. Risks, constraints, and open questions
19. Validation against this plan's success criteria
20. References

The streaming/event publication matrix should include, at minimum:

- Event/stream category
- Trigger
- Source record/resource
- Durable/transient classification
- Publication pattern
- Ordering/cursor need
- Replay/backfill need
- Payload shape
- Authorization/policy requirement
- Delivery guarantee
- DDIL implication
- Test/conformance implication
- Downstream topic handoff
- Notes / unresolved issues

---

## 8. Dependencies

### Must Complete Before Starting

- Overall Glaux Server IDR Research Plan must be available and current.
- Glaux Server Goal and Definition must be available and current.
- `IDR-SRV-001` through `IDR-SRV-034` research reports should be complete or explicitly marked unavailable/deferred.
- Official CSAPI Part 1 and Part 2, OGC API - Features, SensorML, SWE Common, relevant OGC schemas/OpenAPI artifacts, streaming/messaging sources, and project-available AEP-4789 material must be reachable or explicitly marked unavailable.
- Implementation-study and interoperability findings must be reachable or explicitly marked unavailable.
- Research report template must be available.

### Blocks (What This Topic Unlocks)

- `IDR-SRV-036: Control Stream and Command Lifecycle Model`
- `IDR-SRV-037: Feasibility and Command Validation Strategy`
- `IDR-SRV-038: Command Authorization, Safety, and Audit Strategy`
- `IDR-SRV-039: Authentication, Authorization, and API Security Threat Model`
- `IDR-SRV-040: Policy, Releasability, and Cross-Boundary Access Constraints`
- `IDR-SRV-041: DDIL Behavior, Caching, and Synchronization Semantics`
- `IDR-SRV-045: Service Packaging, Containerization, and Deployment Topology`
- `IDR-SRV-046: Local Development and CI Environment Strategy`
- `IDR-SRV-049: Observability, Logging, Metrics, and Operational Diagnostics`
- `IDR-SRV-050: Conformance Harness Strategy`
- `IDR-SRV-052: Rust Test-Driven Architecture and Multi-Layer Test Strategy`
- `IDR-SRV-053: Test Data, Fixtures, Golden Files, and Scenario Corpus Strategy`
- `IDR-SRV-054: Performance, Load, Stress, and Streaming Test Strategy`
- `IDR-SRV-055: Security, Authorization, and Command-Control Test Strategy`
- `IDR-SRV-056: Interoperability Test Matrix for External CSAPI Clients`
- Final overall IDR synthesis report

---

## 9. Research Status Checklist

Update this section as work progresses.

- [ ] Phase 1 complete
- [ ] Phase 2 complete
- [ ] Phase 3 complete
- [ ] Phase 4 complete
- [ ] Phase 5 complete
- [ ] Phase 6 synthesis complete
- [ ] Deliverable draft complete
- [ ] Deliverable reviewed
- [ ] Deliverable accepted

**Actual Research Time:** TBD until complete  
**Completion Date:** TBD until complete

---

## 10. Notes and Open Questions

- This topic defines streaming and event publication strategy, not final implementation code.
- Streaming must complement durable persistence rather than replace it.
- Publication semantics should distinguish client-visible streams from internal operational diagnostics and audit records.
- Implementation-study findings are useful but must not override standards-derived server responsibilities.
- Open question: Which publication protocol should be first implementation versus full-scope readiness?
- Open question: Which event categories require durable replay?
- Open question: How should cursor/resume behavior be represented across protocols?
- Open question: How should policy filtering handle topic names, event gaps, counts, and replay?
- Open question: Which streaming fixtures should become canonical conformance/interoperability test data?
- Risk: Publishing before durable persistence could create unrecoverable client-visible state.
- Risk: Underdefined replay and duplicate handling could break DDIL and reconnect behavior.
- Risk: Overexposing topics or event metadata could leak sensitive operational information.
- Risk: Selecting a heavyweight broker requirement too early could undermine local development and tactical-edge deployment.

---

## References

- Glaux Server Overall IDR Research Plan: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Research/Initial%20Designs/IDR/glaux-server/IDR%20Plans/overall-idr-research-plan.md
- Glaux Server Goal and Definition: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md
- Research Plan Creation Prompt: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-creation-prompt.md
- Research Plan Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-plan-template.md
- Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-report-template.md
- Overall Research Report Template: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/overall-research-report-template.md
- Research Planning Approach: https://github.com/DGIWG-P507/glaux/blob/main/Docs/Governance/research-planning-approach.md
- OGC API - Connected Systems landing page: https://ogcapi.ogc.org/connectedsystems/
- OGC API - Connected Systems - Part 1: Feature Resources: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data: https://docs.ogc.org/is/23-002/23-002.html
- OGC API - Connected Systems public development repository: https://github.com/opengeospatial/ogcapi-connected-systems
- OGC API - Connected Systems OpenAPI artifacts: https://github.com/opengeospatial/ogcapi-connected-systems/tree/master/core/openapi
- OGC API - Features Part 1: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0: https://docs.ogc.org/is/24-014/24-014.html
- OGC schemas: https://schemas.opengis.net/
- MQTT v5 specification: https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html
- Eclipse Mosquitto documentation: https://mosquitto.org/documentation/
- NATS documentation: https://docs.nats.io/
- Apache Kafka documentation: https://kafka.apache.org/documentation/
- CloudEvents specification: https://cloudevents.io/
- Server-Sent Events specification: https://html.spec.whatwg.org/multipage/server-sent-events.html
- WebSocket RFC 6455: https://www.rfc-editor.org/rfc/rfc6455
- RFC 9110 - HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9457 - Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457
- OpenTelemetry documentation: https://opentelemetry.io/docs/
- Rust Tokio documentation: https://docs.rs/tokio/
- Rust axum documentation: https://docs.rs/axum/
- Rust tracing documentation: https://docs.rs/tracing/
- OS4CSAPI organization: https://github.com/OS4CSAPI
- OS4CSAPI client work: https://github.com/OS4CSAPI/ogc-client-CSAPI_2
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/
- OS4CSAPI discussions: https://github.com/orgs/OS4CSAPI/discussions

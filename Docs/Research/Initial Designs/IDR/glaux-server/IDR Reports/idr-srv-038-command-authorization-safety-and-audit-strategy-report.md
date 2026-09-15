# Section 038: Command Authorization, Safety, and Audit Strategy - Research Report

**Topic ID:** IDR-SRV-038<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-038 Command Authorization, Safety, and Audit Strategy](../IDR%20Plans/idr-srv-038-command-authorization-safety-and-audit-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All five core questions and all detailed question groups concerning standards, decision taxonomy, command authority, safety rules, lifecycle integration, feasibility/validation, source and gateway trust, policy and disclosure, audit, diagnostics, DDIL, events, persistence, and verification<br>
**Methodology Used:** Authority-ranked standards extraction; immutable artifact and implementation pins; accepted-baseline reconciliation; threat- and failure-window analysis; subject/object/action/environment authorization modeling; lifecycle gate mapping; audit-integrity modeling; DDIL scenario analysis; fixture and test traceability<br>
**Research Time:** Approximately 25 hours of AI-assisted execution on September 15, 2026<br>
**Approved Standards Baseline:** OGC 23-001 and OGC 23-002 Version 1.0, repository tag `v1.0.0` at [`8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)<br>
**Current Security Guidance Check:** NIST SP 800-53 Rev. 5 Release 5.2.0, NIST SP 800-162 updated 2019, NIST SP 800-207, NIST SP 800-92, RFC 7662, RFC 9334, and RFC 9700 checked September 15, 2026<br>
**Implementation Evidence:** CS-Go `v1.0.4` at [`244f4dd586da685d4d9b75e43f73001028b5bd0e`](https://github.com/SomethingCreativeStudios/connected-systems-go/commit/244f4dd586da685d4d9b75e43f73001028b5bd0e); OpenSensorHub `v2.0.2` at [`235c0eabf24b6d6137b499b4402943d2794b70e6`](https://github.com/opensensorhub/osh-core/commit/235c0eabf24b6d6137b499b4402943d2794b70e6); OS4CSAPI phase-9 at `754411897173c2ec4debaa9bcf4ed9e0f8a9e230`; SECD at `f018fd129bf0d0d1ce75e68198e3ab4d99d937a0`<br>
**Supporting Resources:** Accepted IDR-SRV-001 through IDR-SRV-037, controlled-AEP findings, and upstream-history register Version 1.12<br>
**Document Purpose:** Establish a bounded, implementable command-specific authorization, safety/interlock, approval, dispatch-authorization, disclosure, and accountability baseline without selecting the full enterprise identity architecture or implementing the server<br>
**Author:** OpenAI Codex<br>
**Date:** September 15, 2026<br>
**Last Updated:** September 15, 2026

---

## Evidence and Decision Legend

| Mark | Meaning |
|---|---|
| N | Normative or controlling published-standard evidence |
| A | Accepted Glaux design baseline |
| I | Pinned implementation or interoperability observation |
| E | Engineering or security analysis inferred from cited evidence |
| P | Proposed Glaux decision requiring report acceptance |
| X | Known defect, ambiguity, or unresolved upstream proposal |

The marks classify claims; they are not conformance levels. “Must” and “shall” identify a published obligation or a consequence of an already accepted project decision. “Should” identifies a recommendation awaiting acceptance with this report.

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Command Authorization, Safety, and Audit Extraction Methodology
5. Command Decision Taxonomy
6. Command Authority Model Findings
7. Safety Rule Model Findings
8. Command Lifecycle Authorization and Safety Hook Findings
9. Feasibility, Validation, Source Trust, Gateway, and Dispatch Integration Findings
10. Policy, Releasability, and Diagnostic-Redaction Findings
11. Audit Record Model Findings
12. Transaction, Idempotency, Event Publication, Persistence, Retention, and Synchronization Findings
13. DDIL, Cached Authorization, Stale Policy, Stale Feasibility, and Tactical-Edge Implications
14. Fixture, Conformance, Security Testing, Performance, and Interoperability Test Implications
15. Downstream Topic Handoff Matrix
16. Recommendations
17. Risks, Constraints, and Open Questions
18. Validation Against This Plan's Success Criteria
19. References
Appendices A-C

---

## 1. Executive Summary

Glaux should implement command control as a **separate, deny-by-default decision plane** around the accepted CSAPI Command lifecycle. A caller's successful authentication, permission to invoke an API route, trusted-source status, schema-valid payload, favorable Feasibility result, or broker delivery acknowledgement is never by itself permission to affect a target. A command may be feasible but unauthorized, authorized but infeasible, and feasible and authorized but unsafe. Each decision must remain explicit, evidence-bound, independently auditable, and capable of changing without falsifying CSAPI state. **[N/A/E/P]**

The decision plane should use a capability- and attribute-based model. A command grant is scoped to the authenticated principal and client/service, represented organization or mission authority, operation, target System, immutable ControlStream contract/version, command type and parameter envelope, mission/deployment, time window, gateway or execution path, operating mode, and offline eligibility. Roles are useful administrative groupings, but route-level RBAC alone cannot express target, parameter, time, environment, or gateway constraints. This follows NIST's ABAC model of evaluating subject, object, requested operation, and environment attributes against policy while leaving IDR-SRV-039 to select the enterprise authentication and token architecture.[^1] **[E/P]**

Glaux should distinguish ordinary API authorization from an internal **CommandAuthorityGrant**. The latter proves that a named authority permits a subject to initiate or mediate specified effects against specified targets. It is versioned, revocable, time-bounded, purpose-bound, and not a public CSAPI resource. Separate grants govern feasibility, submission, update, cancellation, approval, override, dispatch, status reporting, and result reporting. A gateway can relay only the target/action envelope granted to it; transport or broker ACLs are defense in depth and never command authorization. **[A/E/P]**

All accepted commands pass through ordered hooks: protected discovery; authentication and source-trust resolution; endpoint authorization; structural/schema/semantic validation; command/target authority; policy and releasability; required feasibility; safety/interlock evaluation; durable operator approval where required; receiving-system acceptance; and a **final pre-dispatch re-evaluation**. Dispatch receives a short-lived, single-use internal ticket bound to the command ID and revision, target, ControlStream contract, policy/safety/trust versions, route/gateway, deadline, and fencing value. Any material input change or expiry invalidates that ticket. This closes the time-of-check/time-of-use window without inventing a public CSAPI state. **[A/E/P]**

Safety should be a versioned rule system, not a side effect of SWE validation. SensorML and SWE Common can express capabilities, inputs, units, choices, and allowed-value constraints; these are essential inputs but do not prove operational safety. Glaux also needs static non-overrideable interlocks, operational-mode and target-state rules, sequence/conflict constraints, rate/repeat limits, geospatial and environmental bounds, mission/deployment constraints, gateway/source freshness, human approval, and DDIL restrictions. Rules return structured internal decisions—allow, deny, require approval, or require fresh evidence. A warning cannot satisfy a mandatory rule. An override is a separately authorized, bounded, reasoned, expiring decision and may affect only rules explicitly marked overrideable. **[N/A/E/P]**

The published CSAPI boundary remains unchanged. Part 1 expects protected functionality and authentication but selects no authentication method; Part 2 inherits those considerations and says `CANCELED` is caused by an authorized user. Neither part defines control ownership, transfer, approval, interlocks, or audit schema.[^2] Upstream issue #58, “Discuss how to implement transfer of control,” remains open, has no linked implementation, and carries a working recommendation to defer or re-scope.[^3] Glaux should therefore identify its control-authority mechanism as an implementation profile, not an OGC conformance requirement. The first profile should prohibit automatic seizure or last-writer-wins transfer. An internal, versioned control-authority lease/grant serializes conflicting controllers; transfer requires explicit revoke/expire then grant, with partition recovery and preemption governed by profile.

Every material request, decision, transition, effect attempt, acknowledgement, status/result assertion, cancellation, approval, override, timeout, redaction, and reconciliation action needs durable audit evidence. The public `CommandStatus` history is domain state, not the audit log. Audit records should be logically append-only, carry event/actor/object/outcome/time and Glaux decision-input provenance, exclude credentials and unnecessary sensitive payloads, and be written atomically with the decision or state transition they evidence. If required command-audit durability is unavailable, Glaux should fail closed before creating authoritative asynchronous work or before dispatching an effect. NIST AU controls establish event selection, record content, time, protection, failure response, generation, retention, and non-repudiation concerns; IDR-SRV-041 will finalize the general audit architecture and retention policy.[^4] **[E/P]**

DDIL operation should not depend on a permanently reachable authorization service. It should depend on locally verifiable, signed, versioned authorization/policy/safety/trust bundles whose issuer, audience, scope, validity, revocation epoch, maximum staleness, time-confidence requirement, and offline operating class are explicit. Glaux should define command classes as online-required, bounded-local-authority, explicitly pre-authorized emergency, or simulation/test-only. Unknown time, expired authority, stale mandatory state, or uncertain revocation must not fail open. Commands retained across disconnection must be re-authorized immediately before dispatch, and stale Feasibility evidence never satisfies a safety rule. **[E/P]**

This report completes Category F research but authorizes no implementation. IDR-SRV-039 remains unauthorized until this report is accepted. IDR-SRV-039 owns general authentication/API threats; 039A owns full zero-trust topology; 040 owns cross-boundary policy; 041 owns general audit; 042/043 own final DDIL and synchronization conflict behavior; and 055 owns the complete command-security test program. **[A]**

### 1.1 Recommended baseline at a glance

| Decision | Rationale | Implementation consequence | Confidence |
|---|---|---|---|
| Deny by default; evaluate subject/object/action/environment | Route permission cannot express target, parameter, mission, time, or state bounds | Typed command decision context and policy interface | High |
| Separate API permission from CommandAuthorityGrant | Access to a collection is not authority to create physical effect | Versioned, revocable internal grants per action and target | High |
| Recheck immediately before dispatch | Authority, policy, target state, and safety inputs may change after admission | Short-lived single-use dispatch ticket with fencing | High |
| SWE/SensorML are inputs, not complete safety policy | Valid units/ranges do not establish operational safety | Layered rule engine with explicit provenance and overrideability | High |
| Approval and override are durable internal evidence, not public statuses | CSAPI has no approval state and exact nine-code boundary is accepted | Keep `PENDING` until a standards-valid decision; protected operator workflow | High |
| No automatic transfer of control | OGC behavior is unresolved; partitions and competing controllers are unsafe | Internal serialized control-authority grant/lease, no public conformance claim | Medium-high |
| Audit is atomic and append-oriented | Effect without durable accountability is unacceptable | Fail closed at authoritative admission/dispatch when audit cannot commit | High |
| Locally verifiable bounded DDIL authority | Always-online checks conflict with tactical operation | Signed/versioned cached bundles and offline command classes | High |
| Public diagnostics are policy-filtered and non-oracular | Reasons can reveal capabilities, rules, topology, identity, or state | Stable generic problem/status codes; details only in protected audit | High |

## 2. Scope and Plan Alignment

### 2.1 Included scope

This report defines command-specific authorization actions and attributes; target/control authority; safety rule categories and evaluation points; operator approval and override; lifecycle, feasibility, validation, source-trust, gateway, dispatch and cancellation integration; command-specific disclosure; audit events and fields; atomicity and idempotency; DDIL local decisions; and downstream verification obligations. It covers direct, gateway-mediated, brokered, simulated, and manual execution paths.

### 2.2 Excluded scope

The report does not select an identity provider, OAuth/OIDC deployment, credential format, PKI hierarchy, enterprise policy language, accreditation baseline, records schedule, SIEM, immutable-storage product, Rust authorization library, service topology, broker, or operator UI. It does not define full cross-boundary information policy, general audit for non-command resources, final DDIL synchronization protocol, or an implementation schedule. Those choices belong to the indexed downstream topics.

It also does not expand the public CSAPI model with approvals, safety decisions, dispatch tickets, control leases, or audit endpoints. Such private records can support the standards API without pretending to be standardized resources.

### 2.3 Required semantic separations

| Concept | Question answered | Does not prove |
|---|---|---|
| Authentication | Who or what presented the request? | permission, trusted representation, or safety |
| API authorization | May this subject invoke this operation on this visible API object? | command authority or physical control |
| Source trust | May Glaux accept assertions from this publisher/gateway in this role? | caller authority over a target |
| Structural/schema validation | Is the representation well formed and conformant to its versioned schema? | semantic correctness, feasibility, or safety |
| Semantic validation | Do references, units, time, and domain values make coherent sense? | current feasibility or authority |
| Feasibility | Could this exact intent plausibly be performed under recorded context? | reservation, permission, safety, or success |
| Policy/releasability | May information/effect cross this boundary for this purpose? | operational safety or target acceptance |
| Command authority | May this actor request this effect on this target under this scope? | target willingness or present safety |
| Target/control authority | Which controller is entitled to direct the target/stream now? | request validity or successful execution |
| Safety | Does the action satisfy all applicable interlocks in current context? | target acceptance or physical outcome |
| Operator approval | Did an authorized person approve a bounded decision? | permanent authority or override of non-overrideable rules |
| Target acceptance | Did the receiving system agree to process it? | dispatch delivery or execution |
| Dispatch authorization | Are all required gates still valid at the effect boundary? | transport delivery, acceptance, or effect |
| Execution status | What authoritative lifecycle evidence did the executor report? | audit completeness or independently observed world state |
| Audit | What request, evidence, decisions, and effects occurred? | correctness merely because a record exists |

### 2.4 Plan coverage

All required report sections are present. Appendix A supplies the mandated command authorization/safety/audit matrix with every requested column. Appendices B and C provide an executable decision sequence, event catalog, and reproducibility record. Section 18 maps every success criterion to report evidence.

## 3. Evidence Base and Authority Classification

### 3.1 Standards baseline

OGC 23-001 and OGC 23-002 Version 1.0 at tag `v1.0.0` are controlling. Part 1 states an expectation that some CSAPI functionality is protected by access control requiring authentication, intentionally does not mandate a particular method, recommends protected transport/storage for sensitive data, and recommends strong revocable asymmetric authentication and signatures for machine clients. Part 2 says all Part 1 security considerations apply. **[N]**

Part 2 defines ControlStreams, Commands, Feasibility, status and results. `PENDING` precedes a decision; `ACCEPTED` can still be followed by rejection or failure; `REJECTED` means the command will not execute; and `CANCELED` denotes cancellation by an authorized user. It supplies no normative role vocabulary, authority grant, ownership-transfer protocol, approval workflow, interlock semantics, audit representation, or reason-redaction rule. `sender` is only an optional string and cannot be trusted as authentication evidence. **[N/E]**

SensorML can describe systems, inputs, parameters, characteristics, capabilities, modes, security/legal constraints, and associated resources. SWE Common describes typed values, units, allowed values, choices, arrays, records, and encodings. Those declarations constrain parsing and may provide safety evidence, but they are publisher-supplied/versioned metadata and do not themselves decide that an action is authorized or safe in a live mission context. **[N/E]**

### 3.2 Security and accountability guidance

NIST SP 800-162 provides the primary abstraction for fine-grained command decisions: subject, object, requested operation, and environment conditions evaluated against policy. NIST SP 800-207 reinforces resource-focused, explicit, continually informed access decisions rather than trust based on network location. These are design guidance, not claims that Glaux is a conformant federal zero-trust implementation.[^1][^5]

NIST SP 800-53 Rev. 5 Release 5.2.0 supplies control objectives used here: access enforcement and least privilege; separation of duties; security attributes; event selection; audit content; response to audit failure; time; audit protection; non-repudiation; retention; generation; and input validation. NIST SP 800-92 remains final practical log-management guidance; its initial Rev. 1 document is not substituted as final guidance. Numeric selections and organizational assignments remain profile/accreditation decisions.[^4][^6]

RFC 9700 is the current OAuth 2.0 security BCP and recommends audience restriction, least privilege, sender-constrained tokens, asymmetric client authentication where feasible, and end-to-end TLS. This report uses those principles but does not select OAuth. RFC 7662 shows that token activity, rights, client, validity, and revocation can be queried while warning that cached introspection has a freshness/security tradeoff. An introspection cache is therefore not indefinite DDIL authority.[^7]

RFC 9334 distinguishes evidence, verification, attestation result, and the relying party's application-specific decision. Glaux applies the same useful separation to gateway/device trust: attestation can inform trust but is not sufficient command authorization; freshness and policy still matter.[^8]

### 3.3 Accepted Glaux constraints

- IDR-SRV-029 requires PostgreSQL authority, scoped idempotency, atomic outbox/inbox, concurrency control, immutable request/evidence capture, and dispatch only after commit.
- IDR-SRV-031 requires one authoritative mutation pipeline and capability registry.
- IDR-SRV-032 separates authenticated principal, publisher instance, represented source, ownership, and trust.
- IDR-SRV-033 prevents simulators and tests from bypassing production safeguards.
- IDR-SRV-035 makes external event publication post-commit, at-least-once, replayable, and policy-filtered.
- IDR-SRV-036 fixes the public boundary at exactly nine CSAPI status codes, preserves multidimensional private orchestration state, forbids arrival-wins projection, distinguishes transport acknowledgement from target acceptance, mediates cancellation, and retains unknown/reconciliation privately.
- IDR-SRV-037 defines Feasibility as advisory and evidence/freshness-bound, prohibits treating it as reservation, and requires revalidation/re-evaluation before command effect.

### 3.4 Implementation and community evidence

OSH demonstrates useful per-resource-family/per-operation permissions and broad command abstractions, but its authentication integration is optional and the evidence does not establish safe command-control defaults, fine-grained parameter/target policy, or full audit. CS-Go demonstrates persistent command resources and test depth, but `v1.0.4` has no REST authentication and permissive CORS; deployment as-is would expose writes and commands. pygeoapi and SECD provide modular/API and interoperability lessons but no complete command-safety authority baseline. These are comparative warnings, not normative deficiencies. **[I]**

The current upstream master remained `3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f` on the September 15 check. Issue #58 remained open and unimplemented with the description asking whether transfer of control belongs in Part 2, a future part, requirements, or recommendations. No material disposition changed, so the upstream-history register remains Version 1.12. **[I/X]**

### 3.5 Controlled-source limitations

The controlled AEP package `AC/224(JCGISR)D(2026)0005`, dated April 27, 2026, SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`, was not redistributed. This report uses only accepted releasable project findings: accountable server-side tasking, traceable authority/source relationships, policy-aware dissemination, operational safety, and DDIL support. It does not infer national rules, roles, markings, or command delegations absent from releasable evidence.

## 4. Command Authorization, Safety, and Audit Extraction Methodology

### 4.1 Extraction frame

Each standards clause, accepted decision, implementation observation, and security source was evaluated against: decision type; lifecycle point; actor/source; target/ControlStream; required authority; feasibility dependency; safety input; policy/disclosure effect; audit evidence; event behavior; DDIL behavior; and test/handoff. Claims were then classified by the legend and reconciled against the accepted lifecycle rather than used to create a second lifecycle.

### 4.2 Evaluation criteria

Options were compared for standards alignment, least privilege, safety under race and stale state, audit integrity, policy non-disclosure, DDIL viability, gateway isolation, implementation complexity, deterministic testing, and interoperability. The selected baseline favors explicit typed decisions and durable evidence over hidden handler logic, and bounded local authority over either unconditional offline access or universal online dependency.

### 4.3 Threat and failure windows

The analysis covered stolen/replayed credentials; overbroad grants; confused deputy and represented-source substitution; target/control-stream enumeration; malicious or compromised gateways; stale metadata, policy, trust, feasibility, and target state; command revision substitution; parameter/unit ambiguity; duplicate delivery; cross-target replay; approval/override abuse; competing controllers; dispatch after revocation; forged/late status; audit suppression/modification; side-channel diagnostics; disconnect/rejoin conflicts; and simulator-to-live escape.

The decisive window is between admission and external effect. A correct admission decision can become invalid before dispatch because the command, target, authority, policy, safety bundle, operating mode, gateway, or time changes. That is why a durable admission decision is necessary but insufficient.

### 4.4 Limitations

This was architecture research, not penetration testing, hazard analysis for a specific actuator, or accreditation. No prototype was necessary to determine the decision boundaries. Exact safety rules, consequence classes, dual-approval thresholds, cryptographic formats, cache durations, audit retention, and fail-safe physical actions need deployment-specific analysis and downstream acceptance.

## 5. Command Decision Taxonomy

### 5.1 Ordered decisions and ownership

| Decision | Owner/interface | Typical result | Client visibility | Re-evaluation |
|---|---|---|---|---|
| Authentication | IDR-039 identity adapter | principal/session or failure | `401` where applicable | each request/session policy |
| Source/gateway trust | source registry/trust evaluator | active/degraded/suspended/revoked/test-only | normally concealed | request, report, dispatch, trust change |
| Discovery/read authorization | API policy enforcement | allow/conceal/deny | resource or policy-safe problem | every query/page/event subscription |
| Request/API authorization | API policy enforcement | allow/deny with obligations | generic `403`/concealed `404` | every operation |
| Structural/schema validation | codec/schema registry | valid/invalid | field-safe `400` | create/update/report |
| Semantic validation | domain validator | valid/invalid/conflict | safe `400`/`409`/`412` | create/update/dispatch if mutable context |
| Feasibility | evaluator from IDR-037 | YES/NO/CONDITIONAL/INDETERMINATE | authorized Feasibility result | when evidence invalidates/expires |
| Command/target authority | command policy engine | allow/deny/obligations | generic denial or terminal safe reason | admission, update, cancellation, dispatch |
| Releasability/policy | IDR-040 policy adapter | allow/redact/conceal/deny | filtered representation/problem | read/write/event/audit export |
| Safety/interlock | safety evaluator | allow/deny/approval/freshness required | usually generic | admission and immediately before dispatch; monitored during execution |
| Operator approval | protected approval workflow | approve/deny/expire/revoke | usually remains `PENDING`; safe final status if denied | after material input change or expiry |
| Target acceptance | authoritative executor | CSAPI `ACCEPTED`/`REJECTED` evidence | CommandStatus | when target responds |
| Dispatch authorization | orchestration gate | single-use ticket/deny | private | every effect attempt |
| Execution status/result | authorized reporter | exact CSAPI evidence | policy-filtered status/result | each assertion |
| Audit | audit writer/verifier | append evidence/failure | protected views only | every material decision/action |

### 5.2 Decision composition

The overall decision is not a Boolean stored on the Command. It is a set of immutable decision records and a current derived gate view. A positive decision is bound to its input manifest and obligations. The gate can be `blocked`, `awaiting_evidence`, `awaiting_approval`, `ready`, `dispatched`, or `reconciliation_required` privately; none is a new public status code.

Mandatory denials dominate. “Require approval” is not allow until a valid approval exists. “Require fresh evidence” is not allow until evidence is renewed. Advisory warnings may accompany allow only when the applicable rule explicitly permits warning disposition. Any uncertainty in a non-overrideable or high-consequence rule denies dispatch.

### 5.3 Client-facing error and lifecycle semantics

- Missing/invalid authentication: `401` plus the applicable challenge, without target details.
- Authenticated but unauthorized: `403`, or `404` when the visibility policy requires concealment.
- Malformed or schema-invalid request: `400`; it creates no Command.
- Current-state conflict: `409`; failed conditional mutation: `412`; a profile requiring a precondition may use `428`; rate enforcement uses `429` with safe retry information.
- A required gate unavailable before durable admission: `503` with `Retry-After` where meaningful; no effect resource is created.
- After an asynchronous Command is created, authoritative validation/policy/safety/target denial maps to `REJECTED` only when the standard meaning “will not execute at all” is certain. Technical execution failure maps to `FAILED` only with the accepted lifecycle semantics.
- Feasibility `NO` remains `COMPLETED` plus result `NO`, not an authorization or safety error.
- Dispatch uncertainty does not invent a public error/status. Preserve last valid status and enter protected reconciliation.

The server should expose stable, low-detail problem types/reason codes and correlation IDs. Full rule, grant, source, gateway, operator, topology, and state evidence stays in protected audit. Bodies, status choices, list counts, cursor behavior, event gaps, and response timing must be tested for disclosure oracles.

## 6. Command Authority Model Findings

### 6.1 CommandAuthorityGrant

The internal grant should contain at least:

- immutable grant ID, version, issuer/authorizing authority, issue time, validity, status, revocation epoch, and supersession link;
- subject principal/service/client and permitted represented organization/mission identity;
- actions such as `feasibility.submit/read`, `command.submit/read/update/cancel`, `command.approve`, `command.override`, `command.dispatch`, `status.append/read`, `result.append/read`, and protected audit actions;
- target System set or selector, ControlStream ID and contract/version, command type and allowed parameter envelope including units;
- mission/deployment, geospatial and temporal scope, operating mode and consequence class;
- allowed execution path/gateway/audience, rate/repeat ceilings, offline eligibility, and obligations such as approval or stronger authentication;
- policy/safety profile references, delegation depth and constraints, and provenance/signature or authoritative registry reference.

The server should resolve broad organizational roles into this bounded decision context. It must never copy an untrusted client `sender` string into the principal field. The public `sender`, if emitted, is a policy-safe identifier derived from authenticated provenance.

### 6.2 Roles and separation of duties

| Role/capability | Allowed responsibility | Prohibited shortcut |
|---|---|---|
| Requester | submit/read within grant | self-approve merely because it submitted |
| Feasibility requester/evaluator | request or assert bounded feasibility | reserve authority or approve command |
| Command approver | approve specified command/revision/rules | alter payload or policy silently |
| Override authority | override explicitly eligible rules within bounded scope | override authentication or non-overrideable interlock |
| Cancellation requester/authority | request or authoritatively decide cancellation per profile | equate request with confirmed cessation |
| Dispatcher service | consume a valid dispatch ticket | choose a new target/action or reuse ticket |
| Gateway relay | deliver/report for named targets and protocols | become controller because it can reach target |
| Receiving executor | accept/reject/execute and report per registered contract | assert unrelated resource status/result |
| Status/result reporter | append exact evidence types for owned targets | mutate Command identity/history |
| Safety/policy administrator | publish reviewed versioned rules | retroactively alter recorded decision inputs |
| Auditor/records manager | view/export/protect governed evidence | issue commands through audit privilege |

High-consequence profiles should separate requester, approver, policy administrator, and override authority and may require two-person approval. Exact thresholds are deployment policy. Service identities need narrower rights than the humans or organizations they represent, preventing a confused-deputy service from applying one caller's authority to another request.

### 6.3 Control authority and transfer

A `CommandAuthorityGrant` answers who may ask; a private `ControlAuthorityLease` answers which controller is currently entitled to direct a target/ControlStream where exclusivity is required. The lease contains holder, scope, issuer, start/expiry, priority/preemption rule, epoch/fencing value, and state. It is not required for every benign or naturally concurrent ControlStream, but the capability registry must state the concurrency/control mode.

The first profile should support `exclusive`, `coordinated`, and `concurrent-safe` modes. Exclusive mode serializes dispatch by lease epoch. Transfer is explicit: revoke/expire the previous lease, resolve in-flight commands, then grant a higher epoch. Automatic network-location priority, latest-write ownership, or silent seizure is prohibited. Preemption, emergency authority, rejoining controllers, and partition recovery remain profile choices. This bounded internal mechanism addresses safety while upstream #58 remains unresolved and must not be advertised as OGC behavior.

### 6.4 Approval and override records

An approval binds approver identity/authority, command ID and exact revision/digest, target and ControlStream versions, approving decision, rule/policy references, scope, time, expiry, conditions, reason, and revocation state. A changed payload, target, contract, policy, safety rule, or expired evidence invalidates it unless the approval policy explicitly defines an unaffected subset.

An override additionally identifies the exact failed rule(s), whether each is overrideable, consequence class, justification, independent approver(s), short validity, permitted attempts, and compensating monitoring. Overrides are never hidden flags. They create security/operator events and immutable audit evidence, and are rejected if used outside their audience, target, command revision, or time.

## 7. Safety Rule Model Findings

### 7.1 Rule sources and precedence

| Rule source | Example | Authority and treatment |
|---|---|---|
| Non-overrideable platform invariant | never dispatch test credential to live actuator | compiled/profiled, change-controlled; deny on failure |
| Versioned administrative safety bundle | maximum slew/rate, conflict rule | authoritative local configuration; signed/versioned |
| SWE/ControlStream schema | type, choice, units, allowed range | required validation input; not sufficient safety |
| SensorML/System metadata | capability, mode, characteristic, interface | versioned evidence; trust/freshness assessed |
| Mission/deployment policy | area/time/task constraints | policy authority; purpose and releasability bound |
| Dynamic target/gateway state | health, mode, active command, link state | authoritative reporter and freshness required |
| Environmental evidence | weather, airspace, hazard zone | source quality/time/spatial scope required |
| Operator decision | explicit approval/override | independently authorized and narrowly bound |
| DDIL operating profile | offline command class/staleness ceiling | local trusted bundle; conservative expiry |

Precedence should be deterministic: non-overrideable invariant; applicable legal/organizational policy; current exclusive-control rule; mandatory safety rules; approval/override obligations; advisory warnings. A lower-precedence allow cannot defeat a higher-precedence deny.

### 7.2 Rule representation

Each rule should have stable ID and version, issuer/source, effective and validity intervals, applicable target/ControlStream/command/parameter/mode/mission scope, consequence severity, required evidence and freshness, predicate, outcome, overrideability, approval class, diagnostic classification, fail-safe action, and test-vector references. The evaluator records the rule digest and evidence manifest, not merely `allowed=true`.

Safety categories include allowed/prohibited command types; target availability and mode; parameter/unit bounds; temporal validity; sequencing and dependencies; conflict with active/in-flight commands; rate/repeat and replay limits; geospatial/geofence bounds; environmental constraints; mission/deployment constraints; human approval; DDIL restrictions; and simulation/test isolation.

### 7.3 Evaluation points

- Before creation: cheap, deterministic checks that can reject without creating a resource—size, format, route, identity, obvious grant absence, schema, immutable target/contract, test/live isolation, and non-sensitive static bounds.
- During admission: complete command/target authority, policy, required feasibility, safety and approval planning; asynchronous processing may retain `PENDING` while protected gates run.
- On update/cancellation: authorize the exact action; re-evaluate every affected rule and invalidate prior approval/ticket where necessary.
- Immediately before dispatch: reload current command revision, grant/lease, policy, safety bundle, target/gateway trust/state, feasibility freshness, time, approval/override, and attempt fence in one coherent decision snapshot.
- During execution: monitor rules expressly designated continuous; if violated, take the configured fail-safe action, which may be alert, inhibit further dispatch, request cancellation, or invoke an independently authorized emergency action. Never assume Glaux can physically stop an actuator absent a verified contract.

### 7.4 Safety decision contract

The internal result should be one of `ALLOW`, `DENY`, `REQUIRE_APPROVAL`, or `REQUIRE_FRESH_EVIDENCE`, plus obligations and safe reason classification. `ALLOW_WITH_WARNING` may exist only for advisory rules and is represented as `ALLOW` plus warning evidence; it is never used to weaken a mandatory failure. The evaluator must be deterministic for a fixed input manifest, time model, and bundle set.

### 7.5 Safety is not validation

An encoded value of 20 degrees may be valid under a Quantity schema and within an advertised `AllowedValues` range, yet unsafe because the platform is stowed, another command is executing, the target is in a prohibited area, a mission window closed, or the gateway state is stale. Conversely, an otherwise safe intent with the wrong unit or malformed choice never reaches safety evaluation. Glaux should preserve the order and record both outcomes.

## 8. Command Lifecycle Authorization and Safety Hook Findings

### 8.1 Admission-to-effect sequence

The canonical sequence is:

1. apply visibility policy before revealing the ControlStream or schema;
2. authenticate principal/client/service and resolve represented source;
3. check request/API permission, rate/size constraints, and source trust;
4. resolve immutable target, ControlStream contract/version, and command identity/revision;
5. decode and validate syntax, schema, semantics, units, references, time, and preconditions;
6. evaluate command/target authority and control lease;
7. evaluate policy/releasability and required Feasibility evidence;
8. evaluate safety rules and establish any approval/override obligations;
9. atomically persist the Command/admission state, decisions, audit, idempotency, and outbox/work where the accepted sync/async contract permits creation;
10. obtain target acceptance/scheduling evidence as defined by the adapter contract;
11. immediately before each effect attempt, re-evaluate all invalidatable decisions and atomically issue/consume a single-use dispatch ticket;
12. append authorized target/gateway status and result evidence and monitor continuous rules;
13. publish only committed, policy-safe lifecycle events; retain complete protected audit.

### 8.2 Public lifecycle mapping

`PENDING` can encompass protected validation, feasibility, safety, and approval work but must not reveal which gate is pending unless policy allows. A known pre-execution denial becomes `REJECTED` with a safe message. `ACCEPTED` requires receiving-system acceptance, not merely Glaux authorization. `SCHEDULED` requires an effective target schedule. `UPDATED` requires authorization and re-gating of the new revision. `CANCELED` requires the configured authoritative cancellation semantics and an authorized user; a cancellation request alone remains private. `EXECUTING`, `COMPLETED`, and `FAILED` require authorized executor evidence and valid transitions.

### 8.3 Re-evaluation triggers

Re-evaluate on command update; target/ControlStream contract revision; grant/lease/policy/safety/trust change; approval/override expiry or revocation; feasibility expiry/invalidation; target/gateway state change; execution-window boundary; queue delay; reconnect; failover; changed route; retry after ambiguous delivery; and before every new external effect attempt. Re-evaluation appends a new decision; it never overwrites the admission record.

### 8.4 Cancellation and emergency action

Cancellation has separate requester and decision authorities. Glaux validates target, command state/revision, cancellation scope, cutoff, and idempotency; records a private request; and dispatches cancellation only under a valid ticket. It emits `CANCELED` only when the registered authority can establish the published meaning. If completion wins a serialized race, the cancellation is refused/recorded rather than rewriting terminal evidence.

Emergency stop or abort should be modeled as its own high-priority command/action with explicit authority, delivery semantics, target support, and audit—not as an unaudited bypass. A safety monitor may invoke it only with a pre-authorized bounded grant.

## 9. Feasibility, Validation, Source Trust, Gateway, and Dispatch Integration Findings

### 9.1 Feasibility integration

Feasibility is advisory evidence. An authorization or safety rule may require a Feasibility result with specified evaluator, decision class, target and command digest, context fingerprint, validity, freshness, or constraints. It must re-evaluate if any bound input changes. `YES` cannot reserve capacity or replace authority; `NO` cannot prove the caller is unauthorized; `INDETERMINATE` may require fresh evidence, approval, or denial according to consequence profile.

The combinations are deliberate:

| Feasible | Authorized | Safe | Disposition |
|---|---|---|---|
| yes | no | unknown | deny without exposing feasibility beyond read policy |
| no | yes | unknown | do not dispatch; reject or await changed state per lifecycle certainty |
| yes | yes | no | deny; safe generic reason; audit exact rule |
| stale/unknown | yes | otherwise plausible | refresh, require approval where explicitly allowed, or deny; never silently assume yes |
| yes | yes | yes | eligible for approval/target acceptance/final dispatch check, not guaranteed execution |

### 9.2 Validation integration

Validation should precede expensive or capability-revealing evaluation where possible. The server records the exact schema/semantic registry versions and normalized parameter digest used. It performs no silent repair or unit conversion. A rule may consume normalized values only if the original and normalized evidence are durably linked and the conversion is deterministic and accepted by the semantic baseline.

### 9.3 Source and gateway trust

Identity, represented authority, and trust remain separate. A gateway may be authenticated and healthy but authorized only to relay, or trusted to report `EXECUTING` but not `COMPLETED`, or scoped to one target/ControlStream/mission window. Trust states should include active, degraded, suspended, revoked, test-only, and simulated. Degraded state applies explicit restrictions; suspended/revoked blocks new effect attempts and assertions; history remains intact. Treatment of in-flight commands is a versioned policy/safety decision.

Gateway assertions carry authenticated gateway identity, represented target/executor identity, route/session/epoch/sequence, target time and server receive time, command/revision/attempt correlation, and raw-artifact digest/reference. The gateway's report is evidence evaluated under its grant; it is not automatically truth.

### 9.4 Dispatch ticket

The private ticket binds:

- command ID, immutable revision and normalized digest;
- target and ControlStream contract/version;
- selected adapter/gateway/audience and authority/lease epoch;
- authorization, policy, safety, feasibility, trust, approval and override decision IDs/digests;
- issue/expiry, maximum attempts, nonce, and queue/worker fence;
- consequence/offline class and required acknowledgement semantics.

It is consumed once by the named dispatcher and cannot alter payload, target, route, or audience. A database-backed same-process dispatcher can use an authoritative row rather than a portable token. A ticket crossing a process or gateway boundary must be integrity-protected and audience-bound. The final encoding belongs to IDR-SRV-039/045.

### 9.5 Direct, brokered, manual, and simulated paths

Every adapter implements the same gate and audit contract. Direct connectivity does not skip dispatch authorization. A broker's authentication, topic ACL, QoS, and acknowledgement do not establish command authority or target acceptance. Manual execution requires a bound work item, operator identity, decision evidence, and explicit status-reporting authority. Simulation/test credentials, targets, namespaces, policies, and event channels are isolated; test-only trust can never produce a live dispatch ticket.

## 10. Policy, Releasability, and Diagnostic-Redaction Findings

### 10.1 Sensitive command information

Potentially sensitive data includes existence of command-capable Systems and ControlStreams; schemas and allowed ranges; target identities and topology; mission/deployment links; live/async state; parameters, requested/execution times, status and results; Feasibility constraints and alternatives; active commands; gateway/source/operator identities; grant and lease scope; safety rules and thresholds; denial reasons; audit records; and aggregate rates. Metadata can reveal capability or intent even when payloads are hidden.

### 10.2 Policy enforcement points

Policy applies before discovery, schema retrieval, collections/counts, links, Feasibility creation/read, Command creation/read/update/cancel, status/result access, event subscription/replay, audit search/export, and observability output. Page cursors, ETags, caches, indexes, and events are bound to the policy view. Redaction must preserve protocol validity without fabricating operational meaning.

### 10.3 Non-oracular behavior

For concealed objects, route, item, and relationship behavior should not distinguish “exists but forbidden” from absent. Stable generic reasons should distinguish only what an authorized caller needs to correct. Detail levels can be:

1. subject-safe: correlation ID, generic category, retryability, and support path;
2. authorized operator: bounded reason and corrective action without sensitive rule internals;
3. security/auditor: exact grant, rule, evidence, source, and policy versions;
4. policy administrator: configuration diagnosis, separately authorized and audited.

Timing, result counts, pagination gaps, event sequence gaps, cache behavior, and rate-limit buckets require the same policy partitioning. Audit export and redaction actions are themselves audited. Redaction creates a derived protected view; it never mutates original evidence.

### 10.4 Handoff to IDR-SRV-040

IDR-SRV-040 must choose the marking/label model, policy combination, cross-boundary guards, discovery concealment rules, aggregation thresholds, obligations, derived-object labeling, downgrade/redaction authority, and policy-view cache/cursor behavior. This report fixes only the command-specific enforcement points and required non-disclosure properties.

## 11. Audit Record Model Findings

### 11.1 Event types

At minimum, append events for request receipt/replay decision; authentication/source resolution reference; validation result; Feasibility request/result use; authorization decision; control-lease decision; policy/releasability decision; safety decision; approval request/decision/revocation/expiry; override request/decision/use; Command creation/update; target acceptance/rejection; dispatch authorization/attempt/delivery/ambiguity; gateway acknowledgement; status/result assertion acceptance/rejection; cancellation request/decision/attempt/outcome; timeout/expiry; monitoring/interlock action; reconciliation/conflict; redaction; audit export; and audit-integrity/failure incidents.

### 11.2 Common record fields

| Field group | Required content |
|---|---|
| Identity | immutable audit event ID; tenant/security domain; event type/schema version |
| Time/place | occurred, observed, and recorded times; time source/confidence; service/node/region |
| Actor | principal, client/service/session, represented source/authority, gateway, executor, operator as applicable |
| Object | Command/Feasibility ID and revision, target System, ControlStream contract/version, status/result IDs |
| Causality | request, idempotency-key digest, correlation, causation, trace, workflow/attempt/fence IDs |
| Decision | decision point, outcome, obligations, safe/internal reason codes, previous/new state |
| Inputs | request/normalized digests, artifact references, policy/safety/grant/lease/trust/feasibility/target-state versions and freshness |
| Effect | adapter/audience, dispatch ticket digest, attempt, acknowledgement class, result reference |
| Governance | classification/releasability/redaction labels, retention/legal-hold class, access tier |
| Integrity | producer/software version, signature/MAC/hash-chain/checkpoint fields, prior-event relation, verification status |

Do not log raw access tokens, private keys, passwords, unneeded credentials, or unrestricted command payloads. Store content-addressed protected artifacts where exact payload evidence is required, with the audit event carrying digest and governed reference. Minimize personal information while retaining accountable identity.

### 11.3 Immutability and correction

All event identity, actor/object binding, time, outcome, decision inputs, and integrity fields are immutable after append. Corrections, late evidence, reclassification, or legal actions append linked events and may change a projection or access view, never the original record. Database append permissions, separate audit-writer roles, hash chaining/checkpoints, signed exports, protected backups, and immutable/WORM archives are layered controls. A hash chain in the same database does not protect against a fully privileged attacker who can rewrite the chain; stronger claims require externally protected checkpoints or storage.

### 11.4 Visibility, search, export, and retention

The public Command/status/result API is not an audit endpoint. Subject-safe receipts may expose a request/correlation identifier; protected operator views expose bounded decision history; security/auditor views expose authorized full evidence; records administrators control retention/export. Searches and exports are policy-filtered, paged with policy-bound cursors, rate-limited, and audited.

Retention uses classes from IDR-SRV-030 and organization-defined periods from IDR-SRV-041. Command domain retention, audit retention, raw artifact retention, metrics retention, legal hold, and cryptographic-key lifetime are independent. Deleting/tombstoning a public resource does not automatically erase required accountability evidence.

### 11.5 Audit failure

For authoritative command creation, approval/override, dispatch authorization, state transition, or cancellation effect, the domain change and minimum audit evidence must commit atomically. If this cannot happen, fail closed before accepting/dispatching. A durable, access-controlled local audit buffer may satisfy the same invariant in DDIL mode; an in-memory logger cannot. During an already executing action, audit failure triggers a protected incident and the configured fail-safe/monitoring response rather than inventing a CSAPI outcome.

## 12. Transaction, Idempotency, Event Publication, Persistence, Retention, and Synchronization Findings

### 12.1 Transaction boundaries

Asynchronous admission atomically writes Command, initial status, command revision, normalized/request artifact references, decision snapshot, audit, idempotency result, work, and outbox. Synchronous private staging atomically writes the hidden work/decision/audit/idempotency record; public publication occurs atomically with its terminal status/result as accepted in IDR-SRV-037. No external effect occurs before the durable pre-effect boundary.

The pre-dispatch transaction locks/checks current revision and orchestration state, evaluates or verifies current gate inputs, appends the decision/audit, and creates/consumes the fenced ticket/attempt. The network call occurs after commit. The completion transaction validates reporter authority, attempt/revision/sequence, legal transition, and result schema, then appends domain evidence, audit, projection, and outbox atomically.

### 12.2 Idempotency and retries

A same-scope/same-key/same-fingerprint retry returns the same logical outcome. It does not create a second Command or duplicate the primary decision events. Security-significant replay observations may append a separate rate-controlled `request_replay_observed` event. Same key with different fingerprint conflicts and is audited. Dispatch retries reuse command/revision/attempt identity according to target dedupe/query capability; they never mint a new logical intent silently.

Every material re-evaluation, decision change, retry attempt, and late/conflicting report is new evidence. Idempotency prevents duplicate effects and resources; it must not erase the history of distinct security events.

### 12.3 Events and observability

Only committed, policy-safe resource lifecycle events enter the external publication log. Internal authorization/safety decisions are audit-only by default. Authorized operator/security channels may carry denial spikes, approval/override activity, revoked-gateway attempts, dispatch ambiguity, interlock activation, audit failure, or reconciliation incidents with restricted detail. Draft Part 3 tasking publication remains disabled under the accepted IDR-SRV-035 profile.

Metrics should cover decision latency and outcome by coarse class; safety rejections; approval wait/expiry; overrides; dispatch failures/ambiguities; gateway failures; cancellation races; stale-policy/feasibility/trust blocks; DDIL decisions; audit append latency/failure/backlog; and redaction. Labels must be bounded and policy-safe—never raw target, principal, command, rule, or credential values in general metrics.

### 12.4 Persistence, retention, and synchronization

PostgreSQL remains the first authoritative store. Logical append-only tables and narrowly privileged functions should separate audit/event writes from projections. Outbox publication follows commit. Archive/checkpoint/export strengthens long-term integrity. IDR-SRV-041/049 will choose backup, restore, verification, and archival mechanics.

Disconnected nodes preserve globally unique event/decision/attempt IDs, issuer/node epochs, causal links, original times plus receive times, and integrity checkpoints. Reconnection imports append evidence, verifies authorization/integrity, deduplicates by identity, and quarantines conflicts. It never applies arrival-wins or rewrites earlier evidence. Final replication/conflict algorithms belong to IDR-SRV-043.

## 13. DDIL, Cached Authorization, Stale Policy, Stale Feasibility, and Tactical-Edge Implications

### 13.1 Local authorization bundle

A locally usable bundle should be signed or otherwise integrity-protected and include bundle ID/version, issuer and trust chain, node/audience, subject/role/grant references, target/ControlStream/action/parameter scope, policy and safety versions, issue/not-before/expiry, maximum staleness, revocation epoch/counter, time-confidence requirements, offline operating classes, delegation restrictions, and fail-safe behavior. It is stored encrypted/protected as required and activated only after verification.

### 13.2 Command operating classes

| Class | DDIL behavior | Examples/constraints |
|---|---|---|
| Online-required | no admission/dispatch without current remote decision/evidence | dynamic high-consequence or cross-boundary commands |
| Bounded local authority | local decision within signed scope, time, state and staleness limits | preplanned target/actions with current local safety evidence |
| Pre-authorized emergency | narrowly scoped, short-lived, independently approved, strongly audited | explicitly defined protective action; never generic override |
| Simulation/test-only | local test plane only; cryptographically/configurationally isolated | deterministic simulator and conformance fixtures |

This class is part of the versioned capability/safety profile, not chosen by the caller.

### 13.3 Staleness and time

Cached policy or introspection results have explicit expiry and invalidation semantics. Unknown revocation, insufficient trusted time, missing required environmental/target state, or an expired bundle denies high-consequence dispatch. Lower-consequence bounded-local commands may proceed only when the approved offline profile says the available evidence remains within maximum staleness. No `last known allow` default exists.

A Feasibility result that expires or loses a bound context is not refreshed by an authorization decision. A Command queued through a disconnect is revalidated and re-authorized before dispatch. A gateway reconnect does not regain authority until its trust, epoch, pending attempts, and control lease reconcile.

### 13.4 Local audit and reconnect

The edge must durably append the same minimum records as the connected server before effect, protect them from ordinary operators and applications, maintain sequence/causal and integrity checkpoints, and monitor storage capacity. On reconnect, it uploads evidence without renumbering or replacing local events. The receiver verifies signatures/digests, policy context and duplicates, then accepts, quarantines, or records conflict. Central disagreement does not retroactively claim the local act never occurred.

### 13.5 Tactical safety posture

Limited bandwidth justifies compact bundles, digests, deltas, and delayed export; it does not justify unauthenticated tasking, indefinite cached authority, silent audit loss, stale feasibility as safety evidence, or merging test/live planes. Numeric staleness, storage, clock, and consequence thresholds remain deployment inputs for IDR-SRV-042/046.

## 14. Fixture, Conformance, Security Testing, Performance, and Interoperability Test Implications

### 14.1 Required fixture corpus

Build immutable fixtures for authorized command; unauthorized visible and concealed target; feasible-but-unauthorized; authorized-but-infeasible; feasible/authorized-but-unsafe; stale Feasibility; approval required/approved/denied/expired/revoked; eligible and ineligible override; non-overrideable interlock; conflicting control lease; transfer/preemption; command revision after approval; policy/rule/trust change before dispatch; direct/gateway/broker/manual/simulated paths; revoked and degraded gateway; forged/duplicate/out-of-order status; cancellation authorized/unauthorized/too-late/racing completion; retry with same/different fingerprint; audit append failure; tamper/correction/export; DDIL valid/expired/unknown-time bundles; reconnect duplicate/conflict; and diagnostic side channels.

Each fixture records exact standards/profile version, identities and represented authorities, grants/leases, policy/safety bundles, target/ControlStream versions, request/normalized digests, Feasibility context, expected HTTP/public status, private decision path, audit events, published events, and permitted diagnostics.

### 14.2 Conformance and profile tests

OGC conformance tests verify that protected deployment behavior does not corrupt the CSAPI resource contract: exact routes, schemas, nine statuses, creation/update/cancellation semantics, status/result ownership, and policy-filtered navigation. Supplemental Glaux profile tests verify gate order, `PENDING`/`REJECTED` mapping, cancellation authority, non-public internal states, dispatch-after-commit, audit atomicity, and no Part 3 inbound tasking claim.

Authorization cannot excuse nonconformant behavior for an authorized fixture. Conversely, an official ATS may require a configured authorized test principal/target; the harness must distinguish a security denial from failure of the standard operation.

### 14.3 Security tests

Test horizontal/vertical privilege escalation; target and parameter-envelope escape; represented-source substitution; confused deputy; cross-target/revision/audience ticket replay; stolen bearer replay protections selected by IDR-SRV-039; lease fencing; approval/override forgery and reuse; rule downgrade; stale cache/revocation; gateway impersonation; status/result writer escalation; broker-topic bypass; simulator-to-live escape; payload/diagnostic injection; enumeration through status/body/time/count/cursor/event behavior; audit suppression/modification/deletion; log secret leakage; export abuse; rate exhaustion; and fail-closed behavior.

Property-based/state-machine tests should generate lifecycle/gate interleavings. Fault injection covers crash before/after each commit and send, partition, clock uncertainty, audit storage exhaustion, evaluator timeout, policy update, key rotation, target reset, duplicate delivery, and late terminal evidence.

### 14.4 Performance tests

Measure p50/p95/p99 and tail decision latency by rule/bundle size and cache state; pre-dispatch recheck latency; transactional audit overhead; approval backlog; outbox lag; concurrent target/lease contention; status/report verification; DDIL local verification; reconnect audit ingestion; and policy-filtered query/event cost. Include saturation and failure behavior. Performance targets cannot justify bypass or fail-open; if a gate exceeds budget, narrow/compile/cache policy safely or restrict the capability.

### 14.5 Interoperability tests

Use CSAPI Explorer, OS4CSAPI, web/mobile clients, publisher/gateway/simulator components, CS-Go, OSH where applicable, and external CSAPI clients. Verify standards-visible behavior with authorized credentials; predictable 401/403/404/400/409/412/429/503 handling; generic denial messages; Command/status/result traversal; cancellation; async polling/events; duplicate retry; and unsupported capability discovery. Gateway tests require identity/authority separation and acknowledgement mapping. Clients must not require private approval, ticket, lease, or audit records.

## 15. Downstream Topic Handoff Matrix

| Topic | Receives from IDR-SRV-038 | Must decide/verify |
|---|---|---|
| IDR-SRV-039 | command actions/attributes, error/disclosure requirements, credential anti-replay needs | authentication, sessions/tokens, API threat model, credential lifecycle |
| IDR-SRV-039A | explicit command PEP/PDP and re-evaluation points | complete zero-trust components, trust algorithm and topology |
| IDR-SRV-040 | sensitive fields and enforcement points | labels, cross-boundary policy, redaction/downgrade and aggregation |
| IDR-SRV-041 | event catalog, fields, atomicity, integrity tiers | general audit schema/service, retention, archive, search/export and verification |
| IDR-SRV-042 | signed cached bundles and command classes | numeric staleness/time/storage rules and complete DDIL semantics |
| IDR-SRV-043 | event identity/causality, lease epochs, reconnect constraints | synchronization protocol, conflict/quarantine/resolution |
| IDR-SRV-044/045 | typed decision ports and dispatch-ticket contract | Rust crates, process/service decomposition, failure isolation |
| IDR-SRV-046/047 | deployment-specific authority/safety/audit dependencies | topology, secrets, keys, environment configuration and hardening |
| IDR-SRV-048 | protected event and safe metric catalog | telemetry pipeline, health, alerting and cardinality |
| IDR-SRV-050/051 | standards/profile split and decision traceability | harness, requirement IDs, evidence reporting |
| IDR-SRV-052/053 | state-machine/fault/fixture requirements | Rust test architecture and governed corpus |
| IDR-SRV-054 | non-bypass performance dimensions | workloads, thresholds, saturation and resilience tests |
| IDR-SRV-055 | full attack and command-control scenarios | security test plan, tools, gates and evidence |
| IDR-SRV-056 | authorized public contract and gateway roles | external client/gateway matrix and demonstrations |
| IDR-SRV-057 | accepted decision summary and residual risks | final synthesis and implementation sequencing |

## 16. Recommendations

1. Adopt a deny-by-default, subject/object/action/environment command policy model; use roles only as inputs.
2. Add internal versioned `CommandAuthorityGrant`, conditional `ControlAuthorityLease`, `ApprovalDecision`, `OverrideDecision`, `GateDecision`, and single-use `DispatchTicket` concepts; expose none as CSAPI resources by default.
3. Define distinct grants for feasibility, submission, update, cancellation, approval, override, dispatch, status, result, read, audit and administration.
4. Enforce protected discovery and schema visibility before revealing command affordances.
5. Layer syntax/schema, semantic validation, authority, policy, feasibility, safety, approval, target acceptance and dispatch authorization without collapsing results.
6. Evaluate command authority and safety at admission and immediately before every effect attempt; invalidate decisions on any bound-input change.
7. Treat SensorML/SWE constraints as versioned inputs, never the whole safety policy; prohibit silent repair/conversion.
8. Define deterministic, versioned rules with provenance, evidence freshness, consequence, overrideability, diagnostic class and tests.
9. Permit overrides only for explicitly eligible rules with independent bounded authority; never override identity or non-overrideable interlocks.
10. Use explicit control leases only where a ControlStream requires exclusive/coordinated control; prohibit last-writer-wins transfer or automatic seizure.
11. Apply the same gate/audit contract to direct, gateway, broker, manual and simulator adapters; never promote transport ACK to domain acceptance.
12. Authorize status/result reporters independently and validate command/revision/attempt/sequence before append.
13. Make audit append-oriented, minimally sufficient, protected and atomic with authoritative command decisions/transitions; fail closed when required evidence cannot be durable.
14. Keep external lifecycle events post-commit and policy-filtered; keep detailed security/safety decisions audit-only unless an authorized channel is explicitly configured.
15. Support DDIL through locally verifiable bounded bundles and explicit command classes; deny on expired/unknown mandatory evidence and recheck before dispatch/reconnect.
16. Implement the fixture and security matrix before enabling live tasking; default live actuation and inbound draft Part 3 tasking to disabled.

Suggested post-research implementation sequence is: typed decision context and event schemas; grant/lease/rule stores and evaluators; transactional audit/outbox integration; approval/override and ticket/fence logic; simulator-only lifecycle harness; protected discovery/read policy; direct/manual adapter; gateway/broker adapters; DDIL bundles and sync; then controlled live-target and interoperability testing. This sequence is guidance, not implementation authorization.

## 17. Risks, Constraints, and Open Questions

| Risk/constraint/open question | Disposition |
|---|---|
| CSAPI does not specify command roles, safety, audit, or transfer | label Glaux mechanisms as profile/private; preserve public standard contract |
| Upstream control-transfer issue remains unresolved | internal conditional lease; no OGC claim; monitor #58 |
| Metadata or diagnostics reveal capability/intent | policy before discovery; stable redacted responses; oracle tests |
| Overbroad role or route permission grants physical control | evaluate exact target/stream/type/parameters/time/mission/path |
| Admission decision becomes stale before effect | single-use short-lived ticket and immediate re-evaluation |
| SWE-valid command is operationally unsafe | layered dynamic rules and explicit evidence freshness |
| Gateway is trusted beyond represented scope | separate identity/trust/authority; exact reporter and relay grants |
| Broker acknowledgement is promoted | adapter contract and tests prohibit promotion |
| Multiple controllers conflict during partition | lease epoch/fence; no last-writer-wins; final policy in 042/043 |
| Operator approval becomes a permanent blanket grant | bind exact revision/context/time; invalidate on change |
| Override becomes hidden bypass | explicit eligible rules, independent authority, expiry, audit and alert |
| Cancellation is claimed before physical certainty | private request; `CANCELED` only under authoritative profile semantics |
| Audit is unavailable at effect boundary | durable local/central atomic evidence or fail closed |
| Privileged administrator can rewrite database and chain | separated roles plus protected external checkpoints/archive; finalize in 041 |
| Audit stores secrets or excessive personal data | digests/references, field minimization, protected artifacts and views |
| Always-online authorization blocks tactical use | bounded locally verifiable bundles and command classes |
| Cached authority silently survives revocation/expiry | validity/revocation epochs/max staleness/time confidence; fail closed by class |
| Stale feasibility is treated as safety | explicit evidence binding and pre-dispatch invalidation |
| Audit/event sync uses arrival-wins | immutable IDs/causality/epochs; verify/dedupe/quarantine |
| Security gates make latency unacceptable | compile/cache safely and benchmark; never bypass; restrict capability if needed |
| Exact consequence classes, rule catalog and dual-control thresholds unknown | deployment/hazard/policy decisions, not invented here |
| Exact identity/token, cryptographic bundle and ticket formats unknown | IDR-039/039A/044/047 |
| Exact audit retention, SIEM and archive products unknown | IDR-030/041/046/049 |

None prevents acceptance of this command-specific architecture. They prohibit enabling live command mutation until downstream security, policy, DDIL, implementation, and test decisions are accepted and demonstrated.

## 18. Validation Against This Plan's Success Criteria

| Success criterion | Result | Evidence |
|---|---|---|
| Distinguish authentication, authorization, trust, feasibility, validation, safety, acceptance, dispatch, execution and audit | Met | §§2, 5, 8; Appendix A |
| Document command authority, target/stream/type/parameter and gateway scope | Met | §§6, 9; Appendix A |
| Document safety categories, lifecycle hooks, repeats, approval, override, DDIL and events | Met | §§7-8, 12-13 |
| Define audit types, fields, immutability, links, retention, redaction, search/export, atomicity, idempotency and sync | Met | §§11-13; Appendix B |
| Define error, diagnostic, policy, fixture, conformance, security, performance and interoperability implications | Met | §§5, 10, 14 |
| Incorporate implementation/community findings non-normatively | Met | §3.4 |
| Produce bounded, decision-usable recommendations | Met | §§1, 16-17 |
| Make downstream handoffs explicit | Met | §15 |
| Provide explicit reproducible references | Met | §19; Appendix C |

Research phases 1 through 6, deliverable drafting, and author review are complete. Plan-owner acceptance remains deliberately unchecked while this report is **In Review**. IDR-SRV-039 is not authorized by this report.

## 19. References

### 19.1 Standards and official technical sources

1. [OGC API - Connected Systems - Part 1: Feature Resources, OGC 23-001, Version 1.0](https://docs.ogc.org/is/23-001/23-001.html).
2. [OGC API - Connected Systems - Part 2: Dynamic Data, OGC 23-002, Version 1.0](https://docs.ogc.org/is/23-002/23-002.html).
3. [Official CSAPI repository tag `v1.0.0`](https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0) and [current upstream issue #58](https://github.com/opengeospatial/ogcapi-connected-systems/issues/58).
4. [OGC SensorML Encoding Standard, OGC 23-000, Version 3.0](https://docs.ogc.org/is/23-000/23-000.html).
5. [OGC SWE Common Data Model Encoding Standard, OGC 24-014, Version 3.0](https://docs.ogc.org/is/24-014/24-014.html).
6. [W3C/OGC Semantic Sensor Network Ontology](https://www.w3.org/TR/vocab-ssn/).
7. [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110) and [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457).
8. [NIST SP 800-162: Guide to Attribute Based Access Control](https://csrc.nist.gov/pubs/sp/800/162/upd2/final).
9. [NIST SP 800-53 Rev. 5, Release 5.2.0](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final).
10. [NIST SP 800-207: Zero Trust Architecture](https://csrc.nist.gov/pubs/sp/800/207/final).
11. [NIST SP 800-92: Guide to Computer Security Log Management](https://csrc.nist.gov/pubs/sp/800/92/final).
12. [RFC 7662: OAuth 2.0 Token Introspection](https://www.rfc-editor.org/rfc/rfc7662.html).
13. [RFC 9334: RATS Architecture](https://www.rfc-editor.org/rfc/rfc9334.html).
14. [RFC 9700 / BCP 240: Best Current Practice for OAuth 2.0 Security](https://www.rfc-editor.org/rfc/rfc9700.html).
15. [W3C PROV Overview](https://www.w3.org/TR/prov-overview/) and [CloudEvents specification](https://github.com/cloudevents/spec/tree/v1.0.2).
16. [PostgreSQL transaction documentation](https://www.postgresql.org/docs/current/tutorial-transactions.html).

### 19.2 Implementation and project sources

17. [Connected Systems Go `v1.0.4`](https://github.com/SomethingCreativeStudios/connected-systems-go/tree/244f4dd586da685d4d9b75e43f73001028b5bd0e).
18. [OpenSensorHub core `v2.0.2`](https://github.com/opensensorhub/osh-core/tree/235c0eabf24b6d6137b499b4402943d2794b70e6).
19. [OS4CSAPI client phase-9](https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/754411897173c2ec4debaa9bcf4ed9e0f8a9e230).
20. [SECD interoperability repository](https://github.com/Sam-Bolling/csapi-server-interop-secd/tree/f018fd129bf0d0d1ce75e68198e3ab4d99d937a0).
21. Accepted [IDR-SRV-014A](idr-srv-014a-osh-csapi-server-implementation-study-report.md), [014B](idr-srv-014b-connected-systems-go-csapi-server-implementation-study-report.md), [014C](idr-srv-014c-pygeoapi-csapi-server-implementation-study-report.md), [014D](idr-srv-014d-secd-csapi-server-implementation-study-report.md), [014E](idr-srv-014e-os4csapi-client-smoke-test-findings-study-report.md), [014F](idr-srv-014f-secd-interoperability-findings-study-report.md), and [014G](idr-srv-014g-os4csapi-discussions-lessons-learned-study-report.md).
22. Accepted IDR-SRV-015 through IDR-SRV-030 resource, representation, validation, persistence, transaction and retention reports in this directory.
23. Accepted [IDR-SRV-031](idr-srv-031-server-write-and-ingestion-model-report.md), [032](idr-srv-032-publisher-to-server-contract-boundary-report.md), [033](idr-srv-033-simulator-to-server-contract-boundary-report.md), [034](idr-srv-034-datastream-observation-and-status-update-semantics-report.md), [035](idr-srv-035-streaming-and-event-publication-strategy-report.md), [036](idr-srv-036-control-stream-and-command-lifecycle-model-report.md), and [037](idr-srv-037-feasibility-and-asynchronous-tasking-strategy-report.md).
24. [OGC Connected Systems upstream-history register, Version 1.12](../IDR%20Evidence/ogc-connected-systems-upstream-history-register.md).
25. Controlled AEP baseline `AC/224(JCGISR)D(2026)0005`, April 27, 2026, SHA-256 `56DC757B6E677B3584E3152A957849F21A24B22854F562613FF283A8B599DA8C`; available only through authorized project custody and accepted releasable findings; not redistributed.

### 19.3 Numbered source notes

[^1]: NIST SP 800-162 defines ABAC authorization by evaluating attributes of subjects, objects, requested operations and sometimes environment conditions against policy/rules/relationships. Final update checked 2026-09-15.
[^2]: OGC 23-001 §6 security considerations and OGC 23-002 front matter; OGC 23-002 §10.11/Table 14 defines `CANCELED` as cancellation by an authorized user. Checked against tag `v1.0.0` on 2026-09-15.
[^3]: Opengeospatial issue #58 remained open on 2026-09-15, labeled `ready for pre-SWG review` and `rec: defer or re-scope`, with no branches or pull requests linked.
[^4]: NIST SP 800-53 Rev. 5 AU-2/3/5/8/9/10/11/12 address event selection, content, logging failure, time, protection, non-repudiation, retention and generation. NIST's page identified Release 5.2.0 as the August 27, 2025 minor release when checked 2026-09-15.
[^5]: NIST SP 800-207 moves access decisions from static network perimeter assumptions toward protection of resources and describes policy decision and enforcement concepts. This report uses the principles; IDR-SRV-039A owns full alignment.
[^6]: NIST SP 800-92 remains final September 2006 guidance for log-management infrastructure and robust operational processes. Its Rev. 1 publication was still an initial public draft on the check date and was not treated as controlling final guidance.
[^7]: RFC 9700 (BCP 240, January 2025) recommends least privilege/audience restriction, sender-constrained access tokens, asymmetric client authentication where feasible, and end-to-end TLS. RFC 7662 §4 requires consideration of security and performance tradeoffs when caching introspection results.
[^8]: RFC 9334 separates device evidence, verifier appraisal, attestation result and the relying party's application-specific authorization decision; it also makes freshness a policy decision and warns that trust evidence alone may be insufficient authorization.

## Appendix A. Command Authorization, Safety, and Audit Matrix

| Command decision type | Lifecycle hook | Actor/source | Target/control stream | Required authority | Feasibility dependency | Safety rule | Policy/releasability | Audit record | Event/publication | Diagnostic exposure | DDIL | Test/security-test | Downstream handoff | Notes/unresolved |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Discovery/schema read | before representation/query | client principal | System/ControlStream/schema | read/discover grant | none | conceal unsafe affordance details | filter before existence/count/link | read decision where required | no event; access telemetry only | resource or concealed 404 | locally cached read policy | enumeration/count/timing/cursor | 039/040/055 | OGC resources remain valid within view |
| Feasibility submit | pre-create/admission | client/service/source | target/ControlStream | feasibility submit + target scope | creates analysis | request bounds/test-live isolation | request/result visibility independent | request, validation, authority | resource events only after commit | generic denial | bounded local evaluator only if profiled | feasible but unauthorized; stale source | 037/039/042 | no reservation or command authority |
| Command submit | pre-create/admission | authenticated requester | exact target/contract/revision | command grant and optional control lease | may be required and fresh | static/dynamic admission rules | check write and later read separately | full admission decisions | created/status after commit only | 400/401/403/404/409/503 safe | offline class/bundle controls | target/parameter/mission escape | 039-042/055 | `sender` derived, not trusted input |
| Operator approval | gate pending | authorized approver | exact command revision | approval class; separation of duties | view only permitted evidence | approval cannot bypass mandatory rule | protect operator/rule details | request/decision/expiry/revoke | protected operator event | usually public remains PENDING | local approval only if bundle permits | forged/replayed/stale approval | 039/040/041/055 | private workflow, no new status |
| Override | before dispatch or rule-specific hook | independent override authority | exact command/rules/attempt scope | explicit override grant | never converts stale feasibility automatically | only overrideable rule; compensating controls | highest diagnostic restriction | request/decision/use | protected alert/event | generic denial | explicit offline eligibility and short expiry | noneligible rule, cross-target reuse | 039A-042/055 | cannot override identity/nonoverrideable interlock |
| Command update | before accepted cutoff | original or separately granted editor | command/revision | update plus target/action scope | invalidate/recompute affected evidence | rerun affected rules | recheck payload/read/event labels | old/new digest, all new decisions | `UPDATED` only on accepted update | safe conflict/precondition reason | offline update only by profile; reconcile | update-after-approval/dispatch, CAS | 036/039-043/050 | invalidates ticket/approval as applicable |
| Target/control lease | admission/pre-dispatch | control authority service/operator | exclusive/coordinated stream | issue/hold/transfer/preempt grant | none | prevent conflicting controllers | existence/holder restricted | grant/revoke/expire/fence | protected control event | conceal holder/topology | partition-safe epoch; conservative rejoin | split brain, stale fence, seizure | 039A/042/043/055 | private profile while issue #58 open |
| Dispatch authorization | immediately pre-effect | orchestrator/dispatcher | exact revision/target/path | dispatch service + current command grant/lease | verify bound freshness | all mandatory rules/current evidence | final effect policy check | gate manifest and ticket | private gate; later lifecycle event | no public internal details | local signed bundle/class/time required | TOCTOU, replay, audience, crash windows | 039A/041-045/055 | ticket single-use and fenced |
| Direct dispatch | postcommit | direct adapter | target protocol endpoint | consume named ticket | none beyond ticket manifest | adapter fail-safe/ack contract | credential/routing policy | attempt/send/ack/ambiguity | private delivery event | no topology | connection failure enters policy path | duplicate/ambiguous delivery | 036/042/045/055 | transport ACK not acceptance |
| Broker dispatch | postcommit | broker adapter | topic/subscriber target | ticket + broker ACL | none beyond ticket | topic/consumer binding | topic itself may be sensitive | publish/ack/consumer evidence | protected operational event | no broker internals | QoS/session not authority | topic injection/ACL bypass/replay | 035/039/042/055 | broker is adapter, not authority |
| Gateway relay | postcommit | authenticated gateway | scoped downstream targets | relay grant bound to gateway/path | gateway evidence freshness if used | gateway/target trust and state | hide route/topology | relay, represented target, epoch, ack | protected gateway event | generic reachability only | reconcile epoch and pending attempts | gateway impersonation/scope escape | 032/039/042/043/056 | identity and represented executor distinct |
| Target acceptance/report | after delivery | registered executor/gateway | command/revision | status-specific reporter grant | none | legal transition/evidence quality | status/message filtering | raw assertion + validation + append | status event after commit | policy-safe status/message | late report verified/reconciled | forged/out-of-order/conflicting status | 036/041-043/055 | only exact semantics map to ACCEPTED |
| Execution monitoring | EXECUTING | target/safety monitor | active command/target | monitor/report and any emergency grant | actual state supersedes prediction | continuous interlocks | state may be highly sensitive | telemetry refs, rule decisions/actions | protected alert; status only if authoritative | generic status | local monitors and durable audit | stale telemetry, false abort, audit loss | 041/042/048/055 | cannot invent physical cessation |
| Result append | execution evidence | authorized result reporter | command/result schema | result append for target/type | none | result schema and safety incident rules | independent result disclosure | raw digest, validator, reporter, link | result event after commit | redact sensitive result | delayed results keep original identity | forged/oversize/schema/confidential result | 040/041/043/056 | result not audit or observed world state |
| Cancellation request | any cancellable nonterminal state | client/operator/service | exact command/revision | cancellation-request scope | none | state/cutoff/race checks | hide capability/reason/actor | request/idempotency/decision | private until authoritative outcome | safe accepted/denied response | queue only if still valid/safe | unauthorized/duplicate/complete race | 036/039-043/055 | request is not CANCELED |
| Cancellation dispatch/outcome | pre-effect/target response | cancellation authority/adapter/executor | active attempt/target | ticket and cancellation decision/report grants | none | abort action has its own safety rules | protect physical/control detail | attempt/ack/outcome | `CANCELED` only when authoritative | generic final message | uncertain delivery enters reconciliation | false cancellation/late completion | 036/041-043/055 | receiving-system cancellation may be REJECTED per standard meaning |
| Timeout/expiry | timer/reconciler | trusted service/time source | command/attempt | timer action/admin policy | invalidate stale evidence | configured fail-safe | reason/time visibility restricted | timer evidence, certainty decision | mapped event only when outcome certain | generic status or none | trusted-time confidence required | time skew/before-after-send race | 037/041-043/055 | no public EXPIRED/TIMED_OUT code |
| Audit search/export/redaction | post hoc | auditor/records/policy admin | protected evidence scope | distinct read/export/redact grant | none | export/storage safety | full policy/label enforcement | query/export/redaction event | no public lifecycle event | tiered protected views | delayed export; local custody | bulk exfiltration/tamper/cursor | 040/041/047/055 | derived redaction never alters original |
| Reconciliation | ambiguity/reconnect | reconciler/operator/gateway | command/attempt/lease epoch | reconciliation role and evidence grants | feasibility irrelevant to occurred effect | inhibit conflicting effects until safe | restrict incident/topology | all candidates, conflict, decision | protected incident; public status only when authoritative | no unknown internal detail | verify/dedupe/quarantine; no LWW | late terminals, duplicate epochs, split brain | 041-043/055 | preserve last valid CSAPI status |

## Appendix B. Command Audit Event Catalog and Decision Invariants

### B.1 Minimum event catalog

`request_received`, `request_replay_observed`, `request_rejected`, `validation_decided`, `feasibility_evidence_evaluated`, `authorization_decided`, `control_authority_decided`, `policy_decided`, `safety_decided`, `approval_requested`, `approval_decided`, `approval_invalidated`, `override_requested`, `override_decided`, `override_used`, `command_created`, `command_updated`, `dispatch_authorized`, `dispatch_denied`, `dispatch_attempted`, `transport_acknowledged`, `target_acknowledged`, `status_assertion_received`, `status_assertion_rejected`, `status_appended`, `result_assertion_received`, `result_appended`, `cancellation_requested`, `cancellation_decided`, `cancellation_attempted`, `interlock_triggered`, `timeout_observed`, `outcome_uncertain`, `reconciliation_decided`, `redaction_decided`, `audit_exported`, and `audit_integrity_incident`.

### B.2 Invariants

1. No external effect attempt exists without a durable command revision, gate manifest, dispatch authorization, audit record, and fenced attempt identity.
2. No positive decision applies outside its bound subject, object, action, environment, evidence versions, time, or audience.
3. No client-supplied identity field becomes authenticated principal or authority evidence.
4. No Feasibility outcome, transport acknowledgement, or target reachability substitutes for authorization or safety.
5. No mandatory-rule failure becomes a warning; no override affects a non-overrideable rule.
6. No approval or ticket survives a material command/context change unless the governing rule explicitly proves it unaffected.
7. No test-only identity, policy, target, route, or evidence produces a live dispatch ticket.
8. No cancellation request becomes `CANCELED` without the configured authoritative semantics.
9. No status/result assertion changes projection until reporter authority, identity, sequence/epoch, transition and schema validate.
10. No correction, reconciliation, retention, redaction, or export rewrites original audit evidence.
11. No disconnected decision exceeds its locally verified command class, bundle scope, validity, staleness, revocation and time-confidence rules.
12. No external publication precedes commit or exposes more than the subscriber's policy view.

## Appendix C. Reproducible Evidence Record

### C.1 Immutable source checks

```powershell
git ls-remote https://github.com/opengeospatial/ogcapi-connected-systems.git refs/tags/v1.0.0 refs/heads/master
git ls-remote https://github.com/SomethingCreativeStudios/connected-systems-go.git refs/tags/v1.0.4
git show v1.0.0:api/part1/standard/sections/clause_6_overview.adoc
git show v1.0.0:api/part2/standard/sections/clause_0_front_material.adoc
git show v1.0.0:api/part2/standard/sections/clause_9_requirements_class_controlstreams.adoc
```

Observed pins on September 15, 2026:

```text
CSAPI Parts 1/2 v1.0.0   8e03b236a049849f2ccc24b4fd9fdce5ff69bed2
CSAPI master              3fd86c73e744b7e2faaf7f1c17366bfb9ff4cd6f
CS-Go v1.0.4              244f4dd586da685d4d9b75e43f73001028b5bd0e
OSH v2.0.2                235c0eabf24b6d6137b499b4402943d2794b70e6
OS4CSAPI phase-9          754411897173c2ec4debaa9bcf4ed9e0f8a9e230
SECD evidence             f018fd129bf0d0d1ce75e68198e3ab4d99d937a0
```

### C.2 Current-source checks

- NIST SP 800-53 page identified Release 5.2.0 dated August 27, 2025.
- NIST SP 800-162 remained final with updates through August 2, 2019.
- NIST SP 800-207 remained final August 2020.
- NIST SP 800-92 remained final September 2006; Rev. 1 remained initial public draft.
- RFC 9700 was BCP 240, published January 2025.
- CSAPI upstream issue #58 remained open, without linked development, on September 15, 2026.

### C.3 Report validation targets

```powershell
rg -n '^## [0-9]+\.' idr-srv-038-command-authorization-safety-and-audit-strategy-report.md
rg -n '^\[\^[0-9]+\]:' idr-srv-038-command-authorization-safety-and-audit-strategy-report.md
git diff --check
```

## Report Completion Checklist

- [x] Topic ID matches the overall research plan index
- [x] Topic research plan is linked and aligned
- [x] All core and detailed research questions are answered or explicitly handed off
- [x] Normative, accepted, implementation, inference, and recommendation evidence are distinguished
- [x] Mutable standards/security/implementation sources identify versions, commits, status, and retrieval date
- [x] Controlled-source and artifact limitations are explicit
- [x] All decision types and authority concepts remain distinct
- [x] The required command authorization/safety/audit matrix includes every prescribed column
- [x] Safety rules, lifecycle hooks, approval, override, transfer/control, diagnostics, events and DDIL are covered
- [x] Audit fields, immutability, atomicity, idempotency, retention, search/export, redaction and sync are covered
- [x] Fixtures, conformance, security, performance and interoperability implications are explicit
- [x] Recommendations are decision-usable and bounded to Glaux Server
- [x] References and evidence checks are reproducible
- [x] Report is ready for plan-owner review
- [ ] Report accepted by plan owner

# Section 039: Authentication, Authorization, and API Security Threat Model - Research Report

**Topic ID:** IDR-SRV-039<br>
**Report Status:** In Review<br>
**Research Plan:** [IDR-SRV-039 Authentication, Authorization, and API Security Threat Model](../IDR%20Plans/idr-srv-039-authentication-authorization-and-api-security-threat-model.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** All whole-server questions concerning protected assets, actors, identities, credentials, API surfaces, trust boundaries, authentication, authorization, threats, controls, deployment profiles, ingestion, streaming, command/control, documentation, DDIL, federation, observability, and verification<br>
**Methodology Used:** Authority-ranked standards extraction; immutable implementation pins; accepted-baseline reconciliation; data-flow and trust-boundary modeling; STRIDE and OWASP API Security Top 10 analysis; misuse-case analysis; deployment-profile comparison; control and verification traceability<br>
**Research Time:** Approximately 31 hours of AI-assisted execution on September 15, 2026<br>
**Approved Standards Baseline:** OGC 23-001 and OGC 23-002 Version 1.0, repository tag `v1.0.0` at [`8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`](https://github.com/opengeospatial/ogcapi-connected-systems/commit/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2)<br>
**Current Security Guidance Check:** OWASP API Security Top 10 2023 and ASVS 5.0.0; NIST SP 800-63-4, SP 800-207, SP 800-162, SP 800-53 Rev. 5 Release 5.2.0, and SP 800-92; RFC 7662, RFC 8705, RFC 8725, RFC 9068, RFC 9449, RFC 9700, and RFC 10017; BCP 195 checked September 15, 2026<br>
**Implementation Evidence:** CS-Go `v1.0.4` at [`244f4dd586da685d4d9b75e43f73001028b5bd0e`](https://github.com/SomethingCreativeStudios/connected-systems-go/commit/244f4dd586da685d4d9b75e43f73001028b5bd0e); OpenSensorHub `v2.0.2` at [`235c0eabf24b6d6137b499b4402943d2794b70e6`](https://github.com/opensensorhub/osh-core/commit/235c0eabf24b6d6137b499b4402943d2794b70e6); OS4CSAPI phase-9 at `754411897173c2ec4debaa9bcf4ed9e0f8a9e230`; SECD at `f018fd129bf0d0d1ce75e68198e3ab4d99d937a0`<br>
**Supporting Resources:** Accepted IDR-SRV-001 through IDR-SRV-038, controlled-AEP findings, and upstream-history register Version 1.12<br>
**Document Purpose:** Establish a decision-usable, secure-by-default whole-server authentication, authorization, and API threat-model baseline without selecting a final identity provider, deployment topology, enterprise policy system, or implementation library<br>
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
| X | Known defect, ambiguity, or unresolved choice |

The marks classify claims; they are not conformance levels. “Must” and “shall” identify a published obligation or a consequence of an accepted project decision. “Should” identifies a recommendation awaiting acceptance with this report.

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Security Extraction and Threat-Modeling Methodology
5. Protected Asset Inventory
6. Actor, Identity, and Credential Taxonomy
7. API Surface and Trust-Boundary Inventory
8. Authentication Strategy Findings
9. Authorization Model Findings
10. Resource-Level and Operation-Level Authorization Findings
11. API Security Threat Model
12. Candidate Control Matrix
13. Ingestion, Source-Registration, Streaming, and Command/Control Security Implications
14. OpenAPI, Schema, Conformance, and Documentation Security Implications
15. DDIL, Federation, Cached Credentials, Revocation, and Offline Authorization Implications
16. Error, Logging, Metrics, Tracing, Redaction, and Observability Implications
17. Fixture, Conformance, Security Testing, Performance, and Interoperability Test Implications
18. Downstream Topic Handoff Matrix
19. Recommendations
20. Risks, Constraints, and Open Questions
21. Validation Against This Plan's Success Criteria
22. References

---

## 1. Executive Summary

Glaux should implement one pluggable, server-owned security boundary that authenticates every protected request, maps external identity evidence into an immutable local security context, and evaluates authorization at the resource, operation, property, event, and effect boundaries. Authentication is evidence about who or what made a request; it is not permission. Network location, TLS termination, a publisher registration, a `sender` field, schema validity, source trust, releasability, command feasibility, safety, and audit evidence likewise answer different questions. Conflating any of them creates an exploitable authorization gap. **[N/A/E/P]**

The authorization baseline should be deny-by-default and hybrid: role-based groupings simplify administration, while attribute- and relationship-based decisions bind the authenticated principal and client to action, resource, source, organization, mission/deployment, security domain, time, policy version, and operating state. Each collection query must constrain the authoritative query itself, and every returned representation, property, link, count, extent, cursor, ETag, subscription event, diagnostic, and generated document must be derived from the same authorized view. Fetching broadly and filtering only at the route or client is prohibited. **[E/P]**

Glaux should support interchangeable authentication profiles rather than select a production identity provider in this topic. Operational profiles should accept standards-based OIDC/OAuth identities for people and applications, short-lived audience-bound access tokens, and high-assurance workload or mutual-TLS identities for sensitive machine paths. Sender-constrained tokens using mTLS or DPoP are strong candidates for high-risk contexts. JWT and opaque-token validation should share one internal contract: JWTs favor local/DDIL verification but require strict algorithm, issuer, audience, type, time, and key checks; introspection improves centralized freshness but creates availability and cache-staleness dependencies. **[E/P]**

Local development is the only profile that may offer explicit unauthenticated operation. It must be named `unsafe-dev`, restricted to loopback or a local IPC boundary, use ephemeral state and keys, disable ingestion from untrusted networks, source registration, administration, federation, and live tasking, and fail startup if combined with an external bind or dangerous capability. CI should use deterministic ephemeral test issuers, keys, certificates, identities, and isolated namespaces—not the unsafe bypass and never real credentials. Demonstrations should exercise the real security abstraction with a controlled issuer and least-privilege test identities. **[E/P]**

The threat model combines trust-boundary data flows, STRIDE, the OWASP API Security Top 10 2023, and Glaux-specific misuse cases. The highest-consequence risks are broken object/property/function authorization; identity or token replay; unsafe source and federated input; query, stream, decompression, and command-flow exhaustion; server-side request forgery through schemas or references; policy-view leakage through links, counts, errors, caches, OpenAPI, and events; insecure proxy/header trust; and bypass of accepted ingestion or command gates. Controls must be enforced in the server and tested negatively; gateways, brokers, adapters, and clients provide defense in depth but are not enforcement authorities. **[E/P]**

Long-lived subscriptions require more than handshake authentication. Glaux must authorize the subscription query, filter each event using current policy, enforce token/session expiry, react to revocation and trust/policy changes within a bounded interval, bind cursors to the authorized view, and close or require reauthentication when continued delivery is no longer allowed. Command paths retain every accepted IDR-SRV-038 gate, including command-specific authority and immediate pre-dispatch re-evaluation; a general API permission or valid access token can never mint physical authority. **[A/E/P]**

DDIL operation should use locally verifiable signed credentials and security bundles with explicit issuer, audience, scope, validity, revocation epoch, maximum staleness, time-confidence requirement, and offline operating class. Unknown time, expired authority, insufficient freshness, or uncertain revocation fails closed for the affected operation. Final disconnected-operation policy belongs to IDR-SRV-042, and synchronization/conflict behavior belongs to IDR-SRV-043. **[A/E/P]**

OpenAPI and conformance metadata are capability disclosure surfaces, not automatically public documentation. Glaux's accepted canonical OpenAPI version remains 3.1.2. OpenAPI 3.2.1, published September 10, 2026, is a monitored target and does not silently replace the accepted baseline. Generated descriptions must come from the enabled capability/security registry, omit secrets and internal topology, declare security truthfully, avoid untrusted request-time external references, and be protected or policy-filtered where deployment capability itself is sensitive. **[A/E/P]**

This report completes the first Category G research topic but authorizes no implementation and no next topic. IDR-SRV-039A remains unauthorized pending project-lead acceptance of this report. It will own detailed zero-trust component placement, enforcement topology, trust zones, and continuous-decision architecture; IDR-SRV-040 owns detailed policy and releasability; IDR-SRV-041 owns general audit architecture; and IDR-SRV-042/043 own final DDIL and synchronization behavior. **[A]**

### 1.1 Recommended baseline at a glance

| Decision | Rationale | Consequence | Confidence |
|---|---|---|---|
| Pluggable server-owned authentication and authorization interfaces | Identity products and deployment shapes differ, but semantics must not | Stable `SecurityContext`, authenticator, principal mapper, and authorizer contracts | High |
| Deny by default; hybrid RBAC plus attributes/relationships | Roles alone cannot express object, source, mission, property, or time bounds | Typed action/resource/environment decisions at every enforcement point | High |
| Claims are inputs, never local authority by themselves | External issuers do not own Glaux resources or source relationships | Versioned local mapping and authorization policy remain authoritative | High |
| Short-lived audience-bound tokens; sender constraint for high risk | Limits replay and token substitution | mTLS/DPoP-capable adapters and exact audience enforcement | High |
| JWT and opaque tokens behind one validation contract | Supports online and DDIL profiles without identity-provider lock-in | Strict validation plus bounded introspection/cache behavior | High |
| Explicit `unsafe-dev`, never an implicit fallback | Developer usability must not become a production bypass | Loopback-only, ephemeral, dangerous capabilities off, fail-fast startup | High |
| Query-, property-, link-, cache-, cursor-, and event-level policy | Route checks do not prevent indirect disclosure | Authorized-view identity propagates through reads and streams | High |
| Re-evaluate long-lived streams and commands | Authorization may change after admission | Expiry timers, revocation/policy hooks, per-event filtering, pre-dispatch check | High |
| TLS 1.3 baseline for new deployments/profiles | Current BCP 195 guidance requires TLS 1.3 for new protocols using TLS | Profile-controlled exceptions must be explicit and tested | High |
| Security fixtures contain no real secrets | Test reproducibility must not create credential exposure | Deterministic ephemeral keys/tokens/certs and secret-scanning gates | High |

## 2. Scope and Plan Alignment

### 2.1 Included scope

This report covers all server-facing and server-originated security boundaries: public CSAPI discovery and resources; resource writes; observation and result ingestion; publisher, adapter, simulator, gateway, and source registration; streaming and event delivery; feasibility and command/control; administration and configuration; health, metrics, logs, and traces; OpenAPI, schema, documentation, and conformance surfaces; service-to-service calls; federation; local, CI, demo, operational, and DDIL deployment profiles; and outbound calls to upstream dependencies.

It defines protected assets, actors, identity concepts, credential classes, authentication and authorization abstractions, resource/action enforcement, threats, candidate controls, safe diagnostics, security evidence, and downstream test obligations.

### 2.2 Explicit exclusions

This report does not select a production identity provider, certificate authority, secrets manager, OAuth authorization server, policy language/engine, web framework, reverse proxy, service mesh, token lifetime, assurance level, organizational role catalog, deployment topology, SIEM, or accreditation control tailoring. It does not implement the server, enable tasking, enable inbound draft Part 3 channels, or claim zero-trust conformance.

IDR-SRV-039A owns detailed zero-trust architecture and enforcement placement. IDR-SRV-040 owns policy labels, releasability, cross-boundary transformations, and disclosure rules. IDR-SRV-041 owns the general audit schema, retention, integrity, and accountability architecture. IDR-SRV-042 and IDR-SRV-043 own final DDIL semantics and synchronization/conflict rules.

### 2.3 Required semantic separations

| Concept | Question answered | Does not prove |
|---|---|---|
| Authentication | Which principal/client presented acceptable evidence? | authority, source trust, releasability, safety |
| Authorization | May that context perform this action on this resource/view now? | data truth, safety, successful effect |
| Source trust | May this publisher/source assert this class of fact? | API permission outside its contract |
| Policy/releasability | May information/effect cross this boundary for this purpose? | identity validity or operational safety |
| Validation | Is the submitted representation structurally and semantically admissible? | source authority, permission, or truth |
| Command authority | May the actor request this bounded effect? | feasibility, safety, or dispatch readiness |
| Safety/interlock | Is the effect acceptable in current operating context? | caller identity or general API permission |
| Audit | What security-relevant evidence was recorded? | correctness solely because a record exists |
| Transport security | Was the channel protected to an authenticated endpoint? | end-user identity or resource authorization |

### 2.4 Plan coverage

Sections 5 through 7 inventory assets, actors, surfaces, flows, and trust boundaries. Sections 8 through 10 evaluate every required deployment/authentication context and define the authorization model. Sections 11 and 12 provide the threat model and the required 15-column matrix. Sections 13 through 17 cover specialized surfaces and tests. Sections 18 through 21 provide handoffs, recommendations, risks, and success-criteria traceability.

## 3. Evidence Base and Authority Classification

### 3.1 Controlling OGC and Glaux evidence

OGC API - Connected Systems Part 1 and Part 2 Version 1.0 at tag `v1.0.0` are controlling. Part 1 expects some functionality to be protected by access control requiring authentication, does not mandate one method, and discusses basic, bearer, API-key, OAuth, and OpenID Connect patterns. It recommends HTTPS for sensitive data and strong, revocable asymmetric authentication/signature approaches for machine clients. Part 2 inherits Part 1 security considerations and identifies cancellation as an action by an authorized user, but neither part defines a complete identity or authorization architecture. **[N]**

SensorML 3.0 recognizes sensitive or confidential descriptions, recommends HTTPS and encryption at rest, and provides `securityConstraints` based on external security models; those constraints can cover a document and SWE extensions can carry finer-grained markings. SWE Common validation constructs describe representations and allowed values. Neither standard turns a label or valid value into an authorization decision. **[N/E]**

Accepted IDR-SRV-029, 031, 032, 033, 035, and 038 are controlling project baselines for transactions, the common write boundary, publisher/source distinctions, simulator isolation, post-commit publication, and command-specific authorization/safety. This report composes those boundaries rather than reopening them. **[A]**

The controlled AEP-4789 evidence reviewed by the project remains non-redistributable. Only accepted releasable findings are used here; no controlled text, operational credential, restricted topology, or protected identifier is reproduced. **[A]**

### 3.2 Security guidance

OWASP's 2023 API list provides current API-specific categories: broken object authorization, broken authentication, broken object-property authorization, unrestricted resource consumption, broken function authorization, unrestricted sensitive business flows, SSRF, security misconfiguration, improper inventory management, and unsafe API consumption. OWASP's threat-modeling guidance supplies the recurring questions—what is being built, what can go wrong, what will be done, and whether it was done well—and supports use of STRIDE. OWASP ASVS 5.0.0 provides verification-oriented requirements, not a replacement architecture. **[E]**

NIST SP 800-207 rejects implicit trust based on network location and treats authentication and authorization as discrete, resource-focused decisions. SP 800-162 supplies the subject, object, requested operation, and environment abstraction used for fine-grained authorization. SP 800-63-4 supplies current digital-identity guidance without fixing Glaux's organizational assurance selections. SP 800-53 Rev. 5 Release 5.2.0 and SP 800-92 provide control and log-management objectives; precise tailoring remains downstream work. **[E]**

RFC 9700, now BCP 240, recommends audience-restricted, least-privilege and sender-constrained OAuth deployments, asymmetric client authentication where feasible, and end-to-end TLS. RFC 8705 and RFC 9449 describe mTLS- and DPoP-bound tokens. RFC 8725 and RFC 9068 require disciplined JWT algorithm/type/issuer/audience/time validation and caution against sensitive visible claims. RFC 7662 defines introspection and exposes the freshness/availability tradeoff of cached active-state decisions. RFC 10017 ranks browser application patterns with backend-for-frontend strongest because tokens remain outside the browser, and requires authorization code with PKCE for browser flows. **[E]**

The current BCP 195 composition includes RFC 9852, which requires TLS 1.3 for new protocols using TLS. Glaux is not inventing a new Internet protocol, but a new deployment/profile should use TLS 1.3 as its normal baseline; any compatibility exception must be explicit, time-bounded, and tested. **[E/P]**

### 3.3 Implementation evidence and its limits

CS-Go `v1.0.4` is useful for API-surface and interoperability study, but its server does not establish a production REST authentication baseline, uses permissive CORS behavior, and does not provide a complete OpenAPI security declaration. OpenSensorHub `v2.0.2` demonstrates granular resource/action permission concepts, but optional or deployment-specific authentication behavior is not evidence of a safe Glaux default. The pygeoapi-based interoperability proof uses fixed demonstration credentials and broad read/CORS behavior appropriate only to its test purpose. **[I]**

OS4CSAPI and SECD remain valuable client, fixture, and failure-shape evidence. No implementation observation overrides published standards or accepted Glaux decisions. Absence of a security control in an interoperability implementation is a gap to test, not permission to omit the control. **[I/E]**

### 3.4 Currency and reproducibility

Standards and guidance were checked September 15, 2026. Immutable source pins are stated in the metadata and Section 22. OpenAPI 3.2.1 is the current published specification as of that check, while accepted IDR-SRV-014 keeps OpenAPI 3.1.2 canonical until the project's release-gated adoption process authorizes a change. The shared upstream-history register remains Version 1.12 because the review found no new tracked CSAPI history requiring a row. **[A]**

## 4. Security Extraction and Threat-Modeling Methodology

The analysis used six passes:

1. Extract standards obligations and accepted project invariants without promoting examples into requirements.
2. Model data flows among caller, edge/proxy, Glaux API, authentication authority, policy decision point, store, publisher/adapter, broker, command gateway, federated server, documentation renderer, and observability sink.
3. Identify protected assets and mark every change of identity, administrative, process, transport, persistence, policy, and security domain as a trust boundary.
4. Apply STRIDE—spoofing, tampering, repudiation, information disclosure, denial of service, and elevation of privilege—at each flow and store.
5. Cross-check OWASP API Security Top 10 2023 and Glaux-specific misuse cases, especially indirect disclosure and physical-effect paths.
6. Trace each control to an enforcement point, safe failure mode, audit/redaction behavior, DDIL implication, negative test, and downstream owner.

The report uses qualitative likelihood and impact because there is not yet an accredited deployment, asset valuation, or organizational threat intelligence set. “Critical,” “high,” or “medium” therefore indicates consequence priority for design sequencing, not a fabricated quantitative risk score. Command effect, cross-domain disclosure, credential compromise, and corruption of authoritative evidence receive the highest consequence weight.

Controls are evaluated against seven properties: server-side enforceability; deny-by-default behavior; deployment portability; DDIL suitability; redaction safety; auditability; and fixture/testability. A control is not credited merely because a proxy, UI, SDK, broker, adapter, or identity provider could perform it. At least one authoritative server enforcement point must exist, with defense-in-depth duplication where valuable.

## 5. Protected Asset Inventory

| Asset class | Examples | Security properties and consequence |
|---|---|---|
| Identity and credential material | passwords, access/refresh tokens, cookies, API keys, private keys, certificates, DPoP keys | Confidentiality and anti-replay; compromise enables impersonation |
| Identity mappings | external subject to local principal/client/source/org mappings | Integrity and provenance; corruption grants unintended authority |
| Authorization and policy state | roles, grants, attributes, relationships, policy/security bundles, revocation epoch | Integrity, freshness, availability; stale or altered state changes access |
| Standards resources | Systems, Deployments, Procedures, Sampling/Control Features, DataStreams, ControlStreams | Confidentiality, integrity, provenance, authorized discovery |
| Dynamic data | Observations, command status/results, system events, stream cursors | Integrity, ordering, policy-filtered confidentiality, availability |
| Write/source authority | publisher registrations, source bindings, sequence/idempotency state, quarantine evidence | Integrity and non-repudiation; spoofed sources contaminate authority |
| Command/effect authority | CommandAuthorityGrant, ControlAuthorityLease, approval, interlocks, dispatch tickets | Integrity, freshness, single use; compromise can cause physical effect |
| Audit and decision evidence | request/decision/effect records, time, actor, correlation, policy versions | Completeness, integrity, controlled confidentiality, availability |
| API and capability metadata | OpenAPI, AsyncAPI, conformance, links, routes, schema catalogs | Integrity and disclosure control; exposes reachable capabilities |
| Service configuration | issuer/audience lists, trust anchors, proxy allowlists, CORS, limits, feature flags | Integrity and secrecy where applicable; misconfiguration defeats controls |
| Data-plane availability | query, write, stream, worker, database, broker, command gateway | Bounded resource use and recovery; exhaustion blocks mission use |
| Observability data | logs, traces, metrics, health/readiness details | Confidentiality and integrity; often aggregates sensitive context |
| Software/supply chain | binaries, containers, dependencies, schema bundles, migration artifacts | Integrity, provenance, reproducibility |
| Deployment topology | internal hostnames, ports, peer identities, trust zones | Confidentiality; assists lateral movement and targeting |

Asset protection is view-dependent. Even a public resource may contain protected properties, links, extents, counts, identifiers, or relationships. Conversely, a secret-free bearer token digest may be appropriate for correlation while the token itself is never retained. Classification decisions belong to IDR-SRV-040; this inventory ensures the enforcement architecture can honor them.

## 6. Actor, Identity, and Credential Taxonomy

### 6.1 Actor and identity classes

| Actor | Required identity distinction | Normal authority boundary |
|---|---|---|
| Anonymous reader | no authenticated principal | Only explicitly public, non-sensitive deployment surfaces |
| Human user | person principal plus session/client | Authorized read/write operations; never inferred command authority |
| Operator | person principal, assurance/context, workstation/client | Bounded operational and approval actions |
| Administrator | person principal plus privileged session and step-up context | Configuration and identity/policy administration; separated duties |
| Web/mobile client | public client instance and represented user | User-delegated actions; browser/mobile token protections |
| Confidential service client | workload/client identity and optional represented user | Service-to-service actions scoped to audience and purpose |
| Publisher/adapter | workload instance plus registered publisher identity | Only registered submission classes and represented sources |
| Source | represented origin of facts | Provenance/authority object; not necessarily the network caller |
| Simulator | isolated workload and scenario/run identity | Synthetic data plane; control plane disabled in production |
| Federated Glaux/CSAPI server | peer workload plus federation relationship/security domain | Bilaterally mapped resources/actions; no transitive trust |
| Command gateway/executor | gateway workload plus target/stream grant | Report/dispatch envelope only; not a human or source substitute |
| Conformance/security client | isolated test-client identity | Explicit test namespace/profile, no production authority |
| Background worker | internal workload identity | Narrow queue/task/store action, no blanket server role |
| Threat actor | stolen client, malicious insider, hostile publisher, compromised peer, unauthenticated Internet client | No legitimate authority beyond any compromised credential's effective scope |

Principal, human identity, client/application, workload instance, publisher, represented source, organization/mission, device/gateway, and target are separate identifiers. Delegation must preserve both the authenticating actor and represented actor. A client-provided `sender`, `owner`, source identifier, organization, role, or policy label is untrusted input until resolved through an authoritative mapping.

### 6.2 Immutable request security context

Successful authentication should yield an immutable internal `SecurityContext` containing, as applicable: local principal ID; client/service ID; credential class and assurance metadata; issuer/security domain; session or token digest identifier; authentication time; intended audience; delegated/represented identity; device/gateway identity; expiry and not-before bounds; revocation/freshness evidence; and request transport/proxy evidence. Raw credentials are excluded. Downstream code receives this context, never reparses bearer tokens or trusts route headers independently.

### 6.3 Credential taxonomy

| Credential/pattern | Appropriate use | Required safeguards | Non-use/default |
|---|---|---|---|
| OIDC user session | operational human/web authentication | code + PKCE, exact redirect URIs, state/nonce, short session, step-up hooks | No implicit or resource-owner-password flow |
| OAuth access token | scoped API delegation | short life, exact issuer/audience/scope, TLS, rotation/revocation | No identity inferred from unvalidated claims |
| mTLS workload identity | publisher, gateway, peer, high-value service path | managed PKI, cert binding, rotation, revocation, mapped service identity | Certificate DN alone is not resource permission |
| DPoP-bound token | public client or replay-sensitive API path | proof validation, nonce/replay handling, key binding | DPoP proof alone is not authentication/authorization |
| JWT access token | local validation and bounded DDIL | fixed algorithms, `typ`, signature, issuer, audience, time, key-use checks | No sensitive claims or algorithm agility from attacker input |
| Opaque token | centrally revocable connected operation | protected introspection, authenticated client, bounded cache | No cache beyond expiry or approved revocation staleness |
| Secure session cookie | BFF/browser session | `Secure`, `HttpOnly`, `SameSite`, CSRF defense, rotation | Never log; never broad-domain cookie |
| API key | bootstrap, dev, narrowly bounded legacy integration | high entropy, header only, hashed at rest, scope, expiry, rotation | Not sole production command identity; never query string |
| Basic credentials | constrained local/test compatibility | TLS, secure store, rotation | Not an operational baseline |
| Signed offline bundle | DDIL credential/policy/trust material | issuer/audience/scope/time/epoch/staleness, protected keys | No indefinite offline grant |

## 7. API Surface and Trust-Boundary Inventory

### 7.1 Surface classification

| Surface | Exposure/default | Principal boundary | Primary threats |
|---|---|---|---|
| Landing, conformance, API definition | Explicitly public only by profile; otherwise authenticated/redacted | Internet/user to API | capability inventory, topology leak, unsafe rendering |
| Standards resource reads | Protected unless deployment declares public view | user/client to query/store/serializer | BOLA, property/link/count leakage, enumeration |
| Standards resource writes | Protected and capability-gated | user/client to write pipeline | BFLA, mass assignment, injection, lost update |
| Dynamic-data query | Protected and quota-bound | user/client to time-series query | resource exhaustion, inference, cursor/view leakage |
| Publisher/adapter ingestion | Registered workload only | external workload/source to admission pipeline | spoofing, replay, unsafe payload, authority confusion |
| Source registration | Administrative/internal by default | administrator to trust registry | privilege escalation, hostile trust anchor |
| Streaming/event subscription | Protected; no anonymous default | subscriber to broker/stream projection | stale authorization, cross-view replay, slow consumer |
| Feasibility and command/control | Strongest profile; disabled until gates proven | operator/client/gateway to decision/effect path | unauthorized effect, replay, confused deputy, TOCTOU |
| Administration/configuration | Internal/privileged, separate audience | administrator to control plane | total compromise, secret/config exposure |
| Health/readiness/metrics/traces | liveness minimal; other signals internal | orchestrator/operator to internals | topology/data/credential leakage, DoS |
| Conformance/test/reset/fault injection | Isolated and disabled by production profile | test identity to test namespace | destructive bypass, production contamination |
| Outbound schema/reference/federation calls | Allowlisted and mediated | server to untrusted network/dependency | SSRF, unsafe consumption, dependency impersonation |

### 7.2 Trust boundaries and security invariants

Trust changes at every external request, proxy hop, identity-provider response, local identity mapping, process/service boundary, queue or broker, storage transaction, federation link, command gateway, observability exporter, browser renderer, and security-domain crossing. “Internal network” is not an identity. A TLS-authenticated proxy or peer is trusted only for an explicit protocol, audience, action set, and header/assertion contract.

The following invariants apply across surfaces:

- Direct backend access cannot bypass edge controls.
- Incoming identity/forwarding headers are stripped unless received from an authenticated allowlisted proxy under an exact contract.
- Query results, derived metadata, caches, cursors, ETags, links, events, and errors remain bound to the authorized view.
- External and federated content is untrusted even when its transport peer is authenticated.
- Administrative, test, simulator-control, and operational data-plane audiences are distinct.
- Authentication or policy service failure never turns a protected route into an anonymous or cached allow outside an explicit bounded DDIL rule.
- A credential proves no more than the exact local mapping and policy decision grant.

## 8. Authentication Strategy Findings

### 8.1 Common authentication architecture

Glaux should expose framework-independent interfaces for credential extraction, authentication, external-to-local principal mapping, token/session status, and security-context construction. Route code should request an authenticated context and an action decision; it should not know whether identity came from OIDC, JWT validation, introspection, mTLS, a session, or a deterministic test issuer. Authentication adapters must return typed failure reasons internally while public errors remain stable and non-oracular.

Trust configuration is data with a lifecycle: accepted issuers, audiences, signature algorithms, keys/trust anchors, certificate profiles, clock policy, introspection endpoints, proxy identities, and principal mappings are versioned, auditable, validated before activation, and replaceable without restarting unsafe fallback behavior. Unknown issuer, audience mismatch, invalid type, disallowed algorithm, expired/not-yet-valid credential, failed proof binding, revoked status, ambiguous principal mapping, or unavailable mandatory validation fails authentication.

JWT validation must fix allowed algorithms out of band; validate signature, `typ` (`at+jwt` where the profile applies), issuer, exact intended audience, expiry, not-before and reasonable issued-at constraints; distinguish access tokens from ID tokens or other JWTs; validate key use and rotation; and reject ambiguous/multiple interpretations. Token claims should remain identifiers and decision inputs, not carry sensitive topology or complete policy. Opaque-token introspection must authenticate the Glaux client, validate the response and subject/audience semantics, and cache only within both token lifetime and a profile-defined revocation-staleness bound.

Sender constraint should be available for replay-sensitive paths. With mTLS-bound tokens, the resource server verifies that the request certificate matches the token binding. With DPoP, it validates proof signature, method, target URI, nonce/jti freshness, access-token hash where required, and key binding. Neither possession proof independently establishes an identity or permission.

### 8.2 Deployment profiles

| Profile | Authentication baseline | Failure/default behavior |
|---|---|---|
| Local development | Explicit `unsafe-dev` principal or ephemeral local issuer; loopback/UDS only | Startup fails on external bind, persistent operational data, federation, source registration, admin exposure, or live tasking |
| CI/unit/integration | Deterministic ephemeral issuer, tokens, certs, sessions, revocation and clock fixtures | Isolated namespace; no real secret and no dependency on developer credentials |
| Demonstration | Controlled test OIDC/OAuth issuer or equivalent real adapter; named least-privilege personas | No shared fixed production-like credential; tasking off or simulator-only |
| Operational human | OIDC authorization code, PKCE, short server session/access token, MFA/step-up hooks | Issuer/audience mismatch or lost freshness fails closed |
| Operational service | Workload identity/mTLS or OAuth client identity, short audience-bound token | No shared service key; asymmetric authentication where feasible |
| Publisher/adapter | Registered workload identity plus separate represented publisher/source mapping | Credential alone cannot create or change source authority |
| Federation | Peer workload identity plus bilateral federation/security-domain mapping | No transitive trust or automatic claim/role import |
| Command gateway | High-assurance workload identity plus accepted command grant/ticket checks | Transport/broker identity cannot substitute for command authority |
| DDIL | Locally verifiable credential and signed trust/policy bundle | Only explicit offline class within time/freshness/revocation bounds |

Unsafe-dev behavior is not an authentication “mode” selected by missing configuration. It is a deliberate, visibly named profile whose guard conditions are verified before bind. Production-capable builds may contain the profile, but operational configuration validation must make accidental activation obvious and fatal.

### 8.3 Browser and mobile clients

For the Glaux web application, a backend-for-frontend is preferred where feasible because OAuth tokens stay outside browser JavaScript. The BFF must use secure, HTTP-only, same-site cookies; implement CSRF protections; rotate sessions; validate origins; restrict outbound resource-server destinations, methods, and headers to an explicit allowlist; and never become an open proxy. If a browser-only client is required, use authorization code with PKCE and consider DPoP, keep tokens short-lived and out of persistent storage where practical, and design for XSS containment. The implicit and resource-owner-password grants are prohibited.

Native/mobile clients use authorization code with PKCE, system browser/claimed HTTPS or platform-protected redirect handling, secure OS credential storage, and short-lived tokens. A mobile installation identity is not a user or command authority.

### 8.4 Reverse proxy and gateway authentication

Proxy-mediated identity is acceptable only when direct backend access is prevented, the proxy authenticates to Glaux (preferably mTLS/workload identity), Glaux strips all externally supplied identity headers, the allowed proxy addresses/identities and forwarded-header hops are exact, and the identity assertion contract is integrity-protected and audience-bound. Glaux must still authorize every resource/action. `X-Forwarded-*` and similar headers are ignored outside that contract.

## 9. Authorization Model Findings

### 9.1 Hybrid model

Glaux should use a hybrid model:

- RBAC groups permissions for administration and operator usability.
- OAuth scopes constrain delegation and token purpose.
- ABAC evaluates subject, client, resource, action, environment, mission, deployment, security domain, time, source, trust, and policy attributes.
- Resource relationships determine ownership, parent/child visibility, publisher/source authority, federation mapping, and target/control relationships.
- Accepted specialized grants, including `CommandAuthorityGrant`, add domain authority that general permissions cannot imply.

External roles/scopes/claims are inputs. A versioned local mapping decides what Glaux principal, client, attributes, and candidate permissions they represent. Policies should return allow/deny plus an internal reason, obligations (property filter, maximum page/rate, redaction, step-up, fresh-policy requirement), policy/attribute versions, and decision expiry. Absence, ambiguity, evaluation error, or unavailable mandatory evidence is deny.

### 9.2 Enforcement points

| Enforcement point | Responsibility |
|---|---|
| Route/protocol admission | Authenticate; reject unsupported capability; coarse action check and limits |
| Query planner/repository | Add mandatory row/resource/source/security-domain predicates before access |
| Write pipeline | Check action, target, source authority, immutable/server-owned fields, conditional/idempotency scope |
| Serializer/link builder | Apply property obligations; omit unauthorized links, alternates, counts, extents, affordances |
| Cache/cursor/ETag layer | Bind artifacts to authorized-view identity and policy/data versions |
| Stream/subscription layer | Authorize selector; filter events; enforce expiry/revocation/re-evaluation |
| Worker/outbox/broker adapter | Carry immutable initiating security evidence; reauthorize effects where required |
| Command decision/dispatch | Preserve all IDR-SRV-038 gates and pre-dispatch re-evaluation |
| Admin/configuration | Separate audience/role, step-up and separation-of-duty hooks |
| Observability/export | Redact before emission; authorize diagnostic access and export destination |

Route-only checks, post-query filtering, client-side hiding, broker ACLs, and gateway checks are insufficient. A nested resource must be authorized in its own context and its parent relationship; changing an identifier in a URL or body must not escape the decision scope.

### 9.3 Decision and cache semantics

An authorization decision should be keyed to principal, client/workload, delegated identity, action, resource or query scope, source/security domain, policy and relevant attribute versions, environment/time class, credential/session identifier, and authorized-view obligations. Cache TTL must not exceed credential expiry, policy decision expiry, revocation freshness, resource relationship freshness, or deployment-profile maximum. Policy, membership, trust, source mapping, resource ownership, or credential changes invalidate affected entries.

Command pre-dispatch authorization is never reused from a generic cache. It follows IDR-SRV-038 and produces its own bounded single-use evidence. For ordinary reads, denial caching must not make transient policy unavailability indistinguishable from an authoritative deny in audit evidence, even if both are exposed generically.

### 9.4 Administrative model

Privileges should be composable and narrow: identity mapping, issuer/trust configuration, source registration, policy administration, security diagnostics, operational administration, audit access, test-control operation, and command approval/override are distinct. Bootstrap authority must be time-bounded, auditable, rotated or removed after provisioning, and inaccessible through ordinary public audiences. Sensitive changes should support dual control or approval where the later accreditation/profile requires it.

## 10. Resource-Level and Operation-Level Authorization Findings

### 10.1 Action vocabulary

Glaux needs stable actions rather than a single “read/write” bit: `discover`, `list`, `read`, `create`, `replace`, `update`, `delete`, `subscribe`, `replay`, `ingest`, `register-source`, `manage-trust`, `view-diagnostics`, `view-audit`, `administer`, `request-feasibility`, `submit-command`, `update-command`, `cancel-command`, `approve-command`, `override-safety`, `dispatch-command`, `report-status`, and `report-result`. The capability registry maps each enabled route/protocol operation to one action and resource type; generated descriptions and tests use the same mapping.

### 10.2 Resource families

| Resource/surface | Required authorization granularity | Important indirect disclosures |
|---|---|---|
| System/Deployment/Procedure | instance, relationship, property/security marking | children, links, geometry, time extent, counts |
| Sampling/Control Feature | instance, target relationship, spatial/property view | target identity, controllability, location |
| DataStream/Observation | stream, source, time/query window, property view | latest/extents, phenomenon coverage, cursor existence |
| ControlStream/Command | target/stream/version/action/parameter envelope and specialized grant | command affordance, status, result, feasibility, safety reason |
| Event/change feed | subscription selector, event type, resource view at delivery | existence/timing of hidden changes, replay watermark |
| Publisher/source | workload, registration, represented source and allowed resource/action | trust status, topology, rejection reasons |
| OpenAPI/conformance/schema | deployment profile and capability disclosure policy | disabled/private routes, auth schemes, internal references |
| Admin/health/metrics | exact operational role and audience | topology, dependency state, identifiers, workload behavior |

List authorization must become query predicates; it is not `list all` followed by deletion. Counts, extents, `numberMatched`, pagination, sort order, timing, existence checks, conditional headers, and cache behavior must describe only the caller's authorized view. Stable identifiers remain opaque and do not confer access. Create permission does not allow assignment of server-owned identity, ownership, source, policy label, approval, status, authority, or audit fields. Update/replace/delete require object-level reauthorization, strong preconditions where already accepted, and separate treatment of relationships and sensitive properties.

### 10.3 Public and protected surfaces

A deployment may intentionally expose a public landing page, conformance list, API definition, schema, or read-only resource subset. Public exposure must be explicit per profile and evaluated for capability/topology disclosure. Liveness should disclose only that the process is alive. Readiness, dependency detail, metrics, traces, configuration, source registration, test controls, and administration are internal/protected by default. CORS only controls browser access; it is never authentication or authorization.

### 10.4 Concealment and response behavior

Unauthenticated access to a protected surface returns `401` with an appropriate challenge. An authenticated but unauthorized operation normally returns `403`; deployments may use `404` when confirming existence would disclose a protected object, but must apply that concealment consistently. Invalid syntax is `400`, precondition/conflict behavior follows accepted write decisions, oversized input is `413`, unsupported media is `415`, rate/resource exhaustion is `429` with bounded retry guidance, and unavailable mandatory security dependencies use a non-oracular `503`. Exact validation status must remain consistent with IDR-SRV-013 rather than inventing a new global `422` rule here.

## 11. API Security Threat Model

### 11.1 Priority threat themes

1. **Broken object, property, and function authorization:** identifier substitution, nested relationship traversal, mass assignment, hidden-property mutation, or access to admin/command routes.
2. **Broken authentication and replay:** stolen bearer tokens, JWT confusion, weak keys, audience substitution, session fixation, certificate mapping errors, DPoP/mTLS binding omission, or stale introspection.
3. **Authority confusion:** treating publisher identity, source ID, federated peer, gateway, simulator, schema validity, or user-supplied `sender` as authority.
4. **Indirect disclosure:** hidden objects inferred through links, counts, extents, events, cursor behavior, ETags, timing, error reasons, documentation, health, logs, or caches.
5. **Unsafe write and effect flow:** bypassing the common pipeline, server-owned-field assignment, idempotency collision, command gate bypass, stale approval, or confused-deputy dispatch.
6. **Resource and business-flow exhaustion:** unbounded spatial/temporal query, page sizes, deep filters, large arrays/geometries, decompression bombs, high-rate subscription/replay, feasibility or command request floods, and slow consumers.
7. **SSRF and unsafe dependency consumption:** arbitrary schema/reference URLs, redirects to internal networks, DNS rebinding, malicious peer resources, or compromised dependency responses.
8. **Injection and content execution:** database/query injection, path traversal, unsafe shell invocation, log/control-character injection, or script-capable Markdown/HTML in generated documentation.
9. **Security misconfiguration and inventory drift:** unsafe-dev external exposure, wildcard credentialed CORS, debug/test endpoints, forgotten version/routes, permissive proxy trust, default secrets, or TLS downgrade.
10. **Audit and observability compromise:** missing decision evidence, attacker-controlled log fields, token/payload capture, high-cardinality secret-bearing metrics, trace propagation to wrong domains, or audit tampering.

### 11.2 STRIDE application

Spoofing is addressed with cryptographic authentication, exact local mapping, proxy boundary controls, and represented-source separation. Tampering is addressed with TLS, signatures/bindings where appropriate, schema and semantic validation, conditional writes, transactional integrity, trusted artifacts, and protected configuration. Repudiation is addressed with actor/client/delegation provenance and atomic security/audit evidence without claiming that logging alone proves truth. Information disclosure is addressed with authorized views, consistent concealment, redaction, safe errors, protected documentation, and view-bound caches. Denial of service is addressed with layered budgets, quotas, timeouts, bounded parsing, pagination, backpressure, admission control, and circuit breaking. Elevation of privilege is addressed with deny-by-default fine-grained actions, local mapping, separation of duties, immutable server-owned fields, and no transitive trust.

### 11.3 Consequence priorities

Unauthorized physical command dispatch, cross-domain release, trust/source-registration compromise, identity/configuration administration compromise, or corruption of authoritative/audit evidence is critical consequence. Broad protected-resource disclosure, ingestion contamination, persistent credential theft, or sustained mission-service outage is high. Enumeration, limited metadata disclosure, and localized resource exhaustion remain material and can become high when composed. This report intentionally avoids numeric scoring until deployment assets, threats, compensating controls, and risk appetite are known.

## 12. Candidate Control Matrix

The following matrix contains every field required by the plan. “039A,” “040,” and similar values are downstream topic IDs, not authorization to start them.

| Asset | API surface | Actor/threat actor | Trust boundary | Threat category | Example misuse | Candidate control | Authentication implication | Authorization implication | Logging/audit | Redaction/diagnostic | DDIL | Test/security-test | Downstream handoff | Notes/unresolved |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Resource objects | Read/item API | authenticated user | caller/API/store | BOLA | change System ID to read hidden System | repository policy predicate plus instance decision | valid identity/context required | `read` on exact object/view | actor, action, object digest, outcome | conceal existence consistently | local policy bundle must cover object | horizontal-ID permutation | 039A,040,055 | Never rely on opaque IDs |
| Protected properties | Read/write API | user/client | serializer/write pipeline | BOPLA/mass assignment | read location or set owner/policy field | property obligations; server-owned-field allowlist | bind client and principal | distinct property/update rules | rejected field classes, not values | omit fields and policy internals | bundle includes property rules | over-posting and response-field oracle tests | 040,052,055 | JSON schema alone is insufficient |
| Admin functions | Admin/test routes | ordinary user/stolen token | public/control plane | BFLA/elevation | invoke source registration or reset endpoint | separate audience/actions; production route disable; step-up hook | privileged identity and session | narrow admin/test action | all attempts and changes | generic denial; protect route inventory | normally online-required | verb/path/role matrix | 039A,047,055 | Avoid “admin=true” monolith |
| Credentials | All protected APIs | token thief | client/network/API | broken authentication/replay | replay bearer token to another audience | short life, exact audience, TLS, sender constraint for high risk | strict JWT/introspection/mTLS/DPoP | credential never grants beyond local policy | token digest/jti only | never log header/cookie/token | bounded local validation | expired, wrong-aud, alg/type, replay cases | 039A,047,055 | Exact lifetimes deferred |
| Browser session | Web/BFF | attacker/site script | browser/BFF/API | CSRF/XSS/session theft | cross-site state change or token exfiltration | BFF, secure cookies, CSRF, CSP/input encoding, outbound allowlist | code+PKCE; session rotation | user/client/action preserved | session digest, origin outcome | no cookie/token; generic UI error | generally connected | CSRF, redirect, XSS, proxy tests | 045,047,055,056 | Direct browser alternative needs DPoP consideration |
| Source authority | Ingestion | hostile/compromised publisher | publisher/API/write pipeline | spoofing/authority confusion | claim another source or resource owner | registered workload-to-publisher/source mapping; common pipeline | authenticate workload instance | `ingest` limited to mapping and type | publisher, source, mapping version, outcome | do not reveal other registrations | signed bounded registration bundle | cross-source, replay, revoked publisher | 040,042,055 | Credential and source remain distinct |
| Trust registry | Source registration | malicious admin/publisher | control plane/registry | elevation/tampering | self-register trusted publisher | protected admin workflow, separation/approval hook, immutable history | strong privileged auth | `register-source` separate from ingest | before/after digests and approvers | protect trust topology | constrained offline changes | self-registration and rollback tests | 039A,041,047,055 | Bootstrap lifecycle unresolved |
| Write state | Resource writes | authorized writer | API/write pipeline/store | validation bypass/race | set status/audit field or overwrite concurrent update | allowlist, schema+semantic validation, If-Match, transaction/idempotency | bind idempotency to security context | action/object/property recheck | request/decision/commit correlation | safe field-level errors | offline conflict policy deferred | over-post, concurrency, duplicate tests | 043,052,055 | Preserve IDR-029/031 |
| Query capacity | Search/dynamic data | abusive client | API/query engine/store | resource exhaustion | huge window, geometry, nesting, sort | cost budgets, page/window/depth limits, timeout, quota, index guard | quota identity/client/IP signals | authorized maximums/obligations | limit class and outcome | no query-plan/internal detail | stricter local budgets | adversarial complexity/load corpus | 046,048,054,055 | Avoid only request-count limiting |
| Parser capacity | All input | unauthenticated/publisher | network/parser | resource exhaustion | decompression bomb, huge array/geometry | body/decompressed-size, depth/item limits, streaming parser, timeouts | authenticate early where possible | capability-specific limits | limit category, payload digest | no echoed payload | local fixed budgets | compression/depth/size fuzzing | 044,052,054,055 | Enforce before allocation |
| Sensitive flow | Feasibility/command/register | automated/stolen client | API/workers/effect | unrestricted business flow | flood expensive analyses or approvals | per-action quotas, concurrency, admission, workflow state limits | client/principal attribution | permission plus contextual quota | attempts/rate/effect outcome | no capability oracle | offline class restricts | sustained and distributed flow tests | 048,054,055 | Valid requests can still be abuse |
| Command authority | Command API/dispatch | user/gateway/insider | API/decision/effect | confused deputy/TOCTOU | general write token causes dispatch | IDR-038 grants, gates, approval/interlock, single-use ticket, pre-dispatch recheck | strongest context/profile | specialized target/action/envelope authority | atomic decision/effect evidence | generic public reason | bounded offline command class | bypass, replay, stale gate, failover tests | 041,042,055 | API permission is not physical authority |
| Stream confidentiality | SSE/MQTT/change feed | subscriber/broker peer | API/broker/subscriber | stale auth/data leak | continue after revocation or replay another view's cursor | per-event filter, expiry timer, reauth/close, view-bound cursor/topic | authenticate handshake and renew/reconnect | subscribe/replay plus current event view | connect/filter/close reason | no hidden resource/event reason | bundle freshness bounds stream | revocation midstream, cursor swap, slow consumer | 040,042,048,054,055 | Broker ACL is defense in depth |
| Cache correctness | HTTP/query/cache | authenticated users | policy/query/cache | cross-tenant/view leak | privileged response served to another user | view-keyed cache/ETag/cursor, `private`/`no-store`, invalidation | include security context identity | include policy/attribute versions/obligations | cache decision metadata | no raw credential in key/log | bounded versioned cache | cross-role and policy-change tests | 040,045,055 | Shared caches need explicit proof |
| Schema/reference fetch | OpenAPI/schema/import | malicious document/user | server/outbound network | SSRF/unsafe consumption | reference loopback/cloud metadata or redirect | no arbitrary request-time fetch; vendored/pinned assets; allowlist scheme/host/IP; DNS/redirect recheck; size/time/digest/quarantine | service identity only to approved endpoints | separate import/admin permission | target class/digest/outcome | hide internal resolution/network | prefer offline bundle | rebinding, redirect, IP literal, large/recursive ref | 045,047,055 | Content still untrusted after fetch |
| File/artifact storage | Import/export/schema cache | malicious identifier | API/filesystem/object store | path traversal/overwrite | use `../` or encoded separator as artifact ID | opaque IDs; canonical resolved-root validation; atomic non-executable storage | authenticated uploader/importer | exact artifact action/namespace | artifact digest/path class | never disclose server path | local store same rule | encoding/symlink/platform path corpus | 044,047,055 | IDs are never filesystem paths |
| Database/query engine | Filters/writes | malicious caller | parser/domain/store | injection | embed operators or crafted text | typed AST, parameterized SQL, allowlisted sort/filter, no shell | identity aids attribution only | action still checked | normalized operation and error class | no SQL/stack trace | same local rule | fuzz and injection corpus | 044,052,055 | Validation and parameterization both required |
| Documentation renderer | OpenAPI/docs | malicious metadata/admin | stored content/browser | stored XSS/info disclosure | script in Markdown description | self-host UI; sanitize/escape active content; CSP; trusted generation | protect docs as profile requires | property/capability-filter definition | render rejection/digest | no secrets/internal endpoints | local static docs | active markup and hidden-route tests | 040,047,055 | OAS warns about rendering risks |
| Proxy identity | All proxied APIs | direct caller/compromised proxy | edge/API | header spoofing | forge principal or client IP header | block direct access; mTLS proxy; strip/rewrite exact headers; trusted-hop list | authenticate proxy and end principal evidence | normal server decision remains | proxy identity/hop/context digest | no raw assertions | local direct mode explicit | direct-backend, duplicate-header, hop-chain tests | 039A,046,055 | Network address alone is not trust |
| CORS/browser boundary | Browser-accessible API | hostile origin | browser/API | misconfiguration/CSRF | credentialed wildcard or overbroad origin | default deny; exact origins/methods/headers; CSRF for cookies | CORS is not authentication | CORS is not authorization | origin decision aggregate | no allowed-origin inventory publicly | usually irrelevant | preflight, null-origin, credential cases | 047,055 | Public APIs may use explicit permissive noncredentialed policy |
| TLS channel | External/internal API | network attacker | every transport hop | spoofing/tampering/disclosure | downgrade or terminate then plaintext untrusted hop | TLS 1.3 baseline, endpoint validation, protected backend hop, rotation | channel may contribute workload identity | never sufficient alone | protocol/cert class, not secrets | generic handshake failure | cached trust anchors need lifecycle | downgrade, hostname, expiry, rotation tests | 039A,046,047,055 | Compatibility exceptions profile-controlled |
| OpenAPI/capability | Definition/conformance | anonymous/recon actor | API/docs | inventory disclosure/drift | discover disabled admin/tasking route | registry-generated profile view, protect/redact, CI route parity | explicit public or authenticated profile | docs visibility is separately authorized | access/version/view digest | omit topology/security internals | bundle locally | route/spec/security-scheme parity | 040,050,055 | Accepted canonical OAS remains 3.1.2 |
| Logs/traces/metrics | Observability | insider/collector compromise | service/exporter/sink | secret/data leakage/tampering | capture token, payload, source or policy reason | pre-sink scrub, allowlisted fields, low-cardinality metrics, protected transport/access | diagnostic identity separate | `view-diagnostics`/audit permissions | immutable linkage and export outcome | token/payload never emitted | local bounded buffer | canary-secret and injection tests | 041,048,055 | Redact before formatter/exporter |
| Federation data | Peer API/sync | compromised/hostile peer | peer/API/local authority | unsafe consumption/transitive trust | import peer role/owner or malicious representation | bilateral mapping, local validation/quarantine, audience binding, no role inheritance | authenticate peer workload | explicit resource/action/domain mapping | peer/source/digest/decision | no other-domain topology | signed offline exchange rules | hostile peer, mapping change, replay tests | 040,042,043,056 | mTLS does not make content true |
| Security config/secrets | Deployment/admin | operator error/attacker | config/store/process | misconfiguration/supply chain | default secret, debug route, wrong issuer | schema validation, secret references, rotation, startup invariants, SBOM/signature/scanning | bootstrap identity controlled | configuration action separated | activation/version/digest | never display secret value | sealed local material lifecycle | unsafe combinations and secret scans | 046,047,055 | Product choices deferred |
| Audit evidence | Audit/store/export | insider/service failure | transaction/store/sink | repudiation/tampering/disclosure | erase denial or log full credential | atomic security event, append-oriented protection, access control, failure policy | preserve actor/client/delegation | audit access distinct from action | this is the evidence plane | minimize/pseudonymize sensitive fields | protected local queue/store | omission, reorder, tamper, sink-failure tests | 041,048,055 | General schema/retention belongs to 041 |

## 13. Ingestion, Source-Registration, Streaming, and Command/Control Security Implications

### 13.1 Ingestion and source registration

All ingestion enters the accepted common write pipeline. Authentication resolves the calling workload; registration resolves the publisher; source mapping resolves what origin it may represent; authorization resolves which operation/resource types and namespaces it may submit; validation resolves admissibility. None substitutes for another. Idempotency keys, batch ownership, sequence/replay state, and quota usage must be scoped to this security context so one publisher cannot collide with or probe another.

Source registration is a control-plane operation, not self-service merely because a publisher authenticates successfully. Register, approve/activate, suspend, rotate identity, change represented sources, and revoke are distinct auditable actions. Revocation prevents new admissions and causes active streams/sessions to re-evaluate; it does not silently rewrite previously accepted provenance. Quarantine records retain safe digests and internal reasons under protected diagnostic access.

### 13.2 Streaming and publication

The initial snapshot/query and the live stream must use the same authorized view. Each subscription binds the principal/client, selector, resource/property obligations, security domain, cursor namespace, policy/attribute versions, credential expiry, and rate/backpressure budget. Policy filtering occurs before an event crosses the subscriber boundary, including through MQTT or another adapter. Topics are not authorization tokens.

At credential expiry or mandatory re-evaluation, the server either validates renewed context according to the protocol or closes and requires reconnect. Policy, trust, source-registration, or membership changes trigger bounded re-evaluation. Replay cannot reveal events no longer authorized; opaque cursors are scoped to the authorized view and cannot be exchanged across principals or policy views. Slow-consumer controls and bounded queues prevent an authenticated subscriber from exhausting the server.

The disabled-by-default outbound experimental Part 3 adapter accepted by IDR-SRV-035 inherits these rules. This report does not enable inbound draft Part 3 channels or claim approved OGC conformance.

### 13.3 Command and control

The command path composes general authentication/API authorization with every accepted IDR-SRV-038 control. General `submit-command` route permission is only an entry gate. `CommandAuthorityGrant`, optional `ControlAuthorityLease`, policy/releasability, validation, feasibility where required, interlocks/safety, approval/override, target/gateway trust, and immediate pre-dispatch re-evaluation remain independent. Dispatch consumes a short-lived single-use ticket bound to the exact command revision, target, contract, decisions, route, deadline, and fencing value.

Gateway authentication proves which gateway is connected; it does not authorize arbitrary targets, synthesize operator identity, accept commands, or report status/results outside its grant. Broker authentication and ACLs are defense in depth. Simulator identities and fixtures cannot bypass gates or reach live targets. Failure of mandatory authentication, authorization, policy, audit, or fresh decision evidence fails closed before authoritative admission or effect according to the accepted transaction boundary.

## 14. OpenAPI, Schema, Conformance, and Documentation Security Implications

### 14.1 Generated API descriptions

The accepted Glaux canonical OpenAPI version remains 3.1.2. OpenAPI 3.2.1 is a monitored current target, not an automatic project upgrade. The capability registry should generate routes, operations, security requirements, conformance declarations, and profile-specific documentation together so a disabled or protected operation cannot remain advertised accidentally. Reusable security schemes describe how a client may authenticate; per-operation Security Requirement Objects express alternatives or explicit public access. They do not prove runtime enforcement or grant access.

OpenAPI explicitly recognizes security filtering as a deployment concern. Glaux may publish a deliberately public capability view, require authentication for a full view, or generate policy/profile-specific views. It must not fabricate conformance or omit required standards semantics merely to hide an implementation defect. When capability disclosure is sensitive, protect the definition rather than making runtime and documentation contradict one another.

Definitions must exclude example secrets, access tokens, cookies, internal issuer administration URLs, private hostnames, trust topology, hidden policy labels, internal database identifiers, command safety rules, stack traces, and operational credentials. Examples use obvious synthetic values. Security scheme descriptions state the intended deployment profile and scopes/actions without promising a specific production IdP.

### 14.2 Schemas, references, and rendering

Schema and documentation references are supply-chain and SSRF boundaries. Production builds should vendor or pin approved OGC/project artifacts with version and digest. Arbitrary request-time external reference resolution is prohibited. An administrative import path, if later supplied, must allowlist protocols, hosts and resolved address classes; revalidate DNS and redirects; bound size, recursion, time, content type and decompression; verify expected digest/signature where available; quarantine before activation; and prevent file/path traversal.

Cycles and deep reference graphs need deterministic bounds. Markdown, HTML, URI, example, title, and description fields are untrusted presentation data. Documentation UIs should be self-hosted where feasible, escape/sanitize active content, use a restrictive content-security policy, and avoid sending definitions or credentials to third-party renderers.

### 14.3 Conformance and test surfaces

OGC conformance requirements remain standards obligations even where authentication method is deployment-selected. The conformance harness receives an authorized fixture principal and profile, discovers only the intended conformance surface, and records authentication requirements as harness configuration. A test-only anonymous bypass would test the wrong server. Reset, seed, clock, fault-injection, and privileged fixture routes remain isolated, separately authenticated, namespace-scoped, and absent/disabled in production profiles.

## 15. DDIL, Federation, Cached Credentials, Revocation, and Offline Authorization Implications

### 15.1 Locally verifiable security state

An operational Glaux node must not silently assume permanent identity-provider, introspection, policy-service, certificate-status, federation, or time-service connectivity. An offline-capable profile should preload signed, versioned credential validation material and trust/policy/security bundles containing issuer, audience, supported credential/algorithm profile, subject/client/source mapping, permitted scope, activation and expiry, revocation epoch, maximum offline age, required time confidence, and allowed offline operation class.

| Offline class | Candidate behavior |
|---|---|
| Online-required | Deny when required authority/freshness cannot be reached; use for administration and unbounded/high-risk changes |
| Bounded local read | Permit only policy-filtered cached/local resources while credential and bundle remain valid |
| Bounded local ingest | Permit registered source submissions within explicit resource, size, rate, time, and replay bounds |
| Bounded local command authority | Only the exact pre-authorized IDR-SRV-038 class; re-evaluate locally immediately before dispatch |
| Simulation/test only | Permit effects solely inside cryptographically/configurationally isolated synthetic namespace |

Unknown or untrusted time, expired credential/bundle, audience mismatch, exceeded maximum staleness, missing mandatory attribute, or uncertain revocation fails closed for the affected class. “Last known allow” is not an indefinite permission. Offline decisions record bundle versions, local clock confidence, revocation epoch, decision expiry, and why connected validation was unavailable. IDR-SRV-042 will finalize externally visible degraded/stale semantics.

### 15.2 Revocation and caching

Connected operation should consume revocation/introspection and policy/trust changes within a bounded profile-defined interval. Cache keys include issuer, token digest, audience and relevant context; cached activity never extends beyond token expiry. Offline revocation uses signed epoch/snapshot/delta material with sequence and anti-rollback protection. Reconnection imports newer security state before expanding authority, re-evaluates long-lived subscriptions and queued effects, and does not retroactively relabel legitimate historic provenance.

Numeric TTLs, grace periods, assurance requirements, emergency authority, and clock-error bounds require deployment risk analysis and prototype validation. The architecture must make them explicit configuration with safe ranges, not hidden constants.

### 15.3 Federation

A federation connection authenticates the peer workload and separately identifies the represented organization/security domain/source. Local bilateral mappings define accepted audiences, resources, actions, labels and transformations. Glaux never imports a peer's roles, ownership, trust, or command authority transitively. Peer assertions and resource content pass normal parsing, validation, policy, source-authority, anti-replay, and quarantine controls.

mTLS plus audience-bound OAuth or signed assertions are candidates, not a final selection. Federation errors and catalogs must not disclose other peers, internal topology, local role names, policy structure, or hidden resource existence. Delayed exchanges preserve issuer/domain, source, sequence, timestamps, credential/bundle versions, signature/digest evidence, and local admission decisions for conflict handling in IDR-SRV-043.

## 16. Error, Logging, Metrics, Tracing, Redaction, and Observability Implications

### 16.1 Public diagnostics

Security failures should use RFC 9457-compatible problem details where applicable with stable type, title/status, a safe generalized detail, and a correlation identifier. They must not expose token-validation steps, accepted issuers, membership, policy expressions, source mappings, command authority, safety/interlock reasons, database/query internals, stack traces, filesystem paths, upstream credentials, or hidden resource existence. More detailed reasons belong in protected decision/audit evidence and are themselves policy-controlled.

Authentication failures should avoid revealing whether a subject, key ID, account, source registration, or resource exists. Rate-limit responses disclose only safe retry information. Validation errors identify usable field/location and rule classes only when that information is authorized and cannot expose hidden schema/capability. Batch diagnostics apply per-item redaction.

### 16.2 Security logging and audit linkage

Security events should carry a generated event/correlation ID; authenticated principal/client/workload and represented identity IDs; credential/session/token digest identifier, never the credential; action/resource type and protected object digest or authorized identifier; decision/outcome; policy/attribute/trust/source-map versions; deployment/security domain; network/proxy evidence in normalized form; time; request/idempotency/transaction/effect links; and internal reason code under protected access.

Authorization headers, cookies, bearer/API keys, private keys, DPoP proofs, password fields, raw identity assertions, full sensitive payloads, query-string credentials, command parameters, sensitive geometry, and policy text are never logged by default. Redaction occurs before formatter, buffer, exporter, crash reporter, or third-party sink. Payload and artifact digests support correlation where appropriate. Log/control characters and untrusted identifiers are encoded to prevent forged lines or fields.

General audit event schema, retention, integrity protection, failure response, time synchronization, access, export and legal/accountability decisions remain IDR-SRV-041. This report requires that security decisions be linkable and that an unavailable mandatory audit commit cannot silently permit high-consequence authoritative work already designated fail-closed by accepted reports.

### 16.3 Metrics, tracing, and health

Metrics should be aggregate and low cardinality: authentication successes/failures by safe reason class and profile, authorization allow/deny/error, rate/size/complexity rejection, active protected streams, reauthorization closure, policy/trust version age, introspection latency/error/cache age, certificate/key expiry horizon, redaction failures, and audit/export backlog. Principal IDs, tokens, resource IDs, free-form error text, URLs with query data, source names, and command IDs are not metric labels.

Trace propagation accepts only validated bounded formats at the edge and regenerates or sanitizes baggage. Security context is referenced by safe IDs, not copied as raw claims. Export destinations are allowlisted, authenticated, policy-approved, and backpressure bounded. Public liveness is minimal; readiness and dependency/security detail are internal and must distinguish degraded identity/policy/audit dependencies without leaking topology.

## 17. Fixture, Conformance, Security Testing, Performance, and Interoperability Test Implications

### 17.1 Fixture corpus

The security fixture set should generate deterministic ephemeral signing keys, certificate chains, token/session IDs, principals, clients, publishers, sources, peers, gateways, roles, relationships, policies, revocations, clock states, and security domains. It needs personas for anonymous, permitted, partially permitted, denied, expired, revoked, wrong-audience, delegated, cross-domain, administrator, operator, publisher, simulator, peer, gateway, and compromised-client cases. No real secret, certificate, endpoint credential, operational identifier, or controlled content may appear.

Malicious fixtures should cover duplicate/ambiguous headers, conflicting claims, algorithm/type confusion, key rotation, invalid proof binding, replay, forged proxy headers, object-ID permutation, hidden-property over-posting, cursor/ETag swapping, large/deep/compressed payloads, filter complexity, path encodings, SSRF targets/redirects/DNS rebinding, active Markdown/HTML, log injection, hostile federation content, and stale/rollback DDIL bundles.

### 17.2 Verification layers

| Layer | Required evidence |
|---|---|
| Unit/property | parser bounds, token/JWT rules, action mapping, policy defaults, cache key separation, redaction invariants |
| Component | authenticators, mapper, authorizer, query predicates, serializers, source registry, stream reevaluation, proxy contract |
| Integration | identity/policy outage, key/cert rotation, revocation, transaction/audit linkage, broker/gateway/federation boundaries |
| Conformance | authorized OGC scenarios with security profile; no test bypass; protected/public metadata behavior |
| Security | OWASP/STRIDE negative matrix, fuzzing, abuse flows, SSRF/path/injection, privilege escalation, secret leakage |
| Performance/load | auth/authorization latency, cache correctness, introspection outage, stream churn, revocation fan-out, adversarial query cost |
| Interoperability | CSAPI Explorer, OS4CSAPI, SECD, publishers/adapters, web/mobile, gateways and peers with explicit credentials/profile |

Every allow test needs a nearby deny or constrained-view test. Tests must verify response body, headers, links, counts, extents, timing class, cache behavior, event stream, logs, traces, metrics and durable evidence—not merely the HTTP status. Fuzz and load failures must remain bounded and must not cause the service to disable authentication, relax authorization, leak diagnostics, or cross-contaminate security contexts.

### 17.3 Interoperability posture

Glaux should publish test-profile instructions and synthetic credentials out of band, advertise supported security schemes truthfully, and distinguish an implementation's inability to authenticate from CSAPI semantic nonconformance. Clients that support only anonymous access may exercise an intentionally public profile but cannot drive protected write, ingestion, administration, federation, or command scenarios. Compatibility must not create an unbounded fallback credential or wildcard CORS policy.

## 18. Downstream Topic Handoff Matrix

| Topic | Required handoff from IDR-SRV-039 | Boundary preserved |
|---|---|---|
| IDR-SRV-039A | SecurityContext and pluggable authn/authz baseline; trust boundaries; enforcement-point inventory; proxy/service/federation identity; continuous re-evaluation needs | Select detailed zero-trust components, placement, zones, and enforcement topology; may propose bounded 039 addendum |
| IDR-SRV-040 | Authorized-view semantics; property/link/count/event/error/document disclosure risks; security-domain and policy-version inputs | Define labels, releasability, transformations, cross-boundary policy and concealment |
| IDR-SRV-041 | Required security event linkage, actors, actions, outcome, decision/version evidence, redaction and fail-closed hooks | Define general audit model, integrity, retention, access, export, time and accountability |
| IDR-SRV-042 | Offline classes; locally verifiable bundles; freshness/time/revocation failures; stream/effect re-evaluation | Define final DDIL-visible semantics, degraded operation and numeric bounds |
| IDR-SRV-043 | Peer/offline provenance, anti-replay, revocation/policy version and reconnect requirements | Define synchronization, conflict resolution and anti-resurrection behavior |
| IDR-SRV-044 | Framework-independent authenticator/mapper/authorizer contracts; parser and crypto-validation requirements | Select Rust language/framework/library strategy after security review |
| IDR-SRV-045 | Enforcement-point list, request context, decision obligations, BFF/outbound mediation needs | Define services/modules and prevent bypass paths |
| IDR-SRV-046 | Deployment profiles, audiences, TLS, proxy, internal/public surfaces, IdP dependency | Select reference deployment topology and HA boundaries |
| IDR-SRV-047 | Issuer/audience/trust/CORS/proxy/unsafe-dev invariants; secret lifecycle and rotation needs | Define configuration/secrets/environment strategy and validation |
| IDR-SRV-048 | Safe event fields, redaction-before-export, metrics cardinality, tracing and health protections | Define logs/metrics/traces/health implementation and sinks |
| IDR-SRV-050 | Authorized conformance fixture and protected documentation/conformance behavior | Define harness without security bypass |
| IDR-SRV-052 | Test seams and invariants for security context, policies, queries, streams and failures | Define Rust multi-layer TDD architecture |
| IDR-SRV-053 | Synthetic identity/credential/threat corpus requirements | Define fixture/golden/scenario corpus with no real secrets |
| IDR-SRV-054 | Query/parser/stream/auth service exhaustion and security-cache performance cases | Define load/stress methodology without relaxing controls |
| IDR-SRV-055 | Full threat/control matrix and negative-test cases; command composition | Define detailed security/authorization/command test strategy |
| IDR-SRV-056 | Client/profile credential negotiation and secure interop scenarios | Define external-client matrix and classify auth vs semantic failures |
| Final synthesis | Recommended baseline, unresolved selections and evidence pins | Decide implementation sequencing only after intervening reports |

No row authorizes its topic. Under the single-topic workflow, acceptance of this report may authorize only the next topic explicitly selected by the project lead.

## 19. Recommendations

1. Accept one server-owned `SecurityContext`, authenticator, principal-mapper, and authorizer abstraction independent of identity product and Rust framework.
2. Deny by default and combine administrative RBAC, delegated scopes, ABAC and resource relationships; treat claims as inputs rather than authority.
3. Enforce decisions at route, query/store, write, serialization/link, cache/cursor, stream/event, worker, admin and command-dispatch boundaries.
4. Adopt short-lived exact-audience credentials, asymmetric client authentication where feasible, and sender-constrained tokens for replay-sensitive/high-risk profiles.
5. Support both strict local JWT validation and protected opaque-token introspection behind one contract, with explicit bounded freshness and revocation semantics.
6. Prefer a BFF for the Glaux web application; otherwise use authorization code with PKCE and evaluate DPoP. Prohibit implicit/password grants.
7. Restrict API keys and Basic authentication to explicit narrow bootstrap, legacy, development or test profiles; never use them as the sole production command identity.
8. Make unauthenticated development an explicit loopback-only `unsafe-dev` profile with ephemeral state and dangerous capabilities disabled; CI uses real security interfaces with ephemeral fixtures.
9. Use TLS 1.3 as the default baseline for new Glaux deployment profiles and protect every hop; define any compatibility exception explicitly.
10. Generate OpenAPI/conformance/security declarations from the capability registry, keep accepted OpenAPI 3.1.2 canonical, and monitor 3.2.1 through release governance.
11. Prohibit arbitrary request-time schema/reference fetching and apply allowlist, re-resolution, bounds, digest and quarantine controls to managed imports.
12. Bind every list, property, link, count, extent, cache, cursor, ETag and event to the authorized view; never fetch broadly and filter only after authority access.
13. Authenticate subscriptions at admission and re-evaluate them at expiry and policy/trust changes; filter each event before delivery.
14. Preserve accepted source/publisher distinctions and every IDR-SRV-038 command gate; neither mTLS, broker ACL nor general API permission is source or command authority.
15. Prepare locally verifiable, signed and versioned DDIL security state with explicit offline classes, time/freshness/revocation bounds, and fail-closed uncertainty.
16. Redact before any log/trace/metric/export boundary, retain safe decision linkage, and verify absence of credentials and protected payloads with canary fixtures.
17. Require negative security tests adjacent to successful cases and include response metadata, caches, events and observability evidence in assertions.
18. Defer final IdP, PKI, policy engine, topology, assurance levels, numeric lifetimes and accreditation tailoring to their owned topics and prototypes.

## 20. Risks, Constraints, and Open Questions

| Item | Consequence | Resolution owner/approach |
|---|---|---|
| Production IdP, authorization server and enterprise directory are unselected | Adapter behavior and operational dependencies unknown | 039A/046 plus stakeholder integration decision |
| Authentication/authorization Rust libraries unselected | Validation correctness and framework integration unproven | 044/045 security-focused prototype |
| PKI/workload identity and certificate revocation design unselected | mTLS operations, rotation and DDIL behavior uncertain | 039A/046/047/042 |
| Exact IAL/AAL/FAL, MFA and step-up requirements unknown | Human/privileged assurance cannot be finalized | Organizational risk/accreditation decision informed by SP 800-63-4 |
| Role, attribute and relationship vocabulary not organizationally approved | Policy administration may diverge | 039A/040 stakeholder workshop and fixtures |
| Token/session/cache/revocation numeric bounds unknown | Availability-security tradeoffs unresolved | Threat-informed prototype/load/DDIL tests in 042/054/055 |
| Browser architecture not selected | BFF operational cost versus browser token exposure remains | 045/046 webapp prototype |
| Fine-grained query pushdown may be complex | Post-filtering can leak counts or exhaust resources | 045/052 database-policy prototype; prohibit unsafe fallback |
| Policy changes during long-lived streams | Unauthorized continuation or availability churn | 040/042 define triggers and bounds; 054/055 test fan-out |
| Public documentation/conformance posture varies by deployment | Capability disclosure or unusable clients | 040/046 define profile views; 050/056 validate |
| Offline time/revocation confidence varies | Stale authority can persist or safe work can stop | 042 defines classes/bounds; default deny for unknown high-risk state |
| Federation partners may use incompatible identity claims | Mapping ambiguity and transitive trust risk | Bilateral local mapping; 039A/040/043/056 |
| Security controls can become availability bottlenecks | IdP/PDP/introspection outage or latency | bounded caches, circuit isolation, local validation where authorized; 046/054 |
| Error concealment can impede operations | Troubleshooting pressure may cause leaks | safe correlation plus protected audit/diagnostic view in 041/048 |
| OAS 3.2.1 is newer than accepted canonical 3.1.2 | Premature upgrade could drift artifacts/tooling | Monitor under IDR-014 release gate; no automatic change |
| Implementation exemplars have permissive/test-only security | Copying behavior could create unsafe defaults | Treat as gap evidence and verify Glaux independently |

Residual risk remains until the selected identity, policy, transport, storage, deployment and observability components are implemented and adversarially tested. Acceptance of this research establishes requirements and recommendations, not an accreditation, production authorization, or claim of security.

## 21. Validation Against This Plan's Success Criteria

| Success criterion | Evidence | Result |
|---|---|---|
| Protected assets, actors, identities, surfaces and trust boundaries with anchors | Sections 3, 5, 6, 7 | Met |
| Authentication/authorization options for all named contexts | Sections 8, 9, 10 and deployment matrix | Met |
| Threats/controls across CSAPI, ingestion, streaming, registration, command, admin, docs, observability and DDIL | Sections 11-16 and 15-column matrix | Met |
| Object/function authorization, safe diagnostics, redaction, audit linkage and source/command implications | Sections 9, 10, 13, 16 | Met |
| Deployment, secrets, TLS, proxy, CORS, observability, fixture, conformance, security, performance and interop implications | Sections 8, 12, 14-17 | Met |
| Implementation/community lessons incorporated non-normatively | Section 3.3 and threat/control conclusions | Met |
| Recommendations decision-usable and Glaux-bounded | Sections 18-20 | Met |
| Downstream handoffs explicit | Section 18 | Met |
| References explicit and reproducible | Metadata, Section 3.4 and Section 22 | Met |

Structural validation confirmed all 22 required numbered sections and all 15 mandated threat-matrix columns. Scope validation confirmed that the report selects neither a production identity provider nor a deployment topology, exposes no controlled AEP content or real credential, preserves accepted CSAPI/OpenAPI and command decisions, and does not authorize implementation or IDR-SRV-039A.

## 22. References

### 22.1 Standards and primary security guidance

- OGC API - Connected Systems - Part 1: Feature Resources, OGC 23-001: https://docs.ogc.org/is/23-001/23-001.html
- OGC API - Connected Systems - Part 2: Dynamic Data, OGC 23-002: https://docs.ogc.org/is/23-002/23-002.html
- Approved CSAPI Version 1.0 source tag: https://github.com/opengeospatial/ogcapi-connected-systems/tree/v1.0.0
- OGC API - Features - Part 1, OGC 17-069r4: https://docs.ogc.org/is/17-069r4/17-069r4.html
- OGC SensorML Encoding Standard 3.0, OGC 23-000: https://docs.ogc.org/is/23-000/23-000.html
- OGC SWE Common Data Model Encoding Standard 3.0, OGC 24-014: https://docs.ogc.org/is/24-014/24-014.html
- OpenAPI Specification 3.2.1 (current monitored target): https://spec.openapis.org/oas/latest.html
- OpenAPI Specification 3.1.1 security considerations (3.1-series evidence): https://spec.openapis.org/oas/v3.1.1.html
- RFC 9110, HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9457, Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457
- RFC 8446, TLS 1.3: https://www.rfc-editor.org/rfc/rfc8446
- BCP 195, Recommendations for Secure Use of TLS and DTLS: https://www.rfc-editor.org/info/bcp195
- RFC 7662, OAuth 2.0 Token Introspection: https://www.rfc-editor.org/rfc/rfc7662
- RFC 8705, OAuth 2.0 Mutual-TLS Client Authentication and Certificate-Bound Access Tokens: https://www.rfc-editor.org/rfc/rfc8705
- RFC 8725, JSON Web Token Best Current Practices: https://www.rfc-editor.org/rfc/rfc8725
- RFC 9068, JWT Profile for OAuth 2.0 Access Tokens: https://www.rfc-editor.org/rfc/rfc9068
- RFC 9449, OAuth 2.0 Demonstrating Proof of Possession: https://www.rfc-editor.org/rfc/rfc9449
- RFC 9700 / BCP 240, Best Current Practice for OAuth 2.0 Security: https://www.rfc-editor.org/rfc/rfc9700
- RFC 10017, OAuth 2.0 for Browser-Based Applications: https://www.rfc-editor.org/rfc/rfc10017
- OpenID Connect Core 1.0: https://openid.net/specs/openid-connect-core-1_0.html
- OWASP API Security Top 10 - 2023: https://api-security.owasp.org/editions/2023/en/0x11-t10/
- OWASP Threat Modeling: https://community.owasp.org/Threat_Modeling
- OWASP Application Security Verification Standard 5.0.0: https://owasp.org/projects/asvs
- NIST SP 800-63-4, Digital Identity Guidelines: https://csrc.nist.gov/pubs/sp/800/63/4/final
- NIST SP 800-207, Zero Trust Architecture: https://csrc.nist.gov/pubs/sp/800/207/final
- NIST SP 800-162, Guide to Attribute Based Access Control: https://csrc.nist.gov/pubs/sp/800/162/upd2/final
- NIST SP 800-53 Rev. 5, Security and Privacy Controls: https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final
- NIST SP 800-92, Guide to Computer Security Log Management: https://csrc.nist.gov/pubs/sp/800/92/final

### 22.2 Project and implementation evidence

- Glaux Server Overall IDR Research Plan: ../IDR%20Plans/overall-idr-research-plan.md
- IDR-SRV-039 Research Plan: ../IDR%20Plans/idr-srv-039-authentication-authorization-and-api-security-threat-model.md
- Accepted IDR-SRV-038 report: ./idr-srv-038-command-authorization-safety-and-audit-strategy-report.md
- Glaux Server Goal and Definition: ../../../../../Plans/glaux-server/glaux-server-goal-and-definition.md
- CS-Go `v1.0.4`: https://github.com/SomethingCreativeStudios/connected-systems-go/tree/v1.0.4
- OpenSensorHub `v2.0.2`: https://github.com/opensensorhub/osh-core/tree/v2.0.2
- OS4CSAPI client research, phase-9: https://github.com/OS4CSAPI/ogc-client-CSAPI_2/tree/phase-9
- SECD interoperability repository: https://github.com/Sam-Bolling/csapi-server-interop-secd
- CSAPI Explorer: https://ogc-csapi-explorer.pages.dev/

### 22.3 Evidence limitations

Implementation sources are observations, not normative security authorities. Mutable web sources were checked on the date stated in the report metadata. The controlled AEP artifact is identified in project governance records and is intentionally not linked or reproduced here. Security guidance establishes design and verification inputs; organizational risk acceptance, control tailoring, cryptographic module requirements, and authorization to operate remain outside this research report.

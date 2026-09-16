# Section 055: Security, Authorization, and Command-Control Test Strategy - Research Report

**Topic ID:** IDR-SRV-055<br>
**Report Status:** Final<br>
**Research Plan:** [IDR-SRV-055 Security, Authorization, and Command-Control Test Strategy](../IDR%20Plans/idr-srv-055-security-authorization-and-command-control-test-strategy.md)<br>
**Overall Research Plan:** [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)<br>
**Research Questions Covered:** security-test architecture and taxonomy; synthetic identity, source, policy and command fixtures; authentication and authorization; object/property/function access; disclosure and non-interference; ingestion and source trust; configuration, secrets and DDIL; command discovery, gates, safety, lifecycle and effects; streaming and abuse; tools, CI, evidence and downstream handoffs<br>
**Methodology Used:** accepted-requirement extraction; threat/control/test trace construction; current primary security-standard and tool review; boundary, misuse, metamorphic and state-machine analysis; artifact-sensitivity review; implementation/community lesson reconciliation; downstream synthesis<br>
**Research Time:** Approximately 48 hours of AI-assisted execution on September 16, 2026<br>
**Accepted Verification Baseline:** IDR-SRV-050 through IDR-SRV-054 independent conformance, normalized traceability, multi-layer Rust TDD, governed deterministic corpus and envelope-bound performance strategies<br>
**Current Tool Evidence:** OWASP ZAP `2.17.0`, cargo-audit `0.22.2`, cargo-deny `0.20.2`, cargo-fuzz `0.13.2`, OPA `1.20.2`, secrecy `0.10.3`, zeroize `1.9.0`; official documentation and release metadata checked September 16, 2026<br>
**Document Purpose:** Define a reproducible implementation security and simulated command-control test strategy without implementing tests, using real credentials or operational data, enabling physical effects, performing unauthorized penetration testing, or claiming accreditation/readiness<br>
**Author:** OpenAI Codex<br>
**Date:** September 16, 2026<br>
**Last Updated:** September 16, 2026

---

## Evidence and Decision Legend

- **[N] Normative:** approved external standard or normatively incorporated artifact.
- **[A] Accepted project baseline:** accepted Glaux report or governing project decision.
- **[D] Direct documentation:** official security, protocol, tool or runtime documentation.
- **[I] Implementation evidence:** another implementation or community result; informative only.
- **[T] Test evidence:** reproducible security observation with frozen target, fixtures and method.
- **[E] Analysis:** reasoned synthesis from identified evidence.
- **[P] Project recommendation:** proposed Glaux decision pending acceptance of this report.
- **[X] Explicit boundary:** excluded claim or later-topic responsibility.

Authentication is not authorization; authorization is not data disclosure; source trust is not command authority; feasibility is not safety; a broker acknowledgement is not a physical effect; a clean scanner report is not security assurance. The test strategy preserves every one of those separations. **[A,E,P]**

---

## Contents

1. Executive Summary
2. Scope and Plan Alignment
3. Evidence Base and Authority Classification
4. Security and Command Test Requirement Extraction Methodology
5. Security Test Taxonomy
6. Test Identity, Role, Source, and Policy Fixture Findings
7. Authentication and Authorization Test Findings
8. Object-Level Access and Policy/Releasability Test Findings
9. Redaction, Side-Channel, Error, Logging, Metrics, and Trace Leakage Test Findings
10. Source Trust, Ingestion, Configuration/Profile, Secret, and DDIL Findings
11. Command Discovery, Authorization, Feasibility, Safety, and Denial Findings
12. Command Lifecycle, Simulated Gateway, Event, and Audit Evidence Findings
13. Abuse, Rate-Limit, Resource-Exhaustion, and Streaming Security Findings
14. Tooling, CI Gates, Evidence, and Redaction Artifacts
15. Downstream Topic Handoff Matrix
16. Recommendations
17. Risks, Constraints, and Open Questions
18. Validation Against This Plan's Success Criteria
19. References

---

## 1. Executive Summary

Glaux should implement a **deny-by-default, trace-driven security test program** in which each accepted requirement or risk is connected to an asset and trust boundary, a threat, a control and enforcement point, positive and negative tests, fixtures, evidence, a CI tier and any deviation. The graph extends IDR-SRV-051 rather than creating a separate security spreadsheet. New routes, actions, resource kinds, properties, stream events and effect adapters must fail CI until they register their policy action/enforcement point and at least one denial case. **[A,E,P]**

The first identity harness has two intentionally different modes. Pure policy and domain tests inject an immutable synthetic `SecurityContext`; those tests prove decision logic, not authentication. HTTP-boundary tests use a small deterministic local issuer/JWKS service with ephemeral per-run keys, loopback or isolated networking, exact issuer/audience/token-type profiles and controllable rotation, expiry and outage. Static bearer tokens remain test-only. A future release test may add one pinned standards-compliant identity-provider adapter, but this report does not select an enterprise identity product. **[A,D,P]**

Authentication tests cover missing and malformed credentials, signature and algorithm confusion, issuer/audience/type errors, time claims and skew, `kid` ambiguity, JWKS rotation/rollback/cache/outage, cross-profile token use, oversized or duplicate claims, and optional certificate/token binding. The server accepts only explicitly configured algorithms, token types, issuers and audiences. Discovery/JWKS retrieval has fixed origins, TLS, redirects, media type, size, key-count and cache bounds; token content can never nominate an arbitrary retrieval URL. Raw tokens, private claims and keys must not appear in responses, logs, audit, traces or reports. **[D,E,P]**

Authorization coverage is semantic, not merely endpoint-based. It crosses subject, client/delegation, action, resource/relationship, property/link, policy view, profile, time/DDIL state, source trust and command evidence. Exhaustive matrices cover high-risk boundaries; pairwise/model-based generation covers the broader cross-product. Horizontal object access, vertical/function access, property writes, mass assignment, alternate methods/representations, relationship traversal and admin/raw/audit surfaces all require explicit negative cases. **[A,E,P]**

Policy-disclosure tests use **twin-world non-interference**: two otherwise identical stores differ only in a hidden record or value, and the unprivileged observer must receive equivalent status, headers, public body, counts, extents, ordering, pagination, cursors, validators, latest values, links, events, documentation and bounded timing class. Expected public facts remain separate from the restricted oracle. Synthetic canaries make accidental leakage detectable without copying hidden values into public golden files. Timing comparison is diagnostic and statistical; it cannot prove absence of all side channels. **[A,E,P]**

Source and ingestion tests prove that authenticated transport identity maps to an authorized publisher/source and cannot self-assert a different identity or authority. They cover unknown, stale, suspended and rotated credentials; replay, duplication and identifier collision; schema/media/size/depth/numeric/Unicode boundaries; decompression and XML hazards where applicable; raw-reference SSRF/path traversal; quarantine isolation; and retention of untrusted policy assertions as provenance rather than authority. Parser and state-machine fuzz failures are minimized and promoted to the governed corpus when safe. **[A,D,P]**

Configuration tests enumerate invalid profile combinations. Authentication cannot be disabled outside `local_unsafe`; static tokens cannot enter demo/operational profiles; public binding cannot silently combine with unsafe local mode; physical command adapters are absent; commands are `disabled` or `simulated`; inbound Part 3 is off; public debug/admin/metrics exposure is rejected; proxy trust is explicit; and secret values are references, not ordinary configuration. A synthetic secret canary is checked across arguments, effective configuration, errors, logs, audit, traces, metrics, reports and panic/crash artifacts. Repository secret scanning is defense in depth and never evidence that no secret exists. **[A,D,P]**

DDIL tests verify signed bundle audience/version/expiry/revocation epoch, anti-rollback and time-confidence rules. Loss of identity or policy connectivity never expands authority. Unknown, stale or indeterminate inputs deny unless an accepted, locally verifiable offline command class explicitly permits action; queued commands are re-authorized and revalidated before dispatch. Audit-capacity loss gates protected actions, and synchronized records never inherit a remote authorization decision. **[A,P]**

Command assurance is an ordered-gate and zero-effect problem. Tests independently vary API permission, structure, semantics, command authority, disclosure policy, feasibility, safety/interlocks, approval, target/gateway trust and the final pre-dispatch decision. No gate may mint evidence for another. Every denied permutation yields a policy-safe public result, protected decision/audit evidence and exactly zero recorded effects. Safety rules are versioned synthetic fixtures with applicability, severity, overrideability, required evidence/freshness and outcome. **[A,E,P]**

Accepted commands receive a short-lived, single-use dispatch ticket bound to command and revision, target, contract, policy/safety/trust versions, route, deadline and fencing epoch. Tests mutate each binding after admission, replay tickets, race cancellation/approval, expire evidence and inject gateway faults. The effect adapter is a non-network `sim://` recorder; builds and test profiles reject other schemes and fail if command code attempts DNS or network access. A gateway simulator can accept, reject, delay, duplicate, reorder, lose or return unknown outcomes without resolving an external endpoint. **[A,P]**

PR gates run deterministic route/action inventory, authentication/authorization/disclosure/configuration/command-safety unit and integration tests, security regressions, cargo-audit, cargo-deny and secret scanning. Nightly jobs broaden property/model tests, fuzzing, source/DDIL/streaming matrices and passive ZAP baseline scanning. Active ZAP API/full scans run only in isolated disposable manual/release-candidate environments with synthetic data, constrained credentials, safe resets and no physical adapter. External assessment and accreditation remain future evidence classes. ZAP, dependency auditing, secret scanning and fuzzing are useful defect detectors, never semantic authorization or assurance oracles. **[D,E,P,X]**

Each run emits a sanitized `SecurityTestResultV1`; restricted captures, if indispensable, have separate access and short retention. Results cross-reference authoritative audit evidence rather than copying it. No unresolved issue blocks implementation of this strategy. Operational identity products, policy content, command targets, accreditation controls and penetration scope require later stakeholder authority. Acceptance authorizes none of IDR-SRV-056, implementation, testing of public/external systems, physical effects or an accreditation/readiness claim. **[P,X]**

## 2. Scope and Plan Alignment

### 2.1 Completed Scope

This report completes the six authorized phases by extracting accepted security obligations, defining test taxonomy and fixtures, specifying authentication/authorization/disclosure and source/configuration/DDIL cases, modeling command gates/lifecycle/effects, evaluating the named tools and producing CI, evidence, matrix, proof and handoff decisions.

### 2.2 Explicit Boundaries

This report does not implement tests or controls; select an operational identity, policy or SIEM product; use real identities, credentials, labels, data, sources or targets; authorize active scanning of public or third-party systems; make a simulator capable of physical dispatch; substitute testing for design review or external assessment; define an ATO/accreditation package; or authorize later research. **[X]**

### 2.3 Research Question Coverage

| Plan theme | Status | Evidence |
|---|---|---|
| scope, sources and traceability | Complete | Sections 3–5 |
| identities, roles, sources, policies and command fixtures | Complete | Section 6 |
| authentication, authorization and object/property access | Complete | Sections 7–8 |
| policy disclosure, redaction and leakage | Complete | Sections 8–9 |
| source, ingestion, profile, secrets and DDIL | Complete | Section 10 |
| command discovery, gates, safety and denial | Complete | Section 11 |
| lifecycle, simulator, events and audit | Complete | Section 12 |
| abuse, resource limits and streams | Complete | Section 13 |
| tools, tiers and evidence artifacts | Complete | Section 14 |
| implementation/community lessons | Complete, informative only | Sections 3 and 14 |
| downstream handoffs | Complete | Section 15 |

## 3. Evidence Base and Authority Classification

### 3.1 Controlling and Accepted Evidence

| Source | Authority/use | Limitation |
|---|---|---|
| CSAPI Parts 1/2 and SensorML/SWE Common | published API, dynamic-data, command and payload semantics | no chosen authentication method, project roles, safety engine or accreditation |
| RFC 9110, 6750, 7517/7519, 8725, 8705, 9700 and 9457 | HTTP, bearer, JWT/JWK, OAuth security, mTLS and problem semantics | deployment-specific identity/policy choices remain project decisions |
| IDR-SRV-031, 038–043 and 047–054 | accepted write, command, authorization, disclosure, audit, DDIL, sync, deployment and verification boundaries | designs still require implementation proof |
| OWASP API Security Top 10 2023 and cheat sheets | risk inventory and defensive test prompts | awareness/guidance, not a Glaux conformance suite |
| NIST SP 800-53 Rev. 5 and SP 800-92 | control/logging categories and evidence prompts | not an authorization to operate |

RFC 8725 requires algorithm verification and validation of issuer, audience and other applicable claims, and warns against one token kind being accepted in another context. RFC 9700 makes audience/privilege restriction and replay resistance central. Glaux is primarily the protected resource server: authorization-server flow tests are out of scope unless Glaux later implements that role. **[N,E]**

### 3.2 Current Tool Evidence

The researched baseline is ZAP `2.17.0`, cargo-audit `0.22.2`, cargo-deny `0.20.2`, cargo-fuzz `0.13.2`, OPA `1.20.2`, secrecy `0.10.3` and zeroize `1.9.0`. Accepted IDR-SRV-044 keeps `jsonwebtoken 11.0.0` as the candidate pin; the existence of `11.1.0` does not silently upgrade that decision. Every implementation pin still requires compatibility and advisory review. OPA is conditional tooling behind the accepted product-neutral policy adapter, not a selected Glaux runtime. **[A,D,X]**

ZAP documents a passive time-limited baseline scan and active API/full scans. Its Automation Framework can import API descriptions, configure authentication, reports and outcome tests. Active scans send attacks and therefore belong only on an explicitly authorized isolated target. GitHub secret scanning recognizes supported patterns and push protection only a subset; large pushes or provider constraints may reduce coverage. These tools supplement deterministic semantic tests. **[D,E]**

### 3.3 Informative Implementation Lessons

Other CSAPI and OpenSensorHub implementations are useful for route inventories, authentication integration shapes and interoperation scenarios, but their behavior is neither normative nor proof of Glaux safety. An implementation disagreement becomes a traceable question against standards and accepted Glaux decisions. **[I,A]**

### 3.4 Evidence Precedence

Published standards control standard semantics; accepted Glaux reports control project architecture; primary tool documentation controls tool capability; executable Glaux tests prove the tested build/profile; implementation observations are hypotheses. Scanner findings and clean scans have no power to override an executable semantic failure. **[N,A,D,T,E]**

## 4. Security and Command Test Requirement Extraction Methodology

Each security obligation becomes a normalized trace chain:

`source/accepted decision -> asset + boundary -> threat/misuse -> control + enforcement point -> test oracle -> fixture/profile -> evidence -> tier + owner -> deviation`

Extraction records whether a statement is normative, accepted, documented, observed, analyzed or recommended. It rejects requirements inferred only from endpoint names or another implementation. Controls are tested at the earliest pure boundary and again at every externally meaningful enforcement boundary. **[A,E,P]**

The method partitions observables into public, authorized, protected-operator and restricted-test evidence. It then applies five oracle forms: exact semantic assertions; structural/allowlist assertions; metamorphic comparisons such as twin worlds; model/state-machine properties; and negative absence/zero-effect assertions. Differential timing is a diagnostic oracle only. High-risk cross-products are exhaustive; lower-risk dimensions use pairwise generation, boundary values and seeded randomized exploration. **[A,P]**

A route/action/field/event/effect inventory is generated from code and compared with the policy registry. Any unregistered surface or action without denial coverage fails CI. The same trace IDs connect IDR-SRV-050 harness cases, IDR-SRV-051 trace nodes, IDR-SRV-052 layers, IDR-SRV-053 fixtures and IDR-SRV-054 abuse workloads. **[A,P]**

## 5. Security Test Taxonomy

| Family | Primary question | Required oracle | Default tier |
|---|---|---|---|
| authentication | is credential proof valid for this exact context? | accept/reject plus safe error and no credential leak | PR |
| authorization | may this actor/client perform this action here now? | deny by default; decision/action/resource match | PR |
| object/function/property | can IDs, routes, relationships or fields bypass policy? | horizontal/vertical/mass-assignment denial | PR |
| disclosure/non-interference | can hidden facts be inferred? | twin-world equivalence and canary absence | PR/nightly |
| source/ingestion | may this source introduce this exact data? | identity binding, validation, quarantine and provenance | PR/nightly |
| configuration/secrets | can an unsafe profile start or leak secret material? | startup rejection and all-surface canary scan | PR |
| DDIL/synchronization | does degraded operation expand authority? | monotonic authority, anti-rollback and revalidation | nightly/RC |
| command/control | can any gate be bypassed or effect escape? | independent gates, valid ticket, zero/one effect | PR/nightly |
| lifecycle/audit | is state/evidence complete and non-invented? | model transitions and ledger reconciliation | PR/nightly |
| stream/replay | does continuing delivery preserve authorization? | subscribe/event/replay re-evaluation | nightly |
| abuse/resource | does attacker-controlled work remain bounded? | limits, isolation, safe 429/503 and recovery | nightly/manual |
| supply/toolchain | are known dependency/policy deviations governed? | pinned reports and owned expiring exception | PR/RC |
| DAST/fuzz/manual | can generated/adversarial probes find defects? | triaged reproducible finding, not a security verdict | nightly/manual |

Security results remain distinct from conformance, correctness, performance and interoperability outcomes even when the same request or workload contributes evidence to several result records. **[A,P]**

### 5.1 Required Security and Command Test Matrix

| Test ID/category | Requirement/risk source | Profile applicability | Identity/source/role fixture | Policy fixture | Command fixture if applicable | Expected behavior | Negative/misuse case | Evidence artifact | Audit expectation | Redaction expectation | CI tier | Downstream handoff | Notes/unresolved |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SEC-AUTHN-JWT | RFC 6750/8725/9700; 039/047 | oidc/composite | synthetic reader/client | allow read | n/a | exact issuer/audience/type/alg/time accepted | none, confusion, wrong iss/aud/type, expired, duplicate/huge claim | sanitized auth result | bounded success/failure class; no token | generic challenge/problem | PR | 056/057 | library behind verifier port |
| SEC-AUTHN-JWKS | RFC 7517/8725; 047 | oidc/composite | local issuer | n/a | n/a | bounded trusted rotation/cache | unknown/duplicate kid, rollback, redirect, SSRF, outage, oversize | issuer/validator trace digest | configuration/availability class | no key/token detail | PR/nightly | 056 | no token-selected URL |
| SEC-AUTHZ-ROUTE | OWASP API5; 039/039A | authenticated profiles | role matrix | action rules | disabled/sim where relevant | every route/action has PEP and denial | alternate method/media/admin path | inventory diff/result | actor/action/resource/outcome | conceal protected topology | PR | 056/057 | new unregistered surface fails |
| SEC-BOLA-GRAPH | OWASP API1; 039/040 | authenticated | owner/non-owner/delegate | relationship/visibility | n/a | only authorized nodes/edges visible | guessed ID, nested link, alternate parent | twin-world facts | deny/conceal decision | no existence/count leak | PR | 056 | traverse through hidden parent denied |
| SEC-PROP-WRITE | OWASP API3; 031/039 | publisher/admin | scoped publisher | field/action obligations | command fields separate | only writable allowed properties persist | mass assignment/server-owned fields/patch alias | before/after digest | attempted fields and decision | no rejected value in errors | PR | 056 | semantic, not serializer-only |
| SEC-POLICY-VIEW | 039A/040 | policy-enabled | public/privileged | reveal/conceal/transform | protected discovery | authorized view precedes query | hidden row changes count/page/latest/ETag | paired world comparison | policy/version/outcome | hidden sentinel absent | PR/nightly | 056/057 | timing diagnostic only |
| SEC-ERROR-OBS | RFC 9457; 041/048 | all | anonymous/denied/admin | diagnostic visibility | denial reasons | safe problem/log/metric/trace | parser/log injection and exception/panic | multi-surface capture | stable correlation, no payload | canaries/credentials absent | PR | 057 | structured encoders |
| SEC-SOURCE | 031/032/034/039 | ingestion | active/revoked/wrong source | source/resource grants | n/a | transport identity bound to source | self-assert other source, replay, rotation race | ingest/provenance record | source/decision/outcome | protected mapping | PR/nightly | 057 | source trust != data policy |
| SEC-INGEST-ABUSE | OWASP API4/7/10; 031 | ingestion | scoped publisher | limits/URL allowlist | n/a | bounded validation/quarantine | deep/huge/Unicode/XML/compression/SSRF/path | limit/fuzz result | rejection class/digest | no payload/reference leak | nightly/manual | 057 | promote safe regressions |
| SEC-PROFILE-SECRET | 044/047/048 | every profile | synthetic config actor | startup constraints | absent/disabled/sim | invalid combinations fail startup | local unsafe public; token/demo; real scheme; exposed debug | config matrix/canary scan | config failure without value | secret absent everywhere | PR | 057 | scanner is supplemental |
| SEC-DDIL | 039A/042/043 | ddil/sync | stale/offline identity | signed bundle/epoch/time | offline classes | no authority expansion; revalidate | rollback, expired/revoked bundle, uncertain clock | bundle/decision digests | evidence quality/version | no policy content | nightly/RC | 057 | remote decision not inherited |
| SEC-STREAM | 035/040/042 | SSE; optional MQTT | subscriber A/B | filter/event visibility | status event fixture | auth at subscribe/event/replay | expiry/revoke/policy change/cursor reuse/wildcard | sequence/cursor facts | connect/change/termination | concealed event absent | nightly | 056/057 | cursor bound to view/principal |
| SEC-CMD-DISC | CSAPI Part 2; 038/047 | disabled/simulated | public/operator | discovery/capability | synthetic ControlStream | advertised surface equals profile | links/schema/control stream leaked | route/OAS/capability diff | discovery decision | hide protected capability | PR | 056 | no physical-adapter affordance |
| SEC-CMD-GATES | 038–042 | simulated only | submitter/approver/operator | authority/disclosure | safe rules/feasibility | all independent gates pass | any one gate deny; no gate substitutes | decision chain/effect count | every material gate | generic external reason | PR | 057 | denial means zero effect |
| SEC-CMD-TOCTOU | 038/042 | simulated only | authorized actors | versioned policy/safety | ticket/revision/target | fresh bound single-use ticket | mutate/replay/expire/wrong target/epoch | ticket digest/effect ledger | invalidation/recheck/attempt | no rule/target secret | nightly | 057 | final pre-dispatch decision |
| SEC-CMD-LIFE | CSAPI Part 2; 036/038 | simulated only | reporter/canceller | reporter/cancel grants | gateway fault script | legal authoritative transitions | invalid/late/duplicate/conflict/timeout/unknown | model trace | assertion + validation + reconciliation | safe public status/message | PR/nightly | 056/057 | never invent effect/cessation |
| SEC-CMD-AUDIT | 038/041 | simulated only | auditor/effect worker | audit access | one permitted effect | durable pre-effect evidence and ledger agreement | audit unavailable/partial commit/duplicate send | audit/event/state/effect reconciliation | complete linked evidence | audit protected and minimized | PR/nightly | 057 | failure closes protected action |
| SEC-ABUSE | OWASP API4/6; 054 | isolated only | attacker/noisy neighbor | quota/priority | simulated bursts | bounded resource use and isolation | auth/query/ingest/subscription/command flood | SecurityTestResult + PerformanceRun | aggregate abuse outcome | no cardinality/payload leak | nightly/manual | 057 | no public stress target |
| SEC-SUPPLY | 044/049 | build/release | build identity | deny/advisory policy | n/a | pins and governed deviations | vulnerable/yanked/license/source drift | audit/deny/SBOM manifest | deviation owner/expiry | no credentials in build logs | PR/RC | 057 | DB freshness recorded |
| SEC-DAST-FUZZ | OWASP; tool docs | disposable isolated | constrained scanner | scanner scope | writes disabled/sim | reproducible triaged findings | active scan external/public or unsafe routes | sanitized ZAP/fuzz report | scan identity/target/time | strip tokens/bodies/URLs as required | nightly/manual/RC | 056/057 | no semantic assurance claim |

## 6. Test Identity, Role, Source, and Policy Fixture Findings

The fixture catalog includes anonymous, authenticated reader, privileged reader, administrator, mapped publisher, operator, command submitter, separate approver, auditor, explicitly denied, expired/revoked and stale/offline identities. A context can include fictional subject, client, delegation chain, organization/mission/purpose attributes, relationships, credential freshness and profile, but roles and scopes never become automatic authority. All identifiers use reserved/example namespaces. **[A,P]**

Pure decision tests inject immutable contexts through the policy port. Router/listener tests authenticate real signed test tokens or test certificates. Ephemeral keys are generated per run; fixture manifests record public-key and configuration digests, not private keys or tokens. A local issuer supports time control, key rotation, outage and malformed responses without implementing user login. **[P]**

Policy fixtures are small signed synthetic bundles: allow, deny, indeterminate, reveal, conceal, transform, stale, expired, wrong audience/version, rollback and conflicting source assertion. Restricted oracle data is stored separately from public expectations. Source fixtures cover active, degraded, suspended, revoked, test-only and simulated sources with explicit resource scopes and rotation epochs. **[A,P]**

Command fixtures contain fictional targets and values, exact schemas/revisions and versioned safety rules. Each rule declares ID/version, applicability, severity, overrideability, evidence/freshness prerequisites and allow/deny/approval/fresh-evidence outcome. Approval binds the exact command revision, scope and expiry and supports revoke. No fixture URL or identifier may resolve to a real target. **[A,P]**

## 7. Authentication and Authorization Test Findings

Authentication cases cover no credential; malformed scheme/token/JWT; invalid signature; `alg=none`, algorithm/key confusion and disallowed algorithm; wrong issuer, audience or explicit token type; expired, premature or implausible issued-at claims and exact skew bounds; missing/unknown/duplicate `kid`; key-use/algorithm mismatch; key rotation, cache, outage and rollback; revoked/disabled identity; token from another profile or token family; duplicate JSON claims, extreme nesting/size and Unicode; and certificate/token binding mismatch where enabled. **[D,P]**

The validator must use a configuration allowlist, not token-selected algorithms or locations. Issuer metadata and JWKS origins are preconfigured; redirect, DNS/IP class, TLS, media type, document size, key count and cache behavior are bounded and tested. Outage policy is explicit and may use only an unexpired trusted cache; it never accepts unverified material. **[D,E,P]**

Authorization denies by default and evaluates every request and continuing action. RBAC is only one input to the accepted attribute/relationship model. Tests bind subject and client/delegation to action, resource, relationship, properties, disclosure view, profile, time, trust and evidence freshness. An authenticated administrator is not implicitly a publisher, policy reader, command approver or executor. **[A,D,P]**

Unauthenticated responses use `401` and an appropriate challenge where the API requires authentication. An authenticated forbidden action uses `403` when revealing the resource/action is permitted. A concealed resource uses the accepted `404` behavior consistently across item, collection, relationship, link, event and error surfaces. Test expectations come from the disclosure decision; they do not globally assume that every denial is `404`. **[N,A,P]**

## 8. Object-Level Access and Policy/Releasability Test Findings

Every Systems, Deployments, Procedures, Sampling Features, Properties, Datastreams, Observations, status/event, ControlStream, Command, feasibility, source-registration, audit and administration surface receives horizontal and vertical authorization cases. Tests guess identifiers, traverse both relationship directions, switch parent paths, use alternate methods/representations, submit server-owned fields and mass assignments, and compare list versus item behavior. HEAD/OPTIONS and generated documentation cannot bypass the same disclosure policy. **[A,P]**

Authorization is pushed into query construction before pagination, total/count, extent, ordering, latest selection, cursor generation, validator/ETag and link construction. Post-fetch filtering is insufficient because it leaks cardinality and distorts navigation. Cursors and validators bind the authorized view, policy version, principal/client and query/filter; cross-principal or cross-policy reuse is rejected without revealing why. **[A,P]**

Twin-world tests construct worlds `W0` and `W1` differing only by one concealed resource/value. For a public observer, the approved observable projection of status, headers, body structure/values, counts, extents, page topology, cursor validity, ETag, latest item, links, OpenAPI/capabilities, stream sequence and protected telemetry must match. Privileged tests separately prove the hidden difference is real. Response-size and latency distributions are compared in stable environments; statistically visible differences trigger investigation but equality never proves universal non-interference. **[E,P]**

## 9. Redaction, Side-Channel, Error, Logging, Metrics, and Trace Leakage Test Findings

The leakage inventory includes body fields and alternate encodings; links and identifiers; count, extent, order, latest and page gaps; status, headers, response length, cache validators and timing; problem details; OpenAPI/examples/capability discovery; health/admin endpoints; log fields/messages; audit; metric labels; traces and baggage; stream filters/events/replay/cursors; and command schema/status/result/reason. Every security-sensitive feature registers its observable surfaces. **[A,P]**

Each scenario injects distinct synthetic canaries for credential, hidden record, policy fact, source mapping and command parameter. Captures are scanned structurally and bytewise for forbidden values and encoded variants. Public golden files describe allowed facts and absence rules; they never contain the restricted oracle. JSON/control-character/newline and terminal escape cases prove structured encoders resist log injection. **[A,P]**

RFC 9457 problem types remain stable and useful without exposing stack traces, queries, key IDs, policy clauses, hidden identifiers, raw payloads or command topology. Logs, metrics and traces use bounded vocabularies and opaque correlation identifiers. Test failures themselves redact inputs before display. Audit is protected authoritative evidence and may contain controlled identifiers required for accountability, but never raw credentials or full payloads by default. **[N,A,P]**

## 10. Source Trust, Ingestion, Configuration/Profile, Secret, and DDIL Findings

### 10.1 Source Trust and Ingestion

Tests prove authenticated transport identity maps through configuration to the allowed source and resources. Payload fields such as `sender`, source ID or policy assertions cannot override that mapping. Cases cover wrong resource, unknown/suspended/revoked/stale source, rotation overlap, replay, duplicate/idempotency, identifier collision, out-of-order input and downgrade. Quarantined or pending data is absent from canonical query, latest, aggregate, stream and command evidence. **[A,P]**

Validation limits cover media type, content length and decoded size; object/array depth and counts; strings, Unicode and numbers; XML entity/external-resource behavior if XML is supported; compression/archive expansion; schema/reference URL allowlists; SSRF, redirects and private/link-local destinations; file/path traversal; timeouts and concurrency. Unsafe downstream responses are treated as untrusted input. Fuzzing targets codecs, JWT/JWK, filters, cursors, problem documents and state machines with deterministic seeds and governed regression promotion. **[D,E,P]**

### 10.2 Profiles, Configuration, and Secrets

The typed configuration matrix rejects unsafe combinations before serving: `local_unsafe` with non-loopback/public binding; authentication disabled in demo/operational profiles; static test tokens outside test; command mode other than disabled/simulated; any physical adapter or non-`sim` target; inbound Part 3; public admin/debug/private metrics; untrusted reverse-proxy headers; unrestricted remote schema/JWKS URLs; or absent/weak required secret references. Startup evidence identifies the rule, not the secret. **[A,P]**

Secrets are mounted/retrieved references, not CLI arguments, ordinary TOML/environment dumps, images or fixtures. `secrecy` and `zeroize` reduce accidental exposure but do not prove process-memory erasure or prevent explicit disclosure. Canary tests scan process metadata available to the harness, effective configuration, errors, stdout/stderr, audit, traces, metrics, test reports and panic/crash artifacts. Repository/provider secret scans, custom deterministic forbidden-pattern rules and review are layered controls with recorded limitations. **[A,D,P]**

### 10.3 DDIL and Synchronization

Tests vary bundle signature, audience, version, expiry, revocation epoch, monotonic counter, local trusted time confidence and evaluator availability. Unknown, expired, rolled-back or indeterminate material denies. Explicit offline classes can rely only on locally verifiable evidence and cannot exceed the last accepted connected authority. Reconnection cannot retroactively legitimize an effect. Queued commands undergo current authorization, disclosure, feasibility, safety, approval and target checks immediately before dispatch. **[A,P]**

Audit capacity and evidence durability are security dependencies: exhaustion or integrity failure closes protected mutations/effects according to the accepted audit design. Synchronization imports facts and provenance, not a remote authorization verdict; conflicts cannot use last-writer-wins to expand authority. **[A,P]**

## 11. Command Discovery, Authorization, Feasibility, Safety, and Denial Findings

Command-disabled profiles must not advertise tasking affordances that cannot be safely exercised. Simulated profiles disclose ControlStreams, schemas, feasibility and command links only to authorized views, and generated OpenAPI/capability documents match the active profile. Physical adapter types and real endpoint schemes are absent from build/profile registries. **[A,P]**

The admission/dispatch sequence tests independent gates: API access; structural and semantic validation; command/target authority; data-policy disclosure; feasibility evidence; safety rules/interlocks; required approval and separation of duties; target/gateway/source trust; durable audit/commit; and final pre-dispatch re-evaluation. A scope, source claim, feasible result, approval, broker acknowledgement or administrator role cannot satisfy a different gate. Gate-order optimizations may reject early but cannot omit evidence needed before effect. **[A,P]**

Each gate has allow, deny, indeterminate, stale, unavailable and changed-after-admission cases as applicable. All denials produce zero effect, a policy-safe response/status and protected structured evidence. Overrides require their own authority, exact rule/scope/revision, reason, expiry and non-overrideable-rule checks; a general administrator cannot improvise one. `ACCEPTED` reflects receiving-system acceptance, not merely Glaux authorization. **[A,P]**

The final ticket is short-lived and single-use, bound to command/revision, target, contract, policy/safety/trust versions, gateway route, deadline and fencing epoch. Tests replace each field, replay after use, expire it, rotate authority, change policy/safety/target health and race approval/cancellation. Any material change invalidates or regenerates the ticket after all gates rerun. **[A,P]**

## 12. Command Lifecycle, Simulated Gateway, Event, and Audit Evidence Findings

Property/state-machine tests generate legal and illegal transitions among the exact nine CSAPI statuses and protected internal states. They cover duplicate submissions, retries/idempotency, update revisions, approval expiry/revoke, cancellation request versus authoritative cancellation, cancellation/completion races, timeout, lost acknowledgement, unknown outcome, duplicate/out-of-order/late/conflicting reports, reconnect and reconciliation. The last valid public status is preserved until authoritative evidence supports another; Glaux never invents execution, completion, failure or physical cessation. **[N,A,P]**

The simulator consists of a non-network `sim://` effect recorder and a scripted gateway response model. The recorder stores normalized command/revision/ticket/effect sequence digests and permits exactly zero or one effect according to the scenario. Test/build controls fail on DNS, socket or non-simulator scheme attempts. The gateway model returns accepted, rejected, delayed, duplicated, reordered, lost and indeterminate responses without resolving an external address. **[P]**

Every material request, decision, revision, approval, ticket, dispatch attempt, acknowledgement, status assertion, cancellation and reconciliation event has a typed audit expectation. Mutations and pre-effect evidence satisfy the atomic/durable boundary in IDR-SRV-041. Tests reconcile state journal, audit journal, event/outbox and effect ledger; missing, duplicate, wrong-actor/action/outcome/version or post-effect-only evidence fails. If authoritative audit cannot be committed, the protected effect is not attempted. **[A,P]**

Public command events contain only policy-authorized status and message. Protected evidence retains reason codes and version/digest links; raw command payloads, safety rules and topology are not copied by default. A test result cross-references audit IDs/checkpoints rather than becoming the audit authority. **[A,P]**

## 13. Abuse, Rate-Limit, Resource-Exhaustion, and Streaming Security Findings

Abuse families cover authentication failures, malformed inputs, expensive filters/spatial/temporal queries, oversized/deep/decompressed bodies, ingestion bursts, connection/subscription/replay churn, slow consumers, command/feasibility bursts and high-cardinality diagnostic attempts. Limits apply by appropriate subject, client, source, address/profile and operation class; one tenant/source cannot consume another's reserved safety/control capacity. Rejection uses safe `429`/`503` behavior and bounded audit/telemetry. Workloads reuse IDR-SRV-054 IDs but produce separate security and performance records. **[A,P]**

Streaming authenticates subscription and authorizes the filter, then re-evaluates at event publication/delivery, replay and material token/policy/source changes. Tests expire/revoke credentials, update policy, conceal a formerly visible object and reconnect with old cursors. Cursors are integrity-protected and bound to principal/client, profile, filter and policy view. Cross-principal reuse and tampering fail without revealing hidden sequence facts. **[A,P]**

If optional MQTT is enabled, tests cover topic ACLs, wildcard expansion, retained messages, QoS duplication/replay, persistent sessions and reconnect. Broker ACLs are defense in depth; application authorization remains authoritative. Part 3 remains outbound-only, and no inbound subscription becomes a command/tasking path. Destructive abuse tests run only in disposable isolated environments with stop conditions from IDR-SRV-054. **[A,P,X]**

## 14. Tooling, CI Gates, Evidence, and Redaction Artifacts

### 14.1 Selected and Conditional Tools

Rust unit/integration tests and cargo-nextest remain primary. Property/state testing and cargo-fuzz cover validators, policies and lifecycles; a local issuer/JWKS fixture covers authentication; real PostgreSQL/testcontainers prove query-level policy; an explicit proxy/TLS harness proves listener assumptions; wire mocks represent only external HTTP failures. cargo-audit and cargo-deny implement accepted dependency/advisory/license/source gates. k6 supplies isolated abuse load under IDR-SRV-054. **[A,P]**

ZAP `2.17.0` is selected as supplemental DAST: passive baseline/nightly first; authenticated active API/full scans only manual/RC on disposable targets. Rules and false positives require owned, reasoned, expiring suppressions. OPA tooling is used only if that adapter exists; `opa test --fail-on-empty`, signed bundle tests and decision-log masking are then mandatory. **[D,P]**

### 14.2 Execution Tiers

| Tier | Required scope | Blocking rule |
|---|---|---|
| every PR | inventory/PEP, authn/authz/object/property/disclosure, config/secret, core source/command/audit tests, regression corpus, cargo-audit/deny, deterministic secret scan | semantic failure, leakage, effect escape, unowned dependency deviation |
| nightly | broad pairwise/property/state matrices, fuzz campaigns, source/DDIL/stream/replay, abuse smoke, passive ZAP baseline | reproducible high/critical semantic defect; triage tool findings |
| manual | isolated active ZAP, targeted fault/TOCTOU, long fuzz/abuse, proxy/mTLS and external IdP adapter | required evidence before affected release/profile claim |
| release candidate | full frozen security manifest, supply/SBOM/provenance, restore/migration security, interop security, all deviations reviewed | incomplete evidence or expired exception blocks release candidate |
| future external/operational | authorized penetration assessment, operational IdP/policy/infra, accreditation controls | governed by later scope and authority; no IDR readiness claim |

### 14.3 Security Evidence Schema

`SecurityTestResultV1` records test, threat, control, requirement and risk IDs; target build/profile/config/corpus/policy/identity fixture digests; tool versions; seed/preconditions/steps; outcome (`pass`, `fail`, `error`, `inconclusive`, `not-applicable`); allowed observed facts; restricted evidence references/digests; audit IDs/checkpoints; effect count/ledger digest; redaction scan; residual limitation; sensitivity, reviewer and timestamps. `error` and `inconclusive` never count as pass. **[P]**

Public CI publishes the sanitized manifest, outcomes and safe reproducer metadata. Raw wire captures, restricted oracles or protected decision details, if indispensable, use separate access and short declared retention. Tokens and private keys are not captured. Reports are scanned before publication; screenshots are not authoritative evidence. **[P]**

### 14.4 Implementation Proofs Required

1. A new route/action/property/event/effect without policy registration and denial coverage fails CI.
2. The local issuer proves strict JWT validation plus JWKS rotation, rollback, outage and algorithm-confusion behavior through the real HTTP boundary.
3. The identity/action/resource/property matrix proves deny-by-default and cannot use a role/scope as substitute authority.
4. Twin-world tests prove the declared public projection—including count, page, extent, latest, cursor and ETag—does not change with hidden facts.
5. Synthetic secret/hidden canaries are absent from responses, documentation, errors, logs, audit, traces, metrics, reports and panic artifacts.
6. Source identity binding, quarantine isolation, SSRF/reference controls and parser/resource limits survive negative and fuzz regression cases.
7. Every invalid profile combination fails before serving and the simulator-only build contains no physical effect adapter or resolvable target scheme.
8. DDIL rollback/staleness/time uncertainty and sync conflict never expand authority; queued work is re-authorized before effect.
9. Stream subscription/event/replay re-evaluation and cursor binding prevent cross-principal/policy-view disclosure.
10. Command gate permutations each yield zero effect on denial; approval separation and safety override rules cannot bypass another gate.
11. Dispatch ticket TOCTOU/replay/fencing and lifecycle/cancellation/unknown-outcome models preserve exactly zero/one effect and authoritative status.
12. State, audit, event/outbox and effect ledgers reconcile across crash/fault points; audit failure closes the effect path; sanitized supply/fuzz/ZAP evidence remains reproducible and triaged.

## 15. Downstream Topic Handoff Matrix

| Recipient | Required handoff | Must not infer |
|---|---|---|
| IDR-SRV-056 interoperability | identity/profile manifest; 401/403/404/disclosure expectations; protected discovery; token/certificate setup; cursor/event policy; command-disabled/simulated cases; safe negative-test permissions | external client may run active attacks or issue physical commands |
| IDR-SRV-057 synthesis | accepted taxonomy/matrix, tool/tier decisions, `SecurityTestResultV1`, twelve proofs, residual risks and future operational boundaries | implementation, accreditation or production security is complete |
| implementation backlog | policy inventory/PEP registry, local issuer, fixture schemas, twin-world harness, simulator/effect ledger, canary scanner and tier manifests | any operational identity/policy/vendor selection |
| conformance/trace/corpus/performance | normalized trace links, separate result types, fixture sensitivities and reusable isolated abuse workload IDs | security outcome substitutes for conformance/performance or vice versa |
| deployment/operations | invalid-profile rules, secret references, protected evidence retention and external-assessment prerequisites | IDR values are operational deployment authorization |

## 16. Recommendations

1. Adopt the trace-driven taxonomy and required matrix as the security-test baseline.
2. Make the route/action/property/event/effect policy inventory and negative-case coverage PR-blocking.
3. Use injected immutable contexts only for pure decision tests; use an ephemeral local issuer/JWKS service for HTTP authentication proof.
4. Require strict issuer/audience/type/algorithm/time/JWKS validation and bounded trusted discovery.
5. Put authorization into queries before all derived metadata and adopt twin-world/canary disclosure tests.
6. Treat sources, policy assertions, feasibility, approvals, broker acknowledgements and administrative roles as distinct evidence, never interchangeable authority.
7. Reject unsafe profile combinations at startup and keep physical adapters/real target schemes absent from the first build.
8. Adopt the versioned synthetic safety-rule fixtures, independent gate permutations, bound single-use dispatch tickets and final pre-dispatch re-evaluation.
9. Require non-network `sim://` effects and reconcile command state, audit, events and effects under fault injection.
10. Adopt `SecurityTestResultV1`, sensitivity-separated artifacts and mandatory publication redaction scans.
11. Run deterministic semantic/supply checks in PR, broad/property/passive scanning nightly and active DAST only in authorized disposable manual/RC environments.
12. Keep external penetration testing, operational products/content and accreditation as explicit future work.

## 17. Risks, Constraints, and Open Questions

| Risk/decision | Treatment | State |
|---|---|---|
| semantic authorization gaps escape endpoint scanners | inventory/PEP and modeled negative tests are authoritative | Resolved strategy |
| mocks prove policy but not listener authentication | separate context injection from local-issuer HTTP tests | Resolved strategy |
| hidden fixtures leak through expectations/artifacts | restricted oracle, public facts and canary scan | Resolved strategy |
| simulator accidentally gains network capability | `sim://` only, no physical adapter, DNS/socket fail test | Resolved strategy |
| stale DDIL material expands authority | signature/audience/version/expiry/epoch/time/anti-rollback and recheck | Resolved strategy |
| DAST damages a live service | active scans only explicitly authorized disposable targets | Resolved strategy |
| scanner/audit clean result overstated | label as supplemental defect detection | Resolved strategy |
| authorization cross-product becomes unbounded | exhaustive high-risk boundaries; governed pairwise/model generation elsewhere | Resolved strategy |
| timing side channels remain environment-dependent | stable statistical diagnostics and residual-risk statement | Constraint |
| exact operational IdP, claims, policy and revocation | adapter contract plus future operational validation | Open stakeholder decision |
| production quotas, identity lifecycle and penetration scope | use synthetic/reference tiers; set only with operational authority | Open operational decision |
| external assessment/accreditation control mapping | future authorized work | Out of scope |

No open item prevents implementing the first deterministic suite. They prevent claims about a particular operational deployment, identity ecosystem, penetration scope or accreditation. **[P,X]**

## 18. Validation Against This Plan's Success Criteria

| Success criterion | Result | Evidence |
|---|---|---|
| scope, source anchors and prior traceability | Met | Sections 2–4 |
| taxonomy and identity/policy/source/command/redaction/audit fixtures | Met | Sections 5–6, 12 |
| authn/authz/object/policy/source/ingest/profile/secrets/DDIL/errors | Met | Sections 7–10 |
| command discovery/authority/feasibility/safety/lifecycle/cancel/timeout/unknown/audit/simulator | Met | Sections 11–12 |
| PR/nightly/manual/RC/future tiers | Met | Section 14.2 |
| tools, evidence, redaction and sensitive-data safeguards | Met | Sections 9 and 14 |
| implementation/community lessons non-normative | Met | Sections 3.3–3.4 |
| decision-usable bounded recommendations | Met | Sections 16–17 |
| explicit downstream handoffs | Met | Section 15 |
| explicit reproducible references | Met | Section 19 |

### Report Completion Checklist

- [x] Required nineteen content areas completed.
- [x] Required fourteen-column matrix completed.
- [x] Identity, policy, source and safe command fixtures defined.
- [x] Positive, negative, misuse, boundary, metamorphic, model and fault tests defined.
- [x] PR, nightly, manual, release-candidate and future evidence tiers separated.
- [x] Twelve implementation proofs defined.
- [x] No real credential, policy label, source data, command target or physical effect authorized.
- [x] Later-topic authorization and readiness/accreditation claims explicitly excluded.

## 19. References

### Governing and Accepted Project Sources

- [Glaux Server Overall IDR Research Plan](../IDR%20Plans/overall-idr-research-plan.md)
- [Glaux Server Goal and Definition](https://github.com/DGIWG-P507/glaux/blob/main/Docs/Plans/glaux-server/glaux-server-goal-and-definition.md)
- Accepted IDR-SRV-031, IDR-SRV-038 through IDR-SRV-043, IDR-SRV-047 through IDR-SRV-054 reports in this directory.
- [OGC API - Connected Systems - Part 1: Feature Resources](https://docs.ogc.org/is/23-001/23-001.html)
- [OGC API - Connected Systems - Part 2: Dynamic Data](https://docs.ogc.org/is/23-002/23-002.html)
- [OGC SensorML 3.0](https://docs.ogc.org/is/23-000/23-000.html)
- [OGC SWE Common Data Model 3.0](https://docs.ogc.org/is/24-014/24-014.html)

### Identity, Protocol, and Security Sources

- [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)
- [RFC 6750: OAuth 2.0 Bearer Token Usage](https://www.rfc-editor.org/rfc/rfc6750)
- [RFC 7517: JSON Web Key](https://www.rfc-editor.org/rfc/rfc7517)
- [RFC 7519: JSON Web Token](https://www.rfc-editor.org/rfc/rfc7519)
- [RFC 8725: JSON Web Token Best Current Practices](https://www.rfc-editor.org/rfc/rfc8725)
- [RFC 8705: OAuth 2.0 Mutual-TLS](https://www.rfc-editor.org/rfc/rfc8705)
- [RFC 9700 / BCP 240: OAuth 2.0 Security Best Current Practice](https://www.rfc-editor.org/rfc/rfc9700)
- [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457)
- [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html)
- [OWASP API Security Top 10 2023](https://api-security.owasp.org/editions/2023/en/0x00-header/)
- [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [OWASP Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
- [OWASP Error Handling Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html)
- [OWASP Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
- [NIST SP 800-53 Rev. 5](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [NIST SP 800-92](https://csrc.nist.gov/pubs/sp/800/92/final)

### Tool and Implementation Sources

- [OWASP ZAP Docker scan documentation](https://www.zaproxy.org/docs/docker/about/)
- [OWASP ZAP Automation Framework](https://www.zaproxy.org/docs/automate/automation-framework/)
- [cargo-audit](https://github.com/rustsec/rustsec/tree/main/cargo-audit)
- [cargo-deny](https://github.com/EmbarkStudios/cargo-deny)
- [cargo-fuzz](https://github.com/rust-fuzz/cargo-fuzz)
- [RustSec Advisory Database](https://rustsec.org/)
- [GitHub secret scanning documentation](https://docs.github.com/code-security/secret-scanning)
- [Open Policy Agent documentation](https://www.openpolicyagent.org/docs/latest/)
- [OPA decision-log masking](https://www.openpolicyagent.org/docs/management-decision-logs#masking-sensitive-data)
- [OS4CSAPI organization](https://github.com/OS4CSAPI)
- [OpenSensorHub](https://github.com/opensensorhub/osh-core)

---

**Research Result:** Complete; report awaiting Glaux Project Lead review.<br>
**Acceptance Boundary:** Acceptance would establish the planning baseline only. It would not implement controls/tests, authorize IDR-SRV-056, permit active testing of public/external targets, enable physical command effects, select operational identity/policy infrastructure, or claim production security, accreditation, conformance or readiness.

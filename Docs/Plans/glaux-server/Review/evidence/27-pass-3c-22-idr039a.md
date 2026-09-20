# Pass 3c, iteration 22 — queue batch 4: committed deep read of IDR-039A

**Date:** 2026-09-19
**Provider/model:** Claude Code; model identified by the runtime environment as Claude Opus 5 (model ID `claude-opus-5`).
**Batch:** batch 4 of 27, the committed deep read of IDR-SRV-039A (zero-trust architecture alignment and enforcement model), from `current_work.next_batch` at planning commit `1b8cdea4e01566fb63ed45e4c1778a11c653d761`.
**Mode:** read-only review of one research report and the Implementation Guide; review artifacts updated and published; no implementation, Goal/Guide/Roadmap, issue, settings or upstream changes. No recommendation was implemented and the queue was not expanded.

## 1. Read coverage, stated precisely

All twenty-five numbered sections were read in full (lines 69-786), together with the evidence and decision legend (lines 23-38) and the status header. The table of contents (lines 39-68) is navigational and was not read as substantive content. Nothing in this report is recorded on the basis of a skim.

Report status: **Final**, accepted by the Glaux Project Lead on September 15, 2026.

## 2. The dependency cross-check the queue required

The recorded scope directed this batch to check that IDR-040 Section 14.1 and IDR-042 Section 14.1, which both cite this report for their offline model, are consistent with what IDR-039A actually specifies. They are. Each cited element exists in the source and says what the citing reports say it says.

| Cited by IDR-040 §14.1 and IDR-042 §14.1 | Present in IDR-039A | Where |
|---|---|---|
| "signed, versioned, anti-rollback policy/trust bundles" | Yes. Bundles "declare issuer, audience/node/security domain, sequence/epoch, policy and schema versions, activation/expiry, maximum offline age, trust/source mappings, permitted operation classes, dependency digests and anti-rollback state" | §16.1 |
| "bounded local authorization envelope" | Yes. Offline operation classes are bounded "within issuer, audience, action, resource, time-confidence, freshness, and revocation-epoch bounds" | §1, §16.1 |
| "Disconnection never expands authority" | Yes, near-verbatim. "Authority does not expand during disconnection" and "Authority never broadens because enterprise services are unreachable" | §1, §16.2 |
| Absent or stale required inputs yield `INDETERMINATE`, then deny, conceal or quarantine | Yes. "Unknown time, expired evidence, missing mandatory state, or exceeded staleness yields deny/indeterminate", with `INDETERMINATE` defined as covering missing mandatory PIP facts and expired decisions and explicitly "not a soft allow" | §1, §11.1 |

The handoff matrix confirms the direction of the dependency from the source side: the IDR-042 row hands over "Local PE/PA, signed anti-rollback bundles, time confidence, offline operation classes, staleness and failure table" with the boundary "Disconnection never broadens authority" (§21, line 644). The IDR-040 row hands over data-centric PIP attributes and PAP lifecycle rather than the bundle model, but that matrix lists what each downstream topic must consume, not an exhaustive list of what it may cite; IDR-040's citation is of the §16.1 model, which exists as described. No inconsistency, and no correction is needed in any of the three reports.

## 3. Accounted against Guide v1.3

### 3.1 The Guide makes no zero-trust claim, and that is the recommendation being followed

A search of the whole Guide finds no occurrence of "zero trust", "ZTA", "PEP", "PDP", "policy engine", "policy decision point", "least privilege" or "workload identity". IDR-039A is cited exactly once, at Guide line 592, as one source for Section 4.10's required behaviour.

That absence is not an omission. The report's first and most emphatic recommendation is that "ZTA-aligned Glaux Server enforcement" be the only unqualified project claim, and that enterprise ZTA, maturity, certification, accreditation or authorization-to-operate claims are prohibited without a competent external authority (§22 recommendation 1; §17.3 prescribes the exact permitted claim wording and forbids "is zero trust"). A Guide that makes no ZTA claim at all is the most conservative position available and is fully consistent with that discipline. The Guide applies the same discipline in the domain it does claim: "Test success is not an OGC certification claim" (line 193) and "Declare a class only when the enabled implementation satisfies all its applicable requirements and prerequisites with adequate tests" (line 907).

### 3.2 Adopted in substance

| IDR-039A item | Guide disposition | Reference |
|---|---|---|
| §11.3 and recommendation 11: no privileged fetch followed by optional application filtering; authorized predicates applied before data is accessed | **Adopted.** "Resolve authorized queryable use and the eligible observation/property/relationship view before evaluating predicates"; authorization precedes queries, counts, extents, links, schema disclosure, latest selection and streaming delivery | Guide lines 469, 598 |
| §11.3: counts, extents, ordering, pagination, cursor validity and error existence behaviour describe only the authorized view | **Adopted.** Continuations are reauthorized, cursors bind endpoint scope and mapping version, and the protected-facts test requires membership, counts and errors not to reveal hidden facts | Guide lines 471, 975, 1019 |
| §1 and §11.1: `INDETERMINATE` is treated as deny; mandatory evidence that is missing, stale or unauthenticated denies rather than degrades to allow | **Adopted in effect.** "no network-facing operation defaults to allow on policy-service failure"; where a richer external policy service is integrated, its timeout and unavailable behaviour must be documented | Guide line 598 |
| §8.4: anonymous is an explicit subject class, not an authentication failure converted into public access | **Adopted.** "A read-only anonymous deployment is an explicit policy option, not a way to enable unauthenticated writes" | Guide line 596 |
| §7.2: arbitrary forwarded headers are prohibited; a gateway hop confers no broad internal trust | **Adopted.** "Trust forwarded origin headers only from configured reverse proxies"; CORS is not access control | Guide lines 364, 596 |
| §14: subscription establishment, per-event view checks, expiry and revocation-driven termination; broker ACLs are secondary and cannot perform per-resource policy | **Adopted.** "Authorize the selected scope and each emitted event"; authority is rechecked during delivery with close on expiry or revocation; MQTT subscribers are permitted only where the whole topic has one enforceable authorized audience | Guide lines 543, 547, 564 |
| §15: command discovery, submission, approval, dispatch and reporting are distinct actions; the final gate runs immediately before dispatch; a gateway acknowledgement is not proof of physical effect | **Adopted.** Policy and configuration are rechecked "immediately before dispatch"; submission permission is separate from permission to report status or results; transport acknowledgement is not execution proof | Guide lines 574, 580, 1037 |
| §13.1: registration is not trust; publisher authentication, source mapping and ingestion authorization are separate | **Adopted.** Publishers are restricted to assigned Systems and streams, status reporters to the adapter or source authority they represent, and a submitter cannot impersonate another sender by writing a field | Guide lines 574, 598 |
| §17: redaction happens before formatter, buffer and exporter; metrics stay low cardinality; logs never carry credentials, raw assertions or payload | **Adopted.** "Do not log secrets, bearer tokens, raw sensitive payloads, or protected policy reasons"; "Avoid resource IDs or arbitrary query strings as metric labels"; effective configuration diagnostics are redacted | Guide lines 602, 663, 667 |
| §18 profiles: development identities are loopback-only and rejected by network-facing configuration; demo and conformance profiles are synthetic and command-disabled | **Adopted.** Explicit development identities are "clearly labeled and rejected by normal network-facing configuration"; unsafe combinations such as public listeners with development authentication are rejected at startup | Guide lines 596, 663 |
| §19: documentation cannot express full object and property policy, so clients must handle authorized subsets and runtime `401`, `403` or concealed `404` | **Adopted.** The status table carries the consistent non-disclosure `404` where policy requires, and schema documents and capabilities receive the same protection as ordinary data | Guide lines 600, 834 |

### 3.3 A fourth source for the F-03 cache instance, and the first to name ETags

The response-cache instance recorded under F-03 in iteration 17 rested on reasoning about validators plus IDR-040 §10.4, and gained IDR-042 §7 and IDR-034 §13.4 in later iterations. IDR-039A states the rule twice and is the first source to name ETags explicitly:

- §11.3: "Counts, extents, ordering, pagination, cursor validity, ETags, `Location`, alternate representations, response timing classes and error existence behavior describe only the authorized view. **Cache keys include that view and relevant policy/evidence versions.**"
- §9.5, PEP registry: "Cache/cursor/ETag PEP | reusable artifact across decisions | **bind to authorized-view and policy/data/security versions**."

Iteration 17 derived the validator half of that instance from RFC 9111 mechanics rather than from any research statement, and noted the Guide's "representation-specific" wording at lines 503 and 739 was ambiguous about whether an authorized view counts as a distinct representation. IDR-039A removes that ambiguity from the research side: it lists ETags among the artifacts that must describe only the authorized view, and it treats the cache, cursor and ETag boundary as a named enforcement point. The instance now rests on four accepted reports. Its substance and recommended fix are unchanged, and it remains a documentation gap rather than a new finding.

### 3.4 Not adopted, as a recorded scope choice

The Guide implements none of the named logical architecture: the policy engine and policy administrator split, the PEP registry with its no-bypass invariant, typed PIP adapters with provenance and freshness envelopes, the PAP bundle lifecycle, the three-state decision envelope with obligations and expiry, the granted-action zone model, the seven deployment profiles, the ZTA alignment manifest, and per-workload rotatable identities beyond the separate publisher and subscriber credentials already required for MQTT examples.

This is the same category F-03 already dispositions, and IDR-039A's own framing supports it. Section 18.2 defers enterprise identity and access management, operational public key infrastructure, external policy decision and administration products, device posture, behavioural analytics, service mesh, cross-domain approval and accreditation to owning organizations and later topics. Section 6.2 states that a modular monolith satisfies the architecture provided in-process calls carry explicit context, and Section 9.4 permits reviewed version-controlled policy fixtures instead of an interactive administration point for a first implementation. Section 20.4 lists the whole approach as a proof-of-concept backlog that is "not authorization to code now." No new finding.

## 4. Observations recorded, not findings

- **`fq-09` gains a sixth source.** §17.2 requires metrics to remain low cardinality using safe reason, action and profile classes, and forbids credentials, raw assertions, full policy, device inventory detail, sensitive resource identifiers, source topology, command targets or payload in logs, traces and audit, with redaction before the formatter, buffer and exporter. The Guide adopts the label discipline at line 667 and the no-secrets rule at line 602. The open part remains the exposure boundary of the metrics endpoint itself.
- **`fq-10` gains a fifth pin.** §25.3 cites Connected Systems Go at `SomethingCreativeStudios`. Five reports and Guide line 1129 now use that organization against IDR-040's single use of `opensensorhub`. Still not adjudicated; the peer-source batch owns it.
- **No new follow-up question.** Nothing in this report raised a question that the existing register does not already carry.

## 5. Findings register changes

No new finding number and no change to any finding's disposition. One existing instance was strengthened:

- **F-03**, response-cache instance: a fourth corroborating source, and the first that names ETags among the artifacts bound to the authorized view.

F-08, F-12, F-15, F-18, F-19, F-20, F-21 and F-22 were checked against this report and need no change. Its command content composes the IDR-038 gates already accounted under F-15 and F-18; its audit and telemetry content duplicates material already recorded under F-19, F-20 and `fq-09`.

## 6. Disposition summary

| Item | Disposition | Where |
|---|---|---|
| IDR-039A read coverage | All 25 sections plus the legend read in full; table of contents excluded as navigational | Section 1 |
| IDR-040 and IDR-042 dependency cross-check | **Consistent.** Every cited element exists in IDR-039A and says what the citing reports say | Section 2 |
| Guide's absence of ZTA vocabulary and claims | Compliance with the report's own claim discipline, not an omission | Section 3.1 |
| Enforcement substance | Substantially adopted across authorization ordering, authorized-view artifacts, fail-closed behaviour, anonymous handling, proxy trust, streaming, commands, source separation and telemetry | Section 3.2 |
| F-03 cache instance | Fourth source; first to name ETags explicitly | Section 3.3 |
| Named ZTA logical architecture | Not adopted; recorded scope choice consistent with F-03 and the report's own deferrals | Section 3.4 |

**Remaining in this check:** one committed deep read, IDR-055, queued as batch 5. Next selected batch is **batch 5 of 27, IDR-055**.

## 7. Statement of limits

This iteration read one research report in full and the Guide text needed to assess it. It did not fetch any issue body, execute batch 17, resolve `fq-10` or `fq-11`, or begin any batch other than batch 4. It implemented no recommendation and added no batch. The cross-check in Section 2 verifies that two downstream citations accurately represent their source; it does not independently validate the offline model against any external authority. The F-03 material here strengthens an existing documentation gap in planning material and is not a report of observed runtime behaviour. `review_complete` remains `false`.

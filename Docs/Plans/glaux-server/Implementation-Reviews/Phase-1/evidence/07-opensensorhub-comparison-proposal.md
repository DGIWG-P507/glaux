# OpenSensorHub comparison proposal

**Status:** proposal ready for the project lead; comparison execution is not yet authorized.<br>
**Step and date:** Phase 1 review, Step 5; 3 October 2026.<br>
**Prepared for:** the Glaux project lead and the assistant carrying out the comparison.<br>
**Authority:** `proceed` after [the Rust follow-up](06-rust-security-followup.md) authorized drafting and publishing this proposal, not installing tools or running either server.<br>
**Prepared by:** Codex (OpenAI), with separate read-only agents examining peer setup and request cases. Exact model versions are not exposed; prior project context was present. A separate non-author reviewer inspects the finished diff and sources; the publication PR records the reviewed commit and outcome. This is not human expert or cross-provider assurance.<br>
**Baselines:** Glaux Server `27955c1b9260cd811ad6bc08f85feab43ad65028`; planning `c58beaa149b49e2f51e1f9d5dc052f36706f9ca6`. Both checkouts were synchronized and all fifteen review gates checked open.

## Recommendation

Run one small comparison of **the same minimal System requests against Glaux and a pinned OpenSensorHub Connected Systems service**. Check the independently expected meaning, not whether the servers produce identical bytes. When they differ, explain whether the standard requires one answer, both answers are permitted, the implementations have different scope, or the evidence is insufficient.

Use eight fixed case groups, two synthetic Systems and one ordinary restart per server. Execute only on disposable GitHub-hosted Linux, in the existing isolated database environment. No company-laptop installation, public demo endpoint, Oracle/Fly service or permanent deployment is proposed. Report the result without repairing either implementation or automatically adding more cases.

The [test-source audit](02-test-source-audit.md) already traced the Phase 1 expectations. The accepted [four-client synthesis][Clients] distinguishes this minimal GeoJSON path from full application compatibility. Reuse those results: do not rerun client studies, launch viewers, or add collection browsing, SensorML implementation, filtering, updates/deletes, observations, commands, streaming, backup/restore or security-policy parity to this comparison.

## Sources and selected peer

The controlling references remain [CSAPI Part 1][Part1], its original schemas at `8e03b236a049849f2ccc24b4fd9fdce5ff69bed2`, [Common Part 1][Common], [Features Part 1][Features] and [HTTP semantics][Http]. Creation uses Glaux's selected Features Part 4 draft at `9ca25f56a58ed822ea8a685a7a41afa7181aaa8b`, not an invented claim that the selected draft is a published CSAPI rule. The existing audit supplies the detailed source chain; launcher review must check the particular assertions against those sources before execution.

Select OSH Core **`v2.0.2`, commit `235c0eabf24b6d6137b499b4402943d2794b70e6`**, reusing accepted [IDR-014A][OshStudy]. This is a deliberate stable comparison pin, not a claim to test current OSH development. The tag has no GitHub Release asset at the checked release-by-tag endpoint; build the pinned source rather than assume a downloadable release package exists.

Source inspection supports a minimal production runtime comprising `sensorhub-service-consys`, `sensorhub-datastore-h2` and their transitive runtime dependencies. OSH's [setup example][OshSetup] composes HTTP, a writable H2 file and the Connected Systems service; its [production entry point][OshLaunch] can start an external configuration. These establish a setup route, **not an executed compatibility result**. Do not run the default distribution configuration unchanged: it selects the legacy SWE API and unrelated services rather than this minimal Connected Systems setup. [Default configuration][OshDefault]

## Fixed request cases

Author fixture A directly from the [original feature/System schema][Schema]:

```json
{"type":"Feature","geometry":null,"properties":{"uid":"urn:glaux:review:phase1:step5:a","name":"Phase 1 comparison A","featureType":"sosa:Sensor"}}
```

Fixture B uses UID `urn:glaux:review:phase1:step5:b`, name `Phase 1 comparison B`, and type `http://www.w3.org/ns/sosa/Platform`. Those different literals catch returning the wrong System. Use the same submitted bytes for each server. Only the configured API root, that server's generated local IDs and the authentication setup differ.

| Group | Requests and independently expected checks | How to judge differences |
|---|---|---|
| 1. Discover | GET the configured API root; record links, API-description navigation and conformance declarations. Glaux's description must expose its actual POST and item GET/HEAD routes. | Common requirements 12–17 and Glaux's discovery contract. Do not require equal conformance lists, titles or OpenAPI documents. No Glaux collection GET exists yet. If OSH points to an external/static description, record that limitation without fetching an unapproved destination; subsequent requests may use the pinned `/systems` route, explicitly not a successful discovery walk. |
| 2. Create and retrieve A | POST A with `Content-Type: application/geo+json`; expect 201 and a usable Location. GET that Location with explicit GeoJSON Accept; check status 200, Feature structure, its own local ID, submitted UID/name/type and self link. | Part 1 requirements 5, 60 and 77–82; selected draft `post-response` A/B. Glaux's empty creation body and UUIDv7 are project choices, not peer-equality requirements. |
| 3. Distinguish B from A | Create/retrieve B; require a distinct local ID and B's independently supplied meaning. Retrieve A again and ensure it was not replaced. | Same sources. Both full URI and CURIE type forms are permitted; record spelling changes and assess permitted equivalence rather than guessing from similar names. |
| 4. Default representation and HEAD | GET A without Accept, then HEAD with explicit GeoJSON Accept. For supported HEAD, require no body and, if Content-Length is supplied, the selected GET representation's length. | RFC 9110 §§9.3.2/8.6. Glaux promises default GeoJSON and HEAD. Record OSH's default and HEAD behavior against its implemented surface; do not silently impose Glaux's format defaults. |
| 5. Missing and invalid IDs | GET one validly encoded but absent ID per server, then one syntactically invalid ID. | Features Part 1 §7.16.3/Table 3 and the Glaux retrieval contract. Establish absent-ID validity against each pinned encoding and fresh owned store before running; a Glaux UUID is not automatically a valid OSH ID. Do not conflate a malformed-ID 400 with missing-resource behavior. |
| 6. Invalid bodies | POST literal `{`; separately POST the valid shape with `properties.uid` omitted. Record rejection, status and body. Glaux expects 400. | Original required-field schema and Glaux's error catalog; RFC 9110 §15.5.1. A successful creation is not rejection. Recheck A/B after the negative cases; unchanged reads do not prove every internal side effect was absent. Do not invent a collection-count check. |
| 7. Format boundary | POST a plain-text body as `text/plain`; GET A asking only for SensorML. Glaux expects 415 and 406 respectively. | RFC 9110 §§15.5.16/15.5.7 and the documented Glaux subset. OSH may support the richer format; that is not a Glaux regression or an obligation to make OSH reject it. Explicit GeoJSON in groups 2–3 remains the common path. |
| 8. Ordinary restart | Stop and restart each serving process once, retaining its own datastore/configuration. Retrieve A/B using their original addresses and compare IDs and meaning with the fixed expectations. | Roadmap 1.5.2, Guide §8.2 and the configured persistence contract. This is not an extra OGC requirement, crash recovery or backup/restore. Keep the H2 file across OSH's restart; an in-memory peer is not a substitute. |

The [creation][Create] and [retrieval][Read] contracts supply Glaux-specific expectations, including its exact type preservation, protected caching, error form and stage-specific link set. Record those separately from shared standards assertions. Different credentials, generated IDs, base paths, additional permitted links, error descriptions and richer peer capabilities are not automatically defects.

## Independent checks and evidence

Use an independently authored HTTP driver and generic JSON interpretation, not either server's request/response types, test fixtures or serializers. A small Java standard-library socket transport can run inside the isolated container and return the raw HTTP/1.1 reply to a Python-standard-library checker on the runner. Request connection closure and bound the complete capture; do not use a client that silently discards an illegal HEAD payload before the checker sees it. This avoids assuming Python is installed inside the database image. The transport supplies no expected resource values and imports no OSH or Glaux model classes.

Before real requests, show the checker rejects known-bad responses: wrong/missing ID, A returned for B, changed UID/name/type, changed geometry, an unsafe Location/self link, and body/length errors for HEAD. Also show that an allowed additional peer link or member does not fail merely because Glaux's current shape is smaller. These are checker controls, not production mutations. Two implementations agreeing with each other can still be wrong; both are judged against fixed expectations.

Retain raw observations before any comparison normalization. Document every allowed normalization: object-member order, per-server base URI/local ID mapping, and the explicitly permitted full-URI/CURIE mappings. Never lowercase UIDs, ignore a missing/mistyped contractual field, or discard a changed value to make the results agree. Validate each server's local ID against its own Location and retained identity; do not normalize that consistency check away. This is a selected field/HTTP comparison, not an entire-schema validator or Annex A conformance run.

Resolve links normally, then require the approved exact scheme, host, port and path boundary. Disable automatic redirects; reject user-info, traversal and response-directed external destinations. Never send Glaux's synthetic authentication header to OSH. An advertised external schema or API document is recorded, not automatically fetched. No real credentials or operational data enter the run.

For each group report: expected answer and controlling source, both actual responses, any legitimate scope difference, and the unresolved question or proposed correction. Preserve setup-blocked, unrun, unsupported and failed outcomes distinctly. A standards ambiguity stays an ambiguity; neither majority agreement among peers nor an assistant's preference settles it.

## Hosted setup and limits

If approved, use **one temporary do-not-merge server PR** containing only diagnostic launchers, authored fixtures/configuration and the minimal workflow addition. Record the approval in the existing action list's Current state before execution. Add the comparison to `database-service`, not a new ungated job. Keep the six lanes, required `Rust bootstrap`, all ordinary checks, action pins, ownership guards and 30-minute lane cap unchanged. Separately review the exact setup diff before running it, and the resulting report before publishing it. Do not merge the diagnostic branch.

1. **Build before isolation.** Use the existing hosted Ubuntu 24.04 route, pinned Glaux Rust toolchain and locked dependencies. Fetch the exact OSH source; collect only its CSAPI/H2 production jars and runtime classpath using a temporary external Gradle init task. No peer tests, UI or replacement implementation. Record the resolved dependency versions and artifact hashes; an OSH source pin is not a fully locked dependency graph. Keep upstream licences/notices with the temporary artifacts; publish diagnostic records, not third-party binary bundles in the Glaux repository. If a dependency needs unavailable credentials or changed sources/versions, stop rather than request tokens, relax access or patch the peer.
2. **Selected additional hosted tools.** Temurin JDK `17.0.16+8` Linux x64, archive SHA-256 `166774efcf0f722f2ee18eba0039de2d685b350ee14d7b69e6f83437dafd2af1`; Gradle `8.10.2`, binary archive SHA-256 `31c55713e40233a8303827ceb42ca48a47267a0ad4bab9177123121e71524c26`. Verify the downloads, record actual versions and use no floating install. The upstream [CI][OshCi] selects Java 17 and its [wrapper][OshGradle] selects this Gradle version. This JDK pin is for isolated reproducibility, not a production security recommendation. [JDK checksum][JdkHash]; [Gradle checksum][GradleHash].
3. **Isolate runtime.** Reuse the task-owned [PostGIS harness][Harness] and its pinned image. Copy the JDK, built jars, configuration and transport helper into that container; do not add host mounts, ports, networks or weaken target validation. Run both services on distinct loopback ports. Use the existing Glaux migration/least-privilege serving-role pattern and a synthetic development identity. OSH uses a fresh writable H2 file, the selected `ConSysApiService` and a proposed `/sensorhub/api` root, with HTTP authentication/access control explicitly disabled only inside this network-disabled fixture. That difference excludes authentication/authorization equivalence from the claim. Limit OSH to `-Xmx128m` and a small H2 cache within the unchanged 768 MiB container limit; the transport JVM is bounded too.
4. **Verify setup, not just process existence.** Confirm database writability, selected module identity, actual routes, retained storage paths, listener addresses and bounded readiness before the cases. The peer's `enableTransactional` flag alone does not establish writability. OSH exposes port rather than bind-host configuration; use its supported external Jetty XML hook for loopback binding and verify the listener. [Service setup][OshSetup]; [HTTP connector/XML][OshHttp]. The classpath collector, XML configuration, JDK compatibility and combined memory fit remain unexecuted setup uncertainties. Do not silently substitute the old SWE API, an in-memory store, another peer version or a publicly reachable service if they fail.

Allow **at most 20 minutes for the entire new diagnostic**, including peer retrieval/build/startup, requests, restart and cleanup, further limited by the lane's remaining time with an evidence-upload margin. These are spending limits, not forecasts. Limit non-readiness HTTP calls to 60 across both servers; readiness polling is separately bounded per startup and by the same total deadline. Use bounded request/body/log sizes and preserve explicit truncation or timeouts; a truncated response cannot count as verified. Runtime requests stay inside the owned network-disabled environment; Internet use is limited to preparation downloads.

Allow **one specifically diagnosed setup/infrastructure retry at most**, preserving the original failure. Do not rerun a behavioral difference until green, increase the time/memory cap, replace dependencies or split automatically into more batches. If the budget expires, retain the exact uncompleted groups. Cleanup covers only validated owned processes, container and temporary paths, including failure paths; cleanup failure is reported and fails the run. A missing comparison, failed checker control, setup failure or unresolved behavioral failure must not become a passing lane. Expected, source-supported capability differences remain visible without being mislabeled as implementation defects.

## Delivery and next decision

The intended next iteration prepares the separately reviewed launcher, runs this bounded comparison and publishes its results. External setup may prevent that outcome; this proposal does not promise compatibility in advance. Preserve the request/response records, tool/source/dependency pins, commands, configuration, outcomes, checksums and decisive logs in the existing numbered evidence folder before hosted artifacts expire. Close the diagnostic PR **without merging** and retain its branch.

Step 5 completes only when all eight groups have been accounted for using actual observations and source-based dispositions; a setup failure or missing execution leaves the affected comparison incomplete. Observed defects can be reported without repairing them. Recommend any Glaux action in the existing findings register; do not change the approved Guide or adopt P1-02/P1-04/P1-05 implicitly.

**A new `proceed` would approve this bounded setup, hosted execution and result report only.** It would not authorize peer repairs, production fixes, a permanent test framework, client-application campaigns, more research, Phase 2 or gate closure. The full gate decision, open recommendations and provider-accounting limitations remain separate. Only the project lead closes [gate 1 (#339)](https://github.com/DGIWG-P507/glaux-server/issues/339).

[Clients]: ../../../../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-066-aleph-connected-systems-ui-client-study-report.md#short-cross-study-note--no-earlier-report-reopened
[OshStudy]: ../../../../../Research/Initial%20Designs/IDR/glaux-server/IDR%20Reports/idr-srv-014a-osh-csapi-server-implementation-study-report.md
[Part1]: https://docs.ogc.org/is/23-001/23-001.html
[Common]: https://docs.ogc.org/is/19-072/19-072.html
[Features]: https://docs.ogc.org/is/17-069r4/17-069r4.html
[Http]: https://www.rfc-editor.org/rfc/rfc9110.html
[Schema]: https://github.com/opengeospatial/ogcapi-connected-systems/blob/8e03b236a049849f2ccc24b4fd9fdce5ff69bed2/api/part1/openapi/schemas/geojson/system.json
[Create]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/system-create.md
[Read]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/docs/system-read.md
[Harness]: https://github.com/DGIWG-P507/glaux-server/blob/27955c1b9260cd811ad6bc08f85feab43ad65028/scripts/database_harness.py
[OshSetup]: https://github.com/opensensorhub/osh-core/blob/235c0eabf24b6d6137b499b4402943d2794b70e6/sensorhub-service-consys/src/test/java/org/sensorhub/impl/service/consys/AbstractTestApiBase.java#L86
[OshLaunch]: https://github.com/opensensorhub/osh-core/blob/235c0eabf24b6d6137b499b4402943d2794b70e6/dist/main/launch.sh
[OshDefault]: https://github.com/opensensorhub/osh-core/blob/235c0eabf24b6d6137b499b4402943d2794b70e6/dist/main/config.json
[OshCi]: https://github.com/opensensorhub/osh-core/blob/235c0eabf24b6d6137b499b4402943d2794b70e6/.github/workflows/gradle_build.yml
[OshGradle]: https://github.com/opensensorhub/osh-core/blob/235c0eabf24b6d6137b499b4402943d2794b70e6/gradle/wrapper/gradle-wrapper.properties
[OshHttp]: https://github.com/opensensorhub/osh-core/blob/235c0eabf24b6d6137b499b4402943d2794b70e6/sensorhub-core/src/main/java/org/sensorhub/impl/service/HttpServer.java#L259
[JdkHash]: https://github.com/adoptium/temurin17-binaries/releases/download/jdk-17.0.16%2B8/OpenJDK17U-jdk_x64_linux_hotspot_17.0.16_8.tar.gz.sha256.txt
[GradleHash]: https://services.gradle.org/distributions/gradle-8.10.2-bin.zip.sha256

# OpenSensorHub comparison results

**Status:** bounded execution finished; Step 5 remains incomplete. Glaux met all eight selected groups. Ten OSH item-dependent cases and its conformance-document navigation were not executed.<br>
**Step and date:** Phase 1 review, Step 5; 4 October 2026.<br>
**Prepared for:** the Glaux project lead and Phase 1 implementers.<br>
**Authority:** the project lead's `proceed` after the [proposal](07-opensensorhub-comparison-proposal.md), recorded before execution in [planning PR #120](https://github.com/DGIWG-P507/glaux/pull/120), merge `8c159b05a4642f90735acecd24e426d12e66df12`.<br>
**Prepared by:** Codex (OpenAI), with a separate runtime author and non-author reviewer. Exact model versions are not exposed; prior project context was present. This is not human expert or cross-provider assurance. The delivery PR records the reviewed report commit.<br>
**Baselines:** Glaux Server `27955c1b9260cd811ad6bc08f85feab43ad65028`; planning `8c159b05a4642f90735acecd24e426d12e66df12`. All fifteen review gates remained open at setup.

## Scope and expectations

The [approved proposal](07-opensensorhub-comparison-proposal.md#fixed-request-cases) fixes eight groups: discovery, System A creation/retrieval, distinct System B, default representation and HEAD, missing/invalid identifiers, invalid bodies, format boundaries and ordinary restart. The two submitted Systems are authored independently, with different UIDs, names and types. Each server is checked against those expectations, not against the other server's output.

The temporary [server PR #360](https://github.com/DGIWG-P507/glaux-server/pull/360) contains only diagnostic files and four workflow lines, inside the existing `database-service` lane after its ordinary checks. It was closed without merging; its branch is retained. No server implementation, peer source, dependency, permanent setting, Guide or Roadmap change is authorized. Phase 2 and gate closure remain separate.

The pinned OSH Core source is `235c0eabf24b6d6137b499b4402943d2794b70e6` (`v2.0.2`). The launcher verifies the proposal's Temurin 17.0.16+8 and Gradle 8.10.2 archive checksums. Its source build collects only CSAPI/H2 production modules and dependencies; actual resolved artifacts are inventoried, not described as a fully locked dependency graph. Runtime uses the unchanged owned, network-disabled PostGIS container with no host mounts or published ports, separate loopback listeners and a persistent H2 file for OSH. Authentication equivalence is outside scope.

Raw HTTP/1.1 requests and responses are retained before generic JSON interpretation, including any illegal HEAD payload. Allowed normalizations are JSON member ordering, per-server origin/local identity mapping and the two explicitly named CURIE/full-URI type pairs. Glaux must preserve its submitted type spelling. Additional permitted peer members are not rejected just because Glaux's initial slice is smaller. This is selected-field checking, not complete schema validation or an Annex A conformance run.

## First attempt: launcher inspection failure

[Run 37175084090](https://github.com/DGIWG-P507/glaux-server/actions/runs/37175084090) used separately reviewed head `db7974114966c5978f2ddea45165bc62f0453149`. Reviewer `/root/osh_proposal_setup` inspected the complete diff, current contracts, pinned peer sources and isolation harness before execution. Pre-execution findings about interruption cleanup and listener handling were corrected in that head.

All ordinary checks passed, including those in the comparison's own lane. The new diagnostic stopped after **116.735 seconds**, before any comparison request. Its effective allowance was 1,029.543 seconds because the existing lane had already consumed part of its unchanged 30-minute limit.

What actually ran:

- All 18 independent response-checker controls and 18 framing/destination controls passed.
- Both tool downloads matched their pinned checksums. OSH's production build completed in 41.421 seconds and produced a 51-jar runtime inventory.
- Glaux started and repeatedly returned HTTP 200 at its configured root. Its listener was observed at `127.0.0.1:18080`, owned by UID 999.
- The launcher's executable-identity inspection repeatedly failed before it could declare readiness. It ran `readlink` on the service process's `/proc` entry as container root, while the service ran as `postgres`. The same inspection inside the graceful-stop guard also failed.
- The unchanged ownership-checked harness nevertheless removed the owned container and verified its absence. **Graceful cleanup is still recorded as failed**; successful container removal does not erase that failure.
- All 534 files listed in the diagnostic evidence manifest hash-match. All 2,325 tracked peer-source hashes matched before/after; Glaux's tracked sources and original corpus also remained unchanged.

The logs establish a diagnostic process-inspection failure, not a Glaux HTTP failure or an OSH behavior result. Linux process-access rules and Docker's default capabilities support the permission explanation, but the failed command did not expose its exact errno, so that mechanism is an inference. The retry uses the service's own UID for inspection and shutdown; it adds no capability and retains exact container, executable, socket-owner and loopback checks. [Linux process information](https://docs.kernel.org/filesystems/proc.html#process-specific-subdirectories), [Docker runtime capabilities](https://docs.docker.com/engine/containers/run/#runtime-privilege-and-linux-capabilities).

All eight groups on both servers remain **unrun in this attempt**. OSH was built but its serving process had not started. No System comparison, H2 persistence result or compatibility claim follows from this attempt.

## Sole diagnosed retry

The retry correction is limited to the launcher's process inspection, with non-signalling positive/wrong-executable/positive controls using its actual shutdown guard. It does not change the requests or expected answers. Its maximum diagnostic allowance is reduced to 1,083 seconds: together with the first attempt, at most 1,199.735 seconds, further bounded by the remaining lane time and upload margin. The existing ordinary checks still run; no automatic retry loop is added.

The same non-author reviewer inspected the original artifact and the complete two-file correction at head `b09898d3bcda3513fe963c704bcb2369384631e2` before [retry run 37176046649](https://github.com/DGIWG-P507/glaux-server/actions/runs/37176046649), with no blocking findings. No further attempt is authorized by this iteration.

The retry ran for **57.288 seconds**, bringing the two diagnostics to **174.023 seconds** in total. It made **29 non-readiness requests** across both servers, below the 60-request cap. Both services started and restarted once on their verified loopback listeners with the same respective datastores. The ownership controls passed for each service; graceful cleanup completed, and the owned container's absence was verified. All 245 diagnostic manifest entries hash-match; all 2,325 peer-source hashes and Glaux's original sources/corpus remained unchanged.

All ordinary checks again passed. The temporary diagnostic, its containing lane and the required `Rust bootstrap` result **failed**, correctly retaining the incomplete comparison. The five other lanes passed. A failed diagnostic is not relabeled green because the ordinary checks passed.

### Actual coverage

The proposal's [fixed-case table](07-opensensorhub-comparison-proposal.md#fixed-request-cases) and [source audit](02-test-source-audit.md) remain the source chain. The table below reports actual observations, not equality to peer output. File numbers refer to `phase1-osh/request-NNN-*.response.bin` inside the unchanged retry archive. `case-results.json` retains the checker output, with its discovery mistake corrected explicitly below.

| Group / expected boundary | Glaux observation | OSH observation and disposition |
|---|---|---|
| 1. Discovery: advertised surface, not equal declarations | Root, linked API description and conformance document returned 200 (010–012). POST and item GET/HEAD were described; `conformsTo` was empty, appropriate to the unfinished classes. | Root returned 200 (029). External API-description links were deliberately not followed. The local conformance link **was advertised but overlooked by our checker**, so its document was not fetched. **Partial**, despite the raw checker's `accounted` label. |
| 2. A: 201, usable Location, correct returned identity and meaning | 201 then 200 at its Location (013–014); independent A values, null geometry and self link matched. | POST returned 201 (030), with `Location: /systems/080g`. Resolving it left the approved API path; no item GET occurred. **Creation response observed; retrieval unrun.** |
| 3. B: distinct identity and values; A unchanged | 201 then 200 for distinct B, then A still correct (015–017). | POST returned 201 with distinct `/systems/0810` (031). B retrieval and A recheck were **unrun** for the same Location reason. |
| 4. Default and HEAD: selected representation and no HEAD body | Default and explicit GET returned GeoJSON A (018–019). HEAD returned 200, zero body bytes and Content-Length 345, matching the explicit GET (020). | Default GET and the HEAD case **unrun**. No claim about OSH's default or HEAD support. |
| 5. Valid absent and malformed IDs: distinguish the requests | Both returned 404 with Problem Details (021–022), matching the Glaux retrieval contract. | Both returned 404 (032–033). The absent ID uses the pinned OSH encoding; the other was deliberately malformed. **Accounted**, not an assertion that all malformed inputs must use 404. |
| 6. Invalid bodies: reject; recheck retained A/B | Literal `{` and missing UID each returned 400; A/B remained correct (023–026). | Literal `{` returned **500**, with a logged EOFException (034); missing UID returned 400 (035). Both A/B rechecks **unrun**. Error handling differs; this does not establish absence of internal side effects. |
| 7. Format boundary: rejection versus richer peer capability | Plain-text POST returned 415; SensorML-only GET returned 406 (027–028), as its documented subset requires. | Plain-text POST returned 400 with an unsupported-format message (036). This is rejection with different specificity, not permission to copy that status into Glaux. SensorML GET **unrun**; no richer-format compatibility claim. |
| 8. Ordinary restart: same addresses and values after restart | Both original URLs returned 200 with unchanged A/B identity and selected fields (044–045). | Process restart succeeded with the H2 file retained, but both post-restart item reads were **unrun**. File retention alone does not prove resource persistence. |

Glaux's **19 case records across eight groups** are accounted for. Its protected System responses also carried `private, no-store`; item GET/HEAD carried `Vary: Accept` and no ETag or Last-Modified. The GET representations contained the expected parentless self link; HEAD had no body. These are the current Glaux contract checks, not imposed peer parity.

OSH's raw 19 records contain six `accounted`, three `behavior-difference-unresolved` and ten `unrun` entries. Those are not nineteen completed checks: one of the six accounted records is the incomplete discovery case. After correcting that interpretation, **only group 5 is fully accounted for**; the other seven groups contain the limits shown above. Case counts include the restart action and multi-request cases, so they are not HTTP-request counts.

### What caused the differences

**Creation links: observed peer behavior, not failed POSTs.** OSH emitted root-relative `/systems/{id}` values even though its service ran at `/sensorhub/api`. Under [RFC 9110 §10.2.2](https://www.rfc-editor.org/rfc/rfc9110.html#section-10.2.2) and [RFC 3986 §5.2.2](https://www.rfc-editor.org/rfc/rfc3986.html#section-5.2.2), the leading slash replaces the base path; it does not preserve the API prefix. The guard therefore correctly refused the resulting destination. We neither requested that out-of-bound URL nor silently repaired it, so there is no observed item 404 or successful retrieval to report.

Pinned OSH [BaseResourceHandler](https://github.com/opensensorhub/osh-core/blob/235c0eabf24b6d6137b499b4402943d2794b70e6/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/resource/BaseResourceHandler.java#L504) constructs that path; [RestApiServlet](https://github.com/opensensorhub/osh-core/blob/235c0eabf24b6d6137b499b4402943d2794b70e6/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/RestApiServlet.java#L219) writes it unchanged. This path does not consult the configured public/proxy API root. Thus simply changing that configuration is not an evidenced cure. This is a link-navigation discrepancy in this pinned deployment, not a general verdict on OSH or a reason to alter Glaux's usable absolute Location in this fixture. Glaux was configured at the origin root, so this run does not test preservation of a nonempty path prefix.

**Malformed JSON: an observed error-handling difference.** The exact request was a one-byte body `{`, with matching Content-Length. The peer log records EOFException from JSON reading and the servlet returns 500. Its [feature deserializer](https://github.com/opensensorhub/osh-core/blob/235c0eabf24b6d6137b499b4402943d2794b70e6/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/feature/AbstractFeatureBindingGeoJson.java#L82) and [JSON binding](https://github.com/opensensorhub/osh-core/blob/235c0eabf24b6d6137b499b4402943d2794b70e6/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/resource/ResourceBindingJson.java#L110) do not map that exception to their invalid-input response; the [generic servlet handler](https://github.com/opensensorhub/osh-core/blob/235c0eabf24b6d6137b499b4402943d2794b70e6/sensorhub-service-consys/src/main/java/org/sensorhub/impl/service/consys/RestApiServlet.java#L247) handles it instead. Glaux's 400 matches its contract and HTTP's client-error meaning. This is not a setup failure, and no peer patch or full-conformance judgment is proposed.

**Discovery: our checker missed a link spelling.** The raw landing response and the recorded link array both contain `rel: "conformance"` pointing to the approved local endpoint. The [diagnostic matcher](https://github.com/DGIWG-P507/glaux-server/blob/b09898d3bcda3513fe963c704bcb2369384631e2/scripts/review_phase1_osh_cases.py#L240) recognized only the full OGC relation URI. Its `not-advertised` observation and `accounted` discovery label are therefore insufficient and corrected here: **advertised, not fetched**. This is an error in our diagnostic, not an OSH absence. The 18 checker controls did not cover this spelling; passing those controls did not prove the whole checker correct. Neither the archive nor the failed run is rewritten.

## Limits and next decision

This iteration establishes useful Glaux behavior on independent fixtures but **does not finish Step 5 or establish OSH interoperability**. No new Glaux implementation defect was demonstrated; P1-02, P1-04 and P1-05 remain open and unadopted. The first setup failure, the retry's peer differences, the checker mistake and every missing request remain explicit. Security equivalence, full schema/conformance validation, crash recovery, richer resource operations and application-client compatibility remain outside scope.

**Recommended next, only on a new `proceed`: one narrow completion run, not another research study.** Reuse these Glaux results. Correct the peer discovery matcher with a discriminating bare-relation control, then run the same fixed OSH cases on a fresh fixture. For item-dependent checks only, permit a clearly labeled diagnostic adapter that maps the exact pinned `/systems/{valid-local-id}` creation path into the configured OSH API prefix. Retain the original header and its navigation discrepancy separately; an adapted read must never become proof that the original Location works. Keep exact-origin/path checks on the resulting URL, reject every other shape, and prove those restrictions with positive and negative controls. Do not patch OSH, relax Glaux expectations or hide the observed malformed-input 500.

That follow-up would use the same source/tool pins and isolated hosted setup, unchanged six lanes and gate, at most 20 diagnostic minutes and 30 non-readiness requests, **one run and no automatic retry**, with separate review before execution and report publication. Close its temporary PR without merging. Stop and report any remaining gaps; do not launch more runs automatically. These are proposed bounds, not current authorization. Gate 1 remains open and Phase 2 blocked until the project lead acts.

## Preserved evidence

Original hosted artifact ZIPs are retained unchanged rather than reconstructed from selected logs. They include the diagnostic's commands, configuration, downloads, dependency inventory, controls, request/response files, source hashes, outcomes and errors, alongside the ordinary lane evidence.

| Attempt | Artifact and metadata | Original bytes / SHA-256 |
|---|---|---|
| Initial setup failure | [ZIP](08-osh-comparison/initial-attempt.zip), artifact `11293720459`; [run metadata](08-osh-comparison/initial-run-metadata.json) | 634,475 / `f3b8fc1e84a88df3a5a9afe776a72f076cefcc688019a16ab10ec49da4ecd059` |
| Sole retry, partial comparison | [ZIP](08-osh-comparison/retry-results.zip), artifact `11293357796`; [run metadata](08-osh-comparison/retry-run-metadata.json) | 436,658 / `7970023df0313f82e5e18b9559b61c2175ca9dc9c6ae3786350629b302114697` |

No local software was installed or build/test/service executed. Only editing, Git operations and reading/downloading the preserved diagnostic records occurred on the laptop. Review recommendations remain unadopted, and only the project lead may close [gate 1 (#339)](https://github.com/DGIWG-P507/glaux-server/issues/339).

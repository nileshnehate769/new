# PHASE 2 — Existing Download Engine Hardening, Critical Security Remediation & Reliability Improvements

You have completed Phase 1 and documented the existing Download Manager Pro architecture.

Now implement Phase 2 directly in the existing repository.

## 1. Primary Objective

Harden the existing Flutter/Dart download engine before introducing the Rust native engine.

This phase must prioritize:

1. Critical security remediation.
2. SSRF and redirect protection.
3. Browser IPC authentication.
4. Download engine consistency.
5. Global bandwidth throttling.
6. Reliable segment persistence and recovery.
7. Post-download workflow integration.

**Do not start implementing the Rust engine in this phase.**

## 2. Mandatory Repository Inspection

Before modifying anything:

- Read all twelve Phase 1 audit documents.
- Inspect the actual implementation of every affected Dart file.
- Trace the download lifecycle from UI to controller, engine, HTTP request, file writing, and database.
- Inspect browser extension and native messaging authentication.
- Inspect existing tests and identify missing regression coverage.
- Identify the current implementation of `RobustSegmentedDownloader` and `SegmentedDownloadTask`.

Do not rely exclusively on the Phase 1 report. Verify every finding against the actual source code.

Preserve existing architecture wherever possible.

## 3. Critical Security Remediation

### SEC-02 — Browser IPC Authentication

Inspect:

- `lib/main.dart`
- `lib/services/browser_integration/browser_ipc_server.dart`
- `browser_extension/`
- `native_host/`

Requirements:

- Remove the empty authentication-token configuration.
- Generate a cryptographically secure random pairing secret.
- Protect persistent secrets using Windows DPAPI or an equally appropriate Windows-protected storage mechanism.
- Never expose the secret in application logs, diagnostic bundles, URLs, or ordinary settings exports.
- Require authentication for sensitive IPC operations.
- Validate message schemas and action allowlists.
- Apply strict request size limits, timeouts, and rate limits.
- Maintain strict Host and Origin validation.
- Prevent unauthorized local processes from controlling downloads.
- Design a secure pairing and token-delivery flow compatible with the existing Chrome and Edge extension.

Do not assume that checking the browser Origin header is sufficient authentication.

Do not break existing native messaging functionality.

### SEC-03 — IPv4-Mapped IPv6 SSRF Protection

Inspect:

`lib/core/network/network_security_validator.dart`

Ensure that the validator correctly rejects:

- IPv4 loopback addresses.
- IPv4 private addresses.
- IPv4 link-local addresses.
- Cloud metadata addresses.
- IPv6 loopback.
- IPv6 unique-local addresses.
- IPv6 link-local addresses.
- IPv4-mapped IPv6 addresses, including `::ffff:127.0.0.1`.
- IPv4-mapped private and link-local destinations.

Add regression tests for IPv4-mapped IPv6 addresses.

Do not rely exclusively on textual URL matching. Normalize and inspect actual resolved address bytes.

Preserve legitimate public HTTP and HTTPS downloads.

### SEC-04 — Redirect SSRF Protection

Inspect:

`lib/services/download_engine/segmented_download_task.dart`

Especially:

`_executeSingleStreamFallback()`

Requirements:

- Disable automatic unrestricted redirects.
- Process redirects manually.
- Set a reasonable maximum redirect count, such as five.
- Validate every redirect URL.
- Resolve and validate destination addresses at each hop.
- Prevent DNS rebinding and DNS time-of-check/time-of-use bypasses by ensuring the connection uses an address that was actually validated.
- Reject redirects to restricted destinations.
- Preserve correct HTTP method and header behavior.
- Never forward sensitive authorization headers to an unrelated origin.
- Close abandoned responses and sockets safely.

Add integration tests using controlled local HTTP servers.

A redirect to loopback or another restricted destination must be rejected.

## 4. Engine Unification

Inspect:

- `dart_download_engine.dart`
- `segmented_download_task.dart`
- `robust_segmented_downloader.dart`
- `segment_worker.dart`
- `direct_offset_file_sink.dart`
- `file_assembler.dart`
- `downloads_controller.dart`

The audit identifies two parallel segmented download implementations.

Evaluate whether `RobustSegmentedDownloader` can safely become the production download implementation.

If compatible:

- Integrate it behind the existing `IDownloadEngine` abstraction.
- Preserve existing public engine APIs where possible.
- Preserve pause, resume, cancel, retry, progress, and completion behavior.
- Retain the single-stream fallback for servers that do not support Range requests.
- Preserve existing file integrity verification.
- Keep existing downloads and database records compatible.

Do not delete the older implementation until the replacement is integrated and verified.

If direct unification would introduce regressions, document the blocker and implement a safe adapter instead.

## 5. Global Bandwidth Throttling

Inspect:

- `bandwidth_throttler.dart`
- `downloads_controller.dart`
- `dart_download_engine.dart`
- `segment_worker.dart`

The audit identifies a disconnected global throttler.

Implement proper bandwidth limiting across all active downloads.

Requirements:

- Global bandwidth limit.
- Per-download bandwidth limit if already supported by the existing design.
- Correct coordination across simultaneous segments.
- Dynamic speed-profile changes without restarting downloads.
- No artificial UI-only speed display.
- Accurate measured throughput.
- No deadlocks or excessive buffering.

Add tests proving that the actual download data flow respects configured limits.

## 6. SQLite Segment Persistence

Inspect:

- `database_service.dart`
- `migrations.dart`
- `sqlite_download_repository.dart`
- `download_segments` schema
- `downloads_controller.dart`

Implement persistent segment information.

Requirements:

- Save segment boundaries.
- Save committed byte offsets.
- Save segment status.
- Save the associated download ID.
- Restore segment information after application restart.
- Validate persisted segment metadata before trusting it.
- Detect missing or corrupted partial files.
- Safely recover interrupted downloads.
- Use database transactions for related updates.
- Maintain backward compatibility with existing database versions.

Never assume that a database byte count proves that the corresponding bytes exist on disk.

Verify file size and actual segment data before resuming.

Do not delete completed user files during recovery.

## 7. Post-Download Workflow Integration

Inspect:

- `workflow_engine.dart`
- `downloads_controller.dart`
- `download_event_monitor.dart`

The audit reports that `WorkflowEngine.dispatchTrigger()` is not connected to download completion events.

Implement reliable workflow dispatch for supported events.

Requirements:

- Trigger workflows at the correct state transition.
- Prevent duplicate execution.
- Handle application restarts safely.
- Do not execute a workflow before the download is finalized.
- Ensure failed downloads do not trigger successful-completion actions.
- Preserve existing workflow conditions and action validation.

## 8. Security and Reliability Regression Tests

Add tests covering:

- IPv4-mapped IPv6 loopback.
- IPv4-mapped private addresses.
- IPv4-mapped link-local addresses.
- Redirect to loopback.
- Redirect to private IP.
- Redirect chain exceeding the maximum.
- DNS rebinding protection.
- Cross-origin authorization-header handling.
- Missing or invalid IPC authentication.
- Unauthorized local IPC requests.
- Oversized IPC payloads.
- Corrupted segment metadata.
- Missing segment files.
- Application restart recovery.
- Global bandwidth throttling across multiple segments.
- Duplicate workflow dispatch prevention.

Use controlled test servers and temporary directories.

Never use dangerous external URLs or access real private network services for security testing.

## 9. Strict Preservation Requirements

Do not:

- Rebuild the Flutter application.
- Redesign the UI.
- Replace the database.
- Remove Chrome or Edge extension support.
- Remove native messaging.
- Introduce the Rust engine.
- Add HLS/DASH or torrent functionality.
- Implement licensing in this phase.
- Delete existing working functionality.
- Change unrelated files without justification.

Preserve existing user settings, download history, completed files, and database compatibility.

## 10. Required Testing

Run the following commands from the actual project directory:

```powershell
flutter pub get
flutter analyze
flutter test
flutter build windows --release
```

If any command fails:

- Record the exact error.
- Identify whether it is caused by your changes or an existing environment issue.
- Fix issues introduced by this phase.
- Do not claim success without actual execution.

Also perform manual verification where possible:

- Start a normal HTTP download.
- Start a segmented download.
- Pause and resume.
- Restart the application and recover an interrupted download.
- Test the global bandwidth limit.
- Test Chrome and Edge extension communication.
- Test invalid IPC authentication.
- Test redirect security.

## 11. Documentation

Create or update:

1. `docs/PHASE_2_IMPLEMENTATION_REPORT.md`
2. `docs/PHASE_2_SECURITY_REPORT.md`
3. `docs/PHASE_2_TEST_REPORT.md`
4. `docs/PHASE_2_FILE_CHANGES.md`

Include:

- Exact files modified.
- Vulnerabilities fixed.
- Implementation details.
- Database changes and migration compatibility.
- Tests added.
- Actual test results.
- Remaining security limitations.
- Any unresolved blockers.

## 12. Final Completion Criteria

Phase 2 is complete only when:

- Critical IPC authentication gaps are addressed.
- IPv4-mapped IPv6 SSRF protection is tested.
- Redirect security is implemented and tested.
- Existing download functionality remains operational.
- The production engine uses a consistent, verified download pipeline.
- Global bandwidth throttling actually affects network transfers.
- Segment metadata is persisted and recoverable.
- Post-download workflows are correctly integrated.
- Flutter analysis and automated tests pass.
- A Windows release build succeeds, or any genuine environmental blocker is documented.

At the end, provide a clear PASS / PARTIAL / BLOCKED status for each objective.

**Begin by inspecting the repository. Implement Phase 2 in controlled steps, test each change, and preserve the existing working application. Do not proceed to Phase 3.**

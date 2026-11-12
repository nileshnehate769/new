# PHASE 1 — DOWNLOAD MANAGER PRO
## Existing Project Audit, Architecture Verification, Security Assessment & Professional Upgrade Blueprint

You are acting as a Senior Windows Desktop Software Architect, Flutter Engineer, Rust Systems Engineer, Network Programming Specialist, Windows Security Engineer and Software QA Lead.

Your task is to conduct a complete professional technical audit of my existing Download Manager Pro project and prepare it for development into an advanced Windows download manager inspired by the best capabilities of Internet Download Manager and Plexo.

This is an existing-project-first assignment.

**DO NOT blindly create a new project. DO NOT replace working code. DO NOT remove existing features simply because you prefer another implementation.**

Your first responsibility is to understand the actual repository, identify what already works, discover architectural problems and establish a safe upgrade strategy.

---

# 1. PRODUCT VISION

Product name: Download Manager Pro

Target platform: Windows 10 and Windows 11, 64-bit.

Application architecture:
- Flutter Desktop frontend.
- Rust native download engine, if justified by the audit.
- Secure local IPC or an appropriate native FFI bridge.
- Windows networking APIs.
- Modular browser integration.
- Secure download storage and recovery.

Commercial model:
- Lifetime licence.
- One-time activation.
- No mandatory monthly subscription.
- Offline download functionality must not depend on a remote licensing server.

Main objective:

Build a powerful and reliable Windows download manager with:

1. IDM-style multi-part downloading.
2. Plexo-style multi-network download aggregation.
3. Intelligent network adapter selection.
4. Adaptive chunk scheduling.
5. Advanced download queue.
6. Browser integration.
7. Media downloading where legally permitted.
8. Torrent support as a separate module.
9. Automatic pause/resume and crash recovery.
10. Professional analytics and bandwidth monitoring.
11. Strong security and privacy protection.

Security must be treated as a core architectural requirement, not an optional feature.

# 2. STRICT EXISTING PROJECT INSPECTION

Before making changes, inspect the actual repository.

Examine:

- Complete folder and file structure.
- pubspec.yaml.
- pubspec.lock.
- All Dart source files.
- All Flutter screens and widgets.
- State management implementation.
- Existing download engine.
- Existing native C++, Rust or other code.
- Windows runner and CMake configuration.
- Windows manifests and permissions.
- Existing plugins and dependencies.
- Browser extensions and messaging components.
- Settings and configuration storage.
- Download history and database.
- Temporary file handling.
- Logging implementation.
- Installer and packaging configuration.
- Existing tests and build scripts.
- Existing documentation.

Do not assume that a feature is implemented merely because a screen or button exists.

Trace important functionality from UI interaction through business logic to the actual network or filesystem operation.

Identify incomplete implementations, mock data, placeholders, nonfunctional buttons, unsafe operations and duplicated logic.

Inspect dependency versions and compatibility before proposing upgrades.

Do not upgrade all dependencies automatically.

# 3. EXISTING FUNCTIONALITY PRESERVATION

Create a complete inventory of existing functionality.

Classify every feature as:

- Working and verified.
- Implemented but not verified.
- Partially implemented.
- Broken.
- UI-only or placeholder.
- Missing.

For every working feature:

- Identify its source files.
- Identify its dependencies.
- Record how it works.
- Document its current behavior.
- Identify potential regression risks.

Preserve working functionality.

Do not:
- Rewrite working modules unnecessarily.
- Rename public APIs without justification.
- Delete existing database records.
- Change existing settings formats without migration.
- Remove existing user functionality.
- Replace working components with placeholder implementations.
- Introduce breaking changes without documenting them.

If a migration is necessary, create a backward-compatible migration strategy.

# 4. ARCHITECTURE EVALUATION

Evaluate the existing architecture and recommend the best upgrade path.

Preferred target:

Flutter Desktop UI + Rust download engine.

However, do not automatically replace a functional existing native engine.

Compare:

A. Existing engine with improvements.
B. Flutter + Rust native library.
C. Flutter + separate Rust engine process.
D. Flutter + C++ native engine.

Evaluate:

- Performance.
- Memory consumption.
- Security isolation.
- Windows compatibility.
- IPC complexity.
- Maintenance.
- Debugging.
- Packaging.
- Dependency stability.

Recommend one architecture with technical justification.

If using a separate Rust process, evaluate Windows named pipes with authenticated connections and strict message validation.

Avoid exposing an unauthenticated localhost HTTP control server.

Define clear boundaries between:

1. Presentation layer.
2. Application service layer.
3. Download engine.
4. Network interface manager.
5. Chunk scheduler.
6. Storage manager.
7. Security controller.
8. Browser integration.
9. Licensing module.
10. Logging and diagnostics.

Define typed commands and events between the frontend and engine.

Include engine lifecycle management, startup failure handling, crash detection and graceful shutdown.

# 5. MULTI-NETWORK DOWNLOAD FEASIBILITY

This is one of the most important product features.

Study the feasibility of combining:

- Wi-Fi.
- Ethernet.
- USB tethering.
- Other independently routed Windows network interfaces.

Evaluate Windows network interface enumeration and socket-level interface binding.

Investigate:

- Windows adapter identification.
- Interface indexes.
- IPv4 and IPv6 handling.
- Source address selection.
- Socket binding.
- Routing behavior.
- DNS resolution per interface.
- Independent network path verification.
- Connection failure detection.
- Interface disconnection.
- Automatic recovery.
- Per-interface bandwidth monitoring.

Determine how the application can send different HTTP range requests through different interfaces.

Evaluate server compatibility:

- HTTP Range support.
- Content-Length.
- Content-Range.
- ETag.
- Last-Modified.
- If-Range.
- Content-Encoding.
- Redirect handling.
- Authentication.
- CDN behavior.
- Server-side IP/session restrictions.

Important:

Do not promise combined bandwidth when the underlying routes are not independent.

Do not assume that multiple adapters automatically provide aggregated bandwidth.

Do not combine chunks from different file versions.

Recommend a safe fallback when multi-network aggregation is unavailable.

Create a technical feasibility report with identified Windows APIs, Rust libraries and implementation risks.

# 6. DOWNLOAD ENGINE DESIGN

Design the future download engine around the following requirements.

HTTP/HTTPS:
- Direct downloads.
- Multi-part downloads.
- HTTP Range requests.
- Adaptive chunk sizing.
- Dynamic concurrency.
- Pause/resume.
- Automatic retries.
- Connection timeout.
- Read timeout.
- Retry backoff.
- Download prioritization.
- Server capability detection.
- File size validation.
- Integrity verification.

Chunk management:
- Unique chunk identification.
- Exact byte ranges.
- Per-chunk state.
- Atomic progress updates.
- Failed chunk reassignment.
- Duplicate-write prevention.
- Safe chunk merging.
- Crash recovery.

Download state:
- Queued.
- Connecting.
- Downloading.
- Paused.
- Retrying.
- Verifying.
- Completed.
- Failed.
- Cancelled.

Ensure that downloaded files are never marked complete until all expected bytes have been verified.

If the remote resource changes during a download, detect the mismatch and restart or safely recover. Never silently combine incompatible chunks.

# 7. SECURITY AUDIT

Perform a threat-model-based security assessment.

Consider:

- Malicious URLs.
- Malicious redirects.
- SSRF.
- DNS rebinding.
- Localhost access.
- Private network access.
- Link-local IP access.
- IPv4 and IPv6 bypasses.
- Path traversal.
- Malicious filenames.
- Symbolic links and reparse points.
- Archive extraction risks.
- Malicious downloads.
- Browser credential exposure.
- Cookie theft.
- IPC impersonation.
- Malformed IPC messages.
- Untrusted browser extensions.
- Dependency supply-chain attacks.
- Update tampering.
- Resource exhaustion.
- Disk exhaustion.
- Log leakage.
- Temporary file leakage.

Mandatory security requirements:

A. URL validation

- Only support explicitly approved URL schemes.
- Reject file:, javascript:, data: and unknown schemes.
- Normalize and parse URLs safely.
- Validate every redirect destination.
- Resolve and validate destination IP addresses.
- Protect against DNS rebinding and address changes.
- Block loopback, private, link-local and reserved destinations by default.
- Prevent IPv4-mapped IPv6 bypasses.
- Recheck destinations when opening connections.
- Do not permit arbitrary filesystem access through a download URL.

B. Filesystem protection

- Use an application-controlled temporary workspace.
- Restrict temporary writes to that workspace.
- Generate safe internal filenames.
- Prevent path traversal.
- Do not trust Content-Disposition filenames.
- Protect against symlink and reparse-point attacks.
- Prevent accidental overwrite of unrelated files.
- Clean up only application-owned temporary files.
- Never automatically execute downloaded files.

C. Browser security

- Do not collect browser cookies unnecessarily.
- Request user authorization before accessing authenticated resources.
- Never log passwords, cookies, bearer tokens or sensitive headers.
- Restrict native messaging to trusted extension identities.
- Validate every message received from browser extensions.
- Keep browser credentials out of general application logs.

D. Process isolation

- Restrict Rust engine privileges.
- Avoid administrator privileges unless specifically required.
- Restrict IPC access to the current user.
- Validate message sizes and schemas.
- Apply resource limits.
- Handle unexpected engine termination safely.

E. Privacy

- Avoid persistent visited-site history unless the user explicitly enables it.
- Provide configurable download history retention.
- Use secure temporary storage.
- Remove abandoned temporary chunks safely.
- Do not collect browsing information for analytics.
- Do not transmit download URLs or personal information to third parties without explicit consent.

Provide a threat model containing:
- Threat.
- Attack scenario.
- Potential impact.
- Mitigation.
- Verification test.

# 8. STORAGE AND RECOVERY DESIGN

Evaluate the current storage architecture.

Recommend:

- Application-managed temporary workspace.
- Download manifest.
- Chunk metadata.
- Atomic state updates.
- Crash recovery.
- Interrupted download recovery.
- Safe final-file assembly.
- Cleanup of stale temporary data.
- Disk space monitoring.
- Integrity verification.

Temporary download data must not be scattered across arbitrary directories.

Never delete user-selected destination files during cleanup.

Document what happens when:
- The application crashes.
- Windows restarts.
- An adapter disconnects.
- The target disk becomes unavailable.
- The server changes the file.
- The user deletes a temporary file manually.
- A download is cancelled.

# 9. BROWSER AND FUTURE MODULE ARCHITECTURE

Inspect existing browser integration.

Document its current state and recommend a modular extension system.

Future targets:

- Chrome.
- Microsoft Edge.
- Firefox.

Future capabilities:

- Download interception.
- Link capture.
- Browser context-menu actions.
- Download manager handoff.
- Authenticated downloads with explicit consent.

Also evaluate the feasibility of independent modules for:

- Video and media detection.
- HLS and DASH downloads where permitted.
- Torrent and magnet downloads.
- Bandwidth monitoring.
- Advanced scheduling.

Do not implement these modules in Phase 1.

# 10. PERFORMANCE AND RESOURCE MANAGEMENT

Evaluate:

- CPU utilization.
- RAM consumption.
- Disk I/O.
- Connection counts.
- Chunk sizes.
- Network utilization.
- Large file handling.
- UI responsiveness.
- Engine communication overhead.

The future application must not freeze when many downloads are active.

All heavy network and filesystem work must remain outside the Flutter UI thread.

Recommend realistic resource limits and configurable concurrency.

# 11. LICENSING AND COMMERCIAL READINESS

Design a future lifetime licensing system.

Requirements:

- One-time licence.
- No mandatory subscription.
- Secure licence verification.
- Device activation management.
- Offline operation after activation.
- Graceful behavior when the activation server is unavailable.
- Secure licence storage.
- Signed application updates.
- No hardcoded secrets.
- No exposed private signing keys.

Do not implement licensing in Phase 1.

# 12. PHASE 1 DELIVERABLES

Create a professional audit report in the existing documentation directory, or create a suitable documentation directory if one does not exist.

Produce the following documents:

1. PROJECT_STRUCTURE.md
   Complete existing repository structure.

2. EXISTING_FEATURE_AUDIT.md
   Every existing feature and its verified status.

3. ARCHITECTURE_AUDIT.md
   Existing architecture, dependencies and identified problems.

4. TARGET_ARCHITECTURE.md
   Recommended future Flutter/native engine architecture.

5. MULTI_NETWORK_FEASIBILITY.md
   Detailed Windows multi-network technical assessment.

6. DOWNLOAD_ENGINE_DESIGN.md
   Proposed HTTP, chunk, scheduling and recovery architecture.

7. SECURITY_AUDIT.md
   Threat model, vulnerabilities, risk severity and mitigations.

8. STORAGE_AND_RECOVERY.md
   Temporary storage, manifests, cleanup and crash recovery.

9. DEPENDENCY_AUDIT.md
   Existing dependencies, compatibility, licensing and risks.

10. DEVELOPMENT_ROADMAP.md
    Ordered development phases and dependencies.

11. TESTING_STRATEGY.md
    Unit, integration, security, network and regression testing.

12. PHASE_1_SUMMARY.md
    Executive summary of findings, blockers, recommendations and next steps.

# 13. PHASE 1 TESTING

Run the existing project's available checks without damaging the working environment.

For Flutter:

- flutter pub get, only if required.
- flutter analyze.
- Existing Flutter tests.
- Windows debug build if the environment supports it.

For Rust, if present:

- cargo check.
- cargo test.
- cargo clippy where applicable.

For C++ components, inspect and execute the available build and test procedures where possible.

Do not upgrade dependencies simply to make a check pass.

Do not delete build configuration or generated files unnecessarily.

If a build fails, record:
- Exact command.
- Error message.
- Relevant source file.
- Root cause if identifiable.
- Recommended fix.

Distinguish pre-existing failures from failures introduced by any Phase 1 change.

# 14. STRICT PHASE 1 CHANGE POLICY

This is primarily an inspection and architecture phase.

Do not begin the full Rust engine implementation.

Do not redesign all application screens.

Do not add new browser extensions.

Do not implement torrent functionality.

Do not implement media downloading.

Do not implement multi-network aggregation yet.

Do not rewrite working code.

Do not make destructive database or filesystem changes.

Documentation and non-invasive diagnostic tests are the primary deliverables.

If a small code change is absolutely necessary to enable a safe audit test, explain the change and its necessity first.

# 15. FINAL REPORT FORMAT

At the end, provide:

A. Existing project condition.
B. Features that already work.
C. Features that need repair.
D. Missing features.
E. Critical security risks.
F. Recommended architecture.
G. Whether the existing project should be upgraded or rebuilt.
H. Multi-network feasibility.
I. Dependency and compatibility issues.
J. Phase 2 implementation scope.
K. Exact files created or modified.
L. Tests executed and their results.
M. Remaining blockers.

For every finding, distinguish verified facts from assumptions and recommendations.

Never claim that something works without testing it.

Never invent files, classes, APIs or test results.

# FINAL INSTRUCTION

Inspect first. Understand second. Document third.

Preserve all working functionality.

Security takes priority over convenience.

Do not move to Phase 2 until the Phase 1 audit documents are complete and the architectural decisions are justified.

The result must be a professional, technically grounded foundation for a commercial Windows download manager, not a superficial code review.

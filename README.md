# CYperX Optimizer — Commercial-Grade License Activation System & Admin Panel

CYperX Optimizer is a commercial-grade, secure Windows desktop application engineered for FiveM / Grand Theft Auto V and universal PC gaming titles. The ecosystem includes an authoritative License Server, an isolated Vendor Admin Panel (`CYperX-Admin.exe`), and the customer desktop suite locked behind Ed25519 digital activation.

---

## 1. System Components & Architecture

The ecosystem consists of three strictly isolated, independently distributed components:

```text
+-----------------------------------------------------------------------------------------+
|                                CYperX License Server                                    |
|         The authoritative authority on license validity, entitlements, device           |
|                limits, hardware fingerprints, and version policies.                     |
|                               (cyperx-license-server)                                   |
+-----------------------------------------------------------------------------------------+
                    ▲                                                 ▲
                    │ REST API (Customer Token)                       │ REST API (Admin Token + TOTP)
                    ▼                                                 ▼
+---------------------------------------+         +---------------------------------------+
|            CYperX Client              |         |          CYperX Admin Panel           |
|  Locked customer desktop application  |         | Dedicated vendor operations tool      |
|  Verifies signed Ed25519 tokens & DPAPI|        | Never included in customer packages   |
|         (CYperX-Optimizer.exe)        |         |          (CYperX-Admin.exe)           |
+---------------------------------------+         +---------------------------------------+
```

### Maven Module Structure

* [cyperx-license-common](file:///c:/xampp/htdocs/cyperX_Optimizer/cyperx-license-common): Shared DTOs (including Mods Catalog & Installation metadata), Crockford Base32 key utilities, Ed25519 signatures, hardware signal collector, and release tooling.
* [cyperx-license-server](file:///c:/xampp/htdocs/cyperX_Optimizer/cyperx-license-server): Spring Boot 3 + PostgreSQL/Flyway server holding private signing keys, HMAC secrets, Argon2id auth, rate limiting, and centralized Mods Catalog API.
* [cyperx-admin](file:///c:/xampp/htdocs/cyperX_Optimizer/cyperx-admin): Standalone JavaFX vendor console (`CYperX-Admin.exe`) with DPAPI session protection, mandatory TOTP, license management, and full Mods Catalog lifecycle management.
* [cyperx](file:///c:/xampp/htdocs/cyperX_Optimizer/cyperx): Commercial customer desktop application with licensing shell, feature entitlements gating, FiveM / GTA V Mods Hub with 1-click safe installation, guarded clock, and Ed25519 release integrity checks.

---

## 2. Core Security & Anti-Bypass Architecture

1. **Server Authority**: The desktop client never decides license validity from a locally editable value. All activations and revalidations are verified server-side.
2. **Ed25519 Cryptographic Signatures**: The server issues tokens in `cyx1.<payloadJsonB64>.<signatureB64>` format. The private signing key exists exclusively on the server. The client holds only public verification keys.
3. **Multi-Signal Device Binding**: Five stable Windows hardware signals (MachineGuid, SMBIOS UUID, CPU identifier, Volume Serial, Network MAC) hashed with product salt. The server derives an HMAC fingerprint and allows activation if $\ge 3$ of 5 signals match, tolerating hardware upgrades.
4. **Local Windows DPAPI Storage**: Activation tokens and device bindings are stored encrypted in `activation.dat` and `device.dat` with SHA-256 integrity tags.
5. **Guarded Monotonic Clock**: Protects against system clock manipulation and rollback during offline grace periods.
6. **Feature Entitlements Engine**: Protected optimization views and operations query `FeatureEntitlementService.hasFeature("...")` which resolves only against valid, signature-verified, device-bound tokens.
7. **Strict Build Separation**: Maven antrun verification and `ReleaseVerifier` ensure zero admin or server classes, and zero bypass tokens exist in the customer package.

---

## 3. Quick Start & Building

### Prerequisites
* Windows 10 (Build 19041+) or Windows 11 (64-bit)
* Java Development Kit (JDK) 21 or 22 configured in `JAVA_HOME`
* Apache Maven 3.9+

### Build the Full Ecosystem
```powershell
# Compile and run test suite across all 4 modules (95+ tests)
mvn clean test

# Build production distribution packages (Native Windows executables + Fat JARs)
.\build-distribution.ps1
```

### Launching Components

#### 1. Customer Application
```powershell
# Native Windows executable (Recommended)
.\dist\customer\CYperX-Optimizer\CYperX-Optimizer.exe

# Quick launcher batch file
.\Launch-CYperX.bat
```

#### 2. Vendor Admin Panel
```powershell
# Native Windows executable (Vendor only)
.\dist\admin\CYperX-Admin\CYperX-Admin.exe

# Quick launcher batch file
.\Launch-Admin.bat
```

#### 3. License Server
```powershell
# Run license server locally
.\Launch-LicenseServer.bat

# Or run with custom environment configuration
java -jar .\dist\server\cyperx-license-server.jar
```

---

## 4. Gaming Performance Overlay (OSD)

CYperX Optimizer includes a production-grade, observation-only gaming performance overlay (HUD) designed for borderless and windowed games (FiveM / GTA V first).

### Key Technical Capabilities
* **Zero Injection / Anti-Cheat Safe**: No DLL injection, no graphics hooks, no game memory reads/writes, no handles beyond `PROCESS_QUERY_LIMITED_INFORMATION`.
* **Microsecond Frame Pacing**: Real-time ETW capture via Intel PresentMon with zero per-frame hot-path allocations.
* **Rigorous Statistics**: Mathematically accurate Average FPS, 1% Low, and 0.1% Low with minimum 100-frame statistical gating (otherwise reported as `N/A`).
* **Hardware Sensors**: Isolated Windows PDH and NVIDIA NVML providers with automatic timeouts, circuit breaker backoff, and DXGI VRAM fallback.
* **Canvas-Rendered HUD**: Dark plate with cyan accents (`#00D4FF`), real-time waveform sparkline, and circular ring gauges with user-set threshold coloring.
* **Interactive Edit Mode**: Move and resize the HUD directly on screen (`Ctrl+Shift+E`), then lock back to click-through mode.
* **SQLite Benchmark Recorder**: Record sessions with per-frame data points, stutter counters, and export to PresentMon-compatible CSV and JSON.

---

## 5. Documentation

* [Exclusive Fullscreen & Presentation Models Guide](file:///c:/xampp/htdocs/cyperX_Optimizer/docs/EXCLUSIVE_FULLSCREEN.md): DWM composition, Flip Model, Fullscreen Optimizations (FSO), and compatibility breakdown.
* [Overlay QA & Testing Guide](file:///c:/xampp/htdocs/cyperX_Optimizer/docs/OVERLAY_TESTING.md): Hardware matrix, error resilience tests, and performance measurement protocol.
* [Deployment Guide](file:///c:/xampp/htdocs/cyperX_Optimizer/docs/DEPLOYMENT_GUIDE.md): PostgreSQL setup, environment variables, systemd service, and TLS reverse proxy configuration.
* [Key Management and Rotation Guide](file:///c:/xampp/htdocs/cyperX_Optimizer/docs/KEY_MANAGEMENT_AND_ROTATION.md): Ed25519 key lifecycle, rotation procedures, and offline manifest signing.
* [Administrator Guide](file:///c:/xampp/htdocs/cyperX_Optimizer/docs/ADMIN_GUIDE.md): Admin Panel operations, TOTP 2FA, 6-step license creation wizard, and hash-chain audit log verification.
* [Production Release Checklist](file:///c:/xampp/htdocs/cyperX_Optimizer/docs/RELEASE_CHECKLIST.md): Step-by-step verification protocol before customer release.

---

## 6. Community Feedback & Official Discord

CYperX Optimizer improves with real feedback from real users. If you notice a bug, have an idea for new gaming optimizations, or want to discuss performance tuning with fellow gamers and engineers, join our Discord:

* **Application Version**: `1.0.0-RELEASE`
* **Official Community Discord**: [https://discord.gg/pmrU2EFkgA](https://discord.gg/pmrU2EFkgA)
* **Bug Reports & Diagnostics**: Generate privacy-redacted diagnostic reports directly within the application (via Settings or `Ctrl+K` -> *Copy Diagnostic Report*) to easily attach when opening a ticket in `#bug-reports`.


# vntapp-macos-build

Build harness only. It builds **VntApp 1.2.13** (Flutter) for **macOS Apple Silicon (arm64)** via GitHub Actions and publishes an unsigned `.dmg`.

- Source: `vnt-dev/VntApp` @ tag `1.2.13` (checked out by the workflow).
- Runner: `macos-15` (Apple Silicon), Flutter 3.24.5, Rust stable.
- Output: `VntApp-1.2.13-macos-arm64.dmg` (see Releases).

The app is **unsigned** (no Apple Developer certificate), so on first launch right-click the app and choose Open to bypass Gatekeeper.

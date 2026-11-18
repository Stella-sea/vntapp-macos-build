# vntapp-macos-build

Build harness for **VntApp 1.2.13** (Flutter) on **macOS Apple Silicon (arm64)** via GitHub Actions. Produces an unsigned `.dmg`.

- Source: `vnt-dev/VntApp` @ tag `1.2.13` (checked out by the workflow).
- Patches in `patches/` are applied after checkout (see `patches/r4-dart.patch`).
- Runner: `macos-15` (Apple Silicon), Flutter 3.24.5, Rust stable.
- Output: `VntApp-1.2.13-macos-arm64-r4.dmg` (see Releases).

## r4 fixes (vs upstream 1.2.13)
- In-app admin request: on macOS, if not running as root, the app offers to relaunch as admin via the system password dialog (osascript). The dialog offers Touch ID on Macs that have it enrolled. Relaunch uses `setsid` so the new root instance fully detaches and survives (r3's `nohup &` was killed when the privilege helper exited, so the app just quit).
- Tray hide now truly hides the window via `windowManager.hide()` (bitsdojo `appWindow.hide()` was minimizing to Dock on macOS).
- Log page no longer spins forever when the log file is missing (`LogReader` handles a missing file; `_isLoading` always resets). Log/work dir is pinned to the app support directory so relative `logs/` paths resolve consistently.
- Permission errors (e.g. `create device ... PermissionDenied`) show a friendly message with a one-tap "管理员重启" action.
- App sandbox entitlement removed so the TUN/utun interface can be created.

Note: creating the virtual network card requires the app process itself to run as root, so a one-time restart as admin is unavoidable (a no-restart design would need a separate privileged helper plus Rust-core changes to accept an externally created TUN fd). After the first successful admin relaunch the app runs as root and stays working until it is quit.

The app is **unsigned** (no Apple Developer certificate), so on first launch right-click the app and choose Open to bypass Gatekeeper.

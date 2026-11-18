# vntapp-macos-build

Build harness for **VntApp 1.2.13** (Flutter) on **macOS Apple Silicon (arm64)** via GitHub Actions. Produces an unsigned `.dmg`.

- Source: `vnt-dev/VntApp` @ tag `1.2.13` (checked out by the workflow).
- Patches in `patches/` are applied after checkout (see `patches/r3-dart.patch`).
- Runner: `macos-15` (Apple Silicon), Flutter 3.24.5, Rust stable.
- Output: `VntApp-1.2.13-macos-arm64-r3.dmg` (see Releases).

## r3 fixes (vs upstream 1.2.13)
- In-app admin request: on macOS, if not running as root, the app offers to relaunch as admin via the system password dialog (osascript), so no Terminal/sudo is needed to create the TUN interface.
- Tray hide now truly hides the window via `windowManager.hide()` (bitsdojo `appWindow.hide()` was minimizing to Dock on macOS).
- Log page no longer spins forever when the log file is missing (`LogReader` handles a missing file; `_isLoading` always resets). Log/work dir is pinned to the app support directory so relative `logs/` paths resolve consistently.
- Permission errors (e.g. `create device ... PermissionDenied`) show a friendly message with a one-tap "管理员重启" action.
- App sandbox entitlement removed so the TUN/utun interface can be created.

The app is **unsigned** (no Apple Developer certificate), so on first launch right-click the app and choose Open to bypass Gatekeeper.

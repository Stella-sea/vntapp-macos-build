# vntapp-macos-build

Build harness for **VntApp 1.2.13** (Flutter) on **macOS Apple Silicon (arm64)** via GitHub Actions. Produces an unsigned `.dmg`.

- Source: `vnt-dev/VntApp` @ tag `1.2.13` (checked out by the workflow).
- Patches in `patches/` are applied after checkout (see `patches/r6-dart.patch`).
- Runner: `macos-15` (Apple Silicon), Flutter 3.24.5, Rust stable.
- Output: `VntApp-1.2.13-macos-arm64-r6.dmg` (see Releases).

## r6 fixes (vs r5)
- Tray: all window hide/show/close now go through `window_manager` only (`system_tray`'s `AppWindow` is gone); `AppDelegate` no longer quits the app when the last window closes, so hiding to tray keeps the process and VPN connection alive. Tray Quit and the close dialog's "close app" path disconnect all connections first, then quit explicitly.
- Admin relaunch: the 800ms blind wait is replaced by a ready-file handshake — the new admin instance writes a ready file with a random token once its window is shown; the old instance only exits after seeing it (30s timeout, otherwise it stays and shows an error).
- Config handover: before relaunch, the root shell copies the console user's SharedPreferences plist (`top.wherewego.vntApp.plist`) to root's plist when the user copy exists and is newer, so configs, unique ID and preferences survive the user -> root switch.
- Still requires the system password on every cold start (no Touch ID / password-free launch yet — that needs a privileged helper, future work).

## r5 fixes (vs upstream 1.2.13)
- In-app admin request: on macOS, if not running as root, the app offers to relaunch as admin via the system password dialog (osascript). The dialog offers Touch ID on Macs that have it enrolled. The relaunch now uses `launchctl asuser` to put the new root process back into the logged-in user's Aqua session (r3's `nohup &` and r4's `setsid` both launched the GUI outside Aqua, so it could not connect to the WindowServer and flashed/quitted). Launch output is logged to `/tmp/vnt_app-root.log`.
- Tray hide now truly hides the window via `windowManager.hide()` (system_tray's `AppWindow.hide()` was minimizing to Dock on macOS).
- Log page no longer spins forever when the log file is missing (`LogReader` handles a missing file; `_isLoading` always resets). Log/work dir is pinned to the app support directory so relative `logs/` paths resolve consistently.
- Permission errors (e.g. `create device ... PermissionDenied`) show a friendly message with a one-tap "管理员重启" action.
- App sandbox entitlement removed so the TUN/utun interface can be created.

Note: creating the virtual network card requires the app process itself to run as root, so a one-time restart as admin is unavoidable (a no-restart design would need a separate privileged helper plus Rust-core changes to accept an externally created TUN fd). After the first successful admin relaunch the app runs as root and stays working until it is quit.

The app is **unsigned** (no Apple Developer certificate), so on first launch right-click the app and choose Open to bypass Gatekeeper.

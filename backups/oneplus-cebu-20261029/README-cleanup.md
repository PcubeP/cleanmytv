# CleanYourTV — handover for OnePlus TV C00 (Cebu)

Done on 2026-10-29. TV: **OnePlus C00 (codename Cebu), Android 11 / SDK 30, firmware RTM8.240401.135**, IP `192.168.0.105`.
Everything here is reversible. Nothing was uninstalled except the Old Movies app you asked to remove.

## What was done

1. **Debloat** — 19 packages disabled with `pm disable-user` (not uninstalled). Telemetry/diagnostics,
   OnePlus content + recommendation rows, OxygenPlay, user guide, TV Manager, screensaver, weather,
   smart-home, camera, and unused streaming preloads (JioCinema, JioPages, Play Movies, Play Games,
   YouTube Music). **Kept by choice:** all casting/mirroring, USB media apps, Bluetooth speaker, Spotify.
   The list lives in `backups/oneplus-cebu-20261029/debloat-mytv.txt`.
2. **Uninstalled** the Old Movies app (`com.acowboys.oldmovies`) — fully removed; reinstall from Play Store if ever wanted.
3. **Launcher** — a clean Apple-TV-style home (`com.gathin.tvhome`) installed and set as default Home;
   stock Google TV launcher (`com.google.android.tvlauncher`) disabled. Survives reboot.
4. **Ad-blocking Private DNS** — set to AdGuard (`dns.adguard-dns.com`, DNS-over-TLS). Blocks ad/tracker
   domains TV-wide (verified: `doubleclick.net` → 127.0.0.1; normal domains resolve). Does not block
   YouTube ads — use SmartTube for those. **Undo:** `$TV "settings put global private_dns_mode opportunistic"`.

## Your dock (bottom row, 6 slots)

| Slot | Tile | Notes |
|---|---|---|
| 1 | **HDMI 1 (PC)** | One-press switch straight to HDMI 1 (your PC). Custom label + banner. |
| 2 | Spotify | |
| 3 | Netflix | |
| 4 | Prime Video | |
| 5 | JioHotstar | |
| 6 | *(empty)* | reserved; add an app later (see below) |

Everything else is in the grid — press **Down** from the dock. Top-right pills: **Inputs** (all HDMI/AV/Live TV),
**Wi-Fi**, **Settings**.

## Undo / restore

All commands run from the repo root `C:\dev\workspace\Personal\cleanyourtv` with the venv Python.
`TV` below = `TV_HOST=192.168.0.105 tools/venv/Scripts/python.exe -I tools/tv.py`

- **Re-enable everything that was disabled (debloat + reverse the list):**
  `TV_HOST=192.168.0.105 tools/restore.sh backups/oneplus-cebu-20261029/debloat-mytv.txt`
- **Re-enable one app:** `$TV "pm enable --user 0 <package>"`
  e.g. weather: `$TV "pm enable --user 0 net.oneplus.weather"`
- **Bring back the stock Google TV launcher as Home:**
  1. `$TV "pm enable --user 0 com.google.android.tvlauncher"`
  2. `$TV "cmd package set-home-activity --user 0 com.google.android.tvlauncher/.MainActivity"`
- **Reinstall Old Movies:** from the TV's Play Store (it was uninstalled, not disabled).

## Change the dock, columns, or wallpaper

Edit `launcher/app/src/main/res/values/config.xml`, then rebuild + reinstall (see below):
- `dock_apps` — the ordered list of dock packages. `fill_dock=false` leaves slot 6 empty.
- `columns` — grid width and dock slot count (currently 6).
- HDMI-1 tile is wired via three keys for package `com.heytap.tv.livetv`:
  `launch_actions` (`=input:com.mediatek.tvinput/.hdmi.HDMIInputService/HW5`), `app_labels` (`=HDMI 1 (PC)`),
  `app_banners` (`=banner_hdmi`, drawable at `launcher/app/src/main/res/drawable/banner_hdmi.xml`).
  HDMI 2 = `HW6`, HDMI 3 = `HW7`.
- Wallpaper/colours without a rebuild: push a theme — `tools/theme.sh themes/<name>` (see `themes/README.md`).

## Rebuild + reinstall the launcher

Portable toolchain is at `C:\tmp\cyt-toolchain` (JDK 17 + Android SDK 34). From `launcher/`:
```
export JAVA_HOME="C:\\tmp\\cyt-toolchain\\jdk\\jdk-17.0.20.1+1"
export ANDROID_HOME="C:\\tmp\\cyt-toolchain\\android-sdk"
TV_HOST=192.168.0.105 ./build-and-install.sh
```
(To reinstall an already-built APK without rebuilding: prefix `SKIP_BUILD=1`.)

## ADB / security (now CLOSED)

Network ADB was turned off (`settings put global adb_enabled 0`) and verified closed — port 5555 refuses
connections (`tools/adb-closed.py 192.168.0.105` → "the TV refused the connection").

- **To tweak the TV again later**, re-enable on the TV: Settings → Device Preferences → Developer options →
  **USB debugging** ON. Then reconnect from the repo root with `TV_HOST=192.168.0.105 tools/venv/Scripts/python.exe tools/tv.py --probe`
  (accept the "Allow USB debugging?" prompt on the TV if it reappears).
- **For maximum security** you can also, on the TV, use Developer options → "Revoke USB debugging
  authorisations" — this forces any computer (including this one) to be re-approved next time.
- Re-verify closed anytime: `TV_HOST=192.168.0.105 tools/venv/Scripts/python.exe tools/adb-closed.py 192.168.0.105`

## Backups (pre-change state)

In `backups/oneplus-cebu-20261029/`: full package list, enabled/disabled lists, `getprop`, all `settings`,
and before/after screenshots (`00-before-stock-home.png` … `05-slot1-focus.png`).
Debloat recovery record (written by disable.sh): `C:\Users\I753694\.cleanyourtv\recovery-192.168.0.105-48b1b224.json`.

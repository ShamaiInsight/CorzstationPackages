# Dashboard connectivity and icons

Release: **2026.10.31-dashboard.1**

This describes the first dashboard release. The newer **2026.10.31-dashboard.2** adds the missing Workout TimeZ app and its artwork; see [WORKOUT-MEDIA-UPDATE.md](WORKOUT-MEDIA-UPDATE.md).

## Changes

- A dedicated strip at the top of the dashboard shows Wi-Fi radio/link status separately from internet availability.
- **Green:** internet online. **Red:** no internet, including a network sign-in warning when NetworkManager identifies a captive portal. **Amber:** checking or local network service unavailable; it does not falsely claim the station is connected.
- Automatic refresh every 10 seconds, plus browser online/offline and return-to-screen events. Slow checks can take longer. Status recovers automatically without reloading the dashboard.
- A Wi-Fi connection or the station's own hotspot is not by itself proof of internet access. Ethernet internet can show online even with Wi-Fi off.
- NetworkManager performs the internet check. If its connectivity checking is unavailable/disabled, the service falls back to validating an HTTPS 204 response from Google or Cloudflare. A firewall blocking both fallback hosts can produce a no-internet warning despite other sites working.
- Supplied artwork is used in both carousel tiles and the selected game's large illustration:
  - TENZ: `10in1_icon_selecable.png`
  - BEEPZ: `bleep_icon_selected.png`
  - PLANKZ Balance: `plank_icon.png`
  - Calibration: `caliionnicon_bluering.png`
- Smaller-screen spacing and carousel vertical alignment prevent the added status strip from crowding the cards.
- Corrected two existing Wi-Fi connection issues: the frontend now accepts the service's `ok` response, and hotspot teardown calls the existing `set_client()` function.

Calibration remains “Coming soon”; this update changes its artwork, not its availability. Game rules, controls and profile requirements are unchanged. Existing missing TimeZ/PongZ artwork and the missing workout-timer page in the supplied archive are outside this update.

## Install on an existing station

The `github-upload` folder contains a freshly rebuilt, matching **package.zip** and **update.json**. Use both files together, upload the ZIP first and JSON last (or commit both together) through your existing updater workflow. No files have been uploaded to GitHub for you.

Alternatively, extract this kit on the Pi, open a terminal in its `CORZ-Pi-Startup` folder, and apply the included package as the normal desktop/station user:

```bash
python3 "$HOME/corz-station/runtime/updater.py" --apply "$PWD/github-upload/package.zip"
```

This assumes the installation is at `$HOME/corz-station`; adjust the path if needed. The existing updater and its service-restart permissions must already be configured. The package includes both the dashboard and **runtime/wifi-api.py**, so do not copy only index.html. The updater restarts changed Python services; if manually copying files instead, restart `corz-wifi` and reload the dashboard. Do not rerun the fresh installer over an existing station.

Unlisted files and local `.env` settings are preserved by the existing updater. Keep a backup before updating.

## Verification

Automated checks: 63 Python tests passed (51 startup/updater/network tests and 12 bridge tests), plus the new Node dashboard/connectivity checks. The release is also applied to an isolated temporary station and its local servers tested. These checks do not change real Wi-Fi settings or contact public connectivity hosts.

Browser checks confirm all four supplied icons load. Real Pi Wi-Fi drops, hotspot/radio transitions, live connectivity probes and systemd restarts still require on-device testing.

The supplied BEEPZ and PLANKZ Node regression scripts have an existing default-host assertion mismatch: they expect `raspberrypi.local`, while the unchanged game files use `localhost` when served locally. Their full suites therefore do not pass as supplied. The game files and these tests have not been changed.

On the Pi, check: Wi-Fi connected with internet; router internet disconnected while Wi-Fi remains connected; Wi-Fi disconnected/off; hotspot mode; and reconnection. Verify both text and colour change, and that a stopped `corz-wifi` service shows “unable to check” rather than a false online state.

# CORZ Station design update — 2026.11.01-dashboard.9

## Install through your existing GitHub updater

1. Extract the GitHub update download.
2. Publish its matching `package.zip` and `update.json` to the same GitHub folder used for your previous updates. Commit both together, or publish JSON last.
3. Allow the station updater to complete installation, then reboot normally.

No Terminal command, installer, desktop setting, new dependency or one-time configuration is required. Do not rerun the power-menu setup. `manifest-readable.json` remains optional and is not needed in GitHub.

## New layout

- Landing: **Configuration** or **Select program**.
- Programs: PLANKZ, BEEPZ, TENZ, LIFTZ and Workout TIMEZ, with your existing supplied game icons. Hall of Fame and player profile remain accessible beneath the cards.
- Configuration: readiness checks, existing Wi-Fi controls, Controller, Program admin, API status, software-update status, player profile, diagnostics and Style.
- Remote: arrows move selection; OK activates; HOME/Back returns one menu level. Games retain their existing exit handling, including PLANKZ confirmation.
- Appearance: Performance (default), Console, Tron and Rebel. Themes and starting game values are saved automatically on this Pi.

The Display/overscan/resolution settings, prototype gym-registration controls, PIN administration, fake firmware version and nonexistent TESTZ program are not included. Wi-Fi reuses the existing working service; no network-service setup is changed.

## Assign the correct numbered Cradle

1. Finish any running game.
2. Open **Configuration → Controller**.
3. Wake your Cradle and choose **Find CORZ devices**. A scan takes approximately five seconds; another ongoing adapter operation may take longer.
4. Find the exact `CORZ_` number belonging to this station, then press OK on it.
5. Wait for the connection status to show that device connected. “Assignment saved” means the selection was stored, not that the radio connection has completed.

The station saves the chosen Bluetooth address in `~/corz-station/runtime/ble-selection.json` and reconnects only to that device. It does not silently switch to a different nearby Cradle when the chosen one is absent. To replace it, scan and select another device. Available scan results expire after 60 seconds; rescan if asked.

Before any explicit assignment, the original automatic connection behavior is preserved for compatibility. Devices already connected to another station may not advertise and may not appear in the scan. Calibration remains in the games, preserving their existing measurement routines.

BLE packet format, sensor UUIDs, sampling, power-off command and game connections stay compatible. Added commands implement discovery and device selection in the existing BLE bridge; no separate service is installed. Device selection must be saved successfully before the current connection is changed. Failure to save is reported on screen.

## Program admin

Defaults apply the next time a game opens; they do not rewrite an in-progress session.

| Program | Configurable defaults |
|---|---|
| PLANKZ | Easy/Medium/Hard difficulty; interface theme |
| BEEPZ | Cycles, repetitions, rest seconds, target-hold seconds; theme |
| TENZ | Sensor range in degrees; theme |
| LIFTZ | Roll/pitch axis, recording seconds, balanced limit; theme |
| Workout TIMEZ | Work seconds, rest seconds, rounds; theme |

Global style is the fallback for any game without an explicit theme. A per-game theme takes priority. Local browser storage retains these settings across package updates at the same dashboard address; clearing Chromium data resets them. Do not switch between localhost/127.0.0.1 addresses if you expect the same browser settings.

API credentials, endpoints, NFC player lookup, PR_1/PR_2 music, Hall of Fame, existing GitHub updater and previously configured power popup are preserved. API and updater configuration screens report actual local service state; they do not submit dummy results, alter credentials or claim to install updates themselves. Score uploads remain implemented for PLANKZ and BEEPZ, not the other three games.

## Validation and Pi checks

Automated tests cover real Chromium navigation, saved defaults, reload persistence, all game themes, iframe focus return, power-menu compatibility, BLE discovery/selection with simulated transports, and isolated package installation. Tests do not transmit Bluetooth commands or submit scores to the live API.

After installing on the Pi, verify:

1. Remote navigation from both main entries and back from a game.
2. Scan two awake numbered Cradles; select the intended one and verify its sensor data in PLANKZ.
3. Reboot, then verify the same Cradle is assigned. Turn it off and confirm another room's Cradle is not automatically substituted.
4. Change PLANKZ default difficulty/theme, reopen PLANKZ, and confirm the new selection. Check your other game defaults too.
5. Check real Wi-Fi, NFC profile lookup, music, power popup, and an actual completed PLANKZ/BEEPZ upload.

Actual Pi remote behavior, adapter concurrency, BLE radio connections and live-server score acceptance cannot be certified by the sandbox tests.

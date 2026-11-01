# Workout TimeZ, Hall of Fame and music update

Release: **2026.10.31-dashboard.2**

- Created matching blue-ring icons: a stopwatch/lightning design for Workout TimeZ and a gold trophy/star for Hall of Fame.
- Connected both icons to their dashboard tiles and large selected-game illustrations.
- Added your supplied `workout-timer.html` unchanged at `app/www/workout-timer.html`, matching the existing Workout TimeZ launch address.
- Added `PR_1.mp3` and `PR_2.mp3` unchanged in `app/www/Music/`. The dashboard's existing shuffled music player discovers these filenames automatically. No new audio autoplay behaviour was added.
- Enabled the timer's existing Exit action to close its dashboard overlay. Existing navigation, game rules and connectivity indicator are retained.

## GitHub update files

Use the **new matching pair** inside `github-upload`:

1. Upload `package.zip` to your existing update repository, replacing the previous ZIP.
2. Upload `update.json` last, replacing the previous JSON (or commit both together).

Do not rename the full startup-kit archive to `package.zip`. Nothing has been pushed to GitHub by the agent.

The rebuilt package contains the workout app, both icons, both music tracks, and the previous connectivity changes. Its manifest version, file size and checksum match the new `update.json`. The existing updater preserves unlisted files and local settings.

## Checks

Dashboard JavaScript compilation and connectivity regression checks are included. The release-install smoke test verifies that the installed local web server serves the workout app, both icons and both MP3 files byte-for-byte.

On the Pi, verify the Workout TimeZ setup/start/pause/exit controls, Hall of Fame icon, and music playback after a key press. Physical remote control, Bluetooth and audio output still require on-device testing. Calibration and PongZ remain unavailable as before.

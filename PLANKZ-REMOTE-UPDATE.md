# PLANKZ remote-only update

Release: **2026.10.31-dashboard.7**

## Play with the remote and Cradle

1. Open PLANKZ from the dashboard.
2. Use Left/Right to choose Easy, Medium or Hard. Up/Down moves between difficulty, Start and Back to Dashboard. OK on a difficulty moves focus to Start.
3. Select **Start · Calibrate Cradle**. Stay away from the Cradle: leave it empty, untouched and still throughout the five-second calibration.
4. When calibration completes, a five-second countdown starts automatically. Now get into position on the Cradle.
5. Play with the accelerometer. The balance circle is centred on the full screen; score, accuracy and active time remain visible. There are no clickable gameplay controls.

The existing neutral-angle/noise calibration, difficulty boundaries, presence checks, score calculation and result API are retained. There is no mouse simulation, connection button, server address field or technical settings form. The game uses the existing station configuration and connects automatically.

## Finish, restart or return home

- The existing two-second hold at either the top or bottom screen edge finishes the session and shows the result. Move away early to cancel the gesture.
- Select **Play again** on the result screen to choose difficulty and recalibrate for a new session.
- HOME, Back or Escape opens a remote-friendly exit confirmation. Keep Playing is selected by default. Choose Leave to Dashboard and confirm with OK to leave.
- Back or HOME again cancels the confirmation. Scoring pauses while it is shown. If opened during calibration, cancelling restarts a complete calibration window so interrupted samples are not mixed.
- Leaving an active session waits for the existing result-save attempt before closing the game. Guest and read-only sessions stay local, as before. Results are not automatically retried to avoid duplicates.
- Lost Cradle data pauses the score and presents remote-operable Continue / Finish options. The connection retries automatically.

## Install

Upload the matching **package.zip** and **update.json** from github-upload. Commit together, or upload the ZIP first and JSON last. The readable manifest is optional.

Let the Pi install dashboard.7, then reboot after the update has finished. The power-button setup already completed for dashboard.6 remains in place: **do not run that setup again**.

## Validation

Real Chromium tests cover remote difficulty changes, automatic calibration and countdown, centred gameplay, simulated BLE scoring, both edge gestures, results, replay, HOME/Back aliases from parent/game focus, confirmation cancellation, result saving before exit, disconnect/reconnect and dashboard arrow input after exit. Layout checked at 1920×1080 and 1280×720.

The Cradle packets and profile API were simulated in the PLANKZ browser tests. Actual accelerometer behaviour, device timing, the real profile API and the physical remote must still be checked on your Pi. Previous Workout TimeZ navigation and the power popup have separate regression checks. Power actions in tests are logged, never executed.

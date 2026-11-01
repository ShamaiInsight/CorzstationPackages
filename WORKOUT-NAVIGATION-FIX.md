# Workout TimeZ exit/navigation fix

Release: **2026.10.31-dashboard.5**

## Reproduced issue

Launching Workout TimeZ, starting a session and leaving using HOME or Back closed the app visually, but Chromium retained keyboard focus inside the hidden iframe. Left/Right then failed to move the dashboard selection.

## Fixes

- Every app close now explicitly releases iframe focus and focuses the dashboard body.
- Fixed a separate Workout TimeZ control issue: its results overlay was initially hidden in CSS, but the input router checked only inline styles. That incorrectly routed game controls into the hidden results screen. Visibility now uses the computed style.
- Existing kiosk flags, NFC/connectivity feedback, icons and both PR music tracks are retained.

## Validation

Real headless Chromium against the local HTTP bridge: start a workout, exit with keyboard Home, Backspace, Escape, the dashboard HOME button, or the timer's own Exit control; then check both Left and Right. Also verify Enter opens LIFTZ after moving Left. No click is used to repair keyboard focus after exiting.

Workout PongZ remains an existing unavailable/coming-soon card to the right of Workout TimeZ; selecting it does not launch an app. Its availability has not been changed.

The browser test and results are in `tools/test_workout_navigation.ts` and `validation/workout-navigation.json`. Physical Pi/remote testing is still required.

## Publish

Upload the matching `github-upload/package.zip` and `github-upload/update.json` to your existing GitHub update location. Commit together or upload JSON last. Reboot after installation to reload the dashboard. No GitHub changes were made on your behalf.

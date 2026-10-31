# Real game-result uploads — dashboard.8

## What changed

The newly supplied write endpoint is now connected:

`POST https://corzconvergence.com/api/public/v1/result`

The Pi exchanges its configured API key for a bearer token using `/session`, reuses it until renewal, and submits:

```json
{
  "tag_id": "2B94E9B3",
  "game": "PLANKZ · EASY",
  "score": 40.68,
  "accuracy": 82.8,
  "session_length_seconds": 49
}
```

This is a format illustration, **not a score to submit**. The real values come from the completed game. Accuracy is a percentage (0–100); duration is rounded to whole seconds, as before. The local bridge receives `game_title` / `duration_seconds` and maps them to `game` / `session_length_seconds`. The captured NFC tag identifies the player, not an invented member UUID.

**PLANKZ and BEEPZ** now upload for selected profiles. Guest results remain local. TENZ, LIFTZ and Workout TimeZ are not changed to upload in this release. Existing simulation titles in BEEPZ remain labelled SIMULATION and are separate from real-device titles.

Exact game names are retained, including difficulty: `PLANKZ · EASY`, `PLANKZ · MEDIUM`, `PLANKZ · HARD`, and the corresponding BEEPZ titles. All-games Hall of Fame includes them; a game filter must match the full name. They are not silently renamed to the historical `Planz` or `Light Cycle` entries.

## Install

Upload the matching `package.zip` and `update.json` from dashboard.8. Commit together, or upload ZIP first and JSON last. Wait for installation to finish, then reboot the Pi. Your existing `.env`, API key, upstream setting, and power-button configuration are preserved. No additional setup script or API key entry is needed.

Tap your tag again after reboot. Profiles selected before the update are not kept across bridge restart.

## Read-only diagnostics — no fake scores

On the Pi:

```bash
python3 ~/corz-station/runtime/check_result_upload.py
```

Expected:

- Bridge: `2026.10.31-dashboard.8`
- Upstream: `https://corzconvergence.com/api/public/v1`
- API key configured: `True`
- Score upload supported: `True`
- Selected profile writable: `True`

This command only reads local status and fresh server statistics. It never submits a result, opens an API-key prompt, or prints credentials. If no profile is selected, tap the tag and run it again. Last local upload is initially null and is retained only until bridge restart.

## Test one real PLANKZ session end to end

1. Tap your registered tag.
2. Before playing, save the current **Easy** statistics:

   ```bash
   python3 ~/corz-station/runtime/check_result_upload.py --baseline ~/plankz-before.json
   ```

3. Launch PLANKZ **Easy**, calibrate, and play a short real session. Finish with the top/bottom hold gesture. Keep the result screen open until it says **Uploaded to OurCorz for [your name]**.
4. Without rebooting, compare fresh server statistics:

   ```bash
   python3 ~/corz-station/runtime/check_result_upload.py --compare ~/plankz-before.json
   ```

The server's session count should increase. The comparison also prints before/after score, accuracy and play time. Best score/accuracy need not increase if this was not a personal best. Other devices can also affect aggregate statistics; use this together with the game's upload confirmation and Hall of Fame. If the server updates statistics asynchronously, wait briefly and compare again—do not resubmit the same result.

For another difficulty/game, use the same exact `--game` on both commands, for example `--game 'PLANKZ · MEDIUM'`. If the API returns 404 for a game with no previous results, record all-games profile statistics using the dashboard instead, then run the check again after the first real session.

The baseline file contains the tag ID and statistics, with owner-only permissions. It contains no key or token; do not share it unnecessarily.

## Failures and duplicate protection

Player identity is checked immediately before submission. A changed/expired selection cannot silently credit another player. The token is refreshed once after an explicit authentication rejection. Network timeouts, server errors and uncertain writes are not automatically retried, to avoid duplicate sessions. Repeated identical local result IDs are suppressed during the running bridge process (last 2048 records); this is not durable server-side idempotency.

If the game says **Save could not be confirmed**, keep the result on screen, check the server profile/Hall of Fame, and inspect:

```bash
journalctl -u ourcorz-bridge -n 60 --no-pager
```

Do not post your API key or bearer token. An “Uploaded” message confirms acceptance of the result request, not an independent guarantee of how the backend calculates rankings.

## Validation limits

Automated tests use a simulated upstream—no test scores are posted to the live leaderboard. They cover the exact documented payload, bearer reuse/renewal, explicit-401 recovery, duplicate/concurrent requests, wrong-player rejection, invalid scores/accuracy/durations, and no automatic retry after uncertain writes. PLANKZ browser controls and release installation have separate checks. The real session above is the final validation against your live API.

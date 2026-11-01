# Kiosk HOME and NFC feedback update

Release: **2026.10.31-dashboard.3**

## Changes

- `kiosk.sh` now launches Chromium with the requested `--password-store=basic` option.
- A shared navigation guard captures HOME/browser-back keys on press **and release**, including inside focused game pages. It forwards those controls to the existing dashboard navigation rather than allowing browser navigation. Existing game pause/exit safeguards are retained.
- The startup splash now uses `location.replace()` so it is not left behind as a browser-back destination.
- The dashboard's top strip now includes NFC reader/lookup status. A visible notification also appears over the dashboard or a game: **tag detected → API loading (with elapsed seconds) → player ready**, or a clear error such as **tag not registered**.
- Reader connection, tag receipt and API response are separate states. “Ready · tap tag” means the serial reader is connected, not that a physical tag has already been read. Once “tag detected” appears, the reader has supplied the tag ID; you can remove the card instead of continuing to wave it.
- Local status polling is now every 500 ms, without overlapping polls. This checks the local bridge, not the remote API on every poll.
- Fixed a display bug where loading and completion shared the same scan sequence, so the player badge could fail to update even after the API returned successfully.
- Serial reading no longer waits for a remote player lookup to complete. New detections are handled without building an unbounded queue of held-card repeats; stale responses cannot replace a newer player.

## Apply the update

Upload the new matching **package.zip** and **update.json** from `github-upload` to your existing update repository. Upload the ZIP first and the JSON last, or commit both together. Do not upload the outer full-kit/archive as `package.zip`.

**After the updater finishes, reboot the Pi once.** The new Chromium launch option is used only when the kiosk launcher starts again; replacing the file alone does not alter an already-running browser or shell loop. Rebooting also loads the new navigation guard, bridge and dashboard together.

The release includes all previous icons, Workout TimeZ, both MP3s and the connectivity indicator. Local `.env` secrets and unlisted files remain preserved by the updater. Nothing has been uploaded to GitHub by the agent.

## Verification and limits

Validation passed: **75 Python tests**, the three targeted dashboard/media/HOME-NFC JavaScript checks, shell syntax checking, and an isolated release installation with real local web-server requests. These cover navigation event capture, iframe routing, NFC progress/error states, player refresh transitions, bridge state/serial handling and packaged assets. Test logs are in `validation/`. The pre-existing BEEPZ/PLANKZ default-host test mismatch noted in the earlier release is not part of these passing checks.

Real NFC hardware timing, the remote's actual HOME signal and Raspberry Pi desktop bindings require an on-device test. The software can report and reduce delays **after the reader sends a tag ID**; it cannot prove that a card is physically in the RF field before that. If the screen remains at “ready · tap tag,” check reader distance/orientation, serial connection and reader firmware. If it says “API loading,” the tag was already detected and the delay is in the API/network path.

If HOME still opens another page after reboot, the remote may be invoking a Raspberry Pi desktop/browser action outside the webpage; that binding will need checking on the device.

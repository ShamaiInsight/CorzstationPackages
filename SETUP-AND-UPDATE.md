# CORZ Pi 5 — fix and test your updater

## Why it tried `/home/joelderoche`

The uploaded updater hard-coded that home directory. The installer tried to
rewrite it for the current user, but the example release JSON also hard-coded
`/home/joelderoche/corz-station/app/www`. A release could override the installer's
correction. Either an old runtime script or such a manifest can explain the error;
the Pi logs would identify which one actually failed.

**This version always targets the installed station folder.** On your account:

```text
/home/corz/corz-station/
├── app/ourcorz_bridge.py
├── app/.env                       ← kept local; never shipped
├── app/www/index.html
├── app/www/Music/                 ← exact capital M
├── runtime/updater.py
├── runtime/bleserver.py
├── runtime/hrserver.py
├── runtime/wifi-api.py
├── updater.json                   ← local update-source settings
└── .updates/backups/              ← changed-file backups
```

## 1. Put the two release files on GitHub

From the kit's `github-upload` folder, upload **these two files**, with exactly
these lowercase names, to the root of `ShamaiInsight/CorzstationPackages`, branch
`main`:

```text
package.zip
update.json
```

Upload `package.zip` first, then `update.json` last, or commit both together.
GitHub filenames are case-sensitive: the old `Package.zip` is a different file.
Do not upload the outer startup-kit ZIP in place of `package.zip`.
The additional `manifest-readable.json` is a reference copy; it is not required
on GitHub. The real install manifest is already inside `package.zip`.

The updater defaults to:

```text
https://raw.githubusercontent.com/ShamaiInsight/CorzstationPackages/main/update.json
```

The JSON names `package.zip` relative to that URL and supplies its SHA-256 and
version. The included release version is **`2026.10.25-test.1`**. If the repository,
branch or account differs, edit `~/corz-station/updater.json`. These unauthenticated
URLs require a public repository. No GitHub changes have been made for you.

## 2. Repair the existing Pi installation

Extract the **new startup kit** on the Pi. Open a terminal inside its
`CORZ-Pi-Startup` folder. Confirm you are logged in as your desktop account:

```bash
whoami
echo "$HOME"
```

Expected: `corz` and `/home/corz`. Then run **without sudo**:

```bash
bash repair-updater.sh
```

It stops the old updater, backs it up, installs the corrected updater and
systemd unit, and grants only the exact restart commands for the five CORZ
services. The updater itself still runs as `corz`, not root. It leaves your app,
API key, `.env`, existing music, databases and Python environment untouched.
It does not start the updater until you choose to do so.

If it reports no existing `~/corz-station`, use the fresh-install instructions in
README.md (`bash install.sh`) instead. Do not delete an existing station folder.
Do not run either script under another account or with `sudo bash`.

Check the configured source, then start:

```bash
nano ~/corz-station/updater.json
sudo systemctl enable --now corz-updater
journalctl -u corz-updater -f
```

Allow the download/restarts to finish. Press Ctrl+C to stop following the log;
this does not stop the service. The test release updates the Python bridge,
frontend and runtime scripts and adds a music file. Other running services
restart only when their script changes. Existing legacy services must already
point at this `corz-station` layout; the repair does not migrate unrelated units.

## 3. Confirm the actual update

On the Pi, open:

**http://localhost:7890/update-test.html**

It checks that:

- The new page and release marker exist.
- The **running Python bridge**, not just its file on disk, reports the release
  version in `bridgeBuild`.
- The updater's installed version matches.
- `Music/CORZ-update-test.mp3` is served. Press Play to test the speakers.

The melody is a short original diagnostic tone. It does not replace your own
songs or automatically become the dashboard's background track.

Terminal checks:

```bash
curl -s http://localhost:8081/api/status | python3 -m json.tool
curl -s http://localhost:7890/api/status | python3 -m json.tool
cat ~/corz-station/.corz_version
systemctl status ourcorz-bridge corz-updater corz-ble corz-hr corz-wifi --no-pager
```

Expect updater `version` and bridge `bridgeBuild` to equal
`2026.10.25-test.1`, with the updater `READY` and an empty `error`.
Restarting `corz-updater` again should say **Already up to date** without
downloading the ZIP. It checks on boot and on service restart, not on a timer.

If the Pi's `.env` sets `WEB_ROOT` elsewhere, the normal dashboard is not serving
`app/www`. Use the default station web root for this test, or deliberately move
your assets/settings back to that layout before testing.

## 4. Make the next release: code, music, folders, games

Work in the editable startup-kit source, not the generated ZIP. Update Python
scripts in `app/` or `runtime/`, and HTML/assets in `app/www/`. Then rebuild:

```bash
python3 tools/build_release.py --version 2026.10.25-test.2
```

This rebuilds both the ZIP and its matching hash/JSON. **Change the version for
every release**, even a test. Version strings are release IDs, not a semantic
ordering: any different ID is installed, including an intentional rollback ID.
Do not just edit the version in `update.json`; both manifests must agree.

To add music, put your MP3 in the source tree before rebuilding:

```text
app/www/Music/PR_1.mp3
app/www/Music/PR_2.mp3
```

The existing dashboard looks for `PR_1.mp3` through `PR_20.mp3` in **Music**.
Use your own licensed files; any available subset is supported. The updater
creates missing folders, replaces explicitly listed files, and preserves all
unlisted files. Do not give an absolute destination like `/home/corz/...`.

Alternatively merge a separate station-relative asset tree:

```bash
# my-assets/app/www/Music/PR_1.mp3 already exists
python3 tools/build_release.py --version 2026.10.25-test.3 --assets ./my-assets
```

Upload the newly generated `github-upload/package.zip` and `update.json`, then:

```bash
sudo systemctl restart corz-updater
```

Manual manifest authors can list individual files, explicit file deletions, and
ordinary nested ZIP bundles. A Unity build ZIP should contain its complete
`index.html`, `Build/` and `TemplateData/` tree. The old username-based Documents
scaffolding and `replace: true` directory wiping are intentionally unsupported.
An optional bundle example is in `release-manifest-example.json`; it is not the
default release. Game-server directories must live under `app/www/`.
Game subprocesses bind only to loopback and use the packaged server definitions
when the updater starts; restart the updater after changing those definitions.

## What “update everything” covers

**Supported:** frontend pages, nested assets/music, the Python bridge, BLE/HR/Wi-Fi
scripts, additional Python helper files, the updater itself, and `kiosk.sh`.
Known server scripts restart automatically when changed; new Python helpers in
`app/` restart the bridge. Additional runtime helpers can request the appropriate
existing service with builder option `--restart-service corz-ble.service`
(or `restart_services` in a manual manifest). A changed `kiosk.sh` takes effect at the
next desktop login/reboot, not in the already-running shell.

**Deliberately not an unattended root installer:** `.env`/keys, `updater.json`,
databases, virtual environments, logs, system packages, pip dependencies, and
systemd units are not overwritten by a release. A new Python dependency or brand
new system service needs a local maintenance step first. Do not expose secrets
in a public ZIP. Control of the release repository effectively controls code
executed by the station account: protect repository access. SHA-256 detects
corrupt/mismatched downloads; it is not a separate publisher signature.

## Failure handling and limits

The whole package is staged, paths checked and Python syntax validated before
live files change. File replacements are atomic individually, not one atomic
swap of the whole station. Backups and an interrupted-transaction journal are
kept under `~/corz-station/.updates/`. On file-copy or normal backend restart
failure, previous changed files and version metadata are restored. A pending
interrupted transaction is recovered at the next service start. Do not delete
`.updates/pending.json` to bypass recovery.

A small stable `updater-launcher.py`, installed by the setup/repair scripts and
excluded from OTA targets, handles recovery before starting the updater. It also
guards self-updates: if the replacement updater crashes or cannot confirm READY
within 90 seconds, it restores the previous changed files/version and starts the
old updater with download checks skipped for that start. Failed restored-service
restarts retain the recovery journal for the next attempt. Files, backup folders
and journal changes are flushed to disk, but no software-only test can guarantee
SD-card survival after a real power cut.

Restart checks confirm systemd reports the service active after a short delay;
they do **not** prove hardware operation or catch every later crash. Bad logic
with valid Python syntax still needs normal testing. A failed updater self-restart
leaves the new version on disk with a warning; manually restart the updater.

Backups are retained, not automatically pruned. Keep sufficient free storage;
remove old backup folders only after a successful boot and hardware check. ZIP
download limit is 512 MiB; total expanded file limit is 2 GiB, with a 128 MiB
per-archive-member memory limit (including an inner bundle ZIP). Larger Unity
builds need a streaming updater revision rather than simply raising that limit.
Updates need room
for the download, staging and changed-file backups. Disconnected boots use local
files and retain the update error in `/api/status` for diagnosis.

For a local no-GitHub test, stop the updater first so its lock is free:

```bash
sudo systemctl stop corz-updater
python3 ~/corz-station/runtime/updater.py --apply "$PWD/github-upload/package.zip"
sudo systemctl start corz-updater
```

The normal local apply restarts changed services. `--no-restart` is for isolated
development tests only; it does not load new code into running servers. The
installer/repair, real Raspberry Pi boot, Bluetooth/NFC, speaker output and live
GitHub download must still be verified on the physical Pi.

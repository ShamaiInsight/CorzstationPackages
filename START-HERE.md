# Your CORZ Pi 5 updater is ready to test

**Latest update:** see [MUSIC-PLAYBACK-FIX.md](MUSIC-PLAYBACK-FIX.md) for the music discovery correction, and [KIOSK-NFC-UPDATE.md](KIOSK-NFC-UPDATE.md) for HOME/NFC changes. **Reboot once after applying this release.** All earlier app/media/connectivity improvements remain included. The bundled `github-upload` release is now **2026.10.31-dashboard.4**. The original updater setup guidance below still applies.

The old updater **and its example manifest** could point at `/home/joelderoche`.
The corrected updater targets the installed `corz-station` folder instead:
**`/home/corz/corz-station` on your Pi.** Release manifests cannot override it.

## 1 · Upload the test release

Inside this kit, open **github-upload**. Upload these two files to the root of
`ShamaiInsight/CorzstationPackages`, branch `main`:

- **package.zip** — lowercase, not the old `Package.zip`
- **update.json** — upload last, or commit both files together

The package already contains its internal `manifest.json`. Do not rename the
outer startup-kit ZIP to `package.zip`. Nothing has been uploaded for you.

## 2 · Repair your current installation

Extract this new kit on the Pi and open a terminal inside `CORZ-Pi-Startup`.
Logged in as **corz**, run **without sudo**:

```bash
bash repair-updater.sh
sudo systemctl enable --now corz-updater
journalctl -u corz-updater -f
```

The repair preserves your app, `.env`/API key, music and database. It installs
the updater and narrowly limited service-restart permissions. A genuinely fresh
Pi uses `bash install.sh` instead; follow README.md for its remaining setup.

## 3 · Check the result

On the Pi open **http://localhost:7890/update-test.html**.

Expected version: **2026.10.31-dashboard.4**. The page verifies the running Python
bridge, updated web files and the new MP3 in `app/www/Music/`. Press Play for a
short test melody. Your dashboard's own tracks use `Music/PR_1.mp3` through
`PR_20.mp3`, with a capital M.

## Next update

Edit the kit's Python/HTML files or add your music, then run:

```bash
python3 tools/build_release.py --version 2026.10.25-test.2
```

Upload the regenerated pair and restart `corz-updater`. Changed Python services
restart automatically. Unlisted files stay intact. Dependencies, new system
services and local secrets still require deliberate local maintenance.

**Validation:** 51 automated tests passed, plus a real local HTTP-server install
test. Physical Pi hardware, actual systemd restarts and live GitHub downloading
still need on-device verification. See SETUP-AND-UPDATE.md for details/recovery.

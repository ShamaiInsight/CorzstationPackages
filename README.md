# CORZ Pi 5 startup kit — updater v2

For Raspberry Pi OS Desktop with labwc/Wayland, using your normal desktop user
(your Pi account is `corz`). No release contains an account-specific install path.
This includes a fresh installer, a repair script for an existing station, and
a GitHub-ready test release. All your supplied active Python servers are included.

**Already installed? Start with [SETUP-AND-UPDATE.md](SETUP-AND-UPDATE.md).**
Run `bash repair-updater.sh`, not the fresh installer. Local app settings remain intact.

## Services and files

| Service | Python file | Function / port |
|---|---|---|
| ourcorz-bridge | app/ourcorz_bridge.py | Dashboard, OurCorz API, NFC serial reader / 7890 |
| corz-ble | runtime/bleserver.py | CORZ_ controller BLE to WebSocket / 8765 |
| corz-hr | runtime/hrserver.py | Standard BLE heart-rate to WebSocket / 8766 |
| corz-updater | runtime/updater.py | GitHub download, install and status / 8081 |
| corz-wifi | runtime/wifi-api.py | Dashboard Wi-Fi controls via NetworkManager / 8082 |
| Desktop launcher | kiosk.sh | Chromium kiosk: splash then dashboard |

Do NOT also run corz_profile_server(1).py: it uses port 7890 too. It is the
older encrypted local-profile/database implementation, not the NFC reader.
Keep its existing database/backups if you need historical local profiles.
The supplied OurCorz bridge replaces that server for this API-based dashboard;
it already contains the NFC worker. test_bridge.py is a test, not a service.

Your bleserver.py acts as a BLE central/client connecting to the CORZ_ device,
and as a WebSocket server for the games. Both BLE programs retry connection;
systemd restarts crashed processes. Their simultaneous scanning/connecting on
one adapter must be verified with your controller and heart-rate device.

## 1. Inspect the current installation

On the Pi:

```bash
cat /etc/os-release
systemctl list-unit-files | grep -Ei 'corz|ble|nfc|homescreen|wifi|heart'
sudo ss -ltnp | grep -E ':7890|:8081|:8082|:8765|:8766'
```

Identify existing units and other autostart jobs that run the same scripts.
Before step 4, stop/disable only those identified conflicting units. For an
actual old unit called NAME.service: sudo systemctl disable --now NAME.service.
Also remove any old Chromium kiosk autostart entry so only one kiosk launches.
Do not disable NetworkManager or bluetooth; they are required system services.

## 2. Install

Extract this ZIP, open a terminal inside CORZ-Pi-Startup and run WITHOUT sudo:

```bash
bash install.sh
```

The installer requests sudo when needed. It creates ~/corz-station, a Python
virtual environment, five service units and a labwc desktop startup entry.
It installs dependencies and gives the desktop user serial/Bluetooth access.
It adds a validated sudoers rule allowing that user to run /usr/bin/nmcli as
root without a password, as required by your Wi-Fi script. This authorizes
network configuration; the Python server itself runs as your normal user.

It does NOT enable/start the five units or reboot. Existing same-name units
are backed up inside corz-station. An existing corz-station folder causes the
installer to stop instead of overwriting a previous install. If an earlier
attempt failed, inspect that folder before moving it aside and retrying.

## 3. Set API key, NFC reader and updater source

```bash
nano ~/corz-station/app/.env
ls -l /dev/serial/by-id/
```

Set OURCORZ_API_KEY. Set NFC_SERIAL_PORT to the actual stable /dev/serial/by-id
path if available. Defaults: /dev/ttyACM0, 115200 baud and /p polling command.
For a GPIO UART instead of USB: in raspi-config disable the serial login
console, enable serial hardware, then typically use /dev/serial0. Confirm the
reader's baud rate/protocol. Only one program may own the NFC serial port.

The API ZIP lacks some original music, icons and extra game pages. Preserve
those assets by copying them into ~/corz-station/app/www without overwriting
the supplied API-integrated pages. See app/README.md for WEB_ROOT alternatives;
if changing WEB_ROOT, copy runtime/splash.html into that web root too.

The updater source is in `~/corz-station/updater.json`. Publish the two files
from `github-upload/` at the repository root before enabling updates; see
SETUP-AND-UPDATE.md. For a no-download first boot set `"manifest_url": ""`.
This kit does not upload anything to GitHub or inspect your live repository.

Releases now use schema 2, with station-relative paths such as
`app/www/index.html`, `runtime/hrserver.py`, and `app/www/Music/PR_1.mp3`.
Absolute paths and `install_base` are refused. Existing schema-1 ZIPs must be
rebuilt with `tools/build_release.py`; no username replacement is needed.

The updater verifies the package SHA-256, stages and validates all files, backs
up changed files, and automatically restarts changed Python services. It can
also replace/restart itself. A failed file installation or backend restart
restores previous files; interrupted transactions are recovered at next start.
It checks once per service start/boot, not continuously. It does not install OS
packages, pip dependencies, systemd units, or modify `.env`, databases or venv.
These maintenance operations still require a deliberate local administrator step.

## 4. Enable services and desktop auto-login

After resolving old-service conflicts:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now bluetooth
sudo systemctl enable --now ourcorz-bridge corz-ble corz-hr corz-updater corz-wifi
sudo raspi-config
```

In raspi-config select Desktop boot with automatic login for the SAME account
used to install. Choose labwc/Wayland if needed. Disable Screen Blanking in
display settings. Menu wording varies by OS release. Then:

```bash
sudo reboot
```

At boot the five system services start, desktop auto-login runs Chromium,
Chromium waits for http://localhost:7890/splash.html, and the splash polls
http://localhost:8081/api/status before opening index.html on the same origin.
The original splash timeout/fallback behavior is retained. Chromium relaunches
if it exits. It is run by the graphical session, with its real Wayland
variables, not from a root system service.

## 5. Verify on the Pi

```bash
systemctl status ourcorz-bridge corz-ble corz-hr corz-updater corz-wifi --no-pager
journalctl -u ourcorz-bridge -u corz-ble -u corz-hr -u corz-updater -u corz-wifi -b -n 100 --no-pager
curl http://localhost:7890/api/status
curl http://localhost:8081/api/status
curl http://localhost:8082/api/wifi/status
tail -n 50 ~/corz-station/kiosk.log
```

Test a cold boot, NFC scan/unplug/replug, controller and heart-rate reconnect,
and a boot without internet. The page can load offline, but OurCorz lookup
requires connectivity. If Bluetooth reports permission errors inspect the
Pi's BlueZ D-Bus policy and group membership; do not run the whole kiosk as root.
Wi-Fi settings may disconnect your SSH session when changing network mode.

## Changes in this kit

- Uses your separately uploaded ourcorz_bridge.py and matching test_bridge.py.
- Renames bleserver(1).py to bleserver.py in the new package only.
- Replaces dashboard/game defaults raspberrypi.local:8765 with localhost:8765
  for this Pi-local kiosk. Fresh Chromium profile avoids old stored URLs.
- Wi-Fi API binds to loopback, checks Host/Origin against the local dashboard,
  and uses sudo -n so a missing permission fails instead of waiting for a password.
  Remote-browser Wi-Fi control is intentionally not configured by this local kit.
- Updater discovers the station directory; a release cannot override its root.
- `setup-updater.sh` installs exact-command restart permissions for five services.
- `repair-updater.sh` upgrades only the updater and its service configuration.
- GitHub test release adds a backend build identifier and a test music file/page.
- Original uploads remain unchanged. Old profile database is not migrated.

The OurCorz public API in this package has no score-submission endpoint;
automatic startup does not add cloud score saving (see app/README.md).

## Disable / rollback

Disable the five units with systemctl disable --now; remove only the
corz-station/kiosk.sh line from ~/.config/labwc/autostart, then log out/reboot.
Restore previous backed-up units if needed and run systemctl daemon-reload.
Remove /etc/sudoers.d/corz-wifi only if no remaining service needs that rule.
Keep your original database and files until migration is verified.

## Validation and reference

See VALIDATION.txt. Real Pi boot, BLE/serial hardware, live API credentials,
NetworkManager changes and GitHub package installation need on-device testing.

Official Raspberry Pi labwc kiosk instructions:
https://www.raspberrypi.com/tutorials/how-to-use-a-raspberry-pi-in-kiosk-mode/

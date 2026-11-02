# dashboard.14 — manual Cradle connections at Bluetooth level

The Pi diagnostics confirmed a Linux Bluetooth connection to CORZ_194FB089
while the CORZ sensor service had no active sensor session. They do not establish
which process initiated that link.

This release adds a Linux Bluetooth guard to the existing CORZ BLE service.
It checks known CORZ devices every second, clears their automatic-trust flag,
and disconnects links that do not have a current explicit staff selection.
It never initiates a connection, removes pairing, or changes non-CORZ devices.
An OS-initiated connection may briefly light the LED before the guard disconnects
it. This enforces manual ownership, not a guarantee that firmware or another
process can never briefly establish a link.

The heart-rate service now explicitly excludes CORZ devices, even if they
advertise the standard heart-rate service. Real heart-rate straps are unchanged.
Logs provided before this fix did not show the heart-rate service connecting.

## Install and test

Upload the matching package.zip and update.json (JSON last), let installation
complete, then reboot. No additional Terminal configuration is required.

1. Leave the previously connected Cradle powered on. Reboot the Pi.
2. Without selecting a device, wait 15 seconds. Its connected light should not
   remain fixed. `bluetoothctl devices Connected` should not list that Cradle.
3. Scan in Controller configuration: discovery alone must not connect.
4. Select the intended Cradle and press OK: its LED and dashboard should agree.
5. Switch to another Cradle without powering off the first, then Disconnect.
6. Reboot once more and repeat step 2.

If it remains connected, collect:

```bash
journalctl -b -u corz-ble.service -u corz-hr.service --no-pager -n 100
bluetoothctl devices Connected
```

Look for bluez-guard messages. Permission/adapter errors are logged and exposed
through the bridge's selectionError status rather than silently reported as
successful enforcement. This uses dbus-fast already installed with Linux Bleak.

Automated tests use simulated Bluetooth state. Actual Pi permissions, Bluetooth
timing, remote control, and the Cradle LED still require real-device testing.

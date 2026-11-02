# dashboard.15 — Cradle orientation and live preview

Open Configuration → Controller. Explicitly select and connect your numbered
Cradle, as before. The small movement preview shows the same corrected axes that
PLANKZ, BEEPZ and TENZ receive.

Keep the Cradle level and select Centre preview if needed. Tilt it right, left,
forward and back. Use Rotate +90° until the directions match. Each press rotates
movement clockwise: 0° → 90° → 180° → 270° → 0°.

Orientation is saved automatically on this Pi, globally for its Cradle controller,
not separately per device. It survives reboot and package updates. Confirm the
preview after switching to a Cradle mounted differently. Set orientation before
starting a game. Each game still performs its own normal neutral calibration;
Centre preview does not change game calibration or scores.

No mouse, text entry, new service or Terminal setup is needed. Use the remote's
arrows and OK. The preview does not connect, reconnect or scan devices. Stale or
disconnected movement is shown as unavailable. Rotation saving errors are shown
without silently changing the active setting.

Install the matching package.zip and update.json from GitHub, JSON last, then
reboot after installation. The dashboard.14 Bluetooth safeguard, API uploads,
NFC/profile screens, music and power menu are retained.

Tests use simulated sensor packets and browser connections. Actual Pi Bluetooth,
physical tilt direction and remote behaviour still need checking on hardware.

# CORZ power menu — dashboard.6

## 1. Publish and install

Upload the matching `package.zip` and `update.json` to your existing GitHub update location. Commit together or upload JSON last. `manifest-readable.json` is optional and included for inspection.

Reboot to let the Pi install **2026.10.31-dashboard.6**. Wait until installation finishes before continuing. The normal `.5` version may appear initially while the updater checks.

## 2. One-time setup on the Pi

Open Terminal as your normal desktop user. Run **without sudo in front**:

```bash
python3 ~/corz-station/runtime/setup_power_menu.py
```

Enter your administrator password if prompted. When setup succeeds, reboot:

```bash
sudo reboot
```

Setup backs up the labwc configuration, redirects `XF86PowerOff`, disables logind's automatic short-press power action, installs the loopback-only broker and grants only the two exact power commands. Backups remain in `~/corz-station/.power-menu-setup/`.

## 3. Use

Press Power → Shutdown / Reboot / Cancel. Cancel is selected initially. Use Left/Right and OK; HOME/Back cancels without exiting the running app. Shutdown/reboot are real actions after selection and OK. Workouts are not automatically paused while the popup is open.

If the old dialog still appears, test the popup separately:

```bash
python3 ~/corz-station/runtime/corz_power.py --request
systemctl status corz-power --no-pager
```

A working manual popup but unchanged remote behaviour suggests a different key signal or desktop configuration; report that result. Long hardware power holds cannot reliably be overridden.

## Undo

```bash
python3 ~/corz-station/runtime/setup_power_menu.py --undo
sudo reboot
```

Undo restores the original desktop config and removes the added service and permissions. It refuses to overwrite subsequent configuration edits.

Browser, request-security and simulated setup/undo tests passed. Actual Pi power-button capture and physical shutdown/reboot remain untested. Test power commands were mocked: no machine was powered off during validation.

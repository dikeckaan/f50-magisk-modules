# <module-name>

One-paragraph summary: what does this module do, who is it for?

## Why

What problem does this solve? What did the user (or device) lack before?
A few sentences about the motivation, not the implementation.

## Install

```sh
# From the bot (recommended if statusbot is already running):
/install_module <id>

# From a phone connected to the hotspot:
#   Magisk Manager → Modules → Install from storage → pick the zip

# From a host with adb access:
adb push <id>-vX.Y.Z.zip /sdcard/
adb shell su -c "magisk --install-module /sdcard/<id>-vX.Y.Z.zip"
reboot
```

## Requires

- Magisk **26.0+** (Delta or newer)
- Android **arm64** (Unisoc UMS9620 / ZTE F50)
- **bin-utils v1.3.0+** — provides `lib/common.sh` shared helpers (most
  modules in this ecosystem depend on it)
- Any module-specific runtime deps go here.

## Configuration

Post-install steps the user must do before the module is useful. Things
like dropping a token file, generating keys, opening a web wizard.

If nothing needs configuration, say "No configuration needed." Don't
fabricate steps.

## Bot integration

How `statusbot` exposes the module to the Telegram chat. Commands like
`/adguard {on|off|status}`, `/traffic_history`, etc. If the module is
standalone (no bot commands), say so.

## Files

| Path | Purpose |
|---|---|
| `/data/<dir>/...`             | Persistent state, not under the Magisk overlay |
| `/data/adb/modules/<id>/`     | Module install dir |
| `/data/adb/modules/<id>/system/bin/<binary>` | Static binary, mounted as `/system/bin/<binary>` |

## Uninstall

Magisk Manager → Modules → Remove. `uninstall.sh` will:
- Stop the daemon (`pkill -f ...`)
- Remove any iptables rules added
- **Keep** `/data/<dir>/` so reinstalling preserves your setup. Wipe
  manually with `rm -rf /data/<dir>` if you want a clean slate.

## License

GPL-3.0 (see [LICENSE](LICENSE)).

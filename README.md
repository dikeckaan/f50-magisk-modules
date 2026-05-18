# f50-magisk-modules

Magisk modules for **ZTE F50** (rooted Android 13, Unisoc UMS9620). Each module is its own repository with `updateJson` support — Magisk Manager auto-detects updates, and the [statusbot](https://github.com/dikeckaan/magisk-zte-f50-statusbot) `/update` command pulls the latest zip from GitHub releases.

## Modules

| Module | Purpose | Repo |
|---|---|---|
| **bin-utils** | Static arm64 binaries (curl, wget, jq, sendat, busybox) + CA bundle | [magisk-zte-f50-bin-utils](https://github.com/dikeckaan/magisk-zte-f50-bin-utils) |
| **cloudflared-tunnel** | Cloudflare Tunnel daemon w/ supervisor + health check | [magisk-zte-f50-cloudflared-tunnel](https://github.com/dikeckaan/magisk-zte-f50-cloudflared-tunnel) |
| **dropbear-ssh** | Dropbear SSH server on port 22222, key-only auth | [magisk-zte-f50-dropbear-ssh](https://github.com/dikeckaan/magisk-zte-f50-dropbear-ssh) |
| **wireless-adb-keeper** | Forces ADB on port 55555 every boot (avoids UFI conflict) | [magisk-zte-f50-wireless-adb-keeper](https://github.com/dikeckaan/magisk-zte-f50-wireless-adb-keeper) |
| **tailscale-control** | Tailscale 1.98.2 (arm64), toggle from bot, adaptive routing | [magisk-zte-f50-tailscale-control](https://github.com/dikeckaan/magisk-zte-f50-tailscale-control) |
| **statusbot** | Telegram bot for everything: status, AT, SMS, performance, IMEI, speedtest, `/update`, etc. | [magisk-zte-f50-statusbot](https://github.com/dikeckaan/magisk-zte-f50-statusbot) |

## Dependency graph

```
statusbot
  └── bin-utils                  (curl, jq, sendat, CA bundle — required)
  └── tailscale-control          (optional, for /tailscale toggle)
cloudflared-tunnel               (independent — for remote SSH/ADB)
dropbear-ssh                     (independent — SSH access)
wireless-adb-keeper              (independent — ADB on alternate port)
```

## Install order

1. **Flash `bin-utils` first** — statusbot will not function without it.
2. **Flash `cloudflared-tunnel`** if you want remote access via Cloudflare Zero Trust. Configure token at `/data/cloudflared/token` post-install.
3. **Flash `dropbear-ssh`** + put your authorized public key at `/data/dropbear/authorized_keys`.
4. **Flash `wireless-adb-keeper`** if you want ADB on port 55555 (avoids UFI-TOOLS conflict).
5. **Flash `statusbot`** — put bot token at `/sdcard/statusbot_token.txt` and chat ID at `/sdcard/statusbot_chat_id.txt` *before* first boot (or set later).
6. **(Optional)** Flash `tailscale-control` if you want Tailscale exit-node functionality. Configure via `/tailscale auth <key>` then `/tailscale on` from the bot.

All modules auto-update via Magisk Manager (uses each module's `updateJson` URL). Bot's `/update` command also fetches updates on demand.

## Auto-release CI

Each module repo has a GitHub Actions workflow (`.github/workflows/release.yml`) that triggers when `module.prop` changes on `main`:
1. Reads `version` and `versionCode` from `module.prop`
2. Builds the module zip (excludes repo-only files)
3. Creates a GitHub Release tagged `vX.Y.Z` with the zip attached
4. Auto-updates `update.json` (referenced by `updateJson` in `module.prop`)

To cut a new release, bump `version=` and `versionCode=` in `module.prop`, commit, push to `main`. CI does the rest.

## License

GPL-3.0 — see [LICENSE](LICENSE) in each repo.

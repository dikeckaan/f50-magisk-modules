# f50-magisk-modules

Magisk modules for **ZTE F50** (rooted Android 13, Unisoc UMS9620) —
the "cep çakısı" (Swiss-army knife) for a pocket cellular gateway.
Each module is its own repository with `updateJson` support; Magisk
Manager auto-detects updates and the
[statusbot](https://github.com/dikeckaan/magisk-zte-f50-statusbot)
`/install_module` + `/update` commands pull releases directly from GitHub.

The bot fetches this index from [`modules.json`](modules.json), which is the
machine-readable catalog. Add a module = add an entry there.

## Modules

| Module | Purpose | Repo |
|---|---|---|
| **bin-utils** | Static arm64 binaries (curl, wget, jq, sendat, bash, busybox) + CA bundle + `lib/common.sh` shared helpers | [magisk-zte-f50-bin-utils](https://github.com/dikeckaan/magisk-zte-f50-bin-utils) |
| **cloudflared-tunnel** | Cloudflare Tunnel daemon w/ supervisor + health check (toybox-compatible) | [magisk-zte-f50-cloudflared-tunnel](https://github.com/dikeckaan/magisk-zte-f50-cloudflared-tunnel) |
| **dropbear-ssh** | Dropbear SSH server on port 22222, key-only auth | [magisk-zte-f50-dropbear-ssh](https://github.com/dikeckaan/magisk-zte-f50-dropbear-ssh) |
| **wireless-adb-keeper** | Forces ADB on port 55555 every boot (avoids UFI conflict) | [magisk-zte-f50-wireless-adb-keeper](https://github.com/dikeckaan/magisk-zte-f50-wireless-adb-keeper) |
| **tailscale-control** | Tailscale 1.98.2 (arm64), toggle from bot, adaptive routing | [magisk-zte-f50-tailscale-control](https://github.com/dikeckaan/magisk-zte-f50-tailscale-control) |
| **statusbot** | Telegram bot (12-language UI): status, AT, SMS, performance, IMEI, speedtest, `/update`, `/install_module`, sha256 verify, plus integrations for every optional module below | [magisk-zte-f50-statusbot](https://github.com/dikeckaan/magisk-zte-f50-statusbot) |
| **traffic-stats** | Pure-shell vnstat-lite — daily per-interface RX/TX accumulator, survives counter resets | [magisk-zte-f50-traffic-stats](https://github.com/dikeckaan/magisk-zte-f50-traffic-stats) |
| **adguardhome** | Network-wide DNS ad-blocker; transparent NAT redirect on br0 filters hotspot clients; web UI on :3000 | [magisk-zte-f50-adguardhome](https://github.com/dikeckaan/magisk-zte-f50-adguardhome) |
| **cell-tools** | Periodic AT-command poller — cell tower DB, IMSI-catcher anomaly detection. Powers bot `/spectrum`, `/imsi_watch`, `/locate` (BeaconDB triangulation) | [magisk-zte-f50-cell-tools](https://github.com/dikeckaan/magisk-zte-f50-cell-tools) |
| **sms-cmd** | Offline SMS command channel. When Telegram is unreachable, an authorised phone can SMS a small command set (status, locate, reboot, panic, kill). Whitelist + shared secret + allowed-command gate. | [magisk-zte-f50-sms-cmd](https://github.com/dikeckaan/magisk-zte-f50-sms-cmd) |
| **tor-relay** | Tor 0.4.9.8 bridge (NOT exit) with bundled OpenSSL/libevent (~9 MB). VPN-aware outbound routing (`direct`/`vpn`) + per-client transparent tor for hotspot clients (`/tor through`). | [magisk-zte-f50-tor-relay](https://github.com/dikeckaan/magisk-zte-f50-tor-relay) |
| **mitm-lab** | ⚠ DANGEROUS — Transparent HTTPS MITM proxy. Custom 4.8 MB Go binary, local self-signed CA, per-client iptables redirect. Installed but DISABLED by default. Cert-pinned apps will break. | [magisk-zte-f50-mitm-lab](https://github.com/dikeckaan/magisk-zte-f50-mitm-lab) |
| **sip-server** [🚧 dev] | ⚠ EXPLORATORY. Embedded UDP/5060 SIP server (~2.4 MB static Go binary). Pairs with the on-device `com.f50.sip` F50SipBridge app. SMS-over-SIP works; cellular-call observation depends on `ims-voice-fix` to survive the F50 screen-off timeout. | [magisk-zte-f50-sip-server](https://github.com/dikeckaan/magisk-zte-f50-sip-server) |
| **ims-voice-fix** [🚧 dev] | ⚠ EXPLORATORY. Bind-mounts a patched `com.spreadtrum.ims` to disable the SCREEN_OFF → call-drop handler. Root cause unconfirmed (counterexample on a second F50 same carrier). See repo for the `screen_off_timeout=infinity` self-test before installing. | [magisk-zte-f50-ims-voice-fix](https://github.com/dikeckaan/magisk-zte-f50-ims-voice-fix) |
| **vpn-gateway** | Fork of Kr328/vpn-gateway. Routes LAN clients through tun0, plus a Tailscale-aware exception (pref 9990) so on-device services reach Tailscale peers correctly without their replies being swallowed by the VPN tunnel. | [magisk-zte-f50-vpn-gateway](https://github.com/dikeckaan/magisk-zte-f50-vpn-gateway) |
| **hotspot-region** | Change the WiFi regulatory region/country code at runtime (default TR, selectable) via `cmd wifi force-country-code` — no file/NVRAM edits, no bootloop risk. ADB CLI + bot `/region`. | [magisk-zte-f50-hotspot-region](https://github.com/dikeckaan/magisk-zte-f50-hotspot-region) |
| **lite-mem** | Memory relief for the low-RAM F50: extra zstd zram swap, vm tuning, configurable debloat, and a shell/bot-toggleable kill of the ZTE goform web panel. CLI `lite-mem status / webui off\|on`, bot `/lite`. | [magisk-zte-f50-lite-mem](https://github.com/dikeckaan/magisk-zte-f50-lite-mem) |

## Dependency graph

```
bin-utils                          (curl + jq + bash + sendat + cacert.pem + lib/common.sh — required by everything below)
  │
  ├── statusbot                    Telegram control surface (REQUIRED)
  │     ├── tailscale-control      optional, /tailscale toggle
  │     ├── traffic-stats          optional, /traffic_history
  │     ├── adguardhome            optional, /adguard + /dns_watch
  │     ├── cell-tools             optional, /spectrum + /imsi_watch + /locate + /ussd
  │     ├── sms-cmd                optional, /sms_cmd offline backup channel
  │     ├── tor-relay              optional, /tor bridge + through-tor
  │     ├── mitm-lab               optional, /mitm transparent HTTPS proxy
  │     ├── ims-voice-fix [dev]    exploratory, IMS apk overlay for screen-off call-drop
  │     └── sip-server     [dev]   exploratory, embedded SIP server (depends on ims-voice-fix)
  │
  ├── cloudflared-tunnel           independent — outbound Cloudflare Zero Trust tunnel for remote SSH/ADB
  ├── dropbear-ssh                 independent — SSH on :22222 (key-only)
  └── wireless-adb-keeper          independent — ADB on :55555 (no telephony conflict)
```

## Install order

1. **`bin-utils`** — flash first; everything else hard-depends on `lib/common.sh`.
2. **`cloudflared-tunnel`** — for remote management. Drop your tunnel token at `/data/cloudflared/token` then reboot.
3. **`dropbear-ssh`** — put your public key at `/data/ssh/authorized_keys`.
4. **`wireless-adb-keeper`** — ADB on port 55555 (avoids UFI-TOOLS clash).
5. **`statusbot`** — Telegram bot. Place token at `/sdcard/statusbot_token.txt` and chat ID at `/sdcard/statusbot_chat_id.txt` before first boot.
6. **Everything else is optional** — install from Telegram once the bot is up:
   - `/install_module list` — show full catalog with installed/available state
   - `/install_module tailscale-control`
   - `/install_module traffic-stats`
   - `/install_module adguardhome`
   - `/install_module cell-tools`
   - `/install_module sms-cmd`
   - `/install_module tor-relay`
   - `/install_module mitm-lab` (⚠ read warnings)
   - `/install_module sip-server` (pair with f50sip-app)
   - `/install_module ims-voice-fix` (🚧 exploratory — read the repo README first; run the screen-off test before deciding)
   - `/install_module hotspot-region` (alias: region)
   - `/install_module lite-mem` (alias: mem / zram — adds zram swap + debloat)

Alias resolution lets you type short forms: `/install_module adguard`, `/install_module ssh`, `/install_module tunnel`, `/install_module ts`, `/install_module traffic`, `/install_module imsi`, `/install_module tor`, `/install_module mitm`, etc.

## Bot command surface (statusbot)

> Subset; see `/help` for the full list (it's ~80 commands).

| Command | Source module | Notes |
|---|---|---|
| `/status` / `/uptime` / `/temp` / `/mem` / `/cpu_freq` | core | Device health |
| `/wifi` / `/dhcp` / `/clients` / `/traffic` | core | Hotspot info |
| `/signal` / `/cellinfo` / `/imei` / `/sms_list` | core (sendat) | Cellular telephony |
| `/update [all\|<id>]` / `/install_module [list\|<id>]` | core | sha256-verified |
| `/tailscale {on\|off\|status\|...}` | tailscale-control | Exit-node toggle |
| `/traffic_history [iface]` | traffic-stats | vnstat-lite |
| `/adguard {status\|on\|off\|log\|url}` | adguardhome | DNS ad-blocker control |
| `/dns_watch {recent\|top\|blocked\|client\|stats}` | adguardhome | Live AGH query log |
| `/spectrum` | cell-tools | Visible cells right now |
| `/imsi_watch {status\|list\|alerts}` | cell-tools | Anomaly events |
| `/locate` | cell-tools | GPS via BeaconDB cell triangulation |
| `/sms_cmd {status\|add\|remove\|secret\|log}` | sms-cmd | SMS offline backup |
| `/tor {status\|on\|off\|route\|fingerprint\|through}` | tor-relay | Bridge + VPN-aware + per-client transparent |
| `/mitm {status\|gen_ca\|ca\|add\|remove\|on\|off\|flows}` | mitm-lab | ⚠ HTTPS decrypt lab |

## Resource footprint

ZTE F50 has **1441 MB RAM** and a 4-core Unisoc UMS9620. Below is the idle resident set + idle CPU share of each module's daemons, as observed on-device for the released ones and estimated (marked 🚧) for those still under development. CPU figures are total-system percent on a 4-core SoC; treat them as "≤" — bursts during work (SMS poll, Telegram callback, DNS query, SIP REGISTER) are higher but short-lived.

| Module | Idle RAM (RSS) | Idle CPU (%) | Notes |
|---|---:|---:|---|
| **bin-utils** | 0 MB | 0% | Static binaries on disk only — no daemon |
| **statusbot** | ~25–30 MB | 0.1–0.3% | bash 5 + curl long-poll loop |
| **cloudflared-tunnel** | ~30 MB | 0.3–0.5% | Go + QUIC keepalive |
| **dropbear-ssh** | ~1 MB (idle) | ~0% | +2–3 MB per active session |
| **wireless-adb-keeper** | 0 MB | 0% | One-shot boot script |
| **tailscale-control** | ~15–20 MB | 0.1–0.2% | tailscaled (Go) keepalive |
| **traffic-stats** | ~1 MB | ~0.1% | Shell sample every 60 s |
| **adguardhome** | ~40–50 MB | 0.2–0.5% | Go + DNS cache; scales with QPS |
| **cell-tools** | ~1–2 MB | ~0.1% | bash AT poll every 60 s |
| **sms-cmd** | ~1–2 MB | ~0.1% | bash `AT+CMGL` poll every 30 s |
| **tor-relay** (idle) | ~25–30 MB | 0.2–0.5% | Bandwidth use adds CPU |
| **mitm-lab** (disabled) | 0 MB | 0% | Default off; ~20 MB when enabled |
| **openeuicc-systemizer** | 0 MB | 0% | system_ext overlay, no daemon |
| **sip-server** 🚧 | ~10–15 MB | ~0.1% | Go UDP/5060 listener |
| **ims-voice-fix** 🚧 | **0 MB** | **0%** | Bind-mount overlay — same process as stock IMS |
| **f50sip-app** (com.f50.sip) | ~30–40 MB | 0.1–0.3% | Foreground service + WakeLock |
| **Typical full stack idle** | **~180–220 MB** | **~2–4%** | ~12–15% of 1441 MB RAM |

`ims-voice-fix` is genuinely free — it doesn't start any new process, it just swaps the on-disk apk that the existing `com.spreadtrum.ims` service already loads. `sip-server` is the cheapest new daemon; `f50sip-app` is the most expensive single piece purely because Android VM overhead per app is ~30 MB minimum.

## modules.json

The bot reads [`modules.json`](modules.json) at install time. Schema v2:

```json
{
  "schema_version": 2,
  "modules": [
    {
      "id": "<short-id>",
      "name": "<display name>",
      "description": "<one-liner>",
      "required": false,
      "arch": "arm64",
      "min_magisk_version": 26000,
      "dependencies": ["bin-utils"],
      "aliases": ["<alias1>", "<alias2>"],
      "update_json": "https://raw.githubusercontent.com/.../main/update.json"
    }
  ]
}
```

To add a new module to the catalog: append an entry to `modules.json` and push. Bot will pick it up within its 10-minute cache TTL.

## Auto-release CI

Each module repo's `.github/workflows/release.yml` is a 12-line caller that delegates to the reusable workflow at [`.github/workflows/release-module.yml`](.github/workflows/release-module.yml) in *this* repo. The reusable workflow:

1. Parses `module.prop` for `id`, `version`, `versionCode`.
2. Skips if a release with that version already exists.
3. Builds the module zip (excludes `.git/`, `.github/`, `LICENSE`, `CHANGELOG.md`, `update.json`, existing `*.zip`).
4. Computes the zip's **SHA-256**.
5. Generates `update.json` with `version`, `versionCode`, `zipUrl`, `sha256`, `changelog`.
6. Commits `update.json` back to `main` (signed as github-actions bot).
7. Creates the GitHub Release with the zip attached and SHA-256 in the notes.

To cut a new release for any module: bump `version=` and `versionCode=` in `module.prop`, commit, push to `main`. The bot's `/update` and `/install_module` verify the SHA-256 against the zip before flashing.

## License

GPL-3.0 — see [LICENSE](LICENSE) in each repo.

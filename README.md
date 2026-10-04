# plery

A single-file, dependency-free Python CLI for managing a 4G MiFi (portable UFI) device over its embedded ZTE-style `/reqproc` web API.

> ⚠️ **Hardware-specific.** This tool targets the API and field names of one specific MiFi/UFI hardware model (4G carrier: China Broadnet, dual-SIM, Web server `Demo-Webs`, admin UI served at `http://192.168.43.1`). It will **not** work on other router brands or firmware with a different API layout. Before running any command, confirm your device responds to `/reqproc/proc_get` — if it does not, this tool is not applicable to your hardware.

## Features

| Command | Description |
|---|---|
| `plery --version` | Version banner |
| `plery login [--password <pw>]` | Log in (password Base64-encoded on the wire, persisted to config) |
| `plery logout` | Log out |
| `plery status` | Signal (RSSI), network, SIM, WAN IP, PPP state, WiFi state |
| `plery traffic` | Monthly up/down traffic + online time |
| `plery devices` | Connected Wi-Fi stations (MAC / IP / hostname) |
| `plery net on\|off` | Connect / disconnect the WAN link |
| `plery limit show` / `limit set --size 205 [--alert 90] [--off]` | Monthly data cap (GB) + alert threshold |
| `plery wifi show` / `password <pw>` / `coverage low\|medium\|high` / `sleep --minutes N` | Wi-Fi SSID state, WPA2 password, transmit power, idle sleep |
| `plery device info` / `reboot` / `shutdown` | Battery, power save, reboot, power off |
| `plery tsw show` / `set --on HH:MM --off HH:MM` / `off` | Timed power on/off |
| `plery macfilter show` / `add <MAC> [--mode 1\|2]` / `del <MAC>` / `mode 0\|1\|2` | Wi-Fi MAC black/white filtering |
| `plery raw 'cmd=...'` / `raw --post 'goformId=...'` | Direct `proc_get` / `proc_post` escape hatch |

Global flags: `--json` (raw API JSON for scripting), `--host <ip>`.

## Requirements

- Python 3.13+ (standard library only — no pip install)
- The target device on the local network, default gateway `192.168.43.1`

## Install

```bash
chmod +x plery
ln -sf "$(pwd)/plery" ~/.local/bin/plery
```

## Configuration

`~/.config/plery/config.json` (mode 0600, auto-created by `plery login --password ...`):

```json
{
  "host": "192.168.43.1",
  "password": "<your-device-admin-password>"
}
```

The password is provided by `plery login --password <pw>` and stored only in the
local config file — no credentials are shipped with this repository.

## Security notes

- The default factory credentials on this hardware are weak; change the admin password
  (`plery login --password <new>`) and the Wi-Fi passphrase
  (`plery wifi password <new>`) if the device is on an untrusted network.
- All API traffic is plain HTTP on the LAN — expected for this kind of embedded device.

## Verification

Read-only commands and idempotent writes (data-cap set to the current value, Wi-Fi
coverage to the current value, MAC filter add/del round-trips, timed power
on/off, WAN connect/disconnect) were exercised against the real device.
`device reboot` / `device shutdown` / `wifi password` are implemented but
deliberately not executed during testing (destructive).

## License

MIT — see [LICENSE](LICENSE.md).

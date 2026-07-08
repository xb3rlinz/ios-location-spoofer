# iOS Location Spoofer

Using the HTTPS decryption function of proxy software, Apple Maps location can be tricked into being located anywhere in the world.

> 📖 **Beginners, read this first!** → [**Beginner-Friendly, Step-by-Step Guide (with Pictures)**](Guide.md)(Step-by-step guide to installation, configuration, and activation, including troubleshooting common problems)

## Reference Projects

This project is based on [acheong08/ios-location-spoofer](https://github.com/acheong08/ios-location-spoofer) The core research. The original project was a standalone iOS app written in Go that used a self-built VPN + MITM proxy to achieve location spoofing.

This repository ported its core logic to JavaScript, adapting it to five proxy platforms: Shadowrocket, Surge, Loon, Quantumult X, and Stash. It requires no compilation or developer account and is ready to use immediately.

### New features compared to the original version

- **Multi-platform support** — Expanding from a single iOS app to five proxy apps, reaching more users.
- **Cell Base Station Coordinate Modification** — The original Go version only modified the WiFi hotspot coordinates, while the JS version additionally handles the coordinate replacement of CellTower (fields 22/24).
- **Multi-response format compatibility** — Automatically detects the container format of Apple's response (ARPC / synthetic / marker / bare) to ensure that it can still be correctly recognized by iOS after modification.
- **Motion state spoofing** — Rewrite motionActivityType and motionActivityConfidence simultaneously to reduce the possibility of being detected by the system.

## What happened?

The iPhone checks Wi-Fi and cell tower signals, then uses the BSSID list to ask Apple for the location of these devices. Apple provides a list of coordinates, and iOS uses these coordinates to calculate its own location.

This setup does something quite simple: **intercept the coordinates Apple sends back along the way and modify them all to the numbers you want.** The iPhone then receives the modified coordinates and calculates the location you specified.

## Which software is supported?

| Software | File | Import Method | Status |
|------|------|---------|------|
Shadowrocket | `ios-location-spoofer.sgmodule` | Configuration → Top right corner + | ✅ Tested and passed |
| Surge | `ios-location-spoofer-surge.sgmodule` Homepage → Modules → Install New Module | ✅ Tested and working |
| Loon | `ios-location-spoofer.lnplugin` | Settings → Plugins → Add Plugin | ✅ Tested and working |
| How much X | `ios-location-spoofer.snippet` | Settings → Rewrite → Add | 🟡 To be tested |
| Stash | `ios-location-spoofer.stoverride` | Overwrite → Install Overwrite | ✅ Tested and passed |

> Users who have tested this are welcome to report their results in the Issue section; if there are any issues, please submit a pull request for correction – please at least specify **which software, which version, what system, and the original error log**.

## How to use

1. Enable HTTPS decryption/MITM switch in the software.
2. Install and trust the CA certificate (Settings → General → VPN & Device Management → Install → Certificate Trust Settings → Enable).
3. Import the module file and check the "Enable" box.
4. Disconnect and reconnect to the VPN, toggle location services on and off.
5. Open the map app to verify.

### Loon Additional Notes

1. Import `ios-location-spoofer.lnplugin` Then, open the plugin configuration page in **Settings → Plugins**.
2. You can directly enter **latitude/longitude**; **address search** is performed by a scheduled task every 15 minutes, which parses and caches the address online (for the first time, please enter the latitude and longitude directly, or save the address and wait for a cron job).
3. Loon's **MITM** must be enabled and the certificate trusted, and the plugin must be within the specified range. `[mitm]` Four domains are effective
4. The plugin includes a **Prepare** request script (settings). `Accept-Encoding: identity`To avoid gzip causing `zip decompress error` (Script timed out)
5. After changing the coordinates, turn positioning on and off; for debugging, turn on the **debugging log** and search in the Loon log. `Location spoofer`

> If the log appears `Evaluate script timeout` or `zip decompress error:-3`Update the plugin and reload Loon, and confirm that all three scripts (Prepare / Response / Geocode cron) are enabled.

## Change coordinates

The default value is Apple Park (37.3349, -122.00902). Change it in the module parameters:

```
latitude=39.9042&longitude=116.4074
```

parameter:

| Name | Default Value | Description |
|------|--------|------|
| `latitude` | 37.3349 | Target Latitude |
| `longitude` | -122.00902 | Target Longitude |
| `address` | (Empty) | Address search (entered in the Loon plugin UI, resolved to latitude and longitude online, taking precedence over manual latitude and longitude) |
| `horizontalAccuracy` | 39 | Horizontal accuracy |
| `verticalAccuracy` | 1000 | Vertical accuracy |
| `altitude` | 530 | Altitude |
| `failOpen` | true | Allow original data to pass if error occurs |
| `debug` | false | Debug log |

## Document List

```
ios-location-spoofer.sgmodule       # Shadowrocket
ios-location-spoofer-surge.sgmodule # Surge
ios-location-spoofer.lnplugin       # Loon
ios-location-spoofer.snippet        # Quantumult X
ios-location-spoofer.stoverride     # Stash
location-spoofer.js                 # 核心脚本（四平台共用）
location-spoofer-qx.js              # QX 专用
location-spoofer-config.json        # 配置样板
使用教程.md                         # 小白保姆级图文教程
location-picker/                    # 进阶（可选）：网页地图选点（Node 或 Cloudflare Worker）
location-picker/worker/             # Cloudflare Worker 版（免 VPS，支持 Loon configUrl）
```

## Advanced: Web Map Point Selection Tool

Frequently changing locations and too lazy to manually check coordinates and adjust parameters? The project comes with built-in... [`location-picker/`](location-picker/) Map point selection tool: Point to map for location, automatic elevation, adjustable precision, compatible with Loon/Shadowrocket. `configUrl` Read.

Three deployment methods:

| Method | Table of Contents | Suitable for |
|------|------|------|
| **Cloudflare Worker — Wrangler CLI** (Recommended) | [`location-picker/worker/`](location-picker/worker/) No VPS required, comes with HTTPS; familiar with command line |
| **Cloudflare Worker — Web Backend** | [`location-picker/cloudflare-webui/`](location-picker/cloudflare-webui/) No VPS required, comes with HTTPS; if you don't want to install npm/Wrangler, just copy the single file.
| Node self-hosting | [`location-picker/server.js`](location-picker/server.js) | Have your own VPS / NAS |

Loon plugin **remote configuration URL** example:

```
https://你的worker.workers.dev/loc.json?token=你的TOKEN
```

## Friendly Links

This project welcomes supervision and feedback from members of the Linux DO community:[LINUX DO](https://linux.do)

## location-picker server configuration

`location-picker/server.js` Controlled by environment variables,`TOKEN` If no process is set, it will exit directly and will not use a weak password as a fallback.

| Variable | Required | Default Value | Description |
|------|---------|--------|------|
| `TOKEN` | **Required** | None | Access password and Shadowrocket module `argument=` The end `configUrl` Inside `token=` Consistency is essential. Recommendation. `openssl rand -hex 24` Generate |
| `PORT` No | `8080` Listening port; ports below 1024 require root access.
| `CERT` | No | Empty | HTTPS certificate fullchain path; `KEY` HTTPS must be configured at the same time.
| `KEY` | No | Empty | HTTPS private key path; `CERT` HTTPS must be configured at the same time.

Startup example:

```bash
# http（最简，先跑通流程再用 https）
TOKEN=$(openssl rand -hex 24) PORT=8080 node server.js

# https（复用 acme.sh 证书；续期无需重启，进程每 12 小时自动热加载）
TOKEN=$(openssl rand -hex 24) PORT=8443 \
CERT=/root/cert/example.com/fullchain.pem \
KEY=/root/cert/example.com/privkey.pem \
node server.js
```

Data file `loc.json` Automatically fall `server.js` Same directory, records current coordinates/altitude/accuracy; already... `.gitignore` If ignored, it will not be mistakenly submitted to the repository.

> ⚠️ **Do not put `TOKEN` Write this in your command-line history**—systemd is recommended. `Environment=` or `.env` + `direnv`,avoid `history` / `ps aux` Give way.

# Location Picker — Cloudflare Worker

Fully compatible with the `../server.js` API. No VPS required, HTTPS included by default, and supports **Loon**, **Shadowrocket**, and **Surge** via `configUrl`.

## Endpoints

| Path | Method | Description |
|------|--------|-------------|
| `/` | GET | Interactive map page (append `?token=` to the URL to enable saving) |
| `/loc.json?token=` | GET | Retrieve the current location as JSON |
| `/set?token=` | POST | Save the selected location |
| `/enable` | POST | Switch back to your real location (tap again to re-enable spoofing) |
| `/health` | GET | Health check (no token required) |

## Deployment

> If you only want to deploy using the Cloudflare Dashboard (copy & paste) without installing npm or Wrangler, see: [`../cloudflare-webui/`](../cloudflare-webui/).

### 1. Install Dependencies

```bash
cd location-picker/worker
npm install
```

### 2. Create a KV Namespace

```bash
npx wrangler kv namespace create LOC_KV
npx wrangler kv namespace create LOC_KV --preview
```

Copy the generated `id` values into the `id` and `preview_id` fields in `wrangler.jsonc`.

### 3. Set an Access Token

```bash
npx wrangler secret put TOKEN
# Enter a secure random string, for example one generated with:
# openssl rand -hex 24
```

For local development, copy `.dev.vars.example` to `.dev.vars` and set:

```text
TOKEN=your_token_here
```

### 4. Deploy

```bash
npm run deploy
```

After deployment, note the Worker URL, for example:

```
https://ios-location-picker.your-account.workers.dev
```

## Loon Configuration

In **Loon**:

**Settings → Plugin → iOS Location Spoofer → Remote Configuration URL**

Enter:

```
https://ios-location-picker.your-account.workers.dev/loc.json?token=your_token
```

Save the configuration, then open the map page in Safari (or any browser on your iPhone):

```
https://ios-location-picker.your-account.workers.dev/?token=your_token
```

Tap a location on the map → **Save Location** → Turn **Location Services** off and back on to apply the new location. (Loon typically refreshes its cache within about 60 seconds.)

## Shadowrocket Configuration

Append the following to the end of your module's `argument=` value:

```
&configUrl=https://ios-location-picker.your-account.workers.dev/loc.json?token=your_token
```

## Custom Domain (Optional)

In the **Cloudflare Dashboard**:

**Workers → Your Worker → Settings → Domains**

Bind a custom subdomain, for example:

```
loc.example.com
```

## Differences from the Node.js Version

- Uses **Cloudflare KV** instead of local files for storage (the free tier is more than sufficient for personal use)
- Cloudflare KV is **eventually consistent**, so after saving a location, Loon may take up to **60 seconds** to refresh its cache
- HTTPS certificates are managed automatically—no manual certificate setup required
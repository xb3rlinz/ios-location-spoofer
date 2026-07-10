# Deploying with the Cloudflare Dashboard

This guide is intended for beginners. It does **not** require installing Node.js, npm, or Wrangler locally, nor does it require understanding a multi-file Worker project.

If you're comfortable using the command line, it's recommended to use the Wrangler deployment method in `../worker/`.

If you just want to copy and paste code into the Cloudflare Dashboard, use the single-file Worker included in this folder:

```text
location-picker/cloudflare-webui/worker.js
```

## What This Guide Does

`location-picker/server.js` is designed for Node.js, VPS, or NAS deployments. It **cannot** be deployed directly to Cloudflare Workers because Workers don't support Node modules such as `http`, `https`, `fs`, or local files like `loc.json`.

The correct Cloudflare architecture is:

```text
Cloudflare Worker   → Serves the map page and API
Cloudflare KV       → Stores the current coordinates
Worker Secret       → Stores the access token (TOKEN)
```

To avoid issues with Cloudflare's online editor and multi-file projects, this folder provides a pre-built, single-file `worker.js`. Simply replace the default Worker code with it.

---

# 1. Create a Worker

1. Open the Cloudflare Dashboard.
2. Go to:

```text
Workers & Pages
```

3. Click:

```text
Create application
```

4. Choose:

```text
Worker
```

**Do not choose Pages.** This project requires APIs and KV storage, not a static website.

5. Name your Worker, for example:

```text
ios-location-picker
```

6. Open the Worker and click:

```text
Edit code
```

---

# 2. Copy the Single-File Worker

1. Open:

```text
location-picker/cloudflare-webui/worker.js
```

2. Copy the entire file.
3. Return to the Cloudflare code editor.
4. Open the default file (usually named `worker.js`).
5. Delete the default Hello World code.
6. Paste the contents of `worker.js`.
7. Click:

```text
Deploy
```

Once deployed, visit:

```text
https://your-worker.workers.dev/?token=YOUR_TOKEN
```

If the map loads, the code was deployed successfully.

Opening the homepage **without** a token should return:

```json
{"error":"bad token"}
```

---

# 3. Create a KV Namespace

1. Return to the Cloudflare Dashboard.
2. Go to:

```text
Workers & Pages
```

3. Open:

```text
KV
```

(or **Workers KV**)

4. Create a namespace.

Suggested name:

```text
LOC_KV
```

---

# 4. Bind the KV Namespace

1. Open your Worker.
2. Go to:

```text
Settings
```

3. Open:

```text
Bindings
```

4. Click:

```text
Add Binding
```

5. Select:

```text
KV Namespace
```

6. Set the variable name **exactly** as:

```text
LOC_KV
```

7. Select the namespace you created.
8. Save.

⚠️ The variable name must be **exactly** `LOC_KV`.

---

# 5. Add the TOKEN Secret

The TOKEN protects access to the map and configuration.

1. Go to:

```text
Settings
```

2. Open:

```text
Variables and Secrets
```

3. Add a new variable.
4. Choose:

```text
Secret
```

5. Name:

```text
TOKEN
```

6. Enter a long, random string.

Example:

```text
Replace this with your own secure random string
```

Avoid weak passwords like birthdays, phone numbers, or `123456`.

If Cloudflare asks you to redeploy, click **Deploy** again.

---

# 6. Verify Everything Works

If your Worker URL is:

```text
https://ios-location-picker.your-account.workers.dev
```

Visit:

```text
https://ios-location-picker.your-account.workers.dev/health
```

You should see:

```json
{"ok":true,"kv":true,"tokenConfigured":true}
```

Meaning:

| Field | Expected | Meaning |
|--------|----------|---------|
| `ok` | `true` | Worker is running |
| `kv` | `true` | KV is correctly bound |
| `tokenConfigured` | `true` | TOKEN secret is configured |

If `kv` is `false`, verify the KV binding is named `LOC_KV`.

If `tokenConfigured` is `false`, add the `TOKEN` secret.

---

# 7. Optional: Use Your Own Domain

Instead of using a long `workers.dev` URL, you can connect a custom domain.

Example:

```text
myloc.example.com
```

Go to:

```text
Settings → Domains & Routes
```

(or **Triggers**)

Choose:

```text
Add Custom Domain
```

Enter:

```text
myloc.example.com
```

If your DNS is already managed by Cloudflare, HTTPS certificates are configured automatically.

Verify by visiting:

```text
https://myloc.example.com/health
```

---

# 8. Shadowrocket Configuration

Append this to the end of your module's `argument=`:

```text
&configUrl=https://your-domain/loc.json?token=YOUR_TOKEN
```

Or, if you're using the default Worker URL:

```text
&configUrl=https://ios-location-picker.your-account.workers.dev/loc.json?token=YOUR_TOKEN
```

The `[MITM]` section remains unchanged.

---

# 9. Using the Map

A common mistake:

> **Searching for a place does NOT save the location.**

Correct workflow:

1. Open:

```text
https://your-domain/?token=YOUR_TOKEN
```

2. Search for a place.
3. The map moves to that area.
4. Tap the map to place a pin.
5. Tap:

```text
Save Location
```

6. Wait for the "Saved" confirmation.

Then verify:

```text
https://your-domain/loc.json?token=YOUR_TOKEN
```

If the coordinates changed, Cloudflare saved them successfully.

---

# 10. Apply the Spoofed Location on iPhone

After saving a location:

1. Disconnect and reconnect Shadowrocket.
2. Turn **Location Services** off and back on.
3. Open Apple Maps to test.

If it still doesn't work, check:

- The Shadowrocket module is enabled.
- HTTPS decryption (MITM) is enabled.
- The CA certificate is installed and trusted.
- `configUrl` was added correctly.
- `loc.json?token=...` returns the updated coordinates.

---

# 11. Frequently Asked Questions

### The homepage loads without a token. Is that safe?

No. The latest version requires a valid TOKEN to access both the homepage and API.

Without a token:

```text
https://your-domain/
```

Returns:

```json
{"error":"bad token"}
```

Likewise:

```text
https://your-domain/loc.json
```

Returns:

```json
{"error":"bad token"}
```

---

### `/health` works, but Shadowrocket doesn't.

Check:

```text
https://your-domain/loc.json?token=YOUR_TOKEN
```

If it still shows the default coordinates:

```text
37.3349, -122.00902
```

You haven't placed a pin and saved it yet.

---

### Searching doesn't change the location.

Searching only moves the map.

You must:

1. Tap the map.
2. Place a pin.
3. Tap **Save Location**.

---

### Cloudflare still shows "Hello World"

You didn't replace the default `worker.js`, or you deployed the wrong Worker.

Open **Edit Code**, replace everything with the provided `worker.js`, then deploy again.

---

### I get `Unexpected token '<'`

You copied the GitHub webpage HTML instead of the actual JavaScript file.

Copy the raw contents of:

```text
worker.js
```

—not the rendered GitHub page.

---

# 12. Final URLs

Map page:

```text
https://your-domain/?token=YOUR_TOKEN
```

Configuration URL:

```text
https://your-domain/loc.json?token=YOUR_TOKEN
```

Shadowrocket parameter:

```text
&configUrl=https://your-domain/loc.json?token=YOUR_TOKEN
```
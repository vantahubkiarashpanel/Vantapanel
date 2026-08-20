
---

## 🔐 Protocols & Transports

Each inbound (link) has two independent **variants**: `vless` and `trojan`. Either or both can be enabled at the same time. Each enabled variant has its own:

- **Transport:** `ws` (WebSocket) or `xhttp-packet-up` / `xhttp-stream-up` (XHTTP)
- **Fingerprint:** any of the supported uTLS fingerprints
- **ALPN:** one of the 6 fixed combinations listed above

When both VLESS and Trojan are enabled on the same inbound, the subscription page and `/sub/<uid>` output will contain a separate config line for **each** enabled protocol (and for each configured alternative address).

### Routing

Because a single inbound can serve two different wire protocols, the auth type is now part of the URL path so the server knows which parser to use:

- WebSocket: `/ws/{auth}/{uuid}` where `{auth}` is `vless` or `trojan`
- XHTTP downlink: `/xhttp/{auth}/{mode}/{uuid}/{session_id}`
- XHTTP packet-up uplink: `/xhttp/{auth}/packet-up/{uuid}/{session_id}/{seq}`
- XHTTP stream-up uplink: `/xhttp/{auth}/stream-up/{uuid}/{session_id}`

> ⚠️ If you're upgrading from an older version of this panel, previously-issued config links (`/ws/{uuid}` without an auth segment) will stop working. Re-copy/re-scan configs from the panel after upgrading.

---

## 🚀 Deploy on Render

### One-click via `render.yaml`

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/VANTA-PROJECT/VANTA_PANEL)

1. Fork or push this repo to GitHub.
2. Go to [render.com](https://render.com) → **New Web Service** → connect your repo.
3. Render will auto-detect `render.yaml` and configure everything.
4. Set your `ADMIN_PASSWORD` environment variable (default: `admin`).

> 💡 **Tip:** For better speed, set the **Region** to **Frankfurt (EU)** in Render settings.

### Manual Setup

| Field | Value |
|---|---|
| **Environment** | Python |
| **Build Command** | `pip install -r requirements.txt` |
| **Start Command** | `python main.py` |

### 🌐 Render & Cloudflare Clean IPs

> **This panel on Render routes through Cloudflare's clean IPs exclusively.**
>
> Render's infrastructure sits behind Cloudflare's network, so all configs will automatically use **Cloudflare clean IP ranges** — which are generally unblocked and stable in restricted regions.
>
> ✅ Use the panel URL directly — Cloudflare CDN handles routing automatically.
>
> If configs don't connect, try manually adding a known Cloudflare clean IP (e.g. `104.21.x.x` or `172.67.x.x`) from the **Clean IP** page in your client instead of the hostname.

---

## 🚂 Deploy on Railway

### One-click deploy

[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/new/template?template=https://github.com/VANTA-PROJECT/VANTA_PANEL)

1. Fork or push this repo to GitHub.
2. Go to [railway.app](https://railway.app) → **New Project** → **Deploy from GitHub repo** → select your repo.
3. Wait for the deployment to finish. You'll be given a URL — that's your service domain. To access the panel, just add `/login` to the end of your domain.

### ⚠️ Railway IP Addresses

> **Railway does NOT use Cloudflare. It uses its own dedicated IP ranges.**
>
> Railway's outbound IPs typically fall in the range **`69.46.46.x`**, so your configs will use Railway's own IPs — not Cloudflare's. These may or may not be accessible depending on your network restrictions.
>
> **If configs don't work on Railway:**
> 1. Check whether the `69.46.46.x` range is reachable from your network.
> 2. Add your own known-working IPs to `railway_ips.txt` (one per line, next to `main.py`) and click the **🚄 Railway IP** button on the **Clean IP** page to bulk-import them into the panel in one click.
> 3. Enable **Fragment Mode** in your v2ray / v2rayNG client (see section below).
> 4. Switch to Render for Cloudflare clean IP routing.

---

## 🌍 Clean IP Page

The **Clean IP** page lets you manage alternative addresses that get appended to every generated config (in addition to the panel's own domain), so clients can fall back to a working IP if the main hostname is blocked.

- **+ Add** — add a single address manually
- **🚄 Railway IP** — bulk-imports every line from `railway_ips.txt` (placed next to `main.py`) in a single request, skipping duplicates
- **Delete All** — clears the list

> There is no default/pre-filled address anymore — the list starts empty until you add your own.

---

## 🔧 Fragment Mode (v2rayNG / v2ray)

If your configurations are not connecting — especially on Railway — enable **Fragment Mode** in your client:

**v2rayNG (Android):**
1. Go to **Settings → Fragment**
2. Enable Fragment and set: Packets `tlshello`, Length `10-30`, Interval `10-20`
3. Reconnect

**v2ray (Desktop):** Add to your `outbound` → `streamSettings`:

```json
"sockopt": {
  "dialerProxy": "fragment",
  "tcpKeepAliveIdle": 100
}

# MHR-CFW - MasterHttpRelay + Cloudflare Worker

[![GitHub](https://img.shields.io/badge/GitHub-MHR_CFW-blue?logo=github)](https://github.com/denuitt1/mhr-cfw)


| [English](README.md) | [Persian](README_FA.md) |
| :---: | :---: |

## Disclaimer

`mhr-cfw` is provided for educational, testing, and research purposes only.

- **Provided without warranty:** This software is provided "AS IS", without express or implied warranty, including merchantability, fitness for a particular purpose, and non-infringement.
- **Limitation of liability:** The developers and contributors are not responsible for any direct, indirect, incidental, consequential, or other damages resulting from the use of this project or the inability to use it.
- **User responsibility:** Running this project outside controlled test environments may affect networks, accounts, proxies, certificates, or connected systems. You are solely responsible for installation, configuration, and use.
- **Legal compliance:** You are responsible for complying with all local, national, and international laws and regulations before using this software.
- **Google services compliance:** If you use Google Apps Script or other Google services with this project, you are responsible for complying with Google's Terms of Service, acceptable use rules, quotas, and platform policies. Misuse may lead to suspension or termination of your Google account or deployments.
- **License terms:** Use, copying, distribution, and modification of this software are governed by the repository license. Any use outside those terms is prohibited.

---

## How It Works

```
Client -> Local Proxy -> Google/CDN front -> GoogleAppsScript (GAS) Relay -> Cloudflare Worker -> Target website
             |
             +-> shows www.google.com to the network DPI filter
```
In normal use, the browser sends traffic to the proxy running on your computer.
The proxy sends that traffic through Google-facing infrastructure so the network only sees an allowed domain such as `www.google.com`.
Your deployed relay then fetches the real website through cloudflare worker and sends the response back through the same path.

This means the filter sees normal-looking Google traffic, while the actual destination stays hidden inside the relay request.

--- 

## How to Use

### 1 - Download project and extract 

```bash
git clone https://github.com/denuitt1/mhr-cfw.git
cd mhr-cfw
pip install -r requirements.txt
```
> **Can't reach PyPI directly?** Use this mirror instead:
> ```bash
> pip install -r requirements.txt -i https://mirror-pypi.runflare.com/simple/ --trusted-host mirror-pypi.runflare.com
> ```


### 2 - Set Up the Cloudflare Worker (worker.js)

1. Open [Cloudflare Dashboard](https://dash.cloudflare.com/) and sign in with your Cloudflare account.
2. From the sidebar, navigate to **Compute > Workers & Pages**
3. Click **Create Application**, Choose **Start with Hello World** and click on **Deploy**
4. Click on **Edit code** and **Delete** all the default code in the editor.
5. Open the [`worker.js`](script/worker.js) file from this project (under `script/`), **copy everything**, and paste it into the Apps Script editor.
6. **Important:** Change the worker on this line to the worker you created:
   ```javascript
   const WORKER_URL = "myworker.workers.dev";
   ```
7. Click **Deploy**.

### 3 - Set Up the Google Relay (Code.gs)

1. Open [Google Apps Script](https://script.google.com/) and sign in with your Google account.
2. Click **New project**.
3. **Delete** all the default code in the editor.
4. Open the [`Code.gs`](script/Code.gs) file from this project (under `script/`), **copy everything**, and paste it into the Apps Script editor.
5. **Important:** Change the password on this line to something only you know, also replace the worker url with your cloudflare worker:
   ```javascript
   const AUTH_KEY = "your-secret-password-here";
   const WORKER_URL = "https://myworker.workers.dev";
   ```
6. Click **Deploy** → **New deployment**.
7. Choose **Web app** as the type.
8. Set:
   - **Execute as:** Me
   - **Who has access:** Anyone
9. Click **Deploy**.
10. **Copy the Deployment ID** (it looks like a long random string). You'll need it in the next step.

> ⚠️ Remember the password you set in step 3. You'll use the same password in the config file below.

### 4 - Run

Click on the `run.bat` file (on windows) or `run.sh` file (on linux) to start the relay.

If you're running for the first time it will prompt a setup wizard where you have to enter the AUTH_KEY and Google Apps Script Deployment ID.
You should see a message saying the HTTP proxy is running on `127.0.0.1:8085`

### 5 - Usage

We recommend using [v2rayN client](https://github.com/2dust/v2rayn) and configuring a socks5 proxy.

You can also use [FoxyProxy](https://getfoxyproxy.org/)'s [Chrome extension](https://chromewebstore.google.com/detail/foxyproxy/gcknhkkoolaabfmlnjonogaaifnjlfnp?hl=en) or [Firefox extension](https://addons.mozilla.org/en-US/firefox/addon/foxyproxy-standard/) to use this proxy in your browser.

### 6 - Test your connection

Open [ipleak.net](https://ipleak.net) in your browser, you should see your ip address set as cloudflare's.

<img width="1454" height="869" alt="image" src="https://github.com/user-attachments/assets/dfd3316d-69b6-4b0e-b564-fdb055dbdafd" />


---

## Adaptive Google IP Failover

Some networks allow one Google frontend IP while slowing or dropping another.
You can now give the proxy a small pool of Google IPs. New TLS connections
skip an IP for a short cooldown when it times out or fails, then automatically
try the next one.

```json
{
  "google_ip": "216.239.38.120",
  "google_ips": [
    "216.239.38.120",
    "216.239.36.120",
    "142.250.80.142",
    "172.217.14.206"
  ],
  "google_ip_fail_cooldown": 120
}
```

`google_ip` is still used as the first/default route. `google_ips` is the
failover pool used by both the HTTP/1.1 connection pool and the HTTP/2
multiplexed transport.

Open the local control center first:

```text
http://127.0.0.1:8085/
```

It links to the two main local pages:

```text
http://127.0.0.1:8085/status
http://127.0.0.1:8085/dashboard
```

Use `/status` to inspect and control runtime health:

```text
http://127.0.0.1:8085/status
```

The dashboard shows the active Google IP, per-IP failures, cooldowns, H2 state,
Apps Script health, cache hits, top hosts, and a troubleshooting FAQ. It also
has safe local controls for clearing cache, clearing IP cooldowns, clearing
script blacklists, resetting runtime stats, and reconnecting H2.

Use `/dashboard` to view and edit `config.json` from the browser. Secret fields
are masked as `***`; leaving them unchanged preserves the current value. Saved
config changes are written to disk and require a proxy restart to apply.

Automation and scripts can read the raw JSON API instead:

```text
http://127.0.0.1:8085/status.json
```

Dashboard controls are local-only by default. If you intentionally share the
dashboard on your LAN, set:

```json
{
  "dashboard_lan_access": true
}
```

### Apps Script Health Scoring

When multiple `script_ids` are configured, the proxy now tracks per-script
request count, failure count, average latency, cooldowns, and a health score.
The stable per-host script mapping is still preserved for session-sensitive
sites, but fan-out fallback prefers healthier deployments first.

### Troubleshooting Reference

The `/status` page includes this same reference so users can diagnose common
problems without opening the README.

| Problem | What to Check / Fix |
|---|---|
| `Config not found` | Run `python setup.py`, or copy `config.example.json` to `config.json` and fill `script_id` plus `auth_key`. |
| Browser shows certificate errors | Install the MITM CA with `python main.py --install-cert`, then fully restart the browser. Firefox needs importing `ca/ca.crt` inside Firefox certificate settings. |
| Telegram works but browser does not load sites | Usually the CA certificate is missing or not trusted. Install `ca/ca.crt`, then close every browser process and reopen. |
| Installed the cert but Chrome/Edge still errors | Chrome and Edge cache certificate state. Close all browser processes from Task Manager/system tray and reopen. |
| `unauthorized` in logs | `auth_key` in `config.json` must exactly match `AUTH_KEY` in `Code.gs`. Re-deploy Apps Script after changing `Code.gs`. |
| Connection timeout | Run `python main.py --scan`, add several reachable IPs to `google_ips`, then clear IP cooldowns from `/status` or restart. |
| `502 Bad JSON` or invalid Worker response | Check `script_id`, Apps Script quota, whether you created a new deployment after editing `Code.gs`, and whether `WORKER_URL` points to your Worker. |
| Slow browsing | Add multiple `script_ids`, keep H2 working, use several healthy `google_ips`, and watch script health scores in `/status`. |
| SOCKS5 works poorly with Telegram | Use Telegram's HTTP proxy mode at `127.0.0.1:8085`. SOCKS5 clients may connect to raw IPs and send non-HTTP bytes that cannot be relayed. |
| YouTube opens but videos do not play | Try `"youtube_via_relay": true`, or check if the SNI-rewrite route is causing SafeSearch/video restrictions. |
| Cloudflare / CAPTCHA loops | Cloudflare Worker egress IP rotates. Deploy `script/upstream_forwarder.js` on a VPS and configure Worker upstream secrets for stable egress. |
| H2 disconnected | H1 fallback is automatic. Use **Reconnect H2** in `/status`; repeated failures can mean H2 ALPN is blocked on that route. |
| Config saved from dashboard but behavior did not change | The dashboard writes `config.json`, but most settings require restarting the proxy to apply. |
| Dashboard not reachable from phone/LAN | Dashboard access is local-only by default. Set `"dashboard_lan_access": true` only on trusted LANs, then restart. |
| Apps Script quota exceeded | Wait for quota reset, reduce `parallel_relay`, add more `script_ids`, and avoid heavy downloads through Apps Script. |

---

## Optional: Stable Exit IP via Upstream Forwarder

CAPTCHAs (Cloudflare Turnstile/bot challenge, reCAPTCHA, hCaptcha) bind tokens
to the IP that solved the challenge. Cloudflare Workers exit through different
edge IPs per request, so verification on the target site fails even when you
solve the challenge. This optional add-on lets the Worker forward all `fetch()`
calls through a small Node server you run on a VPS with a stable IP — giving
the target site one consistent exit address.

### When you need this

- Sites behind Cloudflare's bot challenge keep looping you back to the challenge page.
- Login forms reject you after solving a reCAPTCHA/hCaptcha.
- You need cookie continuity across requests (e.g. `cf_clearance`).

If you don't hit these, leave it unconfigured — the Worker behaves exactly as before.

### Why a separate server is required

Cloudflare Workers don't expose a stable outbound IP — `fetch()` exits through a rotating pool of Cloudflare edge IPs, which is exactly what breaks IP-bound CAPTCHA tokens. Cloudflare's static-egress options (BYOIP, Egress Workers) are Enterprise-tier, so a small VPS with a static IP is the practical workaround. The forwarder is just a thin proxy that re-issues the `fetch()` from a stable address.

### 1. Deploy the forwarder on a VPS

The reference implementation is [`script/upstream_forwarder.js`](script/upstream_forwarder.js).
It needs Node 18+ and no dependencies. Run it behind Caddy or nginx with TLS —
the Worker rejects non-HTTPS forwarder URLs.

```bash
# On your VPS (Ubuntu/Debian example):
sudo apt install -y nodejs   # must be 18+
export AUTH_KEY="some-long-random-string-at-least-32-chars"
export PORT=8787
node script/upstream_forwarder.js
```

Front it with Caddy for auto-TLS:

```
forwarder.example.com {
    reverse_proxy 127.0.0.1:8787
}
```

Quick smoke test:

```bash
curl -X POST https://forwarder.example.com/fwd \
  -H "x-upstream-auth: $AUTH_KEY" \
  -H "content-type: application/json" \
  -d '{"u":"https://httpbin.org/ip","m":"GET","h":{}}'
```

The decoded response body should show the **VPS's IP**.

### 2. Wire the Worker to the forwarder

In the Cloudflare dashboard → your Worker → **Settings → Variables and Secrets**:

| Name | Type | Value |
|---|---|---|
| `UPSTREAM_FORWARDER_URL` | Secret | `https://forwarder.example.com/fwd` |
| `UPSTREAM_AUTH_KEY` | Secret | the same `AUTH_KEY` you set on the VPS |
| `UPSTREAM_FAIL_MODE` | Variable | `closed` (default) — return 502 on forwarder failure. Use `open` to fall back to direct fetch. |
| `UPSTREAM_TIMEOUT_MS` | Variable (optional) | default `25000` |

Save and redeploy the Worker.

### 3. Verify

Browse `https://httpbin.org/ip` through the proxy — you should see the **VPS's IP**, not Cloudflare's. Then revisit a CAPTCHA-protected site that wasn't working — the challenge should now validate.

> The forwarder must require auth. Without `AUTH_KEY` it refuses to start. Anyone with the URL and key can use it as a relay, so keep both secret.


---

## Sources for this project
- https://github.com/masterking32/MasterHttpRelayVPN

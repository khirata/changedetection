# 🕵️‍♂️ Secure Self-Hosted Page Watcher (2026 Edition)

A private website monitoring stack for Raspberry Pi/Docker. This setup handles dynamic (JavaScript) websites, sends alerts to Discord, and is securely accessible via Cloudflare Zero Trust.

## 🚀 Features
- **Changedetection.io**: Core logic and change tracking.
- **SockpuppetBrowser**: Optimized Playwright engine for ARM/RPI hardware.
- **Discord Integration**: Instant alerts via Discord webhooks.
- **Cloudflare Tunnel**: No port forwarding required.
- **Zero Trust Security**: Identity-based login wall (Google/Email).

---

## 🛠 Prerequisites
- **Docker & Docker Compose** installed.
- **Cloudflare Account** with a domain connected.
- **Discord Webhook** (Server Settings > Integrations > Webhooks).

---

## 📂 1. Configuration (`docker-compose.yml`)
Replace `<YOUR_TUNNEL_TOKEN>` with your token from the Cloudflare Zero Trust Dashboard.

```yaml
services:
  # Logic & UI
  changedetection:
    image: ghcr.io/dgtlmoon/changedetection.io:latest
    container_name: changedetection
    restart: unless-stopped
    environment:
      - PLAYWRIGHT_DRIVER_URL=ws://playwright-chrome:3000
    volumes:
      - ./datastore:/datastore
    ports:
      - 5000:5000
    depends_on:
      - playwright-chrome

  # Playwright Browser Engine (ARM optimized)
  playwright-chrome:
    image: dgtlmoon/sockpuppetbrowser:latest
    container_name: playwright-chrome
    restart: unless-stopped
    cap_add:
      - SYS_ADMIN
    environment:
      - MAX_CONCURRENT_CHROME_PROCESSES=5

  # Secure Remote Access
  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    restart: unless-stopped
    command: tunnel --no-autoupdate run --token <YOUR_TUNNEL_TOKEN>
```

---

## 🔔 2. Setup Discord Notifications

1. Copy your Discord Webhook URL.
2. Open `changedetection.io` > **Settings** > **Notifications**.
3. Paste the URL but change `https://` to `discord://`.
* *Example:* `discord://12345/abc-xyz`


4. Click **Send test notification** to verify.

---

## ☁️ 3. Cloudflare Zero Trust Setup

1. **Network Tunnel:** Point your subdomain (e.g., `watches.yourdomain.com`) to `http://changedetection:5000`.
2. **Access Policy:** Create a "Self-hosted" application for that subdomain.
3. **Identity:** Add an "Allow" policy for your specific email address. If using Google, ensure you use the **Social** connector, not Workspace, to allow personal Gmail accounts.

---

## ⚙️ 4. Activation

1. Start the stack: `docker compose up -d`
2. In the UI, go to **Settings > Fetching**.
3. Set **Fetch Backend** to `Playwright Chromium/Javascript`.
4. **Important:** For complex sites, set "Wait seconds before extracting text" to `5`.

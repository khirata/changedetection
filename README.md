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

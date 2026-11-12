# Mitmproxy Homelab Configuration and Retention Report

## 1. Overview

This document records the containerized `mitmproxy` deployment, persistent flow storage, historical-flow loading, and automated 24-hour retention policy. Secret values and private site filters are intentionally supplied at deployment time rather than stored in this public report.

## 2. Docker Compose deployment

Save the following as `compose.yml`:

```yaml
version: '3.8'

services:
  mitmproxy:
    image: mitmproxy/mitmproxy:latest
    container_name: mitmproxy
    restart: unless-stopped
    dns:
      - 1.1.1.1
      - 8.8.8.8
    ports:
      - "4000:8080"
      - "4001:8081"
    volumes:
      - mitmproxy_certs:/home/mitmproxy/.mitmproxy
    command: >
      mitmweb --web-host 0.0.0.0
      --mode regular
      -s /home/mitmproxy/.mitmproxy/load_flows.py
      -w /home/mitmproxy/.mitmproxy/traffic.flow
      --set web_password=${MITMWEB_PASSWORD:?Set MITMWEB_PASSWORD in .env}
      --set view_filter="${MITM_VIEW_FILTER:?Set MITM_VIEW_FILTER in .env}"

volumes:
  mitmproxy_certs:
    name: mitmproxy_certs
```

The deployment exposes the proxy on port `4000` and the web dashboard on port `4001`. The named volume preserves certificates, the addon script, configuration, and `traffic.flow` across container recreation.

## 3. Private deployment variables

Create a local `.env` file beside `compose.yml`. Do not commit it or paste its values into public documentation:

```dotenv
MITMWEB_PASSWORD=<long-random-dashboard-password>
MITM_VIEW_FILTER=<private-mitmproxy-filter>
```

Use a long, unique dashboard password. Set `MITM_VIEW_FILTER` to the private URL/domain filter needed for the homelab; keeping it variable prevents monitored domains from being disclosed in this report. Restrict the `.env` file permissions and add it to `.gitignore`.

Start the service with:

```bash
docker compose up -d
docker compose ps
docker compose logs -f mitmproxy
```

Open the dashboard at `http://localhost:4001` and use the locally configured password.

## 4. Key technical challenges and solutions

### A. Binary `.flow` persistence and dashboard visibility

Mitmproxy stores active traffic in memory. The `-w` option writes traffic to the persistent binary `traffic.flow` file, while the custom `load_flows.py` addon, loaded with `-s`, reads historical flows with `mitmproxy.io.FlowReader` and injects them into the active state when the service starts.

### B. Automated data retention and storage management

Unchecked logging can grow `traffic.flow` indefinitely and leave stale records in the dashboard. The addon applies a 24-hour sliding retention window:

- On startup and hourly, it parses `traffic.flow`, removes entries older than 86,400 seconds, and rewrites the file.
- An hourly asynchronous task removes expired flows from in-memory state.

## 5. Configuration artifacts

- **Compose service:** Runs `mitmweb` in regular proxy mode with ports `4000` and `4001`.
- **Addon script:** `/home/mitmproxy/.mitmproxy/load_flows.py` loads historical flows and performs retention cleanup.
- **Flow log:** `/home/mitmproxy/.mitmproxy/traffic.flow` stores captured flows on the named volume.
- **Credentials and filters:** Supplied through local environment variables only.

## 6. Security notes

- Never commit `.env`, dashboard passwords, captured `.flow` files, or private domain filters.
- Replace the example placeholders before starting the stack; Compose will refuse to start if either required variable is missing.
- Avoid exposing port `4001` directly to the internet. Use a trusted LAN, VPN, or authenticated reverse proxy.
- Treat captured flows as sensitive because they may contain URLs, headers, cookies, request bodies, or other identifying data.

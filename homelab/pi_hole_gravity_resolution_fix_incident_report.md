# Incident Report: Pi-hole Docker Gravity Update Failure & DNS Loop Resolution

**Date:** October 29, 2026  
**System:** Homelab Docker Container (`pihole`)  
**Access Context:** Remote Management via Tailscale VPN  
**Status:** **Resolved**  

---

## 1. Issue Overview
During the addition of new blocklists to a Pi-hole instance running inside a Docker container on an Ubuntu homelab server, the newly added blocklists remained stuck on status **`Unknown`**. Executing the gravity update command (`pihole -g`) stalled indefinitely without fetching any updates.

Attempts to resolve the issue by temporarily changing the router's DNS settings back to standard configurations did not alter the container's behavior.

---

## 2. Root Cause Analysis (RCA)

Executing diagnostic commands inside the container revealed the core failure mechanism:

```bash
adminbry@homelab:~$ docker exec -it pihole curl -I -L "https://media.githubusercontent.com/media/zachlagden/Pi-hole-Optimized-Blocklists/main/lists/nsfw_abp.txt"
curl: (6) Could not resolve host: media.githubusercontent.com (DNS server returned general failure)
```

### Key Findings:
1. **Container DNS Self-Dependency Loop:** Pi-hole was configured to handle its own host/container DNS resolution. When Pi-hole's filtering or gravity state became unstable, the container lost the ability to resolve external domain names (e.g., `media.githubusercontent.com`).
2. **Network Conflict in Docker Compose:** The container's Compose definition attempted to combine `network_mode: host` with explicit container-level `dns:` declarations. In Docker, `network_mode: host` forces the container to share the host system's network stack directly, ignoring Docker bridge-level `dns:` keys.
3. **Upstream Unreachable:** Because the underlying container was unable to resolve external domains through the host's DNS settings, `pihole -g` could not initiate outbound HTTP/HTTPS requests to download updated blocklists.

---

## 3. Remediation & Configuration Steps

### Solution Option A: Host Network Mode (Recommended for Pi-hole)
When using `network_mode: host`, Pi-hole relies directly on the host operating system's `/etc/resolv.conf`.

1. **Remove `dns:` keys from `docker-compose.yml`** to avoid networking conflicts:
   ```yaml
   version: '3.8'

   services:
     pihole:
       container_name: pihole
       image: pihole/pihole:latest
       restart: unless-stopped
       network_mode: host
       cap_add:
         - NET_ADMIN
         - SYS_TIME
       environment:
         TZ: 'Asia/Manila'
         WEBPASSWORD: 'your_secure_password'
       volumes:
         - ./etc-pihole:/etc/pihole
         - ./etc-dnsmasq.d:/etc/dnsmasq.d
   ```

2. **Verify Host DNS Resolution:** Ensure the Ubuntu host server (`/etc/resolv.conf` or `systemd-resolved`) includes an external fallback DNS provider (e.g., `1.1.1.1` or `8.8.8.8`).

---

### Solution Option B: Bridge Network Mode with Port Bindings
If explicitly setting fallback DNS servers inside Docker is preferred, switch from `network_mode: host` to Docker's default bridge network mode:

```yaml
version: '3.8'

services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    restart: unless-stopped
    dns:
      - 1.1.1.1
      - 8.8.8.8
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "80:80/tcp"
      - "443:443/tcp"
    cap_add:
      - NET_ADMIN
      - SYS_TIME
    environment:
      TZ: 'Asia/Manila'
      WEBPASSWORD: 'your_secure_password'
    volumes:
      - ./etc-pihole:/etc/pihole
      - ./etc-dnsmasq.d:/etc/dnsmasq.d
```

---

## 4. Verification & Testing

1. Re-deploy the updated stack:
   ```bash
   docker compose down && docker compose up -d
   ```

2. Verify domain resolution directly within the container:
   ```bash
   docker exec -it pihole curl -I -L "https://media.githubusercontent.com/media/zachlagden/Pi-hole-Optimized-Blocklists/main/lists/nsfw_abp.txt"
   ```
   *Expected Output:* `HTTP/2 200` status header returned successfully.

3. Force a gravity rebuild:
   ```bash
   docker exec -it pihole pihole -g -f
   ```
   *Result:* All blocklists successfully transition from `Unknown` to `Active` with updated domain counts.

---

## 5. Lessons Learned & Prevention
* Always separate the container's outbound management DNS from the DNS service Pi-hole serves to the network.
* Do not combine `dns:` parameters with `network_mode: host` in `docker-compose.yml`.
* Ensure host machines running DNS appliances maintain secondary public upstream resolvers in `/etc/resolv.conf`.
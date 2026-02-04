# ⚡ QUICKSTART — Media Automation Stack

> Use this if you want to get running fast.  
> For architecture, guarantees, and troubleshooting, see **README.md**.

---

## 1️⃣ Prerequisites

- Windows 10
- Docker Desktop (WSL2 backend)
- Ubuntu 24.04 in WSL
- NordVPN account with **service credentials**

---

## 2️⃣ Create Folder Structure

```bash
mkdir -p ~/media-stack/{config,scripts}
```

---

## 3️⃣ Create `.env`

```bash
cat > ~/media-stack/.env <<'EOF'
NORDVPN_USER=your_nord_service_username
NORDVPN_PASS=your_nord_service_password
EOF
```

---

## 4️⃣ docker-compose.yml

Create `~/media-stack/docker-compose.yml`:

```yaml
name: media-stack

services:
  gluetun:
    image: qmcgaw/gluetun:latest
    container_name: gluetun
    cap_add: [NET_ADMIN]
    devices:
      - /dev/net/tun:/dev/net/tun
    environment:
      - TZ=America/New_York
      - VPN_SERVICE_PROVIDER=nordvpn
      - VPN_TYPE=openvpn
      - OPENVPN_USER=${NORDVPN_USER}
      - OPENVPN_PASSWORD=${NORDVPN_PASS}
      - SERVER_COUNTRIES=United States
      - FIREWALL_OUTBOUND_SUBNETS=192.168.0.0/16,10.0.0.0/8
    volumes:
      - ./config/gluetun:/gluetun
    ports:
      - "8080:8080"
      - "6789:6789"
      - "51413:51413/tcp"
      - "51413:51413/udp"
      - "8000:8000"
    restart: unless-stopped

  qbittorrent:
    image: linuxserver/qbittorrent:latest
    network_mode: service:gluetun
    depends_on:
      gluetun:
        condition: service_healthy
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/New_York
      - WEBUI_PORT=8080
    volumes:
      - ./config/qbittorrent:/config
      - /mnt/f/Media:/media
    restart: unless-stopped

  nzbget:
    image: linuxserver/nzbget:latest
    network_mode: service:gluetun
    depends_on:
      gluetun:
        condition: service_healthy
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/New_York
    volumes:
      - ./config/nzbget:/config
      - /mnt/f/Media:/media
    restart: unless-stopped

  prowlarr:
    image: linuxserver/prowlarr:latest
    volumes:
      - ./config/prowlarr:/config
      - /mnt/f/Media:/media
    ports: ["9696:9696"]
    restart: unless-stopped

  sonarr:
    image: linuxserver/sonarr:latest
    volumes:
      - ./config/sonarr:/config
      - /mnt/f/Media:/media
    ports: ["8989:8989"]
    restart: unless-stopped

  radarr:
    image: linuxserver/radarr:latest
    volumes:
      - ./config/radarr:/config
      - /mnt/f/Media:/media
    ports: ["7878:7878"]
    restart: unless-stopped

  uptime-kuma:
    image: louislam/uptime-kuma:1
    volumes:
      - ./config/uptime-kuma:/app/data
    ports: ["3001:3001"]
    restart: unless-stopped
```

---

## 5️⃣ Start Stack

```bash
cd ~/media-stack
docker compose up -d
```

---

## 6️⃣ Rotate Script (Optional but Recommended)

```bash
cat > ~/media-stack/scripts/rotate_vpn.sh <<'EOF'
#!/usr/bin/env bash
set -euo pipefail
cd "$HOME/media-stack"
docker compose restart gluetun
docker compose restart qbittorrent nzbget
EOF

chmod +x ~/media-stack/scripts/rotate_vpn.sh
```

---

## 7️⃣ Web UIs

| Service | URL |
|------|----|
| qBittorrent | http://localhost:8080 |
| NZBGet | http://localhost:6789 |
| Sonarr | http://localhost:8989 |
| Radarr | http://localhost:7878 |
| Prowlarr | http://localhost:9696 |
| Uptime Kuma | http://localhost:3001 |

---

## 🎉 Done

Your stack is live, VPN-protected, and recoverable.

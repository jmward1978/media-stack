# 🧭 WALKTHROUGH.md — Media Automation Stack (Full Step‑by‑Step)

> This is the **complete walkthrough** you asked for: setup → configure UIs → verify → monitor → maintenance.
> It’s written for “follow it exactly” operation.
>
> You already have:
> - `README.md` (reference)
> - `QUICKSTART.md` (fast path)
> - `DISASTER_RECOVERY.md` (worst case)
>
> This file is the **guided tour** in between.

---

## 0) What you need (checklist)

### Required
- Windows 10
- Docker Desktop installed (WSL2 backend)
- WSL2 Ubuntu 24.04 installed
- NordVPN **service credentials** (or other Gluetun-supported provider)
- Media folder exists on Windows: `F:\Media`

### You should already have
- WSL sees your media drive at: `/mnt/f/Media`

Verify in WSL:
```bash
ls /mnt/f/Media
```

---

## 1) Ensure Docker Desktop + WSL are ready

### 1.1 Start Docker Desktop
- Open Docker Desktop in Windows
- Settings → enable: **Start Docker Desktop when you log in**

### 1.2 Verify docker works from WSL
In WSL:
```bash
docker version
docker compose version
```

If those work, continue.

---

## 2) Create stack directory structure (WSL)

```bash
mkdir -p ~/media-stack/{config,scripts}
cd ~/media-stack
```

Sanity check:
```bash
pwd
ls -la
```

---

## 3) Create `.env` with VPN credentials

> ⚠️ Use **NordVPN service credentials**, not your normal login/password.

Create the file:
```bash
cat > ~/media-stack/.env <<'EOF'
NORDVPN_USER=your_service_user
NORDVPN_PASS=your_service_pass
EOF
```

(Cursor blinking is normal. The command ends after you type `EOF` and press Enter.)

Verify:
```bash
sed -n '1,5p' ~/media-stack/.env
```

---

## 4) Create `docker-compose.yml` (authoritative)

### 4.1 Create/edit the file
```bash
nano ~/media-stack/docker-compose.yml
```

Paste **the full compose** from this repo/package:
- Save: **Ctrl + O** → Enter
- Exit: **Ctrl + X**

### 4.2 Compose file you should have

```yaml
name: media-stack

services:
  gluetun:
    image: qmcgaw/gluetun:latest
    container_name: gluetun
    cap_add:
      - NET_ADMIN
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
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://127.0.0.1:9999/health"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 20s

  qbittorrent:
    image: linuxserver/qbittorrent:latest
    container_name: qbittorrent
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
    container_name: nzbget
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
    container_name: prowlarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/New_York
    volumes:
      - ./config/prowlarr:/config
      - /mnt/f/Media:/media
    ports:
      - "9696:9696"
    restart: unless-stopped

  sonarr:
    image: linuxserver/sonarr:latest
    container_name: sonarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/New_York
    volumes:
      - ./config/sonarr:/config
      - /mnt/f/Media:/media
    ports:
      - "8989:8989"
    restart: unless-stopped

  radarr:
    image: linuxserver/radarr:latest
    container_name: radarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/New_York
    volumes:
      - ./config/radarr:/config
      - /mnt/f/Media:/media
    ports:
      - "7878:7878"
    restart: unless-stopped

  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    volumes:
      - ./config/uptime-kuma:/app/data
    ports:
      - "3001:3001"
    restart: unless-stopped
```

---

## 5) Prepare your media folders (Windows/WSL)

You mount `/mnt/f/Media` as `/media` inside containers.

### 5.1 Ensure torrent folders exist on Windows
You said you have:
- `F:\Media\Torrents\Incomplete`
- `F:\Media\Torrents\Complete`

We also want category folders under Complete:
- `F:\Media\Torrents\Complete\tv-sonarr`
- `F:\Media\Torrents\Complete\movies-radarr`

Create them in WSL (safe):
```bash
mkdir -p /mnt/f/Media/Torrents/Complete/tv-sonarr
mkdir -p /mnt/f/Media/Torrents/Complete/movies-radarr
```

Verify:
```bash
ls -la /mnt/f/Media/Torrents/Complete
```

---

## 6) Start the stack

From WSL:
```bash
cd ~/media-stack
docker compose up -d
```

Then check status:
```bash
docker compose ps
```

### Expected
- `gluetun` shows **healthy**
- others show **Up**

---

## 7) Verify VPN is working (Gluetun)

### 7.1 View gluetun logs
```bash
docker logs -f gluetun
```

Look for:
- “Initialization Sequence Completed”
- “Public IP address is …”
- No AUTH_FAILED errors

Press **Ctrl + C** to stop following logs.

### 7.2 Confirm VPN IP from inside gluetun
```bash
docker exec -it gluetun wget -qO- https://ipinfo.io/ip && echo
```

This should show a public IP (typically not your ISP IP).

---

## 8) Open the Web UIs (and log in)

Open in your Windows browser:

- qBittorrent: http://localhost:8080
- NZBGet: http://localhost:6789
- Sonarr: http://localhost:8989
- Radarr: http://localhost:7878
- Prowlarr: http://localhost:9696
- Uptime Kuma: http://localhost:3001

---

## 9) Configure qBittorrent (behind VPN)

### 9.1 Log in
qBittorrent will prompt for credentials.
Use the default credentials from the qB container output/logs if needed:

Check qB logs:
```bash
docker logs qbittorrent | tail -n 80
```

(If you already logged in, skip.)

### 9.2 Set download paths
In qBittorrent Web UI:
- Settings → Downloads
- Default Save Path:
  - `/media/Torrents/Complete`
- Incomplete/TMP path (if enabled):
  - `/media/Torrents/Incomplete`

(If you already set these, skip.)

---

## 10) Configure NZBGet

In NZBGet Web UI:
- Settings → PATHS
  - MainDir (or equivalent): `/media/Torrents/Complete`
  - TempDir: `/media/Torrents/Incomplete`

### Categories (critical)
Create categories so Sonarr/Radarr can route downloads:

- Category: `tv-sonarr`
  - DestDir: `/media/Torrents/Complete/tv-sonarr`
- Category: `movies-radarr`
  - DestDir: `/media/Torrents/Complete/movies-radarr`

(Names must match what you use in Sonarr/Radarr.)

---

## 11) Configure Prowlarr → Sonarr/Radarr sync

### 11.1 Add Sonarr application in Prowlarr
In Prowlarr:
- Settings → Apps → Add App → Sonarr
- URL: `http://sonarr:8989`
- API Key: from Sonarr (Settings → General)
- Test → Save

### 11.2 Add Radarr application in Prowlarr
- Settings → Apps → Add App → Radarr
- URL: `http://radarr:7878`
- API Key: from Radarr (Settings → General)
- Test → Save

### 11.3 Why not localhost?
Inside Docker, `localhost` means “this container,” not another service.
That’s why you use `http://sonarr:8989` etc.

---

## 12) Configure Sonarr

### 12.1 Root folders
Settings → Media Management → Root Folders:
- Add your TV libraries under `/media/...` (mapped from `F:\Media\...`)

Example root folder:
- `/media/Tv`
- `/media/Tv-becca`
(Use the folders you have.)

### 12.2 Download clients
Settings → Download Clients

#### qBittorrent
- Host: `gluetun`
- Port: `8080`
- Username/password: your qB credentials
- Category: `tv-sonarr`

#### NZBGet
- Host: `gluetun`
- Port: `6789`
- Username/password: your NZBGet creds
- Category: `tv-sonarr`

> **Important:** For download clients behind Gluetun, Sonarr should talk to `gluetun` since that’s where the ports are published.

Test each → Save.

---

## 13) Configure Radarr

Settings → Download Clients

#### qBittorrent
- Host: `gluetun`
- Port: `8080`
- Category: `movies-radarr`

#### NZBGet
- Host: `gluetun`
- Port: `6789`
- Category: `movies-radarr`

Test → Save.

### Root folders
Settings → Media Management → Root Folders:
- Set movie roots under `/media/...` (mapped from `F:\Media\Movies`, etc.)

---

## 14) Optional: Migrate existing Sonarr/Radarr/Prowlarr configs

> ✅ SKIP THIS if you’re doing a clean install.
> Do this only if you already have Windows installs and want to bring them into containers.

### 14.1 Stop the Windows services first
In Windows Services:
- Stop Sonarr
- Stop Radarr
- Stop Prowlarr

### 14.2 Copy ProgramData into the new config folders (WSL)
```bash
rsync -a --info=progress2 "/mnt/c/ProgramData/Sonarr/" "$HOME/media-stack/config/sonarr/"
rsync -a --info=progress2 "/mnt/c/ProgramData/Radarr/" "$HOME/media-stack/config/radarr/"
rsync -a --info=progress2 "/mnt/c/ProgramData/Prowlarr/" "$HOME/media-stack/config/prowlarr/"
```

### 14.3 Restart containers
```bash
cd ~/media-stack
docker compose up -d
```

If anything looks weird, do a clean restart:
```bash
docker compose down && docker compose up -d
```

---

## 15) Set up Uptime Kuma (monitoring)

Open:
- http://localhost:3001

Create an admin user.

### 15.1 Add monitors (HTTP)
Use these URLs:

- Sonarr: `http://host.docker.internal:8989`
- Radarr: `http://host.docker.internal:7878`
- Prowlarr: `http://host.docker.internal:9696`
- qBittorrent: `http://host.docker.internal:8080`
- NZBGet: `http://host.docker.internal:6789`

> qB + NZBGet act as **VPN canaries** since they live behind Gluetun.

### 15.2 Maintenance window (recommended)
Add a daily maintenance window for your rotation time (example):
- 04:10 → 04:25
Apply to: qBittorrent, NZBGet (and optionally Prowlarr)

This prevents false alerts during VPN rotation.

---

## 16) VPN rotation automation (cron + one-command)

### 16.1 Create rotate script
```bash
cat > ~/media-stack/scripts/rotate_vpn.sh <<'EOF'
#!/usr/bin/env bash
set -euo pipefail

cd "$HOME/media-stack"

echo "[$(date)] Restarting VPN + downloaders..."
docker compose restart gluetun
docker compose restart qbittorrent nzbget
echo "[$(date)] Rotation complete."
EOF

chmod +x ~/media-stack/scripts/rotate_vpn.sh
```

### 16.2 Add alias (optional)
Add to `~/.bashrc`:
```bash
echo "alias rotatevpn='cd ~/media-stack && ./scripts/rotate_vpn.sh'" >> ~/.bashrc
source ~/.bashrc
```

Now you can run:
```bash
rotatevpn
```

### 16.3 Schedule daily rotation via cron
```bash
(crontab -l 2>/dev/null; echo "15 4 * * * /home/$USER/media-stack/scripts/rotate_vpn.sh >> /home/$USER/media-stack/config/rotate_vpn.log 2>&1") | crontab -
```

Confirm:
```bash
crontab -l
tail -n 50 ~/media-stack/config/rotate_vpn.log
```

---

## 17) Restarting everything (clean way)

Use this as your “fix it” command:
```bash
cd ~/media-stack
docker compose down
docker compose up -d
```

---

## 18) Alternate VPN providers (quick swap)

You can change providers by editing only the `gluetun` environment.

Example (Mullvad, WireGuard) — conceptually:
- `VPN_SERVICE_PROVIDER=mullvad`
- `VPN_TYPE=wireguard`
- `WIREGUARD_PRIVATE_KEY=...`

Refer to Gluetun wiki for exact variables:
https://github.com/qdm12/gluetun/wiki

---

## 19) Jackett as an alternative to Prowlarr

Use **one**, not both.

If using Jackett:
- Remove `prowlarr` service
- Add `jackett` service
- In Sonarr/Radarr, add Torznab indexers using Jackett URL:
  - `http://jackett:9117/torznab/all`

---

## 20) Gist setup (optional, for backup)

Recommended: create **one Gist with multiple files**:
- `docker-compose.yml`
- `README.md`
- `QUICKSTART.md`
- `WALKTHROUGH.md`
- `DISASTER_RECOVERY.md`

Steps:
1. Go to https://gist.github.com/
2. Create each file, paste contents
3. Set gist to Secret (recommended)
4. Create gist

---

## ✅ You’re done

If all UIs load and qB/NZB are green in Kuma, your stack is:
- leak-proof
- recoverable
- automated
- monitorable

If anything blows up, use `DISASTER_RECOVERY.md`.

# 🧭 WALKTHROUGH.md — Full Step-by-Step

> 📘 **Back to overview:** [README.md](./README.md)

This is the complete guided setup: compose → folders → UI config → verification → monitoring → maintenance.

---

## 0) Prereqs

- Windows 10
- Docker Desktop (WSL2 backend)
- WSL2 Ubuntu 24.04
- Media folder on Windows (example): `F:\Media`
- In WSL: `/mnt/f/Media`

Verify:
```bash
ls /mnt/f/Media
docker version
docker compose version
```

---

## 1) Create stack folder

```bash
mkdir -p ~/media-stack/{config,scripts}
cd ~/media-stack
```

---

## 2) Create `.env` (secrets stay out of git)

```bash
cat > ~/media-stack/.env <<'EOF'
NORDVPN_USER=your_service_user
NORDVPN_PASS=your_service_pass
EOF
```

---

## 3) Create `docker-compose.yml`

```bash
nano ~/media-stack/docker-compose.yml
```

Paste from repo and save.

Nano:
- Save: Ctrl+O → Enter
- Exit: Ctrl+X
- Exit without saving: Ctrl+X → N

---

## 4) Start stack

```bash
cd ~/media-stack
docker compose up -d
docker compose ps
```

---

## 5) Verify VPN

Follow logs:
```bash
docker logs -f gluetun
```
Ctrl+C to stop.

Check IP from inside VPN namespace:
```bash
docker exec -it gluetun wget -qO- https://ipinfo.io/ip && echo
```

---

## 6) Folder layout (inside containers)

Your host path `/mnt/f/Media` is mounted as `/media` inside containers.

Create required category folders:
```bash
mkdir -p /mnt/f/Media/Torrents/Complete/tv-sonarr
mkdir -p /mnt/f/Media/Torrents/Complete/movies-radarr
```

---

## 7) Configure qBittorrent

Open http://localhost:8080

Set Downloads:
- Incomplete: `/media/Torrents/Incomplete`
- Complete: `/media/Torrents/Complete`

Set categories (used by Sonarr/Radarr):
- Sonarr: `tv-sonarr`
- Radarr: `movies-radarr`

---

## 8) Configure NZBGet

Open http://localhost:6789

Paths:
- Temp: `/media/Torrents/Incomplete`
- Dest: `/media/Torrents/Complete`

Categories:
- `tv-sonarr` → `/media/Torrents/Complete/tv-sonarr`
- `movies-radarr` → `/media/Torrents/Complete/movies-radarr`

---

## 9) Configure Prowlarr + FlareSolverr

Open http://localhost:9696

If adding indexers returns `Connection refused (localhost:8191)`, FlareSolverr is missing.
This repo includes FlareSolverr in compose.

In Prowlarr:
- Settings → Indexers → (enable Advanced)
- FlareSolverr URL: `http://flaresolverr:8191`

---

## 10) Configure Sonarr/Radarr

Sonarr: http://localhost:8989  
Radarr: http://localhost:7878

Root folders must be **/media/...** paths.

Download Clients (both apps):
- qBittorrent:
  - Host: `gluetun`
  - Port: `8080`
- NZBGet:
  - Host: `gluetun`
  - Port: `6789`

Categories:
- Sonarr: `tv-sonarr`
- Radarr: `movies-radarr`

---

## 11) Monitoring (Uptime Kuma)

Open http://localhost:3001

Add HTTP monitors:
- Sonarr: `http://host.docker.internal:8989`
- Radarr: `http://host.docker.internal:7878`
- Prowlarr: `http://host.docker.internal:9696`
- qB: `http://host.docker.internal:8080`
- NZBGet: `http://host.docker.internal:6789`

---

## 12) Maintenance: VPN rotation

Create script:
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

Alias:
```bash
echo "alias rotatevpn='cd ~/media-stack && ./scripts/rotate_vpn.sh'" >> ~/.bashrc
source ~/.bashrc
```

Run anytime:
```bash
rotatevpn
```

Cron (daily 04:15):
```bash
(crontab -l 2>/dev/null; echo "15 4 * * * /home/$USER/media-stack/scripts/rotate_vpn.sh >> /home/$USER/media-stack/config/rotate_vpn.log 2>&1") | crontab -
```

---

## 13) Restart everything (clean)

```bash
cd ~/media-stack
docker compose down
docker compose up -d
```

---

## 14) Optional: migrate existing Windows ProgramData (skip for clean install)

Stop Windows services first, then copy:
```bash
rsync -a --info=progress2 "/mnt/c/ProgramData/Sonarr/" "$HOME/media-stack/config/sonarr/"
rsync -a --info=progress2 "/mnt/c/ProgramData/Radarr/" "$HOME/media-stack/config/radarr/"
rsync -a --info=progress2 "/mnt/c/ProgramData/Prowlarr/" "$HOME/media-stack/config/prowlarr/"
```

Restart containers:
```bash
cd ~/media-stack
docker compose up -d
```

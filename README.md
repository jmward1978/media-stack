# 📦 Media Automation Stack (Docker + WSL + VPN-Safe)

> **Windows 10 + WSL2 + Docker Desktop**  
> Hardened media automation stack with VPN kill switch, monitoring, and automatic recovery.

This README is the **authoritative reference** for the stack: architecture, guarantees, maintenance, and troubleshooting.
If you want to get running fast, see **QUICKSTART.md**.

---

## 🚀 What This Is

A production-grade media automation stack featuring:

- 🔒 Hard VPN kill switch (Gluetun + NordVPN)
- 🔁 Automatic VPN rotation (cron + safe restarts)
- ⛔ Health-gated startup (no leaks on boot or reconnect)
- 📊 Monitoring with Uptime Kuma
- 🎬 Sonarr / Radarr / Prowlarr
- ⬇️ qBittorrent + NZBGet
- 🧠 No DB hacks, no ignored warnings

---

## 🧱 Architecture Overview

```
Windows 10
└── Docker Desktop
    └── WSL2 (Ubuntu 24.04)
        └── Docker Compose
            ├── gluetun (NordVPN, firewall, kill switch)
            │   ├── qBittorrent (network_mode: service:gluetun)
            │   └── NZBGet      (network_mode: service:gluetun)
            ├── Sonarr
            ├── Radarr
            ├── Prowlarr
            └── Uptime Kuma
```

### Design Guarantees
- VPN down → downloaders are offline
- Downloaders never start unless VPN is healthy
- Windows reboot → stack restores automatically
- VPN rotation → downloaders recover cleanly

---

## 📁 Host Layout

```
~/media-stack/
├── docker-compose.yml
├── .env
├── scripts/
│   └── rotate_vpn.sh
└── config/
    ├── gluetun/
    ├── qbittorrent/
    ├── nzbget/
    ├── sonarr/
    ├── radarr/
    ├── prowlarr/
    └── uptime-kuma/
```

Media storage (Windows):

```
F:\Media
├── Movies
├── TV
├── Torrents
│   ├── Incomplete
│   └── Complete
│       ├── movies-radarr
│       └── tv-sonarr
```

Mounted inside containers as `/media`.

---

## 🔐 Environment Variables

Create `.env`:

```env
NORDVPN_USER=your_nord_service_username
NORDVPN_PASS=your_nord_service_password
```

⚠️ Use **NordVPN service credentials**, not your account email/password.

---

## 🐳 Full docker-compose.yml

(Identical to QUICKSTART — kept here for reference)

```yaml
<SEE QUICKSTART.md>
```

---

## 🔁 VPN Rotation

Manual:
```bash
rotatevpn
```

Scheduled (cron):
```bash
15 4 * * * ~/media-stack/scripts/rotate_vpn.sh >> ~/media-stack/config/rotate_vpn.log 2>&1
```

---

## 📊 Monitoring (Uptime Kuma)

UI:
```
http://localhost:3001
```

Recommended monitors (HTTP):

| Service | URL |
|------|----|
| Sonarr | http://host.docker.internal:8989 |
| Radarr | http://host.docker.internal:7878 |
| Prowlarr | http://host.docker.internal:9696 |
| qBittorrent | http://host.docker.internal:8080 |
| NZBGet | http://host.docker.internal:6789 |

> qBittorrent / NZBGet act as VPN canaries.

---

## 🛠 Helpful Commands

```bash
docker compose up -d
docker compose down
docker compose down && docker compose up -d
docker compose restart gluetun
docker compose ps
docker exec -it gluetun wget -qO- https://ipinfo.io/ip && echo
```

---

## 🧰 Maintenance Notes

- Docker Desktop must start on Windows login
- Containers auto-restart via `restart: unless-stopped`
- Downloaders must restart after VPN restart (handled by script)
- Do NOT monitor Gluetun port 8000 (control port, not health)

---

## 🧯 Troubleshooting

### ❌ “Unable to connect to indexer (localhost:9696)”
**Cause:** Sonarr/Radarr running in Docker cannot reach `localhost` of another container.  
**Fix:** In Sonarr/Radarr → Indexer settings:
- Set Prowlarr URL to: `http://prowlarr:9696`

---

### ❌ “Download client places downloads in /media/... but path does not exist”
**Cause:** Category paths didn’t exist *inside* container.  
**Fix:**
- Ensure `/mnt/f/Media` is mounted as `/media`
- Ensure category subfolders exist:
  ```bash
  /media/Torrents/Complete/movies-radarr
  /media/Torrents/Complete/tv-sonarr
  ```

---

### ❌ Indexers all unavailable
**Cause:** Prowlarr can’t reach trackers or VPN was down.  
**Fix:**
- Verify Gluetun is healthy
- Restart Prowlarr
- Test indexers inside Prowlarr first

---

### ❌ Gluetun restarts but downloaders stay down
**Cause:** Network namespace resets.  
**Fix:** Always restart downloaders *after* Gluetun:
```bash
docker compose restart gluetun qbittorrent nzbget
```
(Handled automatically by rotate script)

---

### ❌ ipinfo.io still works when Gluetun is stopped
**Expected behavior.**
- That’s your **host**, not the containers.
- Container traffic is blocked by Gluetun firewall.

---

## ⏭️ Skippable Sections (Clean Install)

If starting fresh, you can skip:
- Sonarr/Radarr DB migration
- qBittorrent/NZBGet config migration
- Remote Path Mappings

Just start containers and configure via UI.

---

## ✅ Final State

- True VPN kill switch
- Automatic recovery
- Monitoring + alerts
- Safe reboot behavior


See WALKTHROUGH.md for a full guided setup.

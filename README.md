# Media Automation Stack (Docker + VPN Kill Switch)

A production-grade, self-hosted media automation stack designed for **Windows + WSL2**, running fully containerized with **strict VPN enforcement**, health-gated startup, monitoring, and disaster recovery.

This repository focuses on **infrastructure, reliability, and correctness** — not hacks, shortcuts, or unsupported modifications.

---

## Why this stack?

This stack exists to solve a common set of problems with long-running media automation systems:

- Preventing **traffic leaks** when a VPN drops
- Avoiding **double imports, file contention, and corruption**
- Making recovery possible after **Windows reboots, Docker restarts, or VPN failures**
- Ensuring Sonarr/Radarr — not download clients — remain the system of record
- Providing **clear operational docs** you can follow months later

Design goals:

- **Single authority** (Docker, not Windows services)
- **VPN-gated networking** (no VPN = no traffic)
- **Observable health** (you know when things break)
- **Repeatable recovery** (no mystery state)

If you want a stack that behaves predictably and is easy to reason about, this is it.

---

## 📚 Documentation Index

Use the links below depending on what you’re trying to do:

- 🚀 **Quick setup (most people)**  
  👉 [QUICKSTART.md](./QUICKSTART.md)

- 🧭 **Full step-by-step walkthrough (recommended first run)**  
  👉 [WALKTHROUGH.md](./WALKTHROUGH.md)

- 🛠️ **Disaster recovery & rebuild from scratch**  
  👉 [DISASTER_RECOVERY.md](./DISASTER_RECOVERY.md)

- 🐳 **Docker configuration (single source of truth)**  
  👉 [docker-compose.yml](./docker-compose.yml)

- 📝 **Project history & updates**  
  👉 [CHANGELOG.md](./CHANGELOG.md)

---

## What’s included

- Docker + Docker Compose
- Gluetun (VPN enforcement + kill switch)
- Sonarr / Radarr
- Prowlarr (with FlareSolverr)
- qBittorrent (behind VPN)
- NZBGet (behind VPN)
- Uptime Kuma (monitoring)

---

## Quick ports / URLs

- Sonarr: http://localhost:8989
- Radarr: http://localhost:7878
- Prowlarr: http://localhost:9696
- FlareSolverr: http://localhost:8191
- qBittorrent: http://localhost:8080
- NZBGet: http://localhost:6789
- Uptime Kuma: http://localhost:3001

---

## Legal & Usage Notes

This project documents the setup of a self-hosted media automation stack.  
Users are responsible for complying with all applicable laws and the terms  
of service of any providers they use.

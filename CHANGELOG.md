# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
and this project follows semantic versioning.

---

## [1.0.0] – Initial Public Release

### Added
- Hardened Docker Compose stack for Windows + WSL2
- VPN-gated networking using Gluetun
- Sonarr / Radarr with authoritative download handling
- Prowlarr with FlareSolverr support
- qBittorrent and NZBGet running behind VPN
- Health-gated startup and restart behavior
- Uptime Kuma monitoring
- Automated VPN rotation
- Full operational documentation:
  - Quickstart
  - Walkthrough
  - Disaster recovery

### Notes
- No credentials are stored in this repository
- All secrets are provided via `.env` at runtime

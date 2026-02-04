# ⚡ QUICKSTART (Zero Assumptions)

> 📘 **Back to overview:** [README.md](./README.md)

This is the fastest path to get the stack running.

---

## 1) Open WSL

```bash
wsl
```

---

## 2) Create the stack folder

```bash
mkdir -p ~/media-stack/{config,scripts}
cd ~/media-stack
```

---

## 3) Create `.env` (do not commit)

> Use VPN **service credentials** if your provider requires them.

```bash
cat > .env <<'EOF'
NORDVPN_USER=your_service_user
NORDVPN_PASS=your_service_pass
EOF
```

(Cursor blinking is normal. The command ends after you type `EOF` and press Enter.)

---

## 4) Create `docker-compose.yml`

```bash
nano docker-compose.yml
```

Paste the repo’s `docker-compose.yml` contents.

Nano:
- Save: **Ctrl+O**, then **Enter**
- Exit: **Ctrl+X**
- Exit without saving: **Ctrl+X**, then **N**

---

## 5) Start

```bash
docker compose up -d
docker compose ps
```

You should see `gluetun` become **healthy**.

---

## 6) Open UIs

- Sonarr: http://localhost:8989
- Radarr: http://localhost:7878
- Prowlarr: http://localhost:9696
- qBittorrent: http://localhost:8080
- NZBGet: http://localhost:6789
- Uptime Kuma: http://localhost:3001

# 🚨 DISASTER_RECOVERY.md

> 📘 **Back to overview:** [README.md](./README.md)

This is the “something is broken and I need it working again” runbook.

---

## 0) Golden rules

- Your media (`/mnt/f/Media`) is safe.
- Containers are disposable.
- Config folders (`~/media-stack/config/*`) are your state.

---

## 1) Soft restart (most issues)

```bash
cd ~/media-stack
docker compose down
docker compose up -d
docker compose ps
```

---

## 2) VPN recovery (if downloads are dead after VPN restart)

```bash
cd ~/media-stack
docker compose restart gluetun
docker compose restart qbittorrent nzbget
docker compose ps
```

---

## 3) Validate VPN is up

```bash
docker exec -it gluetun wget -qO- https://ipinfo.io/ip && echo
docker logs gluetun | tail -n 80
```

---

## 4) Recreate containers (safe)

```bash
cd ~/media-stack
docker compose down
docker compose pull
docker compose up -d
```

---

## 5) Nuclear (last resort)

> This removes unused Docker data, not your media.
> If you’re unsure, stop here and investigate first.

```bash
docker system prune -f
cd ~/media-stack
docker compose up -d
```

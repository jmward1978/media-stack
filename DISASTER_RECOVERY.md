# 🚨 DISASTER RECOVERY

Golden rule: media is safe.

Soft reset:
```bash
docker compose down
docker compose up -d
```

VPN reset:
```bash
rotatevpn
```

Nuclear:
```bash
docker system prune -f
docker compose up -d
```

---
title : "Tdarr en Proxmox, Blu-ray de Twin Peaks y avisos por Telegram"
slug : "tdarr-proxmox-blu-ray-y-telegram"
date : "2026-10-31"
image : ""
tags : ["tdarr", "proxmox", "lxc", "blu-ray", "makemkv", "telegram", "systemd"]
categories : ["proxmox"]
description : "Convertir las ISOs de Blu-ray de Twin Peaks con Tdarr en un LXC de Proxmox, con avisos de progreso por Telegram y un servicio de extracción que se quedaba congelado."
---

# Montaje

- **Tdarr** corre en el LXC **119** (`tdarr`) de Proxmox. Servidor y nodo `live-lynx` (el de la GPU) en `http://192.168.1.180:8265`.
- Puntos de montaje del LXC (`pct config 119`):

| mp | Host | Contenedor |
|---|---|---|
| mp0 | `/mnt/pve/datos/shared/pelis` | `/mnt/pelis` |
| mp2 | `/mnt/NFS/Pelis` | `/mnt/NFS/Pelis` (la biblioteca real de pepecono) |

- Las ISOs de Blu-ray **no las procesa Tdarr directamente**: primero hay que extraer los títulos a `.mkv` y luego Tdarr los convierte.

# Extracción: `bd-extract-resume.service`

Un servicio systemd en Proxmox recorre las ISOs y extrae capítulo a capítulo, **retomando** donde se quedó si se corta. El 18-07 se quedó colgado en el tercer capítulo y `sudo systemctl restart bd-extract-resume.service` también se congelaba: el proceso de extracción estaba en estado `D` (esperando E/S) y systemd esperaba a que muriera.

Para desatascarlo:

```bash
ssh proxmox 'systemctl status bd-extract-resume --no-pager; ps -eo pid,stat,cmd | grep -i -E "makemkv|ffmpeg|bd" | grep -v grep'
```

```bash
ssh proxmox 'sudo systemctl kill -s KILL bd-extract-resume && sudo systemctl reset-failed bd-extract-resume && sudo systemctl start bd-extract-resume'
```

# Avisos de Telegram

`pepecono:~/Scripts/utilities/tdarr/bd_notify.sh` manda un mensaje con el progreso. Reutiliza el bot y el chat del flujo `tdarr-confirm`, con el token cifrado con sops:

```bash
#!/usr/bin/env bash
set -a
eval "$(sops --decrypt /home/pepe/Scripts/utilities/notificaciones/telegram/tdarr-confirm/.encrypted.env)"
set +a

curl -s "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
     -d chat_id="${TELEGRAM_CHAT_ID}" \
     -d text="$1" \
     -d parse_mode="HTML" > /dev/null
```

Uso: `bd_notify.sh "Twin Peaks: capítulo 3 extraído"`.

Para ver si Tdarr está convirtiendo de verdad (no basta con que esté "arrancado"):

```bash
curl -s http://192.168.1.180:8265/api/v2/status | head -c 400
```

> Nota: la sesión del 18-07 ya no se conserva y hoy no tengo acceso a Proxmox. Las rutas del servicio de extracción y su unidad exacta no las he podido comprobar; ver `systemctl cat bd-extract-resume` en Proxmox.

# Revertir

```bash
ssh proxmox 'sudo systemctl disable --now bd-extract-resume'
```

Borra la unidad de `/etc/systemd/system/` y `bd_notify.sh`. Los `.mkv` extraídos están en la carpeta de entrada de Tdarr.

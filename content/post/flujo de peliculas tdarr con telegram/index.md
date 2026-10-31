---
title : "Flujo de películas pepecono → Proxmox → Tdarr → pepecono con bot de Telegram"
slug : "flujo-de-peliculas-tdarr-con-telegram"
date : "2026-10-31"
image : ""
tags : ["tdarr", "qbittorrent", "telegram", "docker", "ofelia", "proxmox", "vzdump", "sqlite"]
categories : ["proxmox"]
description : "Un orquestador que coge las descargas de qBittorrent, pregunta la categoría por Telegram, las convierte con Tdarr en Proxmox y las devuelve. Y los fallos que tuvo, incluido un re-encode que destrozó una serie."
---

# El flujo

`pepecono:~/Scripts/utilities/tdarr/tdarr_orchestrator.py`, en el contenedor `tdarr-orchestrator`:

1. qBittorrent termina una descarga y su script `telegram-cine-pollo.sh` añade una línea a `tdarr_incoming.tsv`.
2. El orquestador manda un mensaje de Telegram con botones: **categoría** y luego **tipo**.
3. Cuando Proxmox está encendido, copia el archivo con `rsync` a `/mnt/pve/datos/shared/pelis/incoming_general`, la carpeta que vigila Tdarr.
4. Detecta que Tdarr ha terminado (el original desaparece de `incoming_general` y aparece en `transcoded`) y copia el resultado de vuelta a `/mnt/NFS/Pelis/tdarr_output`, por categoría.
5. A los `RETENTION_DAYS` (30) **no borra** el original: Ofelia lanza cada hora `--deferred-delete-once`, que pregunta por Telegram *Borrar ahora / Posponer 7 días*.

Estado en SQLite (`state/tdarr_queue.db`, tabla `jobs`): `pending_category → pending_type → queued_tdarr → converted → copied_back → deleted`, más `declined` y `error`.

## Comandos del bot

| Comando | Qué hace |
|---|---|
| `/status` | tabla resumen: qué hay en cada etapa |
| `/pendientes` | jobs en cola, con botones para actuar sobre cada uno |
| `/sinprogramar` | descargas sin job, con botones para programarlas o ignorarlas |

## docker-compose.yml (sin secretos)

```yaml
services:
  tdarr-orchestrator:
    build: .
    container_name: tdarr-orchestrator
    restart: always
    environment:
      - PYTHONUNBUFFERED=1
      - RETENTION_DAYS=30
      - DB_PATH=/data/tdarr_queue.db
      - INCOMING_FILE=/qbit-appdata/tdarr_incoming.tsv
      - CONTAINER_DOWNLOADS_PREFIX=/downloads
      - HOST_DOWNLOADS_PREFIX=/mnt/NFS/Pelis
      - PROXMOX_SSH_HOST=proxmox
      - PROXMOX_SRC_DIR=/mnt/pve/datos/shared/pelis/incoming_general
      - PROXMOX_OUT_DIR=/mnt/pve/datos/shared/pelis/transcoded
      - LOCAL_OUTPUT_BASE=/output
      - TDARR_API_URL=http://192.168.1.180:8265
    volumes:
      - ./state:/data
      - /home/pepe/contenedores/herramientas/qbittorrent_pelis/appdata:/qbit-appdata
      - /mnt/NFS/Pelis:/mnt/NFS/Pelis
      - /mnt/NFS/Pelis/tdarr_output:/output
      - /home/pepe/.ssh:/root/.ssh:ro
      - /home/pepe/.config/age:/root/.config/age:ro
      - /home/pepe/Scripts/utilities/notificaciones/telegram/tdarr-confirm/.encrypted.env:/run/secrets/telegram.encrypted.env:ro
      - ./.encrypted.env:/run/secrets/qbittorrent.encrypted.env:ro
    labels:
      - ofelia.enabled=true
      - ofelia.job-exec.tdarr-deferred-delete.schedule=0 0 * * * *
      - ofelia.job-exec.tdarr-deferred-delete.command=python3 tdarr_orchestrator.py --deferred-delete-once
```

Los secretos se descifran con sops **dentro del script** y no solo en el `entrypoint.sh`, porque `docker exec`, que usa Ofelia, no pasa por el entrypoint.

# Fallos (agosto y 07-10-2026)

## 1. "El original desapareció" sin haber desaparecido

`proxmox_path_exists()` hacía `ssh proxmox test -e <ruta>` **sin comillas**. Con espacios y paréntesis en el nombre, la shell remota rompía el comando y siempre devolvía "no existe". Los jobs llevaban desde el 9 de agosto atascados.

```diff
-    remote_cmd = f"test -e {remote_path}"
+    remote_cmd = f"test -e {shlex.quote(remote_path)}"
```

Despliegue:

```bash
ssh pepecono 'cd ~/Scripts/utilities/tdarr && docker compose up -d --build && docker logs tdarr-orchestrator --since 30s'
```

## 2. Disco de Proxmox lleno (Tdarr no terminaba nunca)

Tdarr empezaba a convertir pero no podía escribir la salida: `/mnt/pelis` estaba al **100 %**. Había 96 carpetas `tdarr-workDir2-*` a medias. La hipótesis del horario del nodo a cero era falsa.

| `/mnt/pve/datos` | Tamaño |
|---|---|
| `dump/` (backups vzdump) | 273 GB |
| `shared/pelis` | 458 GB |
| `shared/syncthing` | 79 GB |
| `images/` | 80 GB |

Qué se hizo:

```bash
ssh proxmox 'sudo pct exec 119 -- sh -c "rm -rf /mnt/pelis/transcoded/tdarr-workDir2-*"'
```

```bash
ssh proxmox 'sudo pvesh set /cluster/backup/backup-354b179c-9e17 --prune-backups keep-last=2'
```

La retención de backups pasó de 3 a 2 copias (en `/etc/pve/jobs.cfg`). `vzdump --prune-backups` no sirve con el disco lleno, porque primero intenta crear un backup nuevo. Las copias del 23-08 se borraron a mano:

```bash
ssh proxmox 'rm -f /mnt/pve/datos/dump/*2026_08_23*; df -h /mnt/pve/datos'
```

## 3. Recodificaba otra vez lo ya convertido

*La patrulla canina* ya se había recodificado en agosto: entonces la biblioteca de Tdarr apuntaba directamente a `/mnt/NFS/Pelis` con **"borrar original al convertir"**. El resultado (~470 MB/episodio) acabó en `incoming_general` para una **segunda** pasada. El original en alta calidad se perdió.

- Saqué la carpeta de la cola y la devolví a `/mnt/NFS/Pelis` (49 GB, 104 episodios).
- Marqué el job 5 como `copied_back` con una nota para que no vuelva a la cola.
- Hoy la biblioteca de Tdarr solo vigila `/mnt/pelis/incoming_general`. Ninguna otra apunta a la biblioteca real.

**Regla**: Tdarr nunca debe vigilar la biblioteca de verdad con *Delete source* activado. Para ver si un archivo ya pasó por Tdarr, mira `lastTranscodeDate` en `DB2/SQL/database.db` o los `JobReports`.

## 4. Tdarr no usaba la GPU (agosto)

> Nota: la sesión de agosto ya no se conserva; esto es lo que hay que revisar, no el cambio exacto que se hizo.

El nodo con GPU es `live-lynx`. Los plugins tienen que ser de NVENC/QSV (no `libx265`) y el LXC necesita los dispositivos `/dev/dri` o `/dev/nvidia*` en su config. Para comprobarlo en el log del worker: `ffmpeg ... -c:v hevc_nvenc` (o `hevc_qsv`).

# Revertir

```bash
ssh pepecono 'cd ~/Scripts/utilities/tdarr && docker compose down'
```

```bash
ssh proxmox 'sudo pvesh set /cluster/backup/backup-354b179c-9e17 --prune-backups keep-last=3'
```

La cola es una base SQLite: haz una copia de `state/tdarr_queue.db` antes de tocar estados a mano.

---
title : "Syncthing para llevar la carpeta de Mixxx a pepecono y Airsonic"
slug : "syncthing-carpeta-mixxx"
date : "2026-10-31"
image : ""
tags : ["syncthing", "mixxx", "docker", "airsonic", "exfat"]
categories : ["contenedores"]
description : "La carpeta de música para pinchar se envía con Syncthing desde este PC a pepecono, donde la usan Airsonic y mixxx_info. Carpetas, tipos y volúmenes."
---

# Esquema

```
arcolinux /mnt/windows/Mix  ──(sendonly)──▶  pepecono /mnt/NFS/moode/Mixxx  → Airsonic (carpeta "Mixxx")
                                                                           → mixxx_info
arcolinux /mnt/windows/Mix_Playlists ◀─(sendreceive)─▶ pepecono repeat-one/services/mixxx_info/playlists
```

| Carpeta (id) | arcolinux | pepecono (contenedor) | Tipo |
|---|---|---|---|
| `mjugz-pcrud` "Mixxx" | `/mnt/windows/Mix` | `/data/Mixxx` → `/mnt/NFS/moode/Mixxx` | **sendonly** aquí, **receiveonly** allí |
| `mixxx-playlists` "Mixxx Playlists" | `/mnt/windows/Mix_Playlists` | `/data/MixxxPlaylists` → `~/contenedores/repeat-one/services/mixxx_info/playlists` | sendreceive |

`sendonly` / `receiveonly`: este PC es la fuente. Si algo se toca en pepecono, Syncthing no lo devuelve: lo marca como "cambios locales" y se puede revertir desde la web.

# pepecono: el contenedor

`~/contenedores/herramientas/syncthing/docker-compose.yml`:

```yaml
services:
  syncthing:
    image: lscr.io/linuxserver/syncthing:latest
    container_name: syncthing
    hostname: pepecono #optional
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
    volumes:
      - /home/pepe/contenedores/herramientas/syncthing/config:/config
      - /mnt/NFS/lidarr/torrents_backup/watch_torrents:/data/lidarr_torrents
      - /mnt/NFS/Pelis/watch_folder:/data/pelis_torrents
      - /mnt/NFS/moode/Mixxx:/data/Mixxx
      - /home/pepe/contenedores/repeat-one/services/mixxx_info/playlists:/data/MixxxPlaylists
      - /home/pepe/contenedores/repeat-one/data/mfuzz-db:/data/mfuzz-db
    ports:
      - 8384:8384
      - 22000:22000/tcp
      - 22000:22000/udp
      - 21027:21027/udp
    restart: unless-stopped
```

Los volúmenes de `Mixxx` y `MixxxPlaylists` son los nuevos (copia anterior: `docker-compose.yml.bak-claude`). Una carpeta nueva solo se ve dentro del contenedor si está montada: añadir el volumen, recrear el contenedor y luego aceptar la carpeta en la web (`http://pepecono:8384`) con la ruta **del contenedor** (`/data/...`).

```bash
ssh pepecono 'cd ~/contenedores/herramientas/syncthing && docker compose up -d'
```

# arcolinux

Syncthing corre como servicio de usuario. La configuración está en `~/.local/state/syncthing/config.xml`. Dos cosas por ser `/mnt/windows` un disco **exFAT**:

- exFAT no guarda permisos de Unix. Marca *Ignorar permisos* en la carpeta, o cada `chmod` provocará cambios fantasma.
- Los nombres no distinguen mayúsculas: no tengas `Techno/` y `techno/` a la vez.

Ojo: el `~/.config/syncthing/config.xml` que está en chezmoi es una copia antigua de este PC. **No lo apliques en pepecono**, porque allí la configuración real es la del contenedor.

# Lo que lo usa en pepecono

- **Airsonic**: monta `/mnt/NFS/moode/Mixxx` como `/Mixxx` (carpeta de medios "Mixxx"). Ver [playlists de Mixxx en Airsonic](../mixxx-cajones-y-playlists-en-airsonic).
- **mixxx_info** (repeat-one): lee las playlists de `services/mixxx_info/playlists`.

Una canción recién movida tarda en estar disponible: primero la sincroniza Syncthing y luego hay que esperar al escaneo de Airsonic (`mixxx_to_airsonic.py --scan`).

# Revertir

En la web de Syncthing de los dos equipos: *Editar carpeta → Eliminar*. Los archivos no se borran. Después quita los volúmenes `/data/Mixxx` y `/data/MixxxPlaylists` del compose y recrea el contenedor.

---
title : "Sacar el audio de Airsonic en Docker por el PipeWire del host"
slug : "airsonic-docker-audio-pipewire"
date : "2026-10-31"
image : ""
tags : ["airsonic", "docker", "pipewire", "pulseaudio", "alsa", "jukebox"]
categories : ["contenedores"]
description : "El modo jukebox de Airsonic-Advanced reproduce en el servidor. Dentro de Docker no hay tarjeta de sonido, pero se puede usar el socket pulse del host sin recompilar nada."
---

# El problema

El **modo jukebox** de Airsonic reproduce la música en el propio servidor. En pepecono, Airsonic-Advanced corre en Docker (`lscr.io/linuxserver/airsonic-advanced`), y dentro del contenedor no hay ninguna tarjeta de sonido. El host usa **PipeWire**, no ALSA directo.

No hace falta recompilar Airsonic. Java usa ALSA, y ALSA puede mandar a un servidor pulse (el `pipewire-pulse` del host) a través del plugin `alsa-plugins-pulse`.

# Cambios

`~/contenedores/airsonic/docker-compose.yml` (en pepecono), lo añadido:

```yaml
    environment:
      - PULSE_SERVER=unix:/run/user/1000/pulse/native
    volumes:
      - /run/user/1000/pulse/native:/run/user/1000/pulse/native
      - /home/pepe/contenedores/airsonic/config/.asoundrc:/etc/asound.conf
      - /home/pepe/contenedores/airsonic/config/custom-cont-init.d:/custom-cont-init.d
```

1. **El socket pulse del host** (`/run/user/1000/pulse/native`, del usuario `pepe`, UID 1000) se monta en el contenedor, y `PULSE_SERVER` apunta a él. Como el contenedor corre con `PUID=1000`, tiene permiso.

2. **ALSA → pulse.** `config/.asoundrc`, montado como `/etc/asound.conf`:

```
pcm.!default {
    type pulse
}
ctl.!default {
    type pulse
}
```

3. **El plugin.** La imagen de linuxserver es Alpine: no tiene `apt-get` (`exec: "apt-get": executable file not found`) y se pierde lo que instales a mano al recrear el contenedor. Las imágenes de linuxserver ejecutan al arrancar lo que haya en `/custom-cont-init.d`, así que va en `config/custom-cont-init.d/install-alsa-pulse.sh`:

```bash
apk add --no-cache alsa-plugins-pulse
```

```bash
ssh pepecono 'cd ~/contenedores/airsonic && docker compose up -d --force-recreate'
```

# Comprobar

```bash
ssh pepecono 'docker exec airsonic-advanced ls -la /run/user/1000/pulse/native && docker exec airsonic-advanced find /usr -name "libasound_module_pcm_pulse.so"'
```

En Airsonic: *Ajustes → Reproductores* → un reproductor de tipo **Jukebox**. Reproduce algo y mira en pepecono que aparece como cliente:

```bash
ssh pepecono 'pactl list short sink-inputs'
```

Con los túneles PipeWire, ese audio se puede mandar a otro equipo de la casa.

# Revertir

Quita del `docker-compose.yml` la variable `PULSE_SERVER` y los tres volúmenes (socket, `.asoundrc` y `custom-cont-init.d`), y recrea el contenedor.

```bash
ssh pepecono 'cd ~/contenedores/airsonic && docker compose up -d --force-recreate'
```

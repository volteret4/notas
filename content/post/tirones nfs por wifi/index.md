---
title : "Tirones en NFS por WiFi, ahorro de energía y montaje"
slug : "tirones-nfs-por-wifi"
date : "2026-10-31"
image : ""
tags : ["nfs", "wifi", "networkmanager", "fstab", "deadbeef", "nicotine"]
categories : ["network"]
description : "Música FLAC por NFS con tirones, Nicotine+ congelado y DeaDBeeF trabándose al cambiar de pista. Diagnóstico de la WiFi, el montaje y qué está aplicado."
---

# Síntomas

- Tirones al navegar `/mnt/NFS/moode` (biblioteca FLAC en pepecono) con el explorador de archivos, incluso con FLAC de 24 bits.
- Nicotine+ se congelaba al terminar cada descarga, porque la copia a la carpeta montada.
- Después, DeaDBeeF empezaba **todas** las canciones trabado, y a veces la ventana entera se congelaba.

# Diagnóstico (22-10-2026)

| Hallazgo | Dato | Impacto |
|---|---|---|
| El PC va por **WiFi** | `wlp5s0`; el cable `enp6s0` está desconectado | causa principal |
| **Latencia irregular** | ping a pepecono: mín 2 ms, media 17 ms, **máx 120 ms** | cada pico bloquea las operaciones NFS |
| **Ahorro de energía WiFi activado** | `iw dev wlp5s0 get power_save` → `on` | la tarjeta se duerme entre paquetes |
| El router cambia el ancho de canal 20↔40 MHz | en `dmesg`, cada 5–15 min | microcortes |
| Velocidad de lectura | 8,4 MB/s | sobra para FLAC 24/192 (~1 MB/s): el problema es la latencia, no el ancho de banda |

No había retransmisiones ni errores de NFS. Comandos útiles:

```bash
iw dev wlp5s0 link
```

```bash
ping -c 50 -i 0.2 192.168.1.133 | tail -1
```

```bash
nfsstat -m
```

## Nicotine+

Al terminar una descarga, mueve el archivo a la carpeta de destino desde el hilo de la interfaz. Si esa carpeta está en NFS y la WiFi tiene picos de 120 ms, la ventana se queda congelada hasta que acaba la copia. Solución: descargar en disco local y mover por lotes.

```bash
rsync -a --remove-source-files ~/Descargas/nicotine/ /mnt/NFS/moode/Descargas/
```

## DeaDBeeF (29-10-2026)

- PipeWire no registraba xruns: el corte viene de que DeaDBeeF no recibe los datos a tiempo.
- Al empezar cada canción, el FLAC se abre **dos veces**: una para reproducir y otra para el plugin **waveform seekbar**, que lo decodifica entero (25–35 MB) para dibujar la onda.
- Leer el principio de un FLAC tardaba 22–31 ms solo, y **986 ms** con una lectura grande en paralelo, con `rsize` de 1 MB.
- La ventana entera se congela porque el montaje es `hard`: mientras la red no responde, cualquier lectura espera sin límite, y DeaDBeeF lee carátulas y metadatos desde el hilo de la interfaz. `hard` está bien elegido: con `soft` habría errores de lectura y riesgo de dañar archivos.
- Esa noche además la WiFi se degradó: 697 avisos de `missed beacons` desde las 23:23, conectado por 5 GHz a −76 dBm (39 %), cuando la red de 2,4 GHz del mismo router llegaba al 67 %.

La caché del waveform ya existe (`~/.cache/deadbeef/waveform_seekbar/wavecache.db`, unos 500 MB), pero no evita la primera lectura de cada canción.

# Cambios aplicados

**`/etc/fstab`**: la línea vieja de moode está comentada y hay una nueva, con bloques de 128 KB para que las lecturas pequeñas no esperen detrás de bloques de 1 MB:

```
192.168.1.133:/mnt/NFS/moode /mnt/NFS/moode nfs4 rw,hard,vers=4.2,rsize=131072,wsize=131072,timeo=600,retrans=2,_netdev,nofail 0 0
```

Para aplicarla (con DeaDBeeF cerrado):

```bash
sudo umount /mnt/NFS/moode && sudo systemctl daemon-reload && sudo mount /mnt/NFS/moode
```

# WiFi (aplicado el 31-10-2026)

Hasta ese día el ahorro de energía seguía activado (`Power save: on`) y `TeskiyaMaricarmen` estaba **forzada a 5 GHz** (`802-11-wireless.band = a`), la banda con peor señal. Se cambió en NetworkManager, sin tocar `/etc`:

- `802-11-wireless.powersave = 2` (desactivado) en **todas** las conexiones WiFi guardadas: PapasconChocos, PapasconChocos 1, POCO F6, POCO X3 NFC_6781, TeskiyaMaricarmen, TeskiyaMaricarmen 1 y TeskiyaMaricarmen 2.
- `TeskiyaMaricarmen`: `802-11-wireless.band = bg` (2,4 GHz).

```bash
nmcli -t -f NAME,TYPE connection show | grep ':802-11-wireless$' | cut -d: -f1 | while read -r c; do nmcli con modify "$c" 802-11-wireless.powersave 2; done
```

```bash
nmcli con modify TeskiyaMaricarmen 802-11-wireless.band bg
```

El ajuste de ahorro de energía se aplica al reconectar (`nmcli device reapply` no lo admite):

```bash
nmcli con up "PapasconChocos 1" && iw dev wlp5s0 get power_save
```

Resultado: `Power save: off`, ping a pepecono de 2–8 ms. Las conexiones WiFi nuevas que crees tendrán `powersave = default` (activado): repite el `nmcli con modify` en ellas.

Otras mejoras: quitar el waveform seekbar (*Ver → Modo diseño* → clic derecho en la onda → *Reemplazar → Seekbar*) o, la definitiva, **conectar el cable** a `enp6s0`.

# Revertir

En `/etc/fstab`, comenta la línea nueva y descomenta la anterior (`soft,intr,fsc,rsize=65536,...,vers=4.1`). Luego desmonta y vuelve a montar. Hay una copia muy antigua en `/etc/fstab.save` (2024).

Para volver a la WiFi de antes (ahorro de energía por defecto y 5 GHz en TeskiyaMaricarmen):

```bash
nmcli -t -f NAME,TYPE connection show | grep ':802-11-wireless$' | cut -d: -f1 | while read -r c; do nmcli con modify "$c" 802-11-wireless.powersave 0; done
```

```bash
nmcli con modify TeskiyaMaricarmen 802-11-wireless.band a
```

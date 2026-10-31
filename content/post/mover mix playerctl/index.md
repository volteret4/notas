---
title : "mover_mix_playerctl.py, el script que manda canciones a mis carpetas de DJ"
slug : "mover-mix-playerctl"
date : "2026-10-31"
image : ""
tags : ["python", "playerctl", "musicbrainz", "ffmpeg", "mixxx", "airsonic", "cue"]
categories : ["scripts"]
description : "Todo lo que cambió en el script que copia o mueve la canción que suena a /mnt/windows/Mix, con las copias de seguridad de cada cambio para volver atrás."
---

# Qué hace

`~/Scripts/Musica/MoverMusica/mover_mix_playerctl.py` coge la canción que suena (por playerctl, o desde DeaDBeeF o Airsonic) y la manda a una carpeta de `/mnt/windows/Mix` (exFAT, la de Mixxx). Luego:

- elige la carpeta en un menú, con subcarpetas de género (`TECHNO/techno_deep`, `HOUSE/...`) y atajos,
- renombra el archivo a `Artista - Canción (año álbum) [sello]` a partir de los tags,
- pide un comentario opcional,
- regenera los `.m3u` de cada carpeta,
- y lanza en segundo plano `sync_crates.py` (cajones de Mixxx), `yt_sync_mixxx.py` (playlists de YouTube) y `embeed_cover.py` (carátula).

# Cambios, por orden

| Fecha | Cambio |
|---|---|
| 03 → 08 | Credenciales de Airsonic en `.env`, nombre de archivo a partir de la metadata, menú de carga y subcarpetas de género con atajos |
| 05-10 | **Mover en vez de copiar** cualquier canción cuya ruta empiece por `/mnt/windows/Mix/` (antes solo dentro de TECHNO/HOUSE). No busca duplicados. Si eliges la carpeta en la que ya está, solo aplica el comentario |
| 07-10 | Llamada a `sync_crates.py` tras regenerar los `.m3u` |
| 16-10 | **Pistas dentro de un `.cue`**: detecta el cue (incrustado como tag `CUESHEET` o en un `.cue` al lado), averigua la pista que suena por título, número o duración (si no, pregunta) y **corta solo esa pista con ffmpeg** (FLAC/APE/WV/WAV → FLAC; MP3 sin recodificar). Corrige los tags y quita `CUESHEET`, BPM y tonalidad del disco entero. Además, `unquote` no estaba importado y la parte de playerctl siempre fallaba |
| 16-10 | Sincroniza la playlist de YouTube de la carpeta de destino y, al mover, también la de origen. Comentada la llamada a Spotify (su API ya no funciona sin pago) |
| 21-10 | Fecha original vía MusicBrainz (script aparte, `fecha_original_mix.py`) |
| 25-10 | `[Errno 1] Operación no permitida` al copiar desde NFS: los permisos `rwxrwSr-x` (setgid) no caben en exFAT. Nueva `copy_sin_permisos()`, que copia contenido y fechas pero no permisos, usada en `copy_file` y en `shutil.move(..., copy_function=copy_sin_permisos)` |
| 27-10 | **No renombraba al reclasificar** (movía con `rename=False`) y no leía tags escritos como `Artist`/`Title`. Ahora lee los tags sin distinguir mayúsculas, usa el artista del álbum si falta el artista y, si aun así falta algo, deja el nombre original. `[no label]` ya no se pone en el nombre |

## `fecha_original_mix.py`

Para los remasters que confunden: pone en `DATE` la fecha **más antigua** entre la actual, `ORIGINALDATE` (Picard) y el primer lanzamiento de la grabación en MusicBrainz. Nunca retrasa una fecha, y guarda la que había en `RELEASEDATE`.

## `renombrar_mix.py`

Arregla los nombres que quedaron mal por el fallo del 27-10 (encontró unos 1611): espacios dobles, `-  ()` al final, sin artista o sin título, o que empiezan por número de pista.

```bash
~/Scripts/python_venv/bin/python ~/Scripts/Musica/MoverMusica/renombrar_mix.py | less
```

```bash
~/Scripts/python_venv/bin/python ~/Scripts/Musica/MoverMusica/renombrar_mix.py --apply
```

- Con Mixxx **cerrado**. Antes de tocar nada copia la base de datos de Mixxx en `~/.mixxx/backups_renombrar/` y actualiza las rutas para no perder cues ni análisis.
- Si el nombre nuevo ya existe, se lo salta. Si Mixxx tiene una entrada fantasma con ese nombre, intercambia las rutas.
- Apunta cada cambio en `~/.mixxx/backups_renombrar/renombrados.<fecha>.tsv`, y `--deshacer` lo revierte.

## `embeed_cover.py`

Ignoraba el archivo que le pasaba el mover y recorría **todo** Mix (unas 1100 canciones), consultando MusicBrainz cada vez. Ahora procesa solo lo que se le pasa; sin argumentos sigue recorriendo todo Mix.

# Copias de seguridad

En `~/Scripts/Musica/MoverMusica/`:

| Copia | De antes de |
|---|---|
| `mover_mix_playerctl.py.bak.20260826210410` | cambios de agosto |
| `mover_mix_playerctl.py.bak.20261007183246` | llamada a `sync_crates.py` |
| `mover_mix_playerctl.py.bak.20261016201948` | soporte de cue |
| `mover_mix_playerctl.py.bak.20261021220202` | fecha original |
| `mover_mix_playerctl.py.bak.20261027195025` | arreglo del renombrado |
| `fecha_original_mix.py.bak.2026102*` | cada iteración de ese script |
| `~/Scripts/Musica/playlists/mixxx/embeed_cover.py.bak.20261016203210` | — |

El arreglo de permisos del 25-10 se guardó en una copia temporal que ya no existe; está entre `.bak.20261021220202` y `.bak.20261027195025`.

# Revertir

Para volver a una versión concreta:

```bash
cp ~/Scripts/Musica/MoverMusica/mover_mix_playerctl.py.bak.20261016201948 ~/Scripts/Musica/MoverMusica/mover_mix_playerctl.py
```

Para ver qué cambió entre dos copias:

```bash
diff ~/Scripts/Musica/MoverMusica/mover_mix_playerctl.py.bak.20261027195025 ~/Scripts/Musica/MoverMusica/mover_mix_playerctl.py
```

Para deshacer un renombrado masivo:

```bash
~/Scripts/python_venv/bin/python ~/Scripts/Musica/MoverMusica/renombrar_mix.py --deshacer ~/.mixxx/backups_renombrar/renombrados.20261027195611.tsv
```

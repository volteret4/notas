---
title : "Playlists de Mixxx en cajones y en Airsonic"
slug : "mixxx-cajones-y-playlists-en-airsonic"
date : "2026-10-31"
image : ""
tags : ["mixxx", "airsonic", "subsonic", "sqlite", "python", "syncthing"]
categories : ["scripts"]
description : "Dos scripts: uno crea un cajón de Mixxx por cada .m3u de la carpeta Mix y otro copia las playlists de Mixxx al usuario mix de Airsonic."
---

# Contexto

- La música para pinchar está en `/mnt/windows/Mix`, con un `.m3u` por lista, que regenera `mover_mix_playerctl.py` cada vez que muevo una canción.
- Syncthing clona esa carpeta en pepecono (`/mnt/NFS/moode/Mixxx`), y Airsonic la tiene como carpeta de medios **Mixxx**.

# 1. Cajones de Mixxx desde los `.m3u` (`sync_crates.py`)

`~/Scripts/Musica/playlists/mixxx/sync_crates.py` deja un **cajón** (crate) por cada `.m3u` de `/mnt/windows/Mix`, con el mismo nombre y las mismas canciones. La primera vez creó 59.

- Cada cajón queda **idéntico** a su `.m3u`: añade lo que falta y quita lo que sobra. Si borras el `.m3u`, borra el cajón.
- No toca las playlists de Mixxx ni los cajones creados a mano.
- **Canciones movidas de carpeta**: las reconoce por nombre y tamaño y actualiza la ruta, así conservan cues, BPM y análisis. De 441 que faltaban recuperó 404.
- Las canciones totalmente nuevas tienen que entrar antes en la biblioteca de Mixxx (en el escaneo al arrancar), así que entran en su cajón al cerrar Mixxx.
- **Solo escribe con Mixxx cerrado**, y antes guarda una copia de la base de datos en `~/.mixxx/backups_sync_crates/` (las 5 últimas). Si algo falla a mitad, no aplica nada.

```bash
python3 ~/Scripts/Musica/playlists/mixxx/sync_crates.py --dry-run
```

## Cuándo se ejecuta

| Cuándo | Cómo |
|---|---|
| Al mover una canción con `mover_mix_playerctl.py` | lo llama con `--quiet --notify` después de regenerar los `.m3u`. Si Mixxx está abierto, solo avisa |
| Al abrir Mixxx desde el lanzador | `mixxx-sync.sh` sincroniza antes |
| Al cerrar Mixxx | `mixxx-sync.sh` sincroniza después |

`~/Scripts/Musica/playlists/mixxx/mixxx-sync.sh`:

```bash
sync_crates="$(dirname "$(readlink -f "$0")")/sync_crates.py"

python3 "$sync_crates" --quiet --notify
pasuspender -- mixxx "$@" || mixxx "$@"
python3 "$sync_crates" --quiet
```

`~/.local/share/applications/org.mixxx.Mixxx.desktop` sobrescribe el lanzador del sistema para que `Exec` use ese script.

# 2. Playlists de Mixxx en Airsonic (`mixxx_to_airsonic.py`)

`~/Scripts/Musica/playlists/mixxx/mixxx_to_airsonic.py` copia las **playlists con nombre** de Mixxx (no Auto DJ ni el historial) al usuario **`mix`** de Airsonic.

- Lee la base de datos de Mixxx en solo lectura, así que puede estar abierto.
- Una canción en `/mnt/windows/Mix/X` se busca en Airsonic como `Mixxx/X`. Las que Airsonic aún no tiene salen al final como "falta".
- Cada playlist lleva en el comentario la marca `Mixxx: <nombre>`. Así, al volver a ejecutarlo actualiza la misma playlist en el orden de Mixxx en lugar de duplicarla. Las playlists sin esa marca no se tocan.
- Usa GET en la API de Subsonic, porque Airsonic-Advanced bloquea los POST por CSRF.
- Airsonic-Advanced no devuelve la playlist en la respuesta de `createPlaylist`, así que el script la busca después por nombre (la de id más alto).

Credenciales en `~/Scripts/Musica/playlists/airsonic/.env`: `AIRSONIC_URL`, `AIRSONIC_MIX_PASSWORD` y, si el usuario no se llama `mix`, `AIRSONIC_MIX_USER`. El usuario necesita acceso a la carpeta de medios **Mixxx**.

```bash
python3 ~/Scripts/Musica/playlists/mixxx/mixxx_to_airsonic.py --dry-run
```

```bash
python3 ~/Scripts/Musica/playlists/mixxx/mixxx_to_airsonic.py --scan
```

| Opción | Qué hace |
|---|---|
| `--dry-run` | muestra los cambios sin escribir |
| `--scan` | hace que Airsonic escanee antes la biblioteca (tarda más de 10 minutos, recorre toda la biblioteca) |
| `--prune` | borra de Airsonic las playlists que ya no existen en Mixxx |
| `--quiet` | solo el resumen |

Errores corregidos al probarlo: `IndexError: pop from empty list` con carpetas vacías en Airsonic, y la playlist `deep`, que se creó sin la marca la primera vez.

# Revertir

- **Cajones**: con Mixxx cerrado, restaura la última copia.

```bash
cp "$(ls -1t ~/.mixxx/backups_sync_crates/*.sqlite | head -1)" ~/.mixxx/mixxxdb.sqlite
```

- **Lanzador**: borra `~/.local/share/applications/org.mixxx.Mixxx.desktop`.
- **mover_mix_playerctl.py**: quita la llamada a `sync_crates.py` (hay una copia `.bak.<fecha>` del original).
- **Airsonic**: borra las playlists cuyo comentario empieza por `Mixxx:` desde la web, con el usuario `mix`.

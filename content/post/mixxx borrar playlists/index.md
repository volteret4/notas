---
title : "Borrar de golpe las playlists temporales de Mixxx"
slug : "mixxx-borrar-playlists"
date : "2026-10-31"
image : ""
tags : ["mixxx", "sqlite", "bash", "dj"]
categories : ["scripts"]
description : "Un script que borra las listas de reproducción de Mixxx que uso en cada sesión, sin tocar Auto DJ, el historial, las bloqueadas ni los cajones, y con copia de la base de datos."
---

# Para qué

Uso las playlists de Mixxx para sesiones temporales y, una vez grabadas, sobran. Borrarlas una a una desde la interfaz es un rollo.

`~/Scripts/Musica/playlists/mixxx/borrar_playlists.sh` muestra las listas con su número de canciones y pide confirmación:

```bash
~/Scripts/Musica/playlists/mixxx/borrar_playlists.sh
```

Sin preguntar:

```bash
~/Scripts/Musica/playlists/mixxx/borrar_playlists.sh -y
```

# Qué borra y qué no

En la tabla `Playlists` de `~/.mixxx/mixxxdb.sqlite`, la columna `hidden` dice el tipo de lista: `0` es una lista normal, `1` es Auto DJ y `2` es el historial de sesiones (las que tienen fecha). Solo se borran las que cumplen:

```sql
hidden = 0 AND locked = 0
```

- **No toca** Auto DJ, el historial, las listas **bloqueadas** con el candado de Mixxx ni los **cajones** (crates). Para conservar alguna lista, bloquéala antes.
- **Mixxx tiene que estar cerrado.** Si está abierto, el script se niega: Mixxx tiene la base de datos en memoria y pisaría los cambios al cerrar.
- Antes de borrar hace una copia con `sqlite3 .backup` en `~/.mixxx/backups_borrar_playlists/` y guarda las 5 últimas.
- Borra en una transacción: primero `PlaylistTracks` y luego `Playlists`.

# El script

```bash
DB="$HOME/.mixxx/mixxxdb.sqlite"
BACKUP_DIR="$HOME/.mixxx/backups_borrar_playlists"
# hidden=0: listas normales (1 = Auto DJ, 2 = historial); locked=0: no bloqueadas
WHERE="hidden = 0 AND locked = 0"

if pgrep -x mixxx > /dev/null; then
    echo "Mixxx está abierto: ciérralo antes de borrar las listas." >&2
    exit 1
fi

list=$(sqlite3 "$DB" "SELECT name || '  (' || (SELECT count(*) FROM PlaylistTracks WHERE playlist_id = p.id) || ')' FROM Playlists p WHERE $WHERE ORDER BY position")
if [ -z "$list" ]; then
    echo "No hay listas de reproducción que borrar."
    exit 0
fi

echo "$list"
count=$(echo "$list" | wc -l)
if [ "$1" != "-y" ]; then
    read -rp "¿Borrar estas $count listas? [s/N] " answer
    [[ "$answer" == [sS]* ]] || { echo "Cancelado."; exit 0; }
fi

mkdir -p "$BACKUP_DIR"
sqlite3 "$DB" ".backup '$BACKUP_DIR/mixxxdb.$(date +%Y%m%d%H%M%S).sqlite'" || exit 1
ls -1t "$BACKUP_DIR"/mixxxdb.*.sqlite | tail -n +6 | xargs -r rm --

sqlite3 "$DB" "BEGIN; DELETE FROM PlaylistTracks WHERE playlist_id IN (SELECT id FROM Playlists WHERE $WHERE); DELETE FROM Playlists WHERE $WHERE; COMMIT;" && echo "Borradas $count listas. Copia en $BACKUP_DIR"
```

Un alias corto:

```bash
echo "alias mixxx-limpiar='~/Scripts/Musica/playlists/mixxx/borrar_playlists.sh'" >> ~/.zsh_aliases
```

# Recuperar listas borradas

Con Mixxx cerrado, copia la última copia de seguridad encima de la base de datos:

```bash
cp "$(ls -1t ~/.mixxx/backups_borrar_playlists/mixxxdb.*.sqlite | head -1)" ~/.mixxx/mixxxdb.sqlite
```

Ojo: también pierdes cualquier otro cambio en la biblioteca (BPM, cues…) hecho después de esa copia.

# Revertir

```bash
rm ~/Scripts/Musica/playlists/mixxx/borrar_playlists.sh
```

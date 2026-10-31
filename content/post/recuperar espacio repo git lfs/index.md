---
title : "Recuperar espacio, copias ocultas de un repo con Git LFS"
slug : "recuperar-espacio-repo-git-lfs"
date : "2026-10-31"
image : ""
tags : ["git", "lfs", "disco", "limpieza", "du"]
categories : ["git"]
description : "La carpeta .zed de un repo ocupaba 4,4 GB porque dentro había una copia completa del repo de antes de reescribir el historial. Cómo lo encontré, qué borré y qué se perdió."
---

# El problema

`~/gits/pollo/rym_lastfm/.zed` ocupaba muchísimo. Pero Zed es solo un editor: lo único suyo en esa carpeta es `settings.json` (211 bytes).

```bash
du -sh ~/gits/pollo/rym_lastfm/.zed/* ~/gits/pollo/rym_lastfm/.zed/.[!.]* 2>/dev/null | sort -h
```

| Contenido de `.zed` | Tamaño |
|---|---|
| `.git` | 2,1 GB |
| `.old` | 1,4 GB |
| `docs` | 687 MB |
| `db` | 198 MB |

Era **una copia entera del repo** del 19-03-2026, de antes de reescribir el historial (cuando moví la base SQLite a Git LFS). Su `main` estaba "353 delante, 349 detrás" de `origin/main`: mismos mensajes de commit, pero con otros hashes.

# Cómo comprobé que se podía borrar

```bash
git -C ~/gits/pollo/rym_lastfm/.zed log -1 --format='%H %ad %s'
```

```bash
git -C ~/gits/pollo/rym_lastfm/.zed rev-list --left-right --count main...origin/main
```

- **Commits**: el historial viejo está entero en pepecono (`~/gits/pollo/rym_lastfm`), y los mensajes están en el historial actual.
- **Archivos sueltos**: los `.sql` y los scripts viejos son idénticos en `db/` y `.old/`, o se recuperan del historial de git.
- **Bases de datos**: eran versiones anteriores de las actuales de `db/`, que son más nuevas.

# Qué borré (12-10-2026)

| Ruta | Tamaño | Qué era |
|---|---|---|
| `~/gits/pollo/rym_lastfm/.zed/` (todo menos `settings.json`) | 4,4 GB | copia del repo del 19-03 |
| `~/gits/pollo/back_rym_lastfm/` | 4,4 GB | otra copia idéntica (mismo commit del 17-03) |
| `~/gits/pollo/rym_lastfm/.old/db/db_new/lastfm_cache.db` | 1,4 GB | base de datos anterior a `lastfm_cache_rym_new_normalized.db` |

Para conservar la configuración de Zed:

```bash
cp ~/gits/pollo/rym_lastfm/.zed/settings.json /tmp/zed-settings.json && rm -rf ~/gits/pollo/rym_lastfm/.zed && mkdir ~/gits/pollo/rym_lastfm/.zed && mv /tmp/zed-settings.json ~/gits/pollo/rym_lastfm/.zed/settings.json
```

# Lo que se perdió de verdad

- Dos versiones antiguas de las bases de datos (unos 723 MB), que no estaban ni en el repo ni en pepecono. Puede que sigan en el LFS de GitHub (no comprobado).
- `lastfm_cache.db`: `html_temporal.py` y `html_usuarios.py` todavía la usaban. Se adaptaron a la base normalizada: 7 rutas por defecto cambiadas y un error de SQL corregido en `db/db_reader.py` (SQLite no admite `sub(ts)` como alias de columnas).
- Un `db/.env` viejo (ya había uno más nuevo).

# Moraleja

- Antes de copiar un repo "por si acaso" para reescribir su historial, haz `git bundle` o `git clone --mirror` **fuera** del árbol de trabajo. Una copia dentro de una carpeta oculta se olvida y ocupa el doble.
- Borrar archivos grandes del árbol no reduce `.git` si siguen en el historial.

```bash
git count-objects -vH
```

```bash
git lfs ls-files -s | sort -k3 -h | tail
```

# Revertir

No se puede: se borró, no se movió. Para recuperar el historial viejo:

```bash
git clone pepecono:gits/pollo/rym_lastfm ~/rym_lastfm_viejo
```

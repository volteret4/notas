---
title : "Una playlist de YouTube por cada carpeta de Mixxx"
slug : "playlists-de-youtube-desde-mixxx"
date : "2026-10-31"
image : ""
tags : ["youtube", "yt-dlp", "google api", "mixxx", "gist", "python"]
categories : ["scripts"]
description : "yt_sync_mixxx.py crea una playlist privada de YouTube por cada .m3u de mi carpeta de DJ, busca los vídeos con yt-dlp para no gastar cuota y publica un listado en un gist."
---

# Qué hace

`~/Scripts/Musica/playlists/youtube/yt_sync_mixxx.py`:

- Crea una playlist **privada** en YouTube por cada `.m3u` de `/mnt/windows/Mix`, llamada `Mix - <nombre>`.
- Sincroniza por diferencias: añade lo nuevo y quita lo que ya no está en el `.m3u`. **Solo quita vídeos que añadió él**; lo que metas a mano no lo toca.
- Busca los vídeos con **yt-dlp**, que no gasta cuota. La API de YouTube solo se usa para escribir: cada inserción cuesta 50 unidades de las 10.000 diarias, es decir, unas 200 canciones al día.
- Con unas 1.900 canciones, la primera sincronización completa tarda **unos 10 días**. Al acabarse la cuota guarda el estado y la siguiente ejecución sigue donde lo dejó. Mientras tanto, sigue buscando vídeos con yt-dlp para el listado.
- Si YouTube da un 409 (pasa justo después de crear una playlist) o un 5xx, reintenta con espera creciente.

# Archivos

| Archivo | Qué es |
|---|---|
| `~/.config/yt_sync/client_secret.json` | credencial OAuth del proyecto de Google Cloud (copiada de `~/Descargas`) |
| `~/.config/yt_sync/` (token y `state.json`) | autorización y qué vídeo corresponde a cada canción |
| `~/.config/yt_sync/Mix.md` | listado en markdown |

En el proyecto de Google Cloud tiene que estar activada **YouTube Data API v3**, y tu cuenta tiene que estar como usuario de prueba de la pantalla de consentimiento. Si no, sale `403 API not enabled` o `access_denied`.

# Uso

La primera vez abre el navegador para autorizar:

```bash
~/Scripts/python_venv/bin/python ~/Scripts/Musica/playlists/youtube/yt_sync_mixxx.py --dry-run --only dub
```

```bash
~/Scripts/python_venv/bin/python ~/Scripts/Musica/playlists/youtube/yt_sync_mixxx.py
```

| Opción | Qué hace |
|---|---|
| `--dry-run` | muestra lo que haría, sin crear nada (si no ves la playlist, mira si lo lanzaste así) |
| `--only <carpeta>` | solo esa playlist (no distingue mayúsculas) |
| `--retry-missing` | vuelve a buscar las canciones que no encontró |
| `--markdown` | genera `Mix.md` |
| `--publish` | además actualiza el gist |
| `--no-sync` | solo el listado, sin tocar YouTube |

Si una búsqueda trae un vídeo equivocado, se corrige a mano en `~/.config/yt_sync/state.json`.

# El listado en un gist

`Mix.md` tiene la fecha, el total de canciones y cuántas tienen enlace, un índice, una cabecera `##` por carpeta y `###` por subgénero, cada una con su enlace **▶ playlist**, y cada canción enlazada a su vídeo si se encontró. Se publica en un **gist secreto** de `volteret4` que mantiene siempre la misma URL. Con la API gratuita de pastebin, cada cambio crearía un enlace nuevo.

# Integración con el mover

`mover_mix_playerctl.py` lanza la sincronización en segundo plano, con `--publish`, solo para la carpeta de destino (y la de origen si mueves entre carpetas). Si aún no has autorizado, no abre el navegador: avisa con una notificación.

# Revertir

- Borra las playlists `Mix - *` desde YouTube Studio.
- Quita la llamada a `yt_sync_mixxx.py` en `mover_mix_playerctl.py` (la versión de antes es `mover_mix_playerctl.py.bak.20261016201948`).
- Borra el script y la configuración:

```bash
rm -r ~/.config/yt_sync ~/Scripts/Musica/playlists/youtube/yt_sync_mixxx.py
```

- Retira el acceso de la app en tu cuenta de Google (*Seguridad → Apps de terceros*).

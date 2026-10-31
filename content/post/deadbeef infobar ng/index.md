---
title : "Un panel propio para DeaDBeeF con letras, artista, álbum y similares"
slug : "deadbeef-infobar-ng"
date : "2026-10-31"
image : ""
tags : ["deadbeef", "gtk", "c", "plugin", "lyrics", "musicbrainz", "lastfm"]
categories : ["apps"]
description : "El plugin infobar de DeaDBeeF lleva abandonado desde 2014. Lo sustituí por un plugin GTK3 mínimo en C que delega la búsqueda de datos en scripts de Python."
---

# Por qué

El plugin [deadbeef-infobar](https://github.com/DeaDBeeF-Player/deadbeef-infobar) tiene su último commit en 2014. Lo compilé en `~/gits/deadbeef-infobar`, pero no aparecía en DeaDBeeF ni en el modo diseño. En lugar de arreglarlo, hice uno nuevo: **Infobar NG**.

# Cómo está hecho

El plugin en C solo crea el widget y lanza scripts. Los datos los buscan scripts de Python, mucho más fáciles de cambiar.

| Archivo | Qué hace |
|---|---|
| `~/Scripts/Musica/deadbeef/infobar/plugin.c` | registra el widget `infobar_ng` en gtkui y escucha `DB_EV_SONGCHANGED` |
| `infobar/ui.c` | cabecera "Artista — Título" con spinner y un `GtkNotebook` con 4 pestañas de solo lectura: **Letras, Artista, Álbum, Similares** |
| `infobar/runner.c` | ejecuta los scripts en segundo plano con un número de "generación", para descartar resultados de una canción anterior si cambias rápido |
| `letras_ovh.py` | letra desde lyrics.ovh |
| `letras_genius.py` | letra desde Genius si lyrics.ovh falla (token `GENIUS_TOKEN` en `.encrypted.env` con sops) |
| `bio_artista.py` | biografía (Last.fm, MusicBrainz, Wikipedia) |
| `info_album.py` | información del álbum (MusicBrainz, Wikipedia, Last.fm) |
| `similares.py` | canciones similares en Last.fm o, si no, artistas relacionados en MusicBrainz |

Los scripts reciben artista y título (o álbum) como argumentos y escriben texto plano en stdout. El intérprete se puede cambiar con la opción `infobar_ng.python_cmd` de DeaDBeeF; por defecto usan el shebang `~/Scripts/python_venv/bin/python`.

# Compilar e instalar

Necesita las cabeceras de DeaDBeeF (`/usr/include/deadbeef`) y GTK3.

```bash
cd ~/Scripts/Musica/deadbeef/infobar && make install
```

`make install` copia:

- `ddb_infobar_ng.so` en `~/.local/lib/deadbeef/`,
- los cinco scripts en `~/.config/deadbeef/infobar_scripts/`.

Después, reinicia DeaDBeeF, entra en *Ver → Modo diseño* y añade el widget **Infobar NG** donde quieras. En mi layout está como primera pestaña de un `tabs` debajo de la carátula, junto a "propiedades" y "visuals".

Si cambias un script en `~/Scripts/Musica/deadbeef/`, vuelve a ejecutar `make install` para copiarlo a `infobar_scripts`.

# Revertir

En *Modo diseño*, quita el widget y luego:

```bash
rm ~/.local/lib/deadbeef/ddb_infobar_ng.so && rm -r ~/.config/deadbeef/infobar_scripts
```

El clon del plugin viejo (`~/gits/deadbeef-infobar`) no se usa y se puede borrar.

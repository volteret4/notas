---
title : "Odesli con API key y una tabla markdown de enlaces"
slug : "odesli-api-key-y-tabla-markdown"
date : "2026-10-31"
image : ""
tags : ["odesli", "songlink", "python", "copyq", "markdown"]
categories : ["scripts"]
description : "odesli.py lee ODESLI_API_KEY del .env, y odesli_table.py convierte los últimos enlaces copiados en una tabla markdown con un enlace por plataforma."
---

# odesli.py con API key

`~/Scripts/utilities/open_search_links/music_player/odesli.py` coge el álbum que suena (DeaDBeeF o Airsonic) y copia su enlace de song.link/album.link. Va en la **tecla círculo** (keycode 191) en awesome.

Sin clave, la API de Odesli limita mucho las peticiones. Ahora lee `ODESLI_API_KEY` del `.env` de la misma carpeta y la añade como parámetro `key`:

```python
        load_dotenv(dotenv_path="/home/huan/Scripts/utilities/open_search_links/music_player/.env")
        ...
        self.odesli_api_key = os.getenv('ODESLI_API_KEY')
```

```python
            params = {"url": platform_url}
            if self.odesli_api_key:
                params["key"] = self.odesli_api_key
            response = requests.get(api_endpoint, params=params, timeout=15)
```

En `.env`:

```
ODESLI_API_KEY=tu_clave
```

La clave se pide al equipo de Odesli (developers@song.link). Sin ella, el script sigue funcionando igual.

# odesli_table.py

`~/Scripts/utilities/open_search_links/music_player/odesli_table.py` compone una tabla markdown con los últimos álbumes que he buscado:

| Artista | Álbum | Enlaces |
|---|---|---|
| Daft Punk | Discovery | [YT Music](…) · [YouTube](…) · [Spotify](…) · … |

- Lee los últimos elementos del historial de **CopyQ** (30 por defecto, `--scan N` para cambiarlo) y se queda con los que tienen el formato que deja `odesli.py` (`*artista* - álbum: url`), sin duplicados.
- Por cada uno vuelve a consultar Odesli con su propia URL (`song.link`/`album.link` sirven como entrada) para sacar `linksByPlatform`.
- Enlaces por prioridad: YT Music, YouTube, Spotify, Apple Music, Deezer, Tidal, SoundCloud y Amazon Music. Si Odesli no devuelve ninguno (pasa con lo que solo está en Bandcamp, que Odesli no admite), pone el enlace general.
- Copia la tabla con `copyq add` y avisa con una notificación.

```bash
~/Scripts/python_venv/bin/python ~/Scripts/utilities/open_search_links/music_player/odesli_table.py --scan 50
```

Ojo: CopyQ tiene un límite de elementos, y cada `copyq add` puede echar del historial los más antiguos.

Para leer el historial de CopyQ desde un script, el separador y la lectura tienen que ir en la **misma** llamada:

```bash
copyq separator '\n---\n' read 0 1 2
```

# Revertir

`odesli.py` está en el repo de `~/Scripts` sin commitear:

```bash
git -C ~/Scripts diff -- utilities/open_search_links/music_player/odesli.py
```

```bash
git -C ~/Scripts checkout -- utilities/open_search_links/music_player/odesli.py
```

Y para quitar el script de la tabla:

```bash
rm ~/Scripts/utilities/open_search_links/music_player/odesli_table.py
```

---
title : "Surfingkeys en MixesDB y Chosic, y buscar en Nicotine+ desde un atajo"
slug : "surfingkeys-mixesdb-chosic-y-nicotine"
date : "2026-10-31"
image : ""
tags : ["surfingkeys", "javascript", "nicotine", "soulseek", "awesomewm", "plugins"]
categories : ["scripts"]
description : "Atajos de Surfingkeys para saltar al siguiente vídeo de YouTube en las tracklists de mixesdb.com y chosic.com, y un plugin de Nicotine+ para lanzar búsquedas desde mis scripts."
---

# Surfingkeys

Los fragmentos van en la configuración de Surfingkeys (en el navegador Zen), **después** del `` `; `` que cierra `settings.theme = `...``.

## El error típico

```
[SurfingKeys] Error found in settings: SyntaxError: `` literal not terminated before end of script
```

O el código se pegó **dentro** de la plantilla del tema (entre las comillas invertidas), y entonces se trata como CSS y no se ejecuta, o se borró la línea `` `; `` que la cierra. El final del tema debe quedar así, y el JavaScript va debajo:

```
    }
  }
`;

// MixesDB: ...
```

## MixesDB: siguiente y anterior play

En mixesdb.com, las canciones con enlace a YouTube tienen un icono de play diminuto (`i.aff-icon.aff-play`). El script busca cuál está sonando (el que tiene el `iframe` dentro de su `.aff-wrapper`), lo cierra y pulsa el siguiente.

```javascript
const mdbPlays = () => [...document.querySelectorAll('i.aff-icon.aff-play')].filter(el => el.offsetParent !== null);
let mdbIdx = -1;
const mdbCurrent = (plays) => {
    const iframe = document.querySelector('.aff-wrapper-apendix iframe');
    if (iframe) {
        const i = plays.findIndex(p => p.closest('.aff-wrapper').contains(iframe));
        if (i !== -1) return i;
    }
    return mdbIdx;
};
const mdbGo = (step) => {
    const plays = mdbPlays();
    if (!plays.length) return api.Front.showBanner('No hay botones play');
    const cur = mdbCurrent(plays);
    const next = cur === -1 ? (step > 0 ? 0 : plays.length - 1) : cur + step;
    if (next < 0 || next >= plays.length) return api.Front.showBanner('No hay más');
    if (cur !== -1) {
        const w = plays[cur].closest('.aff-wrapper');
        w.querySelectorAll('.aff-wrapper-apendix').forEach(a => a.remove());
        const d = w.querySelector('.aff-details');
        if (d && d.offsetParent !== null) plays[cur].click();
    }
    mdbIdx = next;
    plays[next].scrollIntoView({block: 'center'});
    plays[next].click();
};
api.mapkey('N', 'MixesDB: siguiente play', () => mdbGo(1), {domain: /mixesdb\.com/i});
api.mapkey('P', 'MixesDB: play anterior', () => mdbGo(-1), {domain: /mixesdb\.com/i});
```

- `N` (Shift+n): siguiente. `P` (Shift+p): anterior.
- Al tener `{domain: /mixesdb\.com/i}`, solo cambian en mixesdb. En el resto de webs, `N` sigue repitiendo la búsqueda hacia atrás.

## Chosic: siguiente vídeo del generador de playlists

En `chosic.com/playlist-generator/`, al pulsar el icono de YouTube de una canción, chosic mete un `<iframe id="yt-iframe">` dentro del div cuyo `id` es el `data-song-id` de esa canción. Así se sabe cuál suena.

```javascript
// Chosic playlist generator: siguiente / anterior vídeo de YouTube
const chYts = () => [...document.querySelectorAll('.icon-img-yt[data-track-index]')]
    .filter(el => +el.dataset.trackIndex > 0 && el.offsetParent !== null)
    .sort((a, b) => a.dataset.trackIndex - b.dataset.trackIndex);
let chIdx = -1;
const chCurrent = (yts) => {
    const iframe = document.getElementById('yt-iframe');
    if (iframe) {
        const i = yts.findIndex(y => y.dataset.songId === iframe.parentElement.id);
        if (i !== -1) return i;
    }
    const activa = yts.findIndex(y => y.closest('.song-div.active-song'));
    return activa !== -1 ? activa : chIdx;
};
const chGo = (step) => {
    const yts = chYts();
    if (!yts.length) return api.Front.showBanner('No hay vídeos de YouTube');
    const cur = chCurrent(yts);
    const next = cur === -1 ? (step > 0 ? 0 : yts.length - 1) : cur + step;
    if (next < 0 || next >= yts.length) return api.Front.showBanner('No hay más');
    chIdx = next;
    yts[next].scrollIntoView({block: 'center'});
    yts[next].click();
};
api.mapkey(']y', 'Chosic: siguiente vídeo YT', () => chGo(1), {domain: /chosic\.com/i});
api.mapkey('[y', 'Chosic: vídeo YT anterior', () => chGo(-1), {domain: /chosic\.com/i});
```

Si los selectores dejan de funcionar, es que chosic ha cambiado su `all-tools.js`.

# Buscar en Nicotine+ desde mis scripts

Nicotine+ no acepta búsquedas por línea de comandos, así que uso un **plugin** que vigila un archivo:

1. El script escribe el término en `$XDG_RUNTIME_DIR/nicotine_busqueda` (uno por línea) y lanza `nicotine`. Si ya está abierto, solo trae la ventana al frente.
2. Cada medio segundo, el plugin mira si existe el archivo. Si hay sesión iniciada, lanza la búsqueda global y borra el archivo. Si Nicotine está desconectado, la búsqueda espera a que se conecte.

## El plugin

`~/.local/share/nicotine/plugins/busqueda_externa/PLUGININFO`:

```
Version = "2026-10-09r00"
Authors = ["volteret4"]
Name = "Búsqueda externa"
Description = "Lanza búsquedas escritas por scripts externos en $XDG_RUNTIME_DIR/nicotine_busqueda"
```

`~/.local/share/nicotine/plugins/busqueda_externa/__init__.py`:

```python
# Búsqueda externa: los scripts escriben el término en $XDG_RUNTIME_DIR/nicotine_busqueda
# (un término por línea) y este plugin lo busca en la red global en cuanto hay sesión.

import os

from pynicotine.events import events
from pynicotine.pluginsystem import BasePlugin
from pynicotine.slskmessages import UserStatus

FICHERO = os.path.join(os.environ.get("XDG_RUNTIME_DIR", "/tmp"), "nicotine_busqueda")


class Plugin(BasePlugin):

    def loaded_notification(self):
        self.evento = events.schedule(delay=0.5, callback=self.comprobar, repeat=True)

    def disable(self):
        events.cancel_scheduled(self.evento)

    def comprobar(self):
        if self.core.users.login_status == UserStatus.OFFLINE or not os.path.exists(FICHERO):
            return

        try:
            with open(FICHERO, encoding="utf-8") as f:
                terminos = [linea.strip() for linea in f if linea.strip()]
            os.remove(FICHERO)
        except OSError as error:
            self.log("No se pudo leer %s: %s", (FICHERO, error))
            return

        for termino in terminos:
            self.core.search.do_search(termino, "global")
```

Se activa en Nicotine+ → *Preferencias* → *Plugins* → **Búsqueda externa**. No lo actives editando la configuración con Nicotine abierto, porque la sobrescribe al cerrarse.

## Los scripts

En `~/Scripts/utilities/open_search_links/music_player/_comun.sh`:

```bash
buscar_nicotine() {
    local termino
    termino="$(echo "$1" | sed 's/[][(){}"'"'"'.,:;!?&/_+*-]/ /g; s/  */ /g; s/^ //; s/ $//')"
    [[ -z "$termino" ]] && return 1
    echo "$termino" >> "${XDG_RUNTIME_DIR:-/tmp}/nicotine_busqueda"
    # Si ya está abierto, la segunda instancia solo trae la ventana al frente
    nicotine > /dev/null 2>&1 &
    disown
}
```

Quita la puntuación porque en Soulseek una palabra que empieza por `-` **excluye** resultados, y con títulos tipo "Artista - Canción (X Remix)" se perderían.

- `music_player/nicotine_metadata.sh`: busca lo que suena, con **artista + canción** (no el álbum, que no sirve para remezclas sueltas).
- `busqueda_en.sh nicotine`: busca el texto seleccionado.

## Atajos en awesome

La `N` estaba ocupada, así que comenté `Mod+N` (abrir `~/notes` en Emacs, en `keys_second_brain.lua`) y `Mod+Shift+N` (`lain.util.add_tag()` en `rc.lua`). Ahora:

```lua
    awful.key({ modkey }, "n",
        function() awful.spawn.with_shell("$HOME/Scripts/utilities/open_search_links/busqueda_en.sh nicotine &") end,
        { description = "Buscar en Nicotine+", group = "Buscar en..." }),
    awful.key({ modkey, "Shift" }, "n",
        function() awful.spawn.with_shell("$HOME/Scripts/utilities/open_search_links/music_player/nicotine_metadata.sh &") end,
        { description = "Buscar metadata en Nicotine+", group = "Buscar en..." }),
```

Sigue la lógica del resto de buscadores: `Mod` busca la selección y `Mod+Shift`, lo que está sonando.

# Revertir

- **Surfingkeys**: borra los bloques `MixesDB` y `Chosic` de la configuración.
- **Nicotine**: desactiva el plugin y bórralo.

```bash
rm -r ~/.local/share/nicotine/plugins/busqueda_externa ~/Scripts/utilities/open_search_links/music_player/nicotine_metadata.sh
```

- Quita `buscar_nicotine()` de `_comun.sh` y la rama `nicotine` de `busqueda_en.sh`.
- En `rc.lua`, borra los dos atajos de Nicotine y descomenta el `add_tag` de `Mod+Shift+N` y el `Mod+N` de `keys_second_brain.lua`.

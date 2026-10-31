---
title : "hotkeyz, un buscador de todos mis atajos y aliases"
slug : "hotkeyz-buscador-de-atajos"
date : "2026-10-31"
image : ""
tags : ["rofi", "awesomewm", "zsh", "zellij", "atuin", "fzf", "python", "flask"]
categories : ["scripts"]
description : "Un CSV con todos los atajos de awesomewm, zellij, zsh y los scripts de atuin, un rofi y un fzf para buscarlos y lanzarlos, y una web para editarlo."
---

# Qué es

Con tantos atajos repartidos entre awesomewm, zellij, zsh y atuin, ya no me acordaba de la mitad. `hotkeyz` los junta todos en un CSV y los muestra en un buscador.

| Pieza | Dónde | Cómo se abre |
|---|---|---|
| Generador del CSV | `~/gits/pollo/hotkeyz/scripts/generar_hotkeys_csv.py` | lo lanzan el rofi y `popo` antes de mostrar la lista |
| CSV | `~/.config/hotkeys/hotkeys.csv` | — |
| Rofi | `~/gits/pollo/hotkeyz/scripts/ver_hotkeys.py` | `Super+Shift+V` |
| Terminal (fzf) | `~/gits/pollo/hotkeyz/scripts/popo.py` | `popo` o `popo <fuente>` con Tab |
| Web (Flask) | `~/gits/pollo/hotkeyz/app.py` en `127.0.0.1:5857` | `Super+Ctrl+Shift+V` (`launch.sh --open`) |
| Descripciones editadas en la web | `~/.config/hotkeys/overrides.json` | — |

# Fuentes que lee

- **AwesomeWM**: `rc.lua`, `keys_second_brain.lua` y `keys_shazam.lua`. Los keycodes `#N` se traducen al nombre de la tecla según el teclado (`Super+#86` → `Super+Num +`, `#191` → `F13`).
- **Aliases de zsh**: `~/.zsh_aliases`.
- **bindkeys de zsh**: la sección `# --- TECLAS ---` del `~/.zshrc`.
- **Zellij**: `~/.config/zellij/config.kdl`. La acción se guarda como comando `zellij action ...` completo.
- **Scripts de atuin**: la salida de `atuin scripts list`. Se lanzan con `atuin scripts run NOMBRE`.

Formato del CSV:

```
enabled,source,key,description,action
1,awesomewm/APPS,F12,dropdown tdrop xfce4,
```

Al regenerarlo se conservan la columna `enabled` y las descripciones que hayas cambiado en la web (`overrides.json`). Los atajos repetidos con la misma fuente y tecla no se guardan en `overrides.json`.

# Atajos en awesome

En `~/.config/awesome/rc.lua`, grupo "Menú":

```lua
    awful.key({ modkey, "Shift" }, "v",
        function() awful.spawn.with_shell("python3 $HOME/Scripts/menus/rofi/ver_hotkeys.py") end,
        { description = "buscar hotkeys/aliases", group = "Menú" }),

    awful.key({ modkey, "Control", "Shift" }, "v",
        function() awful.spawn.with_shell("$HOME/gits/pollo/hotkeyz/launch.sh --open") end,
        { description = "editar hotkeys (web)", group = "Menú" }),
```

`~/Scripts/menus/rofi/ver_hotkeys.py` y `generar_hotkeys_csv.py` son **enlaces simbólicos** a los de `~/gits/pollo/hotkeyz/scripts/`, para no tener dos copias.

# `popo` en el terminal

Al final del `~/.zshrc`, con autocompletado por fuente:

```zsh
alias popo='python3 /home/huan/gits/pollo/hotkeyz/scripts/popo.py'

_popo() {
    local -a sources
    if [[ -f "$HOME/.config/hotkeys/hotkeys.csv" ]]; then
        sources=(${(f)"$(awk -F, 'NR>1 && $1=="1" {print $2}' "$HOME/.config/hotkeys/hotkeys.csv" | sort -u)"})
    fi
    _arguments '1: :(${sources[@]})'
}
compdef _popo popo
```

Las acciones de zellij se ejecutan sin terminal, con un límite de 5 segundos, y si fallan muestran el error en vez de quedarse colgadas.

# La web

- **Editor del CSV**: activar o desactivar entradas, con "todos" y "ninguno" por sección, y editar descripciones.
- **Cheatsheets**: se generan a partir del CSV y se exportan a PDF, PNG o ventana flotante. Además trae seis predefinidas en `data/cheatsheets/` (emacs, vim, surfingkeys, zed, zellij y tmux).

Escucha **solo en `127.0.0.1`** y sin `debug`. Al principio estaba en `0.0.0.0` con `debug=True`, que deja la consola de Werkzeug (capaz de ejecutar código) abierta a toda la red.

`launch.sh --open` arranca el servidor en segundo plano si no está corriendo y abre el navegador.

# Revertir

Las copias de antes de los cambios del 23-10:

- `~/gits/pollo/hotkeyz.bak.20261023040026/` (proyecto entero)
- `~/.config/hotkeys/hotkeys.csv.bak.20261023040026`
- `~/.config/awesome/rc.lua.bak.20261023040026`
- `~/Scripts/menus/rofi/ver_hotkeys.py.bak.20261023040026` y `generar_hotkeys_csv.py.bak.20261023040026`

Para volver a la versión de junio:

```bash
rm -r ~/gits/pollo/hotkeyz && cp -a ~/gits/pollo/hotkeyz.bak.20261023040026 ~/gits/pollo/hotkeyz
```

Para quitarlo del todo: borra `~/gits/pollo/hotkeyz`, `~/.config/hotkeys`, los dos enlaces de `~/Scripts/menus/rofi/`, los dos `awful.key` de `rc.lua` y el bloque `POPO` del `.zshrc`.

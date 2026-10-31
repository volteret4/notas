---
title : "Layouts de zellij por proyecto desde una plantilla y rofi"
slug : "zellij-layouts-por-proyecto"
date : "2026-10-31"
image : ""
tags : ["zellij", "rofi", "awesomewm", "terminal"]
categories : ["terminal"]
description : "Una plantilla kdl con el cwd como variable, un script que genera un layout por proyecto y un rofi que abre cualquiera en una pestaña nueva."
---

# Qué hace

Cada proyecto tiene un layout de zellij de una sola pestaña:

```
┌──────────────┬──────────────────────┐
│              │ yazi en el proyecto   │
│   claude     ├──────────────────────┤
│  --continue  │ ssh a pepecono, en la │
│   (40 %)     │ misma ruta            │
└──────────────┴──────────────────────┘
```

Con `Super+Shift+Z` se abre un rofi con todos los layouts y el elegido se abre como pestaña nueva en la sesión de zellij actual.

# Archivos

| Archivo | Qué es |
|---|---|
| `~/.config/zellij/templates/claude_proyecto.kdl` | plantilla con `{{PROJECT_DIR}}` y `{{REMOTE_DIR}}` |
| `~/.config/zellij/templates/rutas_proyectos_claude.txt` | una ruta de proyecto por línea |
| `~/Scripts/menus/rofi/generar_layouts_claude.sh` | genera un `.kdl` por ruta en `~/.config/zellij/layouts/claude/` |
| `~/Scripts/menus/rofi/zellij_claude_tab.sh` | rofi que lista los layouts |
| `~/.config/awesome/rc.lua` | atajo `Super+Shift+Z` |

# La plantilla

`~/.config/zellij/templates/claude_proyecto.kdl`:

```kdl
layout {
    cwd "{{PROJECT_DIR}}"
    pane size=1 borderless=true {
        plugin location="zellij:tab-bar"
    }
    pane split_direction="vertical" {
        pane size="40%" {
            cwd "{{PROJECT_DIR}}"
            command "/home/huan/.local/bin/claude"
            args "--continue"
        }
        pane split_direction="horizontal" {
            pane {
                command "yazi"
                args "{{PROJECT_DIR}}"
            }
            pane {
                command "ssh"
                args "-t" "pepecono" "cd {{REMOTE_DIR}} && exec $SHELL"
            }
        }
    }
    pane size=1 borderless=true {
        plugin location="zellij:status-bar"
    }
}
```

## Quitar las líneas de *Tip*

Las líneas de consejos que no salen en el layout por defecto vienen del plugin `status-bar` cuando tiene **dos líneas de alto** (`size=2`), que es como lo trae el layout `default`. Con `size=1` solo muestra la barra de modos, sin tips.

# El generador

`~/Scripts/menus/rofi/generar_layouts_claude.sh`. `REMOTE_DIR` es la misma ruta cambiando `/home/huan/` por `/home/pepe/`:

```bash
#!/usr/bin/env bash

TEMPLATE="$HOME/.config/zellij/templates/claude_proyecto.kdl"
RUTAS="$HOME/.config/zellij/templates/rutas_proyectos_claude.txt"
DEST="$HOME/.config/zellij/layouts/claude"

rm "$DEST"/*.kdl

while IFS= read -r ruta || [[ -n "$ruta" ]]; do
    [[ -z "$ruta" ]] && continue
    nombre=$(basename "$ruta")
    destino="$DEST/$nombre.kdl"
    if [[ -f "$destino" ]]; then
        echo "Ya existe, omitiendo: $destino"
        continue
    fi
    remote_dir="${ruta/\/home\/huan\//\/home\/pepe\/}"
    sed -e "s|{{PROJECT_DIR}}|$ruta|g" \
        -e "s|{{REMOTE_DIR}}|$remote_dir|g" \
        "$TEMPLATE" > "$destino"
    echo "Creado: $destino"
done < "$RUTAS"
```

Para añadir un proyecto, añade su ruta al `.txt` y vuelve a lanzar el script:

```bash
echo "$HOME/gits/pollo/nuevo" >> ~/.config/zellij/templates/rutas_proyectos_claude.txt && bash ~/Scripts/menus/rofi/generar_layouts_claude.sh
```

Ojo: el script **borra** todos los `.kdl` de `layouts/claude/` antes de generarlos. No edites a mano los layouts de esa carpeta.

# El rofi

`~/Scripts/menus/rofi/zellij_claude_tab.sh` lista los `.kdl` de `~/.config/zellij/layouts/` (primer nivel) y de `layouts/claude/`:

```bash
#!/usr/bin/env bash

LAYOUTS_ROOT="$HOME/.config/zellij/layouts"
LAYOUTS_CLAUDE="$HOME/.config/zellij/layouts/claude"
TEMPLATE="claude_proyecto.kdl"

declare -A layout_paths

while IFS= read -r f; do
    nombre=$(basename "$f" .kdl)
    layout_paths[$nombre]="$f"
done < <(
    find "$LAYOUTS_ROOT"  -maxdepth 1 -name "*.kdl" ! -name "$TEMPLATE"
    find "$LAYOUTS_CLAUDE" -maxdepth 1 -name "*.kdl"
)

seleccion=$(printf '%s\n' "${!layout_paths[@]}" | sort \
    | rofi -dmenu -i -p "Zellij tab: " -theme-str 'window {width: 600px;}')

[[ -z "$seleccion" ]] && exit 0

zellij action new-tab \
    --name "$seleccion" \
    --layout "${layout_paths[$seleccion]}"
```

En `~/.config/awesome/rc.lua`:

```lua
    awful.key({ modkey, "Shift" }, "z",
        function() awful.spawn.with_shell("bash $HOME/Scripts/menus/rofi/zellij_claude_tab.sh") end,
        { description = "Zellij: nueva tab proyecto Claude", group = "APPS" }),
```

Hay una variante para abrir claude dentro de una VM (`claude_vm_proyecto.kdl`, `generar_layouts_claude_vm.sh`, `zellij_claude_vm_tab.sh`); ver [VM para Claude Code](../vm-para-claude-code).

# Revertir

```bash
rm -r ~/.config/zellij/layouts/claude ~/.config/zellij/templates/claude_proyecto.kdl ~/.config/zellij/templates/rutas_proyectos_claude.txt ~/Scripts/menus/rofi/generar_layouts_claude.sh ~/Scripts/menus/rofi/zellij_claude_tab.sh
```

Y quita el bloque `awful.key({ modkey, "Shift" }, "z", ...)` de `rc.lua`.

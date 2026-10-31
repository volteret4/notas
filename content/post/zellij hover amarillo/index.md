---
title : "Zellij 0.45 resalta en amarillo al pasar el ratón"
slug : "zellij-hover-amarillo"
date : "2026-10-31"
image : ""
tags : ["zellij", "tema", "terminal"]
categories : ["terminal"]
description : "Tras actualizar a zellij 0.45, el hover del ratón sale en amarillo chillón. Falta el bloque multiplayer_user_colors en el tema."
---

# El problema

Después de actualizar zellij a la 0.45, al pasar el ratón por un panel (o al agrupar paneles) el borde y el texto de ayuda salían en un **amarillo** que no pegaba con el tema.

# La causa

La 0.45 añadió la sección `multiplayer_user_colors` a los temas. Si el tema no la define, zellij usa sus colores por defecto, y uno de ellos es ese amarillo. El hover y la selección de grupo usan el color del "jugador 1", así que se ve aunque estés solo.

# La solución

Añadir el bloque al tema activo dentro de `themes { ... }` en `~/.config/zellij/config.kdl`, con colores del propio tema. En mi caso, `dracula_2`:

```kdl
        // Without this block zellij falls back to its built-in defaults for any
        // unset player, one of which is a bright yellow that clashes with the
        // rest of the theme (this became visible on mouse hover/group-select
        // after updating to 0.45, which added multiplayer_user_colors).
        multiplayer_user_colors {
            player_1 189 147 249
            player_2 139 233 253
            player_3 80 250 123
            player_4 255 184 108
            player_5 255 121 198
            player_6 255 85 85
            player_7 98 114 164
            player_8 189 147 249
            player_9 139 233 253
            player_10 80 250 123
        }
```

Los valores son RGB separados por espacios. `player_1` (189 147 249) es el morado de Dracula. Con Catppuccin usa sus colores (mauve, sky, green…).

Si prefieres quitar el efecto, hay dos opciones en `config.kdl`:

```kdl
mouse_hover_effects false
advanced_mouse_actions false
```

Hay que cerrar la sesión de zellij para que aplique.

# Revertir

Borra el bloque `multiplayer_user_colors { ... }`. La versión de chezmoi del 23-05 todavía no lo tiene:

```bash
git -C ~/.local/share/chezmoi diff 3228277 2c1adac -- dot_config/zellij/config.kdl
```

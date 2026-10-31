---
title : "Ctrl+R de atuin no funciona dentro de zellij"
slug : "atuin-ctrl-r-zellij"
date : "2026-10-31"
image : ""
tags : ["atuin", "zellij", "zsh", "bindkey"]
categories : ["terminal"]
description : "Tras actualizar atuin y zellij, Ctrl+R dejó de abrir la búsqueda de atuin. Zellij se quedaba la tecla para su modo Resize."
---

# El problema

Un día después de actualizar **atuin** y **zellij**, `Ctrl+R` dejó de abrir el historial de atuin. Pistas:

- `atuin search -i` funcionaba.
- `bindkey "^R"` mostraba `"^R" atuin-search`, o sea que zsh tenía bien la tecla.
- `cat -v` no mostraba **nada** al pulsar `Ctrl+R`: la tecla no llegaba a la shell.
- Con otra letra (`bindkey '^Z' atuin-search`) funcionaba.

# Descartado

- **Downgrade de atuin** (18.19.0 → 18.17.1). No cambió nada, así que volví a la 18.19.0 enseguida. Hoy está en la 18.24.
- **awesomewm**. No tenía ningún atajo con `Ctrl+R`.

# La causa

En mi `config.kdl` el modo Resize está en `Ctrl n`, pero el bloque es `keybinds { ... }` **sin** `clear-defaults=true`. Por eso zellij seguía aplicando sus atajos por defecto, y uno de ellos es `Ctrl r` para Resize. Tras la actualización ese atajo volvió a tener prioridad y se comía la tecla.

# La solución

En `~/.config/zellij/config.kdl`, al final de `keybinds { ... }`:

```kdl
    // Resize mode was remapped to Ctrl n above, but the zellij default for
    // Ctrl r (also Resize mode) stays active since clear-defaults isn't set.
    // Unbind it so Ctrl+R reaches the shell (atuin history search) instead.
    shared_except "locked" {
        unbind "Ctrl r"
    }
```

Cierra **todas** las sesiones de zellij para que aplique (`zellij kill-all-sessions`).

En el `.zshrc`, atuin se carga sin la flecha arriba:

```zsh
eval "$(atuin init --disable-up-arrow zsh)"
```

# Diagnóstico rápido para otras teclas

Si `cat -v` no muestra nada al pulsar una tecla, algo por encima de la shell (zellij, tmux, el terminal o el gestor de ventanas) se la está quedando. Prueba fuera de zellij para saber cuál.

```bash
cat -v
```

# Ojo con chezmoi

Hasta el 31-10 el `config.kdl` que guardaba chezmoi no tenía este `unbind` ni el atajo `Ctrl y` de harpoon, y un `chezmoi apply` deshacía el arreglo. Ya está guardado (commit `94e48bb` de `dot_chezmoi`):

```bash
chezmoi add ~/.config/zellij/config.kdl
```

# Revertir

Borra el bloque `shared_except "locked" { unbind "Ctrl r" }`. Ojo: entonces `Ctrl+R` vuelve a abrir el modo Resize.

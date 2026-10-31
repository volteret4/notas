---
title : "Teclas Supr, Inicio, Fin y borrar palabras en zsh puro"
slug : "teclas-zsh-puro"
date : "2026-10-31"
image : ""
tags : ["zsh", "terminal", "bindkey"]
categories : ["terminal"]
description : "Al quitar oh-my-zsh dejan de ir Supr, Inicio/Fin y Ctrl+flechas. Así se recuperan con bindkey, más borrar por palabra o por tramo de ruta."
---

# El problema

Tras quitar oh-my-zsh (ver [zsh sin oh-my-zsh](../zsh-sin-oh-my-zsh)):

- **Supr** no borraba; solo funcionaba Retroceso.
- **Ctrl+←/→** escribía `D` o `C` en lugar de moverse por palabras.
- **Inicio/Fin** no hacían nada.

oh-my-zsh traía esos `bindkey` en su `lib/key-bindings.zsh`. Sin él, hay que ponerlos a mano en el `.zshrc`.

# La solución

Cada terminal (alacritty, zellij, tmux, rxvt) manda una secuencia distinta para la misma tecla, así que se definen todas en los tres keymaps.

```zsh
WORDCHARS='*?_-.[]~=&;!#$%^(){}<>'  # sin / → Alt+Backspace para en cada componente de ruta

function _backward_kill_full_word() {
    local WORDCHARS="${WORDCHARS}/"
    zle backward-kill-word
}
zle -N _backward_kill_full_word

function _kill_dir() {
    local WORDCHARS="${WORDCHARS/\//}"
    zle kill-word
}
zle -N _kill_dir

function _kill_full_word() {
    local WORDCHARS="${WORDCHARS}/"
    zle kill-word
}
zle -N _kill_full_word

for _keymap in emacs viins vicmd; do
  bindkey -M $_keymap "\e[H"    beginning-of-line          # Inicio
  bindkey -M $_keymap "\e[1~"  beginning-of-line          # Inicio (tmux/rxvt)
  bindkey -M $_keymap "\eOH"   beginning-of-line          # Inicio (xterm alt)
  bindkey -M $_keymap "\e[F"   end-of-line                # Fin
  bindkey -M $_keymap "\e[4~"  end-of-line                # Fin (tmux/rxvt)
  bindkey -M $_keymap "\eOF"   end-of-line                # Fin (xterm alt)
  bindkey -M $_keymap "\e[3~"  delete-char                # Supr
  bindkey -M $_keymap "^H"     _backward_kill_full_word  # Ctrl+Backspace
  bindkey -M $_keymap "\e[3;5~" _kill_full_word           # Ctrl+Supr
  bindkey -M $_keymap "\e[3;3~" _kill_dir                 # Alt+Supr
  bindkey -M $_keymap "\e[1;5C" forward-word  # Ctrl+→
  bindkey -M $_keymap "\e[1;5D" backward-word # Ctrl+←
  bindkey -M $_keymap "\e[5C"   forward-word  # Ctrl+→ (rxvt/tmux)
  bindkey -M $_keymap "\e[5D"   backward-word # Ctrl+← (rxvt/tmux)
  bindkey -M $_keymap "\eOc"    forward-word  # Ctrl+→ (xterm alt)
  bindkey -M $_keymap "\eOd"    backward-word # Ctrl+← (xterm alt)
done
unset _keymap
```

El truco está en `WORDCHARS`, los caracteres que zsh considera parte de una palabra:

| Tecla | Borra | Ejemplo con `cd ~/gits/pollo/claude` y el cursor al final |
|---|---|---|
| Alt+Retroceso | hasta la `/` anterior (zsh por defecto, porque `/` no está en `WORDCHARS`) | `cd ~/gits/pollo/` |
| Ctrl+Retroceso | la ruta entera (añade `/` a `WORDCHARS` solo en esa función) | `cd ` |
| Alt+Supr | hacia delante, hasta la siguiente `/` | — |
| Ctrl+Supr | hacia delante, la ruta entera | — |

# Saber qué secuencia manda una tecla

Si alguna no va en otro terminal, ejecuta `cat -v`, pulsa la tecla y verás la secuencia (`^[` equivale a `\e`). Luego añade una línea `bindkey` con ella.

```bash
cat -v
```

# Revertir

Borra la sección `# --- TECLAS ---` y la línea `WORDCHARS=` del `~/.zshrc`, y abre una shell nueva con `exec zsh`.

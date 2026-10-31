---
title : "Quitar oh-my-zsh y usar zsh puro"
slug : "zsh-sin-oh-my-zsh"
date : "2026-10-31"
image : ""
tags : ["zsh", "terminal", "alias", "atuin"]
categories : ["terminal"]
description : "Cómo pasé de oh-my-zsh a un .zshrc sin framework que mantiene autocompletado, syntax highlighting, dirhistory y copybuffer."
---

# Por qué

El autocompletado con Tab fallaba dentro de oh-my-zsh, y el framework cargaba muchas cosas que no usaba. Lo que me interesaba de él se puede hacer con unas pocas líneas de zsh y dos plugins sueltos.

# Qué queda en el `.zshrc`

El archivo nuevo (`~/.zshrc`) va por secciones:

| Sección | Qué hace | Sustituye a |
|---|---|---|
| Historial | `HISTFILE` distinto por TTY dentro de tmux, `hist_ignore_dups share_history` | — |
| Opciones | `setopt GLOB_DOTS` y `WORDCHARS` sin `/` | — |
| Completado | `autoload -Uz compinit; compinit` | completado de omz |
| Colored man pages | variables `LESS_TERMCAP_*` | plugin `colored-man-pages` |
| Command not found | `source /usr/share/doc/pkgfile/command-not-found.zsh` | plugin `command-not-found` |
| Copybuffer | función `_copybuffer` con xclip en `Ctrl+O` | plugin `copybuffer` |
| Dirhistory | funciones `_dh_*` en `Alt+←→↑↓` | plugin `dirhistory` |
| Teclas | Inicio, Fin, Supr y borrado por palabras | `key-bindings` de omz (ver [teclas en zsh puro](../teclas-zsh-puro)) |
| Atuin | `eval "$(atuin init --disable-up-arrow zsh)"` | — |
| Zoxide | `eval "$(zoxide init zsh)"` | plugin `z` |
| Prompt | `eval "$(starship init zsh)"` | tema de omz |
| Plugins externos | autosuggestions y syntax-highlighting, **siempre al final** | plugins de omz |

Los dos plugins externos:

```bash
source "$HOME/.zsh/zsh-autosuggestions/zsh-autosuggestions.zsh"
source "/usr/share/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh"
```

`zsh-syntax-highlighting` sale del paquete de Arch y `zsh-autosuggestions` es un clon de git en `~/.zsh/`.

```bash
sudo pacman -S zsh zsh-completions zsh-syntax-highlighting pkgfile zoxide starship atuin xclip
git clone https://github.com/zsh-users/zsh-autosuggestions ~/.zsh/zsh-autosuggestions
sudo pkgfile --update
```

Al final descarté el plugin `copyfile`. Los aliases siguen en `~/.zsh_aliases`, que se carga con `source`.

# Sección zinit para sistemas nuevos

Al principio del `.zshrc` hay una sección comentada para hacer lo mismo con **zinit**. Sirve en una máquina sin los paquetes instalados: se descomenta y se comentan las secciones "puras" equivalentes.

```zsh
# ZINIT_HOME="${XDG_DATA_HOME:-$HOME/.local/share}/zinit/zinit.git"
# if [[ ! -d "$ZINIT_HOME" ]]; then
#   mkdir -p "$(dirname $ZINIT_HOME)"
#   git clone https://github.com/zdharma-continuum/zinit.git "$ZINIT_HOME"
# fi
# source "$ZINIT_HOME/zinit.zsh"
#
# zinit light zsh-users/zsh-autosuggestions
# zinit light zsh-users/zsh-syntax-highlighting
# zinit snippet OMZ::plugins/colored-man-pages/colored-man-pages.plugin.zsh
# zinit snippet OMZ::plugins/dirhistory/dirhistory.plugin.zsh
# zinit snippet OMZ::plugins/copybuffer/copybuffer.plugin.zsh
# zinit snippet OMZ::plugins/command-not-found/command-not-found.plugin.zsh
```

# Lo que no se ha tocado

- El paquete `oh-my-zsh-git` **sigue instalado** y la carpeta `~/.oh-my-zsh` sigue ahí. El `.zshrc` simplemente no los carga.
- El paquete y la carpeta de oh-my-zsh siguen ahí porque pepecono, dietpi y doctrina todavía lo usan.

# chezmoi (31-10-2026)

El `.zshrc` nuevo ya está en chezmoi (commit `94e48bb` de `dot_chezmoi`):

- `templates/zshrc/arcolinux` es ahora una copia exacta de `~/.zshrc` (zsh puro).
- `dot_zshrc.tmpl` empieza con un `if`: en arcolinux escribe solo esa plantilla, **sin** `templates/zshrc/comunes`, que llevaba ajustes de oh-my-zsh. El resto de máquinas generan el mismo archivo que antes (comprobado con `chezmoi execute-template`).

```
{{- if eq .chezmoi.hostname "arcolinux" -}}
{{ include "templates/zshrc/arcolinux" }}
{{- else -}}
# ~/.zshrc generado por chezmoi
... (el bloque de antes, sin la rama de arcolinux)
{{ end -}}
```

`chezmoi diff ~/.zshrc` no muestra diferencias.

# Revertir

Volver a la versión de oh-my-zsh, recuperando de git la plantilla y el `.tmpl` anteriores:

```bash
cp ~/.zshrc ~/.zshrc.puro && git -C ~/.local/share/chezmoi checkout 94e48bb~1 -- dot_zshrc.tmpl templates/zshrc/arcolinux && chezmoi apply ~/.zshrc && exec zsh
```

Si cambias `~/.zshrc` a mano, cópialo a `~/.local/share/chezmoi/templates/zshrc/arcolinux`. Es una plantilla, así que `chezmoi re-add` no la actualiza.

Si quieres desinstalar oh-my-zsh del todo:

```bash
sudo pacman -Rns oh-my-zsh-git && rm -rf ~/.oh-my-zsh
```

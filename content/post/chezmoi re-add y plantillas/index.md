---
title : "chezmoi re-add y plantillas por hostname"
slug : "chezmoi-re-add-y-plantillas"
date : "2026-10-31"
image : ""
tags : ["chezmoi", "dotfiles", "git", "zsh"]
categories : ["git"]
description : "Cómo subir a chezmoi los archivos que has cambiado a mano, por qué re-add no toca las plantillas y el arreglo de los aliases, que solo salían en arcolinux."
---

# Subir los cambios hechos a mano

Si editas un archivo que ya controla chezmoi (por ejemplo `~/.config/zellij/config.kdl`), chezmoi no se entera hasta que lo vuelves a añadir:

```bash
chezmoi re-add
```

Sin argumentos, actualiza la copia de **todos** los archivos que han cambiado. Con una ruta, solo ese archivo:

```bash
chezmoi re-add ~/.config/zellij/config.kdl
```

Antes, mira qué cambiaría:

```bash
chezmoi status
```

```bash
chezmoi diff
```

En `chezmoi diff`, el signo `-` es tu archivo en `$HOME` y el `+` es lo que pondría chezmoi.

Como `.chezmoi.toml.tmpl` tiene `autoCommit = true` y `autoPush = true`, `re-add` hace commit y push solo.

## Ojo: `re-add` no toca las plantillas

Los archivos `.tmpl` (`dot_zshrc.tmpl`, `dot_zsh_aliases.tmpl`…) **no se actualizan con `re-add`**: chezmoi no sabe convertir el archivo final otra vez en plantilla. Hay que editar la plantilla, o lo que incluye, a mano:

```bash
chezmoi edit ~/.zsh_aliases
```

O edita directamente `~/.local/share/chezmoi/templates/zsh_aliases/arcolinux` (o `comunes`) y comprueba el resultado con:

```bash
chezmoi cat ~/.zsh_aliases | less
```

# Aliases que solo salían en arcolinux

El `.zshrc` se dividía bien por máquina, pero en pepecono y el resto de servidores el `~/.zsh_aliases` solo traía los comunes. `dot_zsh_aliases.tmpl` solo incluía el bloque adicional si el hostname era `arcolinux`, y en cualquier otro dejaba un comentario:

```diff
 # --- COMÚN ---
 {{- include "templates/zsh_aliases/comunes" }}
 
-# --- ESPECÍFICO POR HOSTNAME ---
-{{- if eq .chezmoi.hostname "arcolinux" }}
-{{- include "templates/zsh_aliases/arcolinux" }}
-{{- else }}
-# Hostname desconocido: {{ .chezmoi.hostname }}
-{{- end }}
+# --- ADICIONALES (igual en todos los hosts) ---
+{{- include "templates/zsh_aliases/arcolinux" }}
```

`dot_zshrc.tmpl` sí distingue por hostname (`arcolinux`, `pepe-cono`, `dietpi`, `doctrina`) y **falla** con un hostname desconocido (`{{ fail ... }}`). Si añades una máquina nueva, añádele su rama.

Para aplicarlo en cada servidor:

```bash
chezmoi update && chezmoi apply && exec zsh
```

Es lo mismo que el alias `chzd`. Para subir los cambios está `chzu`.

# Revertir

El cambio está en el commit `2c1adac` del repo de chezmoi (29-10, junto con otros archivos). Para recuperar solo la plantilla anterior:

```bash
git -C ~/.local/share/chezmoi checkout 2c1adac~1 -- dot_zsh_aliases.tmpl && chezmoi apply ~/.zsh_aliases
```

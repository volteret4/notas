---
title : "Recuperar .. y - en zsh sin oh-my-zsh"
slug : "zsh-punto-punto-y-guion"
date : "2026-10-31"
image : ""
tags : ["zsh", "alias", "terminal"]
categories : ["terminal"]
description : "Sin oh-my-zsh, .. da permiso denegado y - da command not found. Cuatro aliases los recuperan."
---

# El problema

Con oh-my-zsh, escribir `..` subía una carpeta y `-` volvía a la anterior. Sin él:

```
$ ..
zsh: permiso denegado: ..
$ -
zsh: command not found: -
```

Venía de `lib/directories.zsh` de oh-my-zsh, que define estos aliases y activa `auto_cd`.

# La solución

En `~/.zsh_aliases`:

```zsh
alias ..='cd ..'
alias ...='cd ../..'
alias ....='cd ../../..'
alias -- -='cd -'
```

El `--` en el último hace falta porque si no, `alias` interpreta `-` como una opción.

Otra opción es `setopt auto_cd`, que hace que escribir el nombre de una carpeta entre en ella. Así `..` funciona, pero `-` no, así que el alias hace falta igual.

Para navegar por el historial de carpetas también tienes **Alt+←/→** (la función dirhistory del `.zshrc`) y `z` (zoxide).

# Ojo con chezmoi

`~/.zsh_aliases` se genera desde `dot_zsh_aliases.tmpl` (plantillas `templates/zsh_aliases/comunes` y `arcolinux`). Si cambias el archivo a mano, pasa el cambio también a la plantilla o se perderá con el siguiente `chezmoi apply`.

# Revertir

Borra las cuatro líneas de `~/.zsh_aliases` (y de la plantilla de chezmoi si las copiaste).

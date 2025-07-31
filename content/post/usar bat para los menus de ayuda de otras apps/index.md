---
title : "Usar Bat Para Los Menus De Ayuda De Otras Apps"
date : "2025-07-31"
image : ""
tags : ["linux", "bat", "ayuda", "alias"]
categories : [pamplinas]
description : ""
---


Añade esto a tu `.zshrc`
```zsh
 alias -g -- -h='-h 2>&1 | bat --language=help --style=plain'
 alias -g -- --help='--help 2>&1 | bat --language=help --style=plain'
```

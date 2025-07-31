---
title : "Tmux Cerrar Resto De Sesiones"
date : "2025-07-31"
image : ""
tags : []
categories : [básico]
description : "Deja solo la sesión actual viva"
---



> [superuser](https://superuser.com/posts/1161344/timeline)

Prefijo TMUX (p. ej., Ctrl+B) + `:kill-session`

o

`tmux kill-session` (puede ejecutarse desde dentro o desde fuera de una sesión)

Ambos estilos de invocación pueden usar las siguientes banderas:

`-t target-session` destruye la sesión dada
`-a` destruye todas las sesiones excepto la dada o la que está asociada

Ejecutar `kill-session` desde fuera de TMUX elimina la última sesión a la que estaba asociada. `-a` invierte la operación.


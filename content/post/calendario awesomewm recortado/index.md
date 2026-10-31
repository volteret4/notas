---
title : "Calendario de awesomewm recortado"
slug : "calendario-awesomewm-recortado"
date : "2026-10-31"
image : ""
tags : ["awesomewm", "dunst", "lain", "calendario"]
categories : ["arch"]
description : "El calendario del reloj del tema powerarrow-blue solo mostraba la primera semana. Hay que darle alto y ancho fijos y subir el límite de alto en dunst."
---

# El problema

Al pasar el ratón por el reloj del tema **powerarrow-blue** se abre el calendario de `lain.widget.cal`, pero solo se veía la primera semana del mes. Después de un primer ajuste se veían cuatro líneas mal partidas: el mes empezaba en miércoles y el 6 saltaba de línea antes de tiempo.

El calendario es una notificación. Tenía dos límites:

1. **dunst** limitaba el alto de las notificaciones.
2. El `notification_preset` del widget no tenía tamaño, así que el texto en fuente monoespaciada se partía donde quería.

# Cambios

`~/.config/awesome/themes/powerarrow-blue/theme.lua`:

```diff
 theme.cal = lain.widget.cal({
     attach_to = { clock },
     notification_preset = {
         font = "Noto Sans Mono Medium 10",
         fg   = theme.fg_normal,
-        bg   = theme.bg_normal
+        bg   = theme.bg_normal,
+        height = 220,
+        width = 350
     }
 })
```

`~/.config/dunst/dunstrc`, en `[global]`:

```diff
-    #height = (0, 300)
+    height = (0, 600)
```

La fuente tiene que ser **monoespaciada** para que las columnas de los días cuadren.

Para aplicarlo hay que reiniciar dunst y awesome (`Super+Ctrl+R`):

```bash
killall dunst; dunst &
```

# Revertir

Los dos archivos están en chezmoi con la versión anterior (`dunstrc`) o con la nueva (`theme.lua`, desde el 29-10). Para ver las diferencias:

```bash
chezmoi diff ~/.config/dunst/dunstrc ~/.config/awesome/themes/powerarrow-blue/theme.lua
```

Para volver atrás, quita las líneas `height`/`width` del preset en `theme.lua` y vuelve a comentar `height` en `dunstrc`. `chezmoi apply ~/.config/dunst/dunstrc` también sirve, pero deshace además el `mouse_left_click = do_action, close_current` de las [notificaciones de GitHub](../notificaciones-de-github-en-el-escritorio).

---
title : "Dos monitores clonados al arrancar awesomewm"
slug : "xrandr-dos-monitores-al-arrancar"
date : "2026-10-31"
image : ""
tags : ["awesomewm", "xrandr", "sddm", "monitores"]
categories : ["arch"]
description : "Las dos pantallas mostraban lo mismo hasta abrir un terminal con F12. El xrandr estaba en .xinitrc, que SDDM no lee; hay que ponerlo en .xprofile."
---

# El problema

Al iniciar sesión, los dos monitores mostraban lo mismo (espejo). Al pulsar F12, que abre un terminal flotante en awesome, se colocaban bien.

# La causa

La línea de `xrandr` estaba en `~/.xinitrc` y comentada en `~/.config/awesome/autostart.sh`. Pero la sesión arranca con **SDDM**, y los gestores de inicio de sesión no leen `~/.xinitrc`: ese archivo solo lo usa `startx`. Lo que sí leen es `~/.xprofile`.

Hasta que algo forzaba a X a recalcular las pantallas, se quedaban en modo espejo.

# La solución

Crear `~/.xprofile`:

```bash
export XDG_CURRENT_DESKTOP=Awesome
xrandr --output HDMI-A-0 --mode 1920x1080 --pos 0x0 --rotate normal --primary --output DisplayPort-0 --mode 1920x1080 --pos 1920x0 --rotate normal
```

Los nombres de salida (`HDMI-A-0`, `DisplayPort-0`) salen de:

```bash
xrandr --listmonitors
```

| Archivo | Lo lee | Estado |
|---|---|---|
| `~/.xinitrc` | `startx` | tiene la misma línea (sin cambios) |
| `~/.xprofile` | SDDM, LightDM, GDM | **nuevo** |
| `~/.config/awesome/autostart.sh` | awesome | líneas viejas de xrandr comentadas (sin cambios) |

`~/.xprofile` no está en chezmoi. Si lo quieres guardar:

```bash
chezmoi add ~/.xprofile
```

# Revertir

```bash
rm ~/.xprofile
```

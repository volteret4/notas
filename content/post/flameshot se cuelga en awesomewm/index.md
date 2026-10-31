---
title : "Flameshot se cuelga sin error en awesomewm"
slug : "flameshot-se-cuelga-en-awesomewm"
date : "2026-10-31"
image : ""
tags : ["flameshot", "x11", "xdg-desktop-portal", "awesomewm", "capturas"]
categories : ["apps"]
description : "Desde flameshot 14 las capturas se piden al portal xdg, que en awesome no responde. Con useX11LegacyScreenshot=true vuelve a capturar directamente en X11."
---

# El problema

Flameshot dejó de funcionar. Incluso `flameshot full` se quedaba colgado **sin mostrar ningún error**. El daemon estaba registrado en D-Bus, pero la captura no terminaba nunca.

# La causa

Desde la **versión 14.0**, en X11 flameshot ya no captura la pantalla directamente: se la pide al servicio de capturas del escritorio, el portal `org.freedesktop.portal.Screenshot`. En mi sistema el portal lo da `xdg-desktop-portal-gtk`, y en awesome esa petición **nunca recibe respuesta**.

Con `gdb` se veía el hilo principal esperando en un bucle de eventos anidado. El propio programa trae el mensaje `Screenshot portal timed out after 30 seconds`.

(La actualización de qt6-base a 6.12 del mismo mes era una pista falsa.)

# El arreglo

En la sección `[General]` de `~/.config/flameshot/flameshot.ini`:

```ini
useX11LegacyScreenshot=true
```

Vuelve a usar la captura directa de X11. Es lo que recomienda flameshot para gestores de ventanas sin portal (i3, xmonad, awesome…). Avisa de que está *deprecated* y podría desaparecer en versiones futuras.

Reinicia el daemon y prueba:

```bash
pkill flameshot; flameshot & disown
```

```bash
flameshot full -p /tmp
```

Si algún día quitan la opción, la alternativa es tener un portal que funcione en awesome (`xdg-desktop-portal` + `xdg-desktop-portal-gtk`, con `XDG_CURRENT_DESKTOP` y un `portals.conf` que asigne `org.freedesktop.impl.portal.Screenshot`).

# Revertir

Borra la línea `useX11LegacyScreenshot=true` de `flameshot.ini` (hay una copia de la configuración en `~/.config/flameshot/flameshot.ini.bak.20261023184500`) y reinicia flameshot.

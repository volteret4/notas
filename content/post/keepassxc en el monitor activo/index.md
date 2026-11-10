---
title : "KeePassXC en el monitor activo y Super+Q a la bandeja"
slug : "keepassxc-monitor-activo-awesomewm"
date : "2026-11-10"
image : ""
tags : ["keepassxc", "awesomewm", "xdotool", "awesome-client", "multimonitor"]
categories : ["apps"]
description : "Con dos monitores, KeePassXC siempre aparecía en el mismo. No era una regla de rc.lua: es una app de instancia única y awesome salta a donde está la ventana. Además, Super+Q la manda a la bandeja sin cerrar la base de datos."
---

# Síntoma

- Con Super+F4 (`~/Scripts/Apps/kp_busqueda.sh`), KeePassXC siempre salía en el mismo monitor, aunque estuviera trabajando en el otro.
- Super+Q la minimizaba en vez de quitarla de en medio.

# Causa

No había ninguna regla en `rc.lua` que la atara a un monitor o etiqueta. Las únicas que le afectaban eran:

- `class = "keepassxc*", role = "gimp-image-window"`: un resto de una regla de GIMP que nunca coincidía.
- `"KeePassXC"` en la regla de flotantes centradas: solo la pone flotante y centrada en la pantalla donde se crea.

El motivo real:

1. KeePassXC es de **instancia única**. La ventana se crea una vez, en el monitor y la etiqueta enfocados en ese momento, y se queda ahí.
2. Las siguientes veces, `kp_busqueda.sh` solo vuelve a mostrar esa ventana y la activa con `xdotool windowactivate`.
3. Cuando una ventana pide activarse, awesome te lleva a su monitor y su etiqueta, no la trae a donde estás.

Un primer intento fue moverla con `awful.screen.focused()` justo antes de activarla, y no bastó. Al restaurarse, KeePassXC coge el foco, y para entonces awesome ya considera enfocado el monitor de KeePassXC. Hay que apuntar el monitor **antes** de lanzarla.

Lo de Super+Q: en `~/.config/keepassxc/keepassxc.ini` estaban `MinimizeOnClose=true` y `MinimizeToTray=true`, pero también `ShowTrayIcon=false`. Sin icono en la bandeja no hay dónde esconderla, así que solo se minimiza. Y con `MinimizeOnClose=false` se cierra el programa, y con él la base de datos.

# Cambios

## Script de Super+F4

`~/Scripts/Apps/kp_busqueda.sh` queda así:

```bash
#!/usr/bin/env bash
# Abrir/traer keepassxc al frente, en el monitor activo, con la búsqueda enfocada

# Guardar el monitor enfocado ANTES de lanzar: al restaurarse, keepassxc
# coge el foco y awesome pasaría a considerar enfocado su monitor original
scr=$(awesome-client "return require('awful').screen.focused().index" | awk '{print $2}')

# Lanzar o traer al frente (instancia única: una 2ª llamada muestra la ventana existente)
if pgrep -x keepassxc >/dev/null; then
    keepassxc >/dev/null 2>&1
else
    setsid -f keepassxc >/dev/null 2>&1
fi

# Esperar a que la ventana principal sea visible (máx ~10 s)
wid=""
for _ in $(seq 50); do
    wid=$(xdotool search --onlyvisible --name 'KeePassXC$' 2>/dev/null | tail -n1)
    [ -n "$wid" ] && break
    sleep 0.2
done
[ -z "$wid" ] && exit 1

# Traer la ventana a la etiqueta visible de ese monitor y centrarla
awesome-client "local awful = require('awful'); for _, c in ipairs(client.get()) do if c.window == $wid then c.minimized = false; c:move_to_tag(screen[${scr:-1}].selected_tag); awful.placement.centered(c) end end" >/dev/null 2>&1

# Activar vía EWMH
xdotool windowactivate --sync "$wid"
sleep 0.15

# Esperar a soltar Super y mandar Ctrl+F sin modificadores colgados
xdotool keyup Super_L Super_R
xdotool key --clearmodifiers ctrl+f
```

Para ver qué monitor y etiqueta tiene la ventana:

```bash
awesome-client "local r='' for _, c in ipairs(client.get()) do if c.class=='KeePassXC' then r=r..c.window..' scr='..c.screen.index..' tag='..(c.first_tag and c.first_tag.name or 'nil') end end return r"
```

## Regla muerta en rc.lua

He borrado la regla `{ rule = { class = "keepassxc*", role = "gimp-image-window" }, properties = { maximized = true } }`. Lo comprobé con:

```bash
awesome -k -c ~/.config/awesome/rc.lua
```

## Super+Q a la bandeja

En la interfaz es "Minimizar en lugar de salir de la aplicación" y el icono de bandeja, pero es más fácil editar el `.ini`. Hay que hacerlo **con KeePassXC cerrado del todo** (Ctrl+Q), porque guarda los ajustes al salir:

```bash
sed -i 's/^MinimizeOnClose=false/MinimizeOnClose=true/; s/^ShowTrayIcon=false/ShowTrayIcon=true/' ~/.config/keepassxc/keepassxc.ini
```

Debe quedar así:

```ini
MinimizeOnClose=true
MinimizeToTray=true
ShowTrayIcon=true
```

La barra de awesome ya tiene `wibox.widget.systray()`, así que el icono tiene dónde aparecer. Queda así:

- **Super+F4:** abre KeePassXC o la saca de la bandeja en el monitor donde estoy.
- **Super+Q:** la esconde en la bandeja y la base de datos sigue abierta.
- **Ctrl+Q dentro de KeePassXC:** sale de verdad.

Posts relacionados: [Flameshot se cuelga en awesomewm](/p/flameshot-se-cuelga-en-awesomewm/) y [Hotkeyz, buscador de atajos](/p/hotkeyz-buscador-de-atajos/).

# Deshacer

```bash
cp ~/Scripts/Apps/kp_busqueda.sh.bak ~/Scripts/Apps/kp_busqueda.sh
```

Con KeePassXC cerrado:

```bash
cp ~/.config/keepassxc/keepassxc.ini.bak ~/.config/keepassxc/keepassxc.ini
```

La regla borrada de `rc.lua` está en la copia `~/.config/awesome/rc.lua.bak.<fecha>` de este cambio. Recupérala de ahí si la quieres.

---
title : "Bloq Mayús reasignado y las mayúsculas se quedan activadas"
slug : "bloq-mayus-atascado"
date : "2026-10-31"
image : ""
tags : ["teclado", "xmodmap", "xkb", "caps lock"]
categories : ["arch"]
description : "Si quitas la función de Bloq Mayús y las mayúsculas se quedan activadas, ya no hay tecla para apagarlas. Cómo desactivarlas desde la terminal."
---

# El problema

Tengo **Bloq Mayús** reasignado (no hace de bloqueo de mayúsculas). Un día se quedaron las **MAYÚSCULAS ACTIVADAS** y no había forma de quitarlas, porque la tecla que las desactiva ya no existe.

Pasa cuando el estado de bloqueo se activa antes de cargar la reasignación (al arrancar, al cambiar de teclado, al reconectar un teclado USB…) o lo activa otro programa (`xdotool`, una VM, un escritorio remoto).

# En mi configuración

`~/.Xmodmap`:

```
!Turn Caps Lock into another Control key
remove Lock = Caps_Lock
!keycode 66 = Hyper_L
!add mod3 = Caps_Lock 
!keysym Caps_Lock = mod3
!clear Control
!add Control = Control_L Control_R
clear lock
```

`remove Lock = Caps_Lock` y `clear lock` quitan la tecla del modificador *Lock*. A partir de ahí, si el estado *Lock* está activo, ninguna tecla lo apaga.

# Desactivar las mayúsculas atascadas

Mira el estado:

```bash
xset q | grep -i caps
```

Apágalas mandando la tecla directamente al servidor X. `xdotool` se salta la reasignación física, así que funciona aunque la tecla esté haciendo de Super:

```bash
xdotool key Caps_Lock
```

Es lo que usé (18-09-2026). Después `xset q` debe mostrar `Caps Lock:   off`.

Si no tienes xdotool, otra opción es reiniciar la distribución y volver a cargar el `.Xmodmap`:

```bash
setxkbmap es && xmodmap ~/.Xmodmap
```

Con una opción de xkb (`caps:super`, `caps:ctrl_modifier`…) en vez de xmodmap, la tecla original no se pierde del todo:

```bash
setxkbmap -layout es -option caps:super
```

# Revertir

Nada que revertir: los comandos solo cambian el estado actual del teclado.

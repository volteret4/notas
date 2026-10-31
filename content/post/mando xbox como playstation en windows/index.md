---
title : "Windows reconoce el mando de Xbox como uno de PlayStation"
slug : "mando-xbox-como-playstation-en-windows"
date : "2026-10-31"
image : ""
tags : ["windows", "mando", "xbox", "xinput", "ds4windows", "steam input", "hidhide"]
categories : ["pamplinas"]
description : "De un día para otro, los juegos mostraban los botones de PlayStation con un mando de Xbox. Suele ser un programa que emula un DualShock o un perfil de Steam Input. Cómo encontrarlo."
---

# El problema

Desde un día concreto (02-09-2026), Windows y los juegos trataban mi **mando de Xbox** como uno de **PlayStation**: botones △○✕□ y controles cambiados en cada juego.

Windows no hace eso por sí solo: un mando de Xbox es **XInput** de forma nativa. Si aparece como DualShock, es que hay un **mando virtual** emulado encima.

# Sospechosos habituales

| Programa | Qué hace | Dónde mirarlo |
|---|---|---|
| **DS4Windows** | emula un DualShock 4 con ViGEmBus | icono en la bandeja, perfil "DS4" |
| **Steam Input** | puede presentar el mando como PlayStation si está marcado *Soporte de configuración de PlayStation* | Steam → *Ajustes → Mando* |
| **ViGEmBus / HidHide** | el driver del mando virtual y el que oculta el físico | *Administrador de dispositivos → Dispositivos del sistema* |
| Emuladores (PCSX2, RPCS3…) | algunos instalan drivers DS4 | *Panel de control → Programas* |

Comprobar desde PowerShell qué mandos ve Windows:

```powershell
Get-PnpDevice -Class HIDClass,XnaComposite -PresentOnly | Where-Object FriendlyName -match "Xbox|XINPUT|Wireless Controller|DualShock|Game" | Select-Object Status,FriendlyName,InstanceId
```

Si sale un *Wireless Controller* (Sony) además del *Xbox Controller*, el primero es el virtual. Quitarlo es tan simple como cerrar o desinstalar el programa que lo crea.

`joy.cpl` (*Dispositivos de juego*) muestra la lista que usan los juegos antiguos.

> Nota: la sesión del 03-09 ya no se conserva y hoy no puedo entrar en Windows, así que no sé cuál de estos fue la causa ni qué se cambió exactamente.

# Revertir

Si se desinstaló DS4Windows o ViGEmBus, se reinstalan desde sus releases de GitHub. Si fue un ajuste de Steam Input, se vuelve a marcar en *Ajustes → Mando*.

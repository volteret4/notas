---
title : "Super Mario Galaxy en Dolphin con un mando de Xbox"
slug : "dolphin-mario-galaxy-mando-xbox"
date : "2026-10-31"
image : ""
tags : ["dolphin", "emulacion", "wii", "windows", "mando"]
categories : ["pamplinas"]
description : "Galaxy necesita Wiimote con Nunchuk. El perfil tenía el mando Classic, el stick en WASD y el puntero en el ratón. Así quedó la asignación para un mando de Xbox."
---

# El problema

En Windows (Dolphin en `C:\Program Files\Emulators\Dolphin-x64`), *Super Mario Galaxy* no reconocía bien el mando. El **Wiimote 1** tenía tres fallos:

- La extensión era el **Classic Controller**, que Galaxy no admite: necesita **Nunchuk**.
- El stick del Nunchuk estaba en **W/A/S/D** del teclado, pero el dispositivo era el mando de Xbox, así que Mario no se movía.
- El puntero estaba en el **ratón** y el giro en el clic derecho.

Además, en el mando de GameCube la cruceta abajo estaba en `Pad W` (izquierda) en vez de `Pad S`.

# La asignación

Dispositivo: `SDL/0/XInput Controller #1`. Extensión: **Nunchuk**.

| Mando Xbox | Wii | En Galaxy |
|---|---|---|
| Stick izquierdo | Stick del Nunchuk | moverse |
| A | A | saltar |
| X | Agitar Wiimote | giro (ataque) |
| RT | B | disparar trozos de estrella |
| LT | Z | agacharse |
| LB | C | centrar la cámara |
| Stick derecho | Puntero (IR) | apuntar y recoger trozos de estrella |
| Pulsar stick derecho | Recentrar puntero | — |
| Start / Back / Guía | + / − / Home | — |
| Cruceta | Cruceta | cámara |

`WiimoteNew.ini` completo:

```ini
[Wiimote1]
Device = SDL/0/XInput Controller #1
Buttons/A = `Button S`
Buttons/B = `Trigger R`
Buttons/1 = `Button N`
Buttons/2 = `Button E`
Buttons/- = Back
Buttons/+ = Start
Buttons/Home = Guide
D-Pad/Up = `Pad N`
D-Pad/Down = `Pad S`
D-Pad/Left = `Pad W`
D-Pad/Right = `Pad E`
IR/Up = `Right Y+`
IR/Down = `Right Y-`
IR/Left = `Right X-`
IR/Right = `Right X+`
IR/Recenter = `Thumb R`
Shake/X = `Button W`
Shake/Y = `Button W`
Shake/Z = `Button W`
Extension = Nunchuk
Nunchuk/Buttons/C = `Shoulder L`
Nunchuk/Buttons/Z = `Trigger L`
Nunchuk/Stick/Up = `Left Y+`
Nunchuk/Stick/Down = `Left Y-`
Nunchuk/Stick/Left = `Left X-`
Nunchuk/Stick/Right = `Left X+`
Nunchuk/Stick/Calibration = 100.00 141.42 100.00 141.42 100.00 141.42 100.00 141.42
Source = 1
Extension/Attach MotionPlus = False
[Wiimote2]
Device = DInput/0/Keyboard Mouse
Source = 0
[Wiimote3]
Device = DInput/0/Keyboard Mouse
Source = 0
[Wiimote4]
Device = DInput/0/Keyboard Mouse
Source = 0
[BalanceBoard]
Device = DInput/0/Keyboard Mouse
Source = 0
```

Los archivos están en `%APPDATA%\Dolphin Emulator\Config\` (`WiimoteNew.ini` y `GCPadNew.ini`). Dolphin tiene que estar **cerrado** al editarlos, o los sobrescribe.

# Si algo no va

- **El mando no responde**: *Controles → Wii Remote 1 → Configurar*, y elige el mando en *Dispositivo*.
- **El puntero vuelve al centro al soltar el stick**: activa *Entrada relativa* en la pestaña del puntero (IR).

# Revertir

Las copias de seguridad están en `%APPDATA%\Dolphin Emulator\Config\` con el sufijo `.bak.20261008015303`. Con Dolphin cerrado, desde PowerShell:

```powershell
cd "$env:APPDATA\Dolphin Emulator\Config"; Copy-Item WiimoteNew.ini.bak.20261008015303 WiimoteNew.ini -Force; Copy-Item GCPadNew.ini.bak.20261008015303 GCPadNew.ini -Force
```

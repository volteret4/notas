---
title : "Configurar Wakeonlan Via Wifi"
date : "2025-07-31"
image : ""
tags : []
categories : [network]
description : "Despierta dispositivos usando la tarjeta wifi"
---


> [askubuntu](https://askubuntu.com/questions/1332960/how-to-check-if-my-device-supports-wake-on-wlan)

El proceso se puede resumir de la siguiente manera:

Instalar iw, una herramienta para configurar dispositivos inalámbricos Linux:

```sh
sudo apt install iw
```

Enumerar los dispositivos inalámbricos disponibles:

```sh
iw list
```

Mostrar el estado de activación inalámbrica del dispositivo phy0 (ajustar el dispositivo según el resultado del commando anterior):

```sh
iw phy0 wowlan show
```

La sección de compatibilidad con WoWLAN muestra los modos de WoWLAN compatibles.

Habilitar WoWLAN para el dispositivo phy0 con magic-packet:

```sh
sudo iw phy0 wowlan enable magic-packet
```


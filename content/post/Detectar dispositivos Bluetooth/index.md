---
title : "Detectar Dispositivos Bluetooth"
date : "2025-07-31"
image : ""
tags : ["linux", "bluetooth", "devices"]
categories : [network]
description : "Usa el terminal para conectar a dispositivos bluetooth"
---


[Origen](https://stackoverflow.com/questions/73615751/how-to-detect-and-enable-a-bluetooth-adapter)


El kernel de Linux registrará todos los dispositivos conectados a él durante el inicio y puede encontrarlos con el siguiente commando.

```bash
dmesg | grep -i blue
```

Puedes usar `bluetoothctl` para arrancar la app y luego usar:

```sh
power on
agent on
scan on
connect MAC_ADDRESS
trust MAC_ADDRESS

---
title : "Desactivar Servicio De Impresión"
date : "2025-07-31"
image : ""
tags : []
categories : [básico]
description : "Deshabilita cups con systemd"
---


> [stackexchange](https://unix.stackexchange.com/questions/480082/how-to-disable-cups-service-on-reboot-with-systemd) 

Suelo conectarme a una red con muchas impresoras. Cuando se detectan impresoras, aparecen muchos mensajes molestos en GNOME. Uso la impresora rara vez, así que preferiría mantener CUPS desactivado la mayor parte del tiempo. Detener CUPS funciona y elimina las notificaciones molestas:

```sh
systemctl stop cups
```

Me gustaría desactivarlo al arrancar. Sorprendentemente, después de desactivarlo,

```sh
systemctl disabled cups
```

CUPS sigue ejecutándose después de reiniciar. El commando de estado

```sh
systemctl status cups

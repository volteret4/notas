---
title : "Mandar el audio de Windows a un Linux con PipeWire (Scream y AirPlay)"
slug : "audio-de-windows-a-linux-pipewire"
date : "2026-10-31"
image : ""
tags : ["pipewire", "scream", "shairport-sync", "airplay", "tuneblade", "windows", "audio"]
categories : ["network"]
description : "Dos formas de oír en pepecono lo que suena en Windows: Scream (tarjeta de sonido virtual por red) y AirPlay con TuneBlade y shairport-sync sobre PipeWire. Con los errores que salieron."
---

# Contexto

pepecono tiene PipeWire con el módulo pulse por TCP (puerto **4713**) y zeroconf, para que los otros Linux manden audio con un túnel. Para Windows hay dos opciones.

`~/.config/pipewire/pipewire-pulse.conf.d/` en pepecono:

```
# auth.conf
pulse.properties = {
    server.address = [
        "unix:native"
        "tcp:4713"
    ]
}

pulse.rules = [
    {
        matches = [ { tcp.port = 4713 } ]
        actions = {
            check-access = false
        }
    }
]
```

```
# zeroconf.conf
pulse.modules = [
    { name = libpipewire-module-zeroconf-publish }
]
```

El fallo de los túneles entre Linux venía de **IPv6**: avahi anunciaba la dirección `fe80::...` y el túnel no llegaba a conectarse.

# Opción 1: Scream

[Scream](https://github.com/duncanthrax/scream) es una tarjeta de sonido virtual para Windows que manda el audio por la red. En Linux se recibe con `scream` (en pepecono está en `/usr/bin/scream`).

## Instalarlo en Windows

El driver no está firmado por Microsoft, así que hay que activar el **modo de prueba** antes de instalarlo:

```bat
bcdedit /set testsigning on
```

Reinicia, instala el driver (`Install-x64.bat` como administrador) y deja el modo de prueba **activado**. Si lo desactivas, el driver deja de cargar.

## Unicast en lugar de multicast

Por defecto Scream manda en **multicast** (239.255.77.77:4010). Con pi-hole y un router TP-Link no llegaba, aunque en Windows los vúmetros se movían. La solución es mandar en **unicast** a la IP de pepecono, con valores en el registro.

La clave `Options` **no existe**: hay que crearla.

```bat
reg add "HKLM\SYSTEM\CurrentControlSet\Services\Scream\Options" /v UnicastIPv4 /t REG_SZ /d 192.168.1.133 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Services\Scream\Options" /v UnicastPort /t REG_DWORD /d 4010 /f
```

Reinicia Windows (o desactiva y activa el dispositivo de audio Scream).

## Recibir en Linux

```bash
scream -o pulse -u -p 4010
```

`-u` escucha en unicast. Con PipeWire, `-o pulse` aparece como un cliente más en `pavucontrol`.

# Opción 2: AirPlay con TuneBlade + shairport-sync

TuneBlade, en Windows, manda el audio del sistema a receptores AirPlay. En pepecono el receptor es `shairport-sync` con salida a PipeWire.

`/etc/shairport-sync.conf`:

```
general =
{
	interface = "enp1s0";
	port = 5005;
};
```

- `interface = "enp1s0"`: sin esto, shairport se anunciaba **en cada interfaz**, incluidas las redes de Docker, y TuneBlade no conectaba con ninguna.
- `port = 5005`: el puerto por defecto, el **5000**, estaba ocupado por un Flask (`ss -tlnp | grep 5000`).

Para que use el PipeWire del usuario `pepe` en lugar de ALSA, en `/etc/systemd/system/shairport-sync.service.d/override.conf`:

```ini
[Service]
Environment="PIPEWIRE_RUNTIME_DIR=/run/user/1000"
ExecStart=
ExecStart=/usr/bin/shairport-sync -o pw
```

Sin esto salía `the default ALSA device is inaccessible -- "hw:0" used instead.`: shairport, como usuario de sistema, no tiene acceso al PipeWire de la sesión. Para depurarlo a mano:

```bash
sudo PIPEWIRE_RUNTIME_DIR=/run/user/$(id -u pepe) shairport-sync -o pw -vv
```

Necesita `nqptp` (AirPlay 2) y `avahi-daemon` activos.

Un caso raro: con otro cliente Linux reproduciendo por túnel, el túnel del primero desaparecía de `pavucontrol`. Cada cliente tenía que tener un nombre de *sink* distinto.

# Estado actual (31-10-2026)

`shairport-sync` está en **failed** desde el 28-10 (`start-limit-hit`). El `pipewire.service` de usuario de pepe solo arranca por socket, así que si no hay sesión de `pepe` no hay PipeWire al que conectarse. Para volver a ponerlo en marcha:

```bash
ssh pepecono 'sudo systemctl reset-failed shairport-sync && sudo systemctl restart shairport-sync && journalctl -u shairport-sync -n 20 --no-pager'
```

Si falla por PipeWire, activa *lingering* para que la sesión de `pepe` arranque sin login:

```bash
ssh pepecono 'sudo loginctl enable-linger pepe'
```

# Revertir

En Windows:

```bat
reg delete "HKLM\SYSTEM\CurrentControlSet\Services\Scream\Options" /f
bcdedit /set testsigning off
```

Y desinstala el driver desde el *Administrador de dispositivos*.

En pepecono:

```bash
ssh pepecono 'sudo rm -r /etc/systemd/system/shairport-sync.service.d && sudo systemctl daemon-reload && sudo systemctl disable --now shairport-sync'
```

Quita también `interface` y `port` de `/etc/shairport-sync.conf`.

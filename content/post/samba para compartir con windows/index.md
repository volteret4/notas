---
title : "Compartir carpetas de Linux con Windows por Samba (cuando NFS no va)"
slug : "samba-para-compartir-con-windows"
date : "2026-10-31"
image : ""
tags : ["samba", "smb", "windows", "nfs", "roms"]
categories : ["network"]
description : "El cliente NFS de Windows dio guerra, así que compartí las ROMs de pepecono por Samba, con unidad fija en Windows. Configuración y cómo deshacerlo."
---

# Por qué Samba

Quería jugar en Windows (RetroArch y ES-DE) a las copias de mis juegos que están en pepecono. Probé NFS, pero el cliente NFS de Windows 10 (*Servicios para NFS*) daba problemas de permisos y de nombres de usuario. Samba es nativo en Windows y funcionó a la primera.

# En pepecono (Arch)

```bash
sudo pacman -S samba
```

`/etc/samba/smb.conf`. El original se guardó como `smb.conf.original` y `smb.conf.arcolinux`:

```ini
[ROMs]
   path = /home/pepe/ROMs
   browseable = yes
   read only = no
   guest ok = no
   valid users = pepe
[Pelis]
   path = /mnt/NFS/Pelis
   browseable = yes
   read only = yes
   guest ok = no
   valid users = pepe
```

El recurso `[Pelis]` lo añadí más tarde (13-09) para montar las películas en Windows.

Samba tiene sus propias contraseñas:

```bash
sudo smbpasswd -a pepe
```

```bash
sudo systemctl enable --now smb
```

Comprobar:

```bash
testparm -s && smbclient -L //localhost -U pepe
```

# En Windows: unidad permanente

```bat
net use G: \\192.168.1.133\ROMs /user:pepe /persistent:yes
```

Windows pide la contraseña de Samba. Para que la recuerde, marca *Recordar credenciales* o guárdala con `cmdkey`:

```bat
cmdkey /add:192.168.1.133 /user:pepe /pass
```

Para desmontar:

```bat
net use G: /delete
```

## La unidad no aparece en el Explorador

Si haces `net use` desde una consola **"Ejecutar como administrador"**, la unidad solo la ven los procesos elevados. El Explorador y los reproductores, que corren sin privilegios, no la ven. Lo mismo pasa si la mapeas por **SSH**: es otra sesión de Windows, aunque sea el mismo usuario.

Hay que hacerlo en el escritorio, en una consola **normal**:

```bat
net use Z: /delete
net use Z: \\192.168.1.133\Pelis /user:pepe /persistent:yes
```

Para comprobar la red desde PowerShell: `Test-NetConnection 192.168.1.133 -Port 445`.

## Error 1219

```
Error de sistema 1219. No se permiten varias conexiones a un servidor o a un recurso compartido por el mismo usuario usando más de un nombre de usuario.
```

Windows solo permite **un usuario por servidor**. Si ya hay una conexión a `\\192.168.1.133` con otro usuario (o como invitado), hay que cerrarla antes:

```bat
net use * /delete /y
```

Otra opción es usar el nombre en vez de la IP para la segunda conexión (`\\pepe-cono\Pelis`), porque Windows lo considera otro servidor.

# Estado actual (31-10-2026)

`/home/pepe/ROMs` **ya no existe** en pepecono: las ROMs se movieron a `C:\ROMs` en Windows (ver [ES-DE](../es-de-multidisco-y-accesos-de-steam)). El recurso `[ROMs]` sigue en `smb.conf`, pero apunta a una carpeta vacía. Se puede quitar, o dejarlo si vuelves a tener una copia allí.

# Revertir

En Windows:

```bat
net use G: /delete
cmdkey /delete:192.168.1.133
```

En pepecono:

```bash
ssh pepecono 'sudo systemctl disable --now smb && sudo cp /etc/samba/smb.conf.original /etc/samba/smb.conf'
```

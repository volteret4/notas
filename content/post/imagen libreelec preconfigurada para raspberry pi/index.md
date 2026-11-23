---
title : "Imagen de LibreELEC ya configurada para la Raspberry Pi 4"
slug : "imagen-libreelec-preconfigurada-raspberry"
date : "2026-11-23"
image : ""
tags : ["libreelec", "kodi", "raspberry", "mke2fs", "imagen"]
categories : ["scripts"]
description : "Cómo saco una imagen .img.gz de LibreELEC que arranca ya configurada (addons, cuentas, biblioteca, SSH solo con clave) a partir de una instantánea de /storage, sin root, y por qué LibreELEC no deja hacerlo de la forma obvia."
---

# Idea

Tengo una Raspberry Pi 4 con LibreELEC 12 arrancando desde un USB, enchufada a la tele. Después de dejarla a mi gusto (addons, cuentas, biblioteca de Kodi, mando, SSH solo con clave) quería poder regrabar el USB y que arrancase **ya configurada**, sin repetir nada a mano.

La imagen sale de una instantánea de `/storage` de la Pi funcionando, metida en la imagen oficial. Se construye en el PC sin `sudo`; solo la grabación necesita root.

Relacionado: [la Raspberry como teclado Bluetooth para la TV](/p/raspberry-teclado-bluetooth-tv/) y [tarjetas NFC para Kodi](/p/sobrinos-tarjetas-nfc-kodi/).

# Síntoma

Lo obvio era grabar la imagen oficial y copiar la configuración en la partición `STORAGE` antes del primer arranque. Hacerlo así tiene tres problemas:

1. LibreELEC **no amplió la partición**: se quedó en 32 MB de los 128 GB del USB.
2. **SSH no arrancaba**: el puerto 22 rechazaba las conexiones aunque en los ajustes aparecía activado.
3. Al pasar a `fakeroot` + `mke2fs -d` para no necesitar root, fallaba con `Permiso denegado mientras se abría "shadow"`.

# Causa

## La ampliación de STORAGE

El primer arranque lo hace `/usr/lib/libreelec/fs-resize`, y solo si existe `/storage/.please_resize_me`:

- Si ya hay `/storage/.kodi`, `/storage/.config` o `/storage/.cache`, se niega: *«Resizing is not permitted - the system has already been initialised»*.
- Si no los hay, amplía la partición con `parted` y **la vuelve a formatear con `mke2fs`** (mismo UUID y etiqueta).

O sea, lo que metas en `STORAGE` antes del primer arranque o impide la ampliación o se borra. La ampliación la tengo que hacer yo.

## SSH

Para el primer arranque sin pantalla activé SSH añadiendo `ssh` a `cmdline.txt`. Pero mientras ese parámetro está en el kernel, el addon de ajustes de LibreELEC ignora su propia configuración y, cada vez que arranca Kodi, renombra `/storage/.cache/services/sshd.conf` a `sshd.disabled`. Sin ese fichero, en el siguiente arranque sshd no se lanza.

La configuración buena de SSH es la que el addon guarda en `/storage/.kodi/userdata/addon_data/service.libreelec.settings/oe_settings.xml` (`ssh_autostart` y `ssh_secure`). A partir de ahí genera `sshd.conf`, con `PasswordAuthentication no` si `ssh_secure` vale 1. Así que en la imagen `cmdline.txt` va **sin** `ssh`.

## fakeroot y shadow

`/storage/.cache/shadow` tiene permisos `000`. `fakeroot` simula el dueño, pero no se salta los permisos reales del disco: después de extraer el tar, `mke2fs` no puede leer ese fichero.

`mke2fs` (e2fsprogs 1.47) también acepta **un tar** en `-d`. Así no hay que extraer nada y se conservan dueño (root) y permisos tal cual vienen en el tar.

# Cambios

Necesita `mke2fs` 1.47 o más nuevo, `mtools`, `sfdisk`, `blkid` y `gzip`.

## 1. Instantánea de /storage

Con Kodi parado unos segundos para que sus bases de datos estén coherentes, y sin cachés ni logs (el `tar` de LibreELEC es el de busybox: `--exclude` con `./` delante y `.` como origen):

```bash
ssh root@<pi> 'systemctl stop kodi; sync'
```

```bash
ssh root@<pi> 'cd /storage && tar -cf - --exclude=./.kodi/temp --exclude=./.kodi/addons/packages --exclude=./.cache/mesa_shader_cache_db --exclude=./.cache/log --exclude=./logfiles --exclude=./screenshots --exclude=./backup --exclude=./.update .' > storage.tar
```

```bash
ssh root@<pi> 'systemctl start kodi'
```

En mi caso son unos 50 MB.

## 2. La imagen

Lo hace `imagen/construir.sh`. Los pasos clave:

```sh
gunzip -c LibreELEC-RPi4.aarch64-12.2.1.img.gz > img
P1=$(sfdisk -J img | python3 -c 'import json,sys;print(json.load(sys.stdin)["partitiontable"]["partitions"][0]["start"])')
P2=$(sfdisk -J img | python3 -c 'import json,sys;print(json.load(sys.stdin)["partitiontable"]["partitions"][1]["start"])')

# UUID y etiqueta de STORAGE: cmdline.txt arranca con disk=UUID=..., hay que conservarlos
dd if=img of=p2.orig bs=512 skip="$P2" status=none
UUID=$(blkid -o value -s UUID p2.orig)
LABEL=$(blkid -o value -s LABEL p2.orig)

# STORAGE de 1 GB al final de la imagen
truncate -s $(( P2 * 512 + 1024 * 1024 * 1024 )) img
echo ', +' | sfdisk --quiet --no-reread --no-tell-kernel -N 2 img

# autostart.sh y la marca de ampliación, añadidos al tar como root
mkdir -p extra/.config
install -m 755 autostart.sh extra/.config/autostart.sh
: > extra/.pollo_grow
tar -rf storage.tar -C extra --owner=0 --group=0 ./.config/autostart.sh ./.pollo_grow

# el sistema de ficheros se rellena directamente desde el tar (sin root ni fakeroot)
mke2fs -q -F -t ext4 -m 0 -L "$LABEL" -U "$UUID" -d storage.tar p2.img 1024M
e2fsck -fn p2.img
dd if=p2.img of=img bs=512 seek="$P2" conv=notrunc status=none

# partición de arranque (FAT) sin montarla: 4K60 y cmdline.txt sin «ssh»
FAT="img@@$(( P1 * 512 ))"
mcopy -n -i "$FAT" ::cmdline.txt cmdline.txt
sed -i 's/ ssh$//; s/ ssh / /' cmdline.txt
mcopy -o -i "$FAT" cmdline.txt ::cmdline.txt
mcopy -n -i "$FAT" ::config.txt config.txt
grep -q '^hdmi_enable_4kp60' config.txt || printf '\nhdmi_enable_4kp60=1\n' >> config.txt
mcopy -o -i "$FAT" config.txt ::config.txt

gzip -c img > pollo_media-$(date +%Y%m%d).img.gz
```

La instantánea no lleva `.please_resize_me`, así que LibreELEC no toca `STORAGE` al arrancar.

La imagen lleva credenciales (los ajustes de los addons, la clave SSH autorizada y las claves de host), así que la guardo con permisos `600` y no la publico.

## 3. Ampliar STORAGE al grabar

`flash.sh` comprueba que el disco es USB y no el del sistema, pide escribir `SI` y graba. Después amplía `STORAGE` **sin el USB montado**, que es lo seguro, y quita la marca:

```sh
gunzip -c pollo_media-latest.img.gz | dd of="$DEV" bs=4M conv=fsync status=progress
partprobe "$DEV"
parted -s "$DEV" resizepart 2 100%
e2fsck -fy "${DEV}2"
resize2fs "${DEV}2"
mount "${DEV}2" /mnt && rm -f /mnt/.pollo_grow && umount /mnt
```

## 4. Por si se graba con otra herramienta

Si la imagen se graba con Raspberry Pi Imager o con `dd` a pelo, `STORAGE` se queda en 1 GB. Para eso va `/storage/.config/autostart.sh`, que LibreELEC ejecuta al arrancar. ext4 admite crecer montado:

```sh
#!/bin/sh
if [ -e /storage/.pollo_grow ]; then
    (
        rm -f /storage/.pollo_grow
        PART=$(grep " /storage " /proc/mounts | cut -d" " -f1)
        DISK=$(echo "$PART" | sed 's/p\?2$//')
        printf 'Yes\n100%%\n' | parted ---pretend-input-tty "$DISK" resizepart 2 && resize2fs "$PART"
        df -h /storage
    ) >> /storage/.pollo_grow.log 2>&1 &
fi
```

Solo lo intenta una vez. Si falla, la Pi funciona igual con 1 GB de `STORAGE`.

De momento he comprobado la imagen por dentro (abajo); todavía no la he grabado y arrancado en la Pi, ni con `flash.sh` ni con la ampliación en caliente.

# Comprobar

Que la partición tiene el UUID correcto y no tiene errores:

```bash
blkid p2.img && e2fsck -fn p2.img
```

Que los ficheros importantes conservan dueño y permisos:

```bash
debugfs -R "stat /.ssh/authorized_keys" p2.img | grep -E "Mode|User"
```

Que `cmdline.txt` ya no lleva `ssh`:

```bash
mcopy -n -i "img@@$(( P1 * 512 ))" ::cmdline.txt - | cat
```

# Deshacer

Para volver a LibreELEC sin configurar, graba la imagen oficial; el primer arranque amplía y formatea `STORAGE` como siempre:

```bash
gunzip -c LibreELEC-RPi4.aarch64-12.2.1.img.gz | sudo dd of=/dev/sdX bs=4M conv=fsync status=progress
```

Para quitar solo la ampliación en caliente de una Pi ya grabada:

```bash
ssh root@<pi> 'rm -f /storage/.config/autostart.sh /storage/.pollo_grow'
```

---
title : "Luks"
date : "2025-07-31"
image : ""
tags : []
categories : [seguridad]
description : "Encripta una unidad"
---


## LUKS

> [Origen](https://www.zeppelinux.es/como-anadir-cambiar-o-eliminiar-contrasena-en-discos-cifrados-con-luks/)

### Listar particiones encriptadas

```sh
lsblk -p

    NAME                              MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
    /dev/sda                            8:0    0 931,5G  0 disk  
    ├─/dev/sda1                         8:1    0   512M  0 part  /boot/efi
    ├─/dev/sda2                         8:2    0   732M  0 part  /boot
    └─/dev/sda3                         8:3    0 930,3G  0 part  
      └─/dev/mapper/sda3_crypt        253:0    0 930,3G  0 crypt     
        ├─/dev/mapper/mint--vg-root   253:1    0 929,3G  0 lvm   /
        └─/dev/mapper/mint--vg-swap_1 253:2    0   976M  0 lvm   [SWAP]
    /dev/sr0
```

### Añadir nuevas contraseñas

` sudo cryptsetup luksAddKey /dev/sda3`

### Backups

```sh
sudo timeshift --create  
sudo timeshift --list  
sudo timeshift --restore --snapshot "2020-02-19_18-32-36"
sudo timeshift --delete  --snapshot '2014-10-12_16-29-08'
```

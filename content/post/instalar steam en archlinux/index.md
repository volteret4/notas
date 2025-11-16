---
title : "Instalar Steam En Archlinux"
date : "2025-11-16"
image : ""
tags : ["pacman", "steam"]
categories : ["Arch"]
description : ""
---


Edit la configuración de `pacman`
```shell
% vim /etc/pacman.conf
```

Elimina el comentario de la sección `[multilib]`

```shell
#[multilib]
#Include = /etc/pacman.d/mirrorlist
```

Quedando asi

```shell
[multilib]
Include = /etc/pacman.d/mirrorlist
```

Guarda el archivo y actualiza

```shell
% pacman -Syyu
```

Ya puedes instalar Steam:

```shell
% pacman -S steam
```
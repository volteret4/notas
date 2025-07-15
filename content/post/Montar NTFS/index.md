---
title : "Montar NTFS"
date : "2025-07-15"
image : ""
tags : []
categories : [básico]
description : "Montar disco NTFS en linux"
---


## Instalar
```bash
sudo apt install fuse
sudo apt install ntfs-3g
```
## Montar
```bash
sudo mount -t ntfs /dev/sdXy /path
```

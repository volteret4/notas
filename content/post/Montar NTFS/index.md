---
title : "Montar Ntfs"
date : "2026-01-17"
image : ""
tags : ["filesystems"]
categories : ["terminal"]
description : ""
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

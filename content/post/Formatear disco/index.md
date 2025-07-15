---
title : "Formatear Disco"
date : "2025-07-15"
image : ""
tags : []
categories : [básico]
description : ""
---


## Formatear disco

``` bash
# Crear nueva tabla de particiones 
sudo fdisk /dev/sdXY
	# Orden: g, n, w

# Formatear disco
mkfs [options] [-t type fs-options] device [size]

	sudo mkfs -t ext4 /dev/sdb1  # EJEMPLO
```

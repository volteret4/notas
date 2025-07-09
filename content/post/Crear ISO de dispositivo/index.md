---
title : "Crear ISO De Dispositivo"
date : "2025-07-10"
image : ""
tags : []
categories : [básico]
description : "Copia tus unidades fisicas en una ISO."
weight : 2
---


Crea una imagen de una unidad como un disco duro o una tarjeta de memoria.

``` bash
sudo dd if=/dev/sdX of=name-of-iso.iso status="progress"

---
title : "Imagemagick Convertir Imagen A Tono De Grises"
date : "2025-07-31"
image : ""
tags : ["linux", "magick", "image", "grey-scale", "blanco-y-negro"]
categories : [imágenes]
description : ""
---


> [Origen](https://stackoverflow.com/questions/13317753/convert-rgb-to-grayscale-in-imagemagick-command-line)

Usa esta linea cambiando `<img_in>` por la imagen a convertir.

```bash
magick <img_in> -colorspace Gray <img_out>

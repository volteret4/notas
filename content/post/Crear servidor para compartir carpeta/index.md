---
title: "Crear Servidor Para Compartir Carpeta"
date: "2025-07-10"
image: ""
tags: []
categories: [básico]
description: "Descarga archivos de tu servidor rápidamente"
---

Crea un servidor http en una carpeta para poder descargar archivos rápidamente

```sh
python3 -m http.server 8000 --bind 0.0.0.0 --directory /ruta/al/directorio
```

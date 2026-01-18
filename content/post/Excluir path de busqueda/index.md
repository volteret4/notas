---
title : "Excluir Path De Busqueda"
date : "2026-01-18"
image : ""
tags : ["archivos", "búsqueda", "shell"]
categories : ["Pamplinas"]
description : ""
---


Para poder borrar todo el contenido de una carpeta excepto una ruta concreta se puede:

```bash
find $carpeta -maxdepth 1 -type d ! -iname ${ruta} -delete
```

no termina de funcionar asi

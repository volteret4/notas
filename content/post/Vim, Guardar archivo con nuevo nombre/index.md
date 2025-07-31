---
title : "Vim, Guardar Archivo Con Nuevo Nombre"
date : "2025-07-31"
image : ""
tags : []
categories : [básico]
description : ""
---


> [Origen](https://www.cyberciti.biz/faq/how-to-save-existing-file-to-a-new-file-save-as-in-vi-vim/)

Si quieres forzar el guardado de un archivo con otro nombre en vim puedes usar:

```vim
:w !sudo tee /etc/httpd/cyberciti.com.conf
```

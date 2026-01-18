---
title : "Copiar Archivo De Un Contenedor De Proxmox"
date : "2026-01-18"
image : ""
tags : ["archivos", "commands", "shell"]
categories : ["Proxmox"]
description : ""
---


Con este commando puedes copiar un archivo a tu host
```sh
pct pull <vmid> <path> <destination>

# por ejemplo
pct pull 135 /root/contenedores/pollo.zip pollo.zip
```


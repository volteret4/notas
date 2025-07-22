---
title : "Git. Retroceder En Github"
date : "2025-07-22"
image : ""
tags : []
categories : [git]
description : "Restaurar commit"
---



```sh
 git restore archivo1 archivo2
```

Si además de restaurarlos quieres deshacer cualquier cambio que hayas _staged_ con `git add`, usa:

```sh
 git restore --staged archivo1 archivo2 git restore archivo1 archivo2

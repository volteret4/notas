---
title : "Kopia Server Add User"
date : "2025-07-31"
image : ""
tags : ["linux", "kopia", "server", "user"]
categories : [básico]
description : ""
---


Añade un nuevo usuario así desde el servidor, reemplazando el texto entre llaves:

```docker
 docker exec {kopia_container_name} kopia server user add {user}@{hostname}
```


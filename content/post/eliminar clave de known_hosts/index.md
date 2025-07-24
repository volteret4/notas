---
title : "Eliminar Clave De Known_hosts"
date : "2025-07-24"
image : ""
tags : ["linux", "ssh", "known_hosts"]
categories : [básico]
description : ""
---


Para eliminar las claves que referencien a dietpi:
```sh
   ssh-keygen -f "/home/$USER/.ssh/known_hosts" -R "dietpi"

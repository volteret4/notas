---
title : "Demasiados Intentos De Conexión Ssh"
date : "2026-01-17"
image : ""
tags : ["ssh"]
categories : ["Network"]
description : ""
---


[source](https://stackoverflow.com/questions/59429697/ssh-too-many-authentication-failures-error-when-trying-to-connect-to-raspberry)


Si tienes demasiadas llaves en el agente ssh, se prueban todas con el servidor, y se puede alcanzar el limit antes de acertar la correcta, pudiendo evitarlo asi:

```bash
ssh -o IdentityAgent=none -i private_key_file_for_raspberry ...
```

O puedes modificar la configuracion de ssh para ese host editando `~/.ssh/config`

```bash
HOST raspi42
   hostname raspberry.myhome
   user pi
   IdentityAgent none
   IdentityFile private_key_file_for_raspberry
```


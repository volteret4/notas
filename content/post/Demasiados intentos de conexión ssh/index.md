---
title : "Demasiados Intentos De Conexión Ssh"
date : "2025-07-31"
image : ""
tags : ["linux", "ssh", "fail", "error"]
categories : [network]
description : ""
---


[source](https://stackoverflow.com/questions/59429697/ssh-too-many-authentication-failures-error-when-trying-to-connect-to-raspberry)

The `known_host` file isn't related to your problem.  
It's seems to be the problem, that your agent knows too many keys.

You can try it with

```bash
ssh -o IdentityAgent=none -i private_key_file_for_raspberry ...
```

You can add this to your config file, too.

```bash
HOST raspi42
   hostname raspberry.myhome
   user pi
   IdentityAgent none
   IdentityFile private_key_file_for_raspberry
```


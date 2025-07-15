---
title : "Mac Address Spoofing"
date : "2025-07-09"
image : ""
tags : []
categories : [network]
description : "Cambia la MAC en Linux"
---


> [Origen](https://wiki.archlinux.org/title/MAC_address_spoofing)

## Macchanger

Otro método utilize macchanger (también conocido como GNU MAC Changer). Ofrece diversas funciones, como cambiar la dirección para que coincida con un proveedor específico o aleatorizarla completamente.

Instala el paquete `macchanger`.

La suplantación se realiza por interfaz. Especifica el nombre de la interfaz de red como interfaz en cada uno de los siguientes commandos.

La dirección MAC se puede suplantar con una dirección completamente aleatoria:

```bash
macchanger -r interface
```

Para aleatorizar solo los bytes específicos del dispositivo de la dirección MAC actual (es decir, para que, si se verifica la dirección MAC, se registre como del mismo proveedor), ejecuta el commando:

```bash
macchanger -e interface
```

Para cambiar la dirección MAC a un valor específico, ejecuta:

```bash
macchanger --mac=XX:XX:XX:XX:XX:XX interface
```

Donde XX:XX:XX:XX:XX:XX es la MAC a la que deseas cambiar.

Finalmente, para restaurar la dirección MAC a su valor de hardware original y permanente:

```bash
macchanger -p interface
```

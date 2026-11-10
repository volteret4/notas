---
title : "Cambiar de WiFi solo mientras DeaDBeeF está abierto"
slug : "wifi-automatica-deadbeef"
date : "2026-11-10"
image : ""
tags : ["wifi", "networkmanager", "deadbeef", "nfs", "systemd"]
categories : ["network"]
description : "Un servicio de usuario pasa a la red de 2,4 GHz (menos lag con el NFS) al abrir DeaDBeeF y vuelve a la de 5 GHz al cerrarlo. Además: el proceso se llama deadbeef-main y había perfiles WiFi duplicados atados a una tarjeta antigua."
---

# Idea

Uso dos redes:

- **TeskiyaMaricarmen** (5 GHz): la normal, más rápida para todo lo demás.
- **PapasconChocos** (2,4 GHz): con menos lag. Va mejor con DeaDBeeF tirando de la música del NFS (`/mnt/NFS/moode`).

Quería usar PapasconChocos solo con DeaDBeeF abierto, sin cambiar a mano. Las dos redes van al mismo router (192.168.1.1), así que el NFS y Tailscale siguen funcionando después del cambio.

# Script

`~/.local/bin/wifi-deadbeef` mira cada 2 segundos si DeaDBeeF está abierto:

- Si está abierto y estoy en TeskiyaMaricarmen, pasa a PapasconChocos.
- Al cerrarlo vuelve a TeskiyaMaricarmen, pero solo si fue el script quien hizo el cambio. Si elijo PapasconChocos a mano, la respeta.

```bash
#!/bin/bash
# Cambia a PapasconChocos (2.4 GHz, menos lag) mientras deadbeef está abierto
# y vuelve a TeskiyaMaricarmen (5 GHz) al cerrarlo.
# Solo vuelve atrás si fue este script quien cambió (respeta cambios manuales).
LENTA="PapasconChocos"
RAPIDA="TeskiyaMaricarmen"
INTERVALO=2
cambiado=0

# Devuelve el SSID activo (no el nombre del perfil, que puede ser "TeskiyaMaricarmen 4")
activa() {
    local uuid
    uuid=$(nmcli -t -f UUID,TYPE con show --active | awk -F: '$2=="802-11-wireless"{print $1; exit}')
    [ -n "$uuid" ] && nmcli -g 802-11-wireless.ssid con show "$uuid"
}

while true; do
    if pgrep -x 'deadbeef|deadbeef-main' >/dev/null; then
        if [ "$(activa)" = "$RAPIDA" ]; then
            echo "deadbeef abierto → $LENTA"
            nmcli con up "$LENTA" >/dev/null && cambiado=1
        fi
    elif [ "$cambiado" = 1 ]; then
        if [ "$(activa)" = "$LENTA" ]; then
            echo "deadbeef cerrado → $RAPIDA"
            nmcli con up "$RAPIDA" >/dev/null
        fi
        cambiado=0
    fi
    sleep "$INTERVALO"
done
```

Se lanza con un servicio de usuario, `~/.config/systemd/user/wifi-deadbeef.service`:

```ini
[Unit]
Description=Usar PapasconChocos mientras deadbeef está abierto

[Service]
ExecStart=%h/.local/bin/wifi-deadbeef
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

Para activarlo:

```bash
chmod +x ~/.local/bin/wifi-deadbeef && systemctl --user daemon-reload && systemctl --user enable --now wifi-deadbeef.service
```

Al cambiar de red, la WiFi se corta 2-5 segundos. El NFS de moode está montado como `hard`, así que DeaDBeeF se queda esperando y luego sigue.

# Por qué no funcionaba al principio

## El proceso se llama `deadbeef-main`

`pgrep -x deadbeef` no encontraba nada con DeaDBeeF abierto. Al lanzarlo, el nombre del proceso pasa a ser `deadbeef-main`:

```bash
cat /proc/$(pgrep -f '^deadbeef' | head -1)/comm
```

Por eso el script busca los dos nombres: `pgrep -x 'deadbeef|deadbeef-main'`.

## Perfiles WiFi duplicados

La conexión activa se llamaba `TeskiyaMaricarmen 4`, no `TeskiyaMaricarmen`. Había perfiles `TeskiyaMaricarmen` del 1 al 4 y `PapasconChocos 1`. Por eso el script compara por SSID y no por nombre de perfil.

La causa: los perfiles originales estaban atados a la tarjeta antigua, `wlp6s0`. La actual es `wlp5s0`. NetworkManager no podía usarlos (`No suitable device found for this connection`) y creaba copias nuevas.

Para verlo:

```bash
nmcli -t -f NAME,TYPE con show | grep ':802-11-wireless$' | cut -d: -f1 | while read -r c; do echo "$c: $(nmcli -g connection.interface-name con show "$c")"; done
```

Primero comprobé que la contraseña guardada en las copias era la misma. Después quité la atadura de los perfiles originales, me pasé a ellos y borré las copias:

```bash
nmcli con modify TeskiyaMaricarmen connection.interface-name "" && nmcli con modify PapasconChocos connection.interface-name ""
```

```bash
nmcli con up TeskiyaMaricarmen && for c in "TeskiyaMaricarmen 1" "TeskiyaMaricarmen 2" "TeskiyaMaricarmen 3" "TeskiyaMaricarmen 4" "PapasconChocos 1"; do nmcli con delete "$c"; done
```

TeskiyaMaricarmen mantiene la prioridad 5 (PapasconChocos tiene 0), así que al arrancar se conecta a la de 5 GHz. Los dos perfiles siguen con el ahorro de energía desactivado ([YouTube lento en Zen](/p/youtube-lento-zen-ahorro-energia-wifi/)).

## El journal no muestra nada

`journalctl --user -u wifi-deadbeef` dice `No journal files were found`: en este sistema no se guarda el journal de usuario. Para ver si funciona, mira la red activa:

```bash
nmcli -t -f NAME,TYPE con show --active | grep wireless
```

# Deshacer

```bash
systemctl --user disable --now wifi-deadbeef.service && rm ~/.config/systemd/user/wifi-deadbeef.service ~/.local/bin/wifi-deadbeef && systemctl --user daemon-reload
```

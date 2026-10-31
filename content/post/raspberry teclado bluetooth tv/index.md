---
title : "Una Raspberry Pi como teclado Bluetooth para una TV Hisense (VIDAA)"
slug : "raspberry-teclado-bluetooth-tv"
date : "2026-10-31"
image : ""
tags : ["raspberry", "bluetooth", "bluez", "hid", "l2cap", "python", "vidaa"]
categories : ["network"]
description : "Cómo convertir una Raspberry Pi 4 en un teclado HID Bluetooth para una tele Hisense con VIDAA, controlada por TCP desde otro equipo. Incluye todos los tropiezos de BlueZ."
---

# Qué es

`~/gits/pollo/mando_bluetooth`:

- **`rpi/`**: en la Raspberry Pi 4 (`chino`). Se presenta a la TV como un teclado Bluetooth y escucha órdenes por **TCP en el puerto 5555**.
- **`beelink/client.py`**: en el mini PC (Beelink). Manda las teclas a la Raspberry: `python3 client.py <ip-rpi>`.

Protocolo (JSON, uno por línea):

```
{"action": "key",      "key": "UP"}
{"action": "media",    "key": "VOLUP"}
{"action": "combo",    "mod": "LCTRL", "key": "C"}
{"action": "type",     "text": "hola"}
{"action": "pin",      "code": "123456"}
{"action": "consumer", "usage": 96}
```

# Preparar la Raspberry (`rpi/setup.sh`)

```bash
sudo bash rpi/setup.sh
```

1. Instala `python3-dbus python3-gi bluez bluez-tools`.
2. Crea un drop-in de systemd para `bluetoothd`. No se toca la unidad de la distro:

```ini
# /etc/systemd/system/bluetooth.service.d/hid.conf
[Service]
ExecStart=
ExecStart=/usr/libexec/bluetooth/bluetoothd --compat --noplugin=input
```

- `--noplugin=input`: si no, el plugin *input* de BlueZ se queda los PSM HID 0x11 y 0x13 y el servidor no puede abrirlos (`Cannot bind L2CAP PSM`).
- `--compat`: habilita el socket SDP antiguo para `sdptool`.

3. `rfkill unblock bluetooth` (venía bloqueado). En `/boot/config.txt` no puede haber `dtoverlay=disable-bt`.
4. Clase de dispositivo **teclado** (`0x000540`), visible y nombre `RPi-BT-Keyboard`.

# Lo que falló por el camino

| Problema | Causa y solución |
|---|---|
| `sdptool browse local` vacío tras `sdptool add HID` | `sdptool` no sirve para HID en BlueZ 5. El registro SDP se hace por D-Bus con `ProfileManager1` (`pairing_agent.register_hid_sdp()`) |
| La clase del dispositivo volvía a la de antes | BlueZ pisa la de `hciconfig class`. Hay que usar `btmgmt --index 0 class 5 64` (periférico + teclado) |
| La TV muestra un **código de 6 dígitos** y falla la conexión | La TV muestra el código y la Raspberry tiene que **teclearlo**. El agente de emparejamiento (`pairing_agent.py`, `KeyboardDisplay`) espera a que llegue el PIN con `{"action":"pin","code":"..."}` en lugar de aceptar solo |
| Tres emparejamientos repetidos | por borrar y volver a emparejar. Limpia con `bluetoothctl remove <MAC>` en los dos lados |
| El móvil lo veía y la TV no | condición de carrera: los sockets L2CAP tienen que estar abiertos **antes** de ponerse visible (`prepare_l2cap_servers()` va antes de `piscan`). `--no-sdp` solo sirve para probar el descubrimiento |

Diagnóstico útil:

```bash
sudo btmon
```

```bash
sudo systemctl cat bluetooth | grep ExecStart
```

```bash
hciconfig hci0 -a
```

# Arrancar

```bash
sudo python3 rpi/bt_hid_server.py --host 0.0.0.0 --port 5555
```

En la TV: *Configuración → Conexiones → Bluetooth* → buscar → `RPi-BT-Keyboard` → teclear el código que muestra con `{"action":"pin",...}` desde el cliente.

# Teclas especiales de la TV

El descriptor HID tiene un tercer informe (*Report ID 3*) de **Consumer Control** con un código libre, para probar teclas propias de cada tele sin tocar el descriptor. En `rpi/hid.py` están los candidatos que probé para la Hisense:

| Alias | Código | Para |
|---|---|---|
| `INPUT`, `INPUT2`, `INPUT3`, `INPUT4` | 0x0060, 0x0086, 0x00A0, 0x0183 | menú de fuentes |
| `YOUTUBE` | 0x0196 (AL Internet Browser) | abrir YouTube |
| `APPS`, `SMART` | 0x023C, 0x0223 | lanzador |
| `CH_UP`, `CH_DOWN`, `POWER`, `SLEEP` | 0x009C, 0x009D, 0x0030, 0x0032 | — |

El "favorito" que lleva a HDMI3 es la misma idea: probar códigos hasta dar con el que usa VIDAA.

# Revertir (en la Raspberry)

```bash
sudo rm /etc/systemd/system/bluetooth.service.d/hid.conf && sudo systemctl daemon-reload && sudo systemctl restart bluetooth
```

```bash
bluetoothctl remove <MAC-de-la-TV>
```

Y en la TV, olvida el dispositivo `RPi-BT-Keyboard`.

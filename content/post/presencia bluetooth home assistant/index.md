---
title : "Detector de presencia por Bluetooth con aviso a Home Assistant"
slug : "presencia-bluetooth-home-assistant"
date : "2026-10-31"
image : ""
tags : ["bluetooth", "l2ping", "home assistant", "python", "raspberry"]
categories : ["network"]
description : "Un script en la Raspberry que hace l2ping a los móviles conocidos y crea en Home Assistant un binary_sensor por dispositivo, con notificación al llegar o salir."
---

# Cómo funciona

`~/gits/pollo/mando_bluetooth/rpi/presence.py` (en la misma Raspberry que hace de [teclado Bluetooth](../raspberry-teclado-bluetooth-tv)):

- Cada `scan_interval` segundos hace `l2ping -c 1 -t 5 <MAC>` a cada dispositivo de la lista. **No hace falta que esté emparejado** ni visible: basta con que tenga el Bluetooth encendido.
- Lo marca **ausente** tras `absent_threshold` fallos seguidos, para no dar falsos "se ha ido".
- En cada cambio ejecuta `on_arrive` u `on_leave` (comandos de shell, opcionales) y avisa a Home Assistant.

# Configuración

Copia `rpi/presence_config.json.example` a `rpi/presence_config.json`:

```json
{
    "devices": {
        "AA:BB:CC:DD:EE:FF": {
            "name": "Mi movil",
            "on_arrive": "",
            "on_leave": ""
        }
    },
    "scan_interval": 15,
    "absent_threshold": 3,
    "home_assistant": {
        "url": "http://192.168.1.X:8123",
        "token": "TU_TOKEN_DE_LARGA_DURACION",
        "notify_service": "notify.mobile_app_tu_movil"
    }
}
```

El token es un *token de acceso de larga duración* (Home Assistant → tu perfil → *Seguridad*). `presence_config.json` está en `.gitignore`.

# Qué crea en Home Assistant

Con la API REST (`POST /api/states/<entity>`) crea o actualiza **un `binary_sensor` por dispositivo**:

```
binary_sensor.bt_presence_mi_movil   on / off
  friendly_name: BT Mi movil
  device_class: presence
  mac: AA:BB:CC:DD:EE:FF
  source: mando_bluetooth
```

Si hay `notify_service`, además llama a `POST /api/services/notify/<servicio>` con el mensaje de llegada o salida.

Ojo: las entidades creadas por `/api/states` **no sobreviven a un reinicio** de Home Assistant hasta el siguiente cambio de estado. Si las usas en automatizaciones, el script las vuelve a publicar en el siguiente ciclo.

# Consultarlo desde fuera

El servidor HID (`bt_hid_server.py`) carga el detector y acepta en el puerto 5555:

```
{"action": "presence"}                        -> estado de todos (JSON)
{"action": "presence", "mac": "AA:BB:..."}    -> presence:AA:BB:...:present|absent
```

# Revertir

```bash
rm ~/gits/pollo/mando_bluetooth/rpi/presence.py ~/gits/pollo/mando_bluetooth/rpi/presence_config.json
```

Quita del servidor la importación de `PresenceDetector`, borra en Home Assistant las entidades `binary_sensor.bt_presence_*` (*Herramientas para desarrolladores → Estados*) y revoca el token.

Los cambios no están commiteados en el repo:

```bash
git -C ~/gits/pollo/mando_bluetooth status --short
```

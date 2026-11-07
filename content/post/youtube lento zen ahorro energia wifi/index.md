---
title : "YouTube lento en Zen: el ahorro de energía de la WiFi había vuelto"
slug : "youtube-lento-zen-ahorro-energia-wifi"
date : "2026-11-07"
image : ""
tags : ["wifi", "networkmanager", "youtube", "zen-browser", "ublock", "violentmonkey"]
categories : ["network"]
description : "YouTube cargaba fatal en Zen. La culpa no era de uBlock: un perfil WiFi nuevo había reactivado el ahorro de energía. Qué se cambió y cómo deshacerlo."
---

# Síntoma

YouTube cargaba fatal en Zen Browser. La primera sospecha era uBlock Origin, pero lo había usado en Firefox sin problemas.

# Causa

El ahorro de energía de la WiFi ya estaba desactivado desde el 31-10-2026 ([Tirones en NFS por WiFi](/p/tirones-nfs-por-wifi/)), pero **perfil a perfil**. NetworkManager acabó usando un perfil nuevo, `TeskiyaMaricarmen 3`, con `powersave = default`, y el ahorro de energía volvió a activarse.

Conviene comprobarlo con estos dos comandos:

```bash
iw dev wlp5s0 get power_save
```

```bash
nmcli -t -f NAME,TYPE connection show | grep ':802-11-wireless$' | cut -d: -f1 | while read -r c; do echo "$c: $(nmcli -g 802-11-wireless.powersave con show "$c")"; done
```

# Antes y después

Ping al router (`ping -c 40 -i 0.25 192.168.1.1`), con la misma señal débil (-77 a -79 dBm en 5 GHz):

| | Mínimo | Media | Máximo |
|---|---|---|---|
| Ahorro de energía activado | 19,9 ms | 71,6 ms | 165 ms |
| Ahorro de energía desactivado | 1,4 ms | 4,3 ms | 31 ms |

La velocidad de descarga ya era suficiente (unos 39 Mbit/s): el problema era la latencia.

# Cambios

## WiFi

Desactivado en el perfil que faltaba:

```bash
nmcli con modify "TeskiyaMaricarmen 3" 802-11-wireless.powersave 2
```

Este ajuste se aplica al reconectar. Para aplicarlo en el momento, sin reconectar:

```bash
sudo iw dev wlp5s0 set power_save off
```

Para que no vuelva a pasar con perfiles nuevos, se puede fijar para todos en `/etc`:

```bash
printf '[connection]\nwifi.powersave = 2\n' | sudo tee /etc/NetworkManager/conf.d/wifi-powersave-off.conf && sudo systemctl reload NetworkManager
```

El perfil sigue en 5 GHz (`band = a`), donde la señal es peor que en 2,4 GHz.

## Navegador

- **Violentmonkey:** desactivado un script de letras, y parece que ayudó bastante (impresión, sin medir).
- **Dark Reader:** excluido en YouTube.
- **uBlock Origin:** sin cambios. Usa las listas por defecto y la mejora llegó sin tocarlo.

# Deshacer

Volver al valor por defecto en el perfil:

```bash
nmcli con modify "TeskiyaMaricarmen 3" 802-11-wireless.powersave 0
```

Quitar el ajuste global, si se aplicó:

```bash
sudo rm /etc/NetworkManager/conf.d/wifi-powersave-off.conf && sudo systemctl reload NetworkManager
```

Aplicar los cambios reconectando (corta la red unos segundos):

```bash
nmcli con up "TeskiyaMaricarmen 3"
```

Activar el ahorro de energía al momento:

```bash
sudo iw dev wlp5s0 set power_save on
```

En el navegador: reactivar el script en el panel de Violentmonkey y, en YouTube, volver a activar Dark Reader desde su icono.

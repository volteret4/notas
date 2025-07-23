---
title : "Wake On LAN"
date : "2025-07-23"
image : ""
tags : ["wakeonlan", "lan", "Linux"]
categories : [network]
description : "Despierta un dispositivo de tu red"
---


## Configure the host waking up

```sh
sudo ethtool -s eth0 wol g
```

Access BIOS and enable Wake On Lan.


## How Do I Send WOL Magic Packets Under Linux?

```sh
wakeonlan MAC-Address-Here

wakeonlan "74:56:3C:5D:E6:1D" # proxmox
wakeonlan "30:65:ec:a8:9b:37" # kodi
```
OR  
```sh
etherwake MAC-Address-Here
```

## Crea un servicio para activar wol al inicio


```bash ❴lineNos="true" wrap="true" title="/etc/systemd/system/wol.service"❵
[Unit]
Description=Enable Wake On Lan
Requires=network.target
After=network.target

[Service]
Type=oneshot
ExecStart = /sbin/ethtool --change enp1s0 wol g

[Install]
WantedBy=basic.target
```


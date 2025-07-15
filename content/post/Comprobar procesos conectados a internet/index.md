---
title: "Comprobar Procesos Conectados A Internet"
date: "2025-07-15"
image: ""
tags: []
categories: [network]
description: "Detectar las conexiones de cada aplicación"
---

```bash
sudo netstat -tulnp  # requiere instalar net-tools (en debian)
sudo ss -tulnp

# Puertos especificos:
sudo fuser -n tcp <puerto>      # se puede cambiar tcp por udp
ps -p $(sudo fuser -n tcp 80)   # muestra nombre de los PID

# Conexiones activas
sudo lsof -i -n -P

# Usuario concreto
lsof -u username

# Archivos abiertos por un proceso
lsof -p 1234

# Procesos abiertos en un directorio
lsof +D /ruta/al/directorio

# Actualiza la salida cada 5 segundos
lsof -i -n -P -r 5
```

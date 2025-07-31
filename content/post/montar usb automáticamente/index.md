---
title : "Montar Usb Automáticamente"
date : "2025-07-31"
image : ""
tags : ["linux", "usb", "automount", "mount"]
categories : [scripts]
description : ""
---


## 1. Editar regla:

`sudo nano /etc/udev/rules.d/99-usb-mount.rules`

`ACTION=="add", SUBSYSTEM=="block", ENV{DEVTYPE}=="partition", ENV{ID_BUS}=="usb", TAG+="systemd", ENV{SYSTEMD_WANTS}="usb-mount@%k.service"`


## 2. Crear Servicio

`sudo nano /etc/systemd/system/usb-mount@.service`

```sytemd
[Unit]
Description=Mount USB device %I
After=dev-%i.device
BindsTo=dev-%i.device

[Service]
Type=oneshot
ExecStart=/usr/local/bin/mount-usb.sh /dev/%I
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

## 3. Creamos Script 

`sudo nano /usr/local/bin/mount-usb.sh`

```python
#!/usr/bin/env python3
import os
import sys
import subprocess
import time
import logging
from datetime import datetime

# Configurar logging
log_file = "/var/log/usb-mount.log"
logging.basicConfig(
    filename=log_file,
    level=logging.DEBUG,
    format='%(asctime)s - %(levelname)s - %(message)s'
)

def mount_usb_and_restart_container():
    logging.info("Script iniciado")
    
    if len(sys.argv) != 2:
        logging.error("Error: Se requiere el nombre del dispositivo como argumento")
        sys.exit(1)
    
    device = sys.argv[1]
    mount_point = "/mnt/usb"
    
    logging.info(f"Procesando dispositivo: {device}")
    
    # Verificar si el dispositivo ya está en fstab
    try:
        with open('/etc/fstab', 'r') as f:
            fstab_content = f.read()
            if device in fstab_content:
                logging.info(f"El dispositivo {device} ya está definido en fstab. Ignorando.")
                sys.exit(0)
    except Exception as e:
        logging.error(f"Error al verificar fstab: {e}")
    
    # Verificar si el dispositivo ya está montado en otro lugar
    try:
        result = subprocess.run(["findmnt", "-S", device], stdout=subprocess.PIPE, text=True)
        if result.returncode == 0:
            logging.info(f"El dispositivo {device} ya está montado en otro lugar. Ignorando.")
            sys.exit(0)
    except Exception as e:
        logging.error(f"Error al verificar montajes: {e}")
    
    # Esperar un momento para asegurar que el dispositivo esté disponible
    time.sleep(2)
    
    # Crear punto de montaje si no existe
    os.makedirs(mount_point, exist_ok=True)
    logging.info(f"Punto de montaje creado/verificado: {mount_point}")
    
    # Verificar si ya hay algo montado en el punto de montaje
    try:
        result = subprocess.run(["findmnt", mount_point], stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True)
        if result.returncode == 0:
            logging.info(f"Ya hay un dispositivo montado en {mount_point}. Desmontando...")
            subprocess.run(["umount", mount_point], check=True)
    except Exception as e:
        logging.warning(f"Aviso al verificar punto de montaje: {e}")
    
    # Montar el nuevo dispositivo
    try:
        # Mostrar el dispositivo que vamos a montar
        logging.info(f"Intentando montar: {device} en {mount_point}")
        subprocess.run(["ls", "-la", device], check=False)
        
        # Intentar montar
        mount_result = subprocess.run(
            ["mount", device, mount_point], 
            check=False,
            stdout=subprocess.PIPE,
            stderr=subprocess.PIPE,
            text=True
        )
        
        if mount_result.returncode == 0:
            logging.info(f"Dispositivo {device} montado en {mount_point}")
        else:
            logging.error(f"Error al montar: {mount_result.stderr}")
            sys.exit(1)
    except Exception as e:
        logging.error(f"Excepción al montar el dispositivo: {e}")
        sys.exit(1)
    
    # Esperar un momento para asegurar que el montaje se completó
    time.sleep(2)
    
    # Reiniciar el contenedor de filebrowser
    try:
        logging.info("Reiniciando contenedor filebrowser")
        result = subprocess.run(
            ["docker", "restart", "filebrowser"], 
            check=False,
            stdout=subprocess.PIPE,
            stderr=subprocess.PIPE,
            text=True
        )
        
        if result.returncode == 0:
            logging.info("Contenedor filebrowser reiniciado con éxito")
        else:
            logging.error(f"Error al reiniciar el contenedor: {result.stderr}")
    except Exception as e:
        logging.error(f"Excepción al reiniciar el contenedor filebrowser: {e}")
        sys.exit(1)
    
    logging.info("Script completado correctamente")

if __name__ == "__main__":
    try:
        mount_usb_and_restart_container()
    except Exception as e:
        logging.error(f"Error no controlado: {e}")
```

## 4. Hacer el script ejecutable
`sudo chmod +x /usr/local/bin/mount-usb.sh`

## 5. Recargar reglas udev
`sudo udevadm control --reload-rules`

## 6. Recargar systemd

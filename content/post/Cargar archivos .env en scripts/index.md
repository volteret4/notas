---
title : "Cargar Archivos .Env En Scripts"
date : "2026-01-17"
image : ""
tags : ["variables"]
categories : ["Scripts"]
description : ""
---



- **Crea el archivo `.env`**  
    Este archivo contendrá tus variables de entorno:

```sh
# Archivo .env 
VAR1="valor1"
VAR2="valor2"
```


- **Escribe el script Bash que cargará `.env`**  
    Puedes usar `source` para cargar las variables del archivo en el entorno del script:

```sh
# Cargar variables desde .env 
source "$(dirname "$0")/.env"  

# Usar las variables 
echo "VAR1 es: $VAR1" echo "VAR2 es: $VAR2"
```

## Python
Puedes usar el paquete python-dotenv con `pip install python-dotenv`.r

```python
import os
from dotenv import load_dotenv

load_dotenv()

API_KEY = os.gentenv ('API_KEY')
```
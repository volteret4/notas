---
title : "Instalar Invidious"
date : "2025-07-31"
image : ""
tags : ["linux", "invidious", "youtube", "docker", "yt"]
categories : [contenedores]
description : "Instala este fantástico frontend para youtube"
---


Clona el repositorio
```bash
git clone https://github.com/iv-org/invidious.git
cd invidious
```

Para obtener `po_token` y `visitor_data` ejecuta:
```sh
 docker run quay.io/invidious/youtube-trusted-session-generator
```


Para obtener el hmac:
```sh
 openssl rand -base64 21
```

Necesitas editar el docker-compose.yml para añadir estos tres tokens


Tambien puedes correr estos commandos y cargar las variables desde un .env:

```sh
#!/bin/bash

# Ejecutar el contenedor y capturar la salida
OUTPUT=$(docker run quay.io/invidious/youtube-trusted-session-generator)

# Extraer los valores usando grep y sed o awk
VISITOR_DATA=$(echo "$OUTPUT" | grep 'visitor_data:' | sed 's/.*visitor_data: *//')
PO_TOKEN=$(echo "$OUTPUT" | grep 'po_token:' | sed 's/.*po_token: *//')

# Verifica que se hayan extraído correctamente
if [[ -z "$VISITOR_DATA" || -z "$PO_TOKEN" ]]; then
    echo "Error: No se pudieron extraer los datos correctamente."
    exit 1
fi

# hmac
hmac="$(openssl rand -base64 21)"

# Guardarlos en un archivo .env
cat <<EOF > .env
VISITOR_DATA=$VISITOR_DATA
PO_TOKEN=$PO_TOKEN
HMAC=$hmac
EOF

echo ".env generado correctamente:"
cat .env


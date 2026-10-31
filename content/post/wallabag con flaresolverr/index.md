---
title : "Wallabag con FlareSolverr para las webs protegidas por Cloudflare"
slug : "wallabag-con-flaresolverr"
date : "2026-10-31"
image : ""
tags : ["wallabag", "flaresolverr", "cloudflare", "docker", "proxy", "mitm"]
categories : ["contenedores"]
description : "Wallabag no podía guardar páginas con el challenge de Cloudflare o muros de cookies. Lo hago salir por un proxy que reintenta esas peticiones con FlareSolverr (Chrome headless)."
---

# El problema

Algunos artículos se guardaban en Wallabag vacíos o con la página de "Checking your browser" de **Cloudflare**, o con el aviso de cookies en lugar del texto.

Wallabag no deja usar FlareSolverr directamente, pero sí un **proxy HTTP**. La idea, basada en [FlareSolverrProxy](https://github.com/Lenni0451/FlareSolverrProxy):

```
wallabag ──proxy──▶ flaresolverr-proxy:8888 ──directo──▶ web
                         │ si responde con el challenge de Cloudflare
                         └──▶ flaresolverr:8191 (Chrome) ──▶ web
```

Para ver las páginas HTTPS, el proxy hace **MITM** con una CA propia. Wallabag solo confía en esa CA, y el proxy sí verifica TLS de verdad contra las webs reales.

# Cambios en `pepecono:~/contenedores/herramientas/`

## `docker-compose-herramientas.yml`

Servicios nuevos:

```yaml
  # FLARESOLVERR: wallabag sale por flaresolverr-proxy; si una web devuelve
  # el challenge de Cloudflare, el proxy la repite vía FlareSolverr (Chrome).
  flaresolverr:
    image: ghcr.io/flaresolverr/flaresolverr:latest
    container_name: flaresolverr
    environment:
      - LOG_LEVEL=info
      - TZ=Europe/Madrid
    restart: unless-stopped
  flaresolverr-proxy:
    build: ./flaresolverr-proxy
    image: flaresolverr-proxy:local
    container_name: flaresolverr-proxy
    environment:
      - FLARESOLVERR_URL=http://flaresolverr:8191/v1
      - FLARESOLVERR_TIMEOUT=60000
    volumes:
      - ./flaresolverr-proxy/ca:/ca
    labels:
      - com.centurylinklabs.watchtower.enable=false
    depends_on:
      - flaresolverr
    restart: unless-stopped
```

Volúmenes añadidos al servicio `wallabag`:

```yaml
      - ./wallabag/config_prod.yml:/var/www/wallabag/app/config/config_prod.yml:ro
      - ./flaresolverr-proxy/ca/ca.pem:/etc/flaresolverr-proxy-ca/ca.pem:ro
```

Ninguno publica puertos al host: solo se ven en la red de Docker.

## `flaresolverr-proxy/`

- `Dockerfile`: `python:3.13-slim` con `openssl` y `requests`, y expone el 8888.
- `proxy.py`: hace la petición directa (20 s). Si la respuesta es un challenge de Cloudflare, la repite con FlareSolverr (`request.get`, hasta 60 s). Para HTTPS genera al vuelo un certificado por host firmado por la CA.
- `ca/ca.pem` y `ca/ca.key`: la CA. Si no existe, la genera al arrancar (validez de 100 años). **`ca.key` no debe salir de esa carpeta.**

## `wallabag/config_prod.yml`

Se monta desde fuera para que **sobreviva a Watchtower**, que recrea el contenedor. Al final:

```yaml
httplug:
    clients:
        wallabag_core:
            config:
                defaults:
                    timeout: 90
                    verify: "/etc/flaresolverr-proxy-ca/ca.pem"
                    proxy:
                        http: "http://flaresolverr-proxy:8888"
                        https: "http://flaresolverr-proxy:8888"
```

`timeout: 90` deja margen a FlareSolverr, que tarda en resolver el challenge.

Aplicar:

```bash
ssh pepecono 'cd ~/contenedores/herramientas && docker compose -f docker-compose-herramientas.yml up -d --build flaresolverr flaresolverr-proxy wallabag && docker exec wallabag bin/console cache:clear --env=prod'
```

# Comprobar

```bash
ssh pepecono 'docker logs flaresolverr-proxy --since 10m | tail -20'
```

Guarda en Wallabag una web con Cloudflare: en el log del proxy debe verse que reintenta con FlareSolverr.

# Revertir

Hay copia del compose de antes: `docker-compose-herramientas.yml.bak-claude-20260930-124709`.

```bash
ssh pepecono 'cd ~/contenedores/herramientas && docker compose -f docker-compose-herramientas.yml stop flaresolverr flaresolverr-proxy && cp docker-compose-herramientas.yml.bak-claude-20260930-124709 docker-compose-herramientas.yml && docker compose -f docker-compose-herramientas.yml up -d --remove-orphans wallabag'
```

Ojo: esa copia es anterior también a cualquier otro cambio hecho en el compose después del 30-09.

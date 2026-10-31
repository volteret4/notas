---
title : "escuchowsky, recomendaciones musicales con Last.fm en AWS"
slug : "escuchowsky-recomendaciones-lastfm"
date : "2026-10-31"
image : ""
tags : ["lastfm", "listenbrainz", "flask", "docker", "aws", "nginx", "modsecurity", "crowdsec"]
categories : ["contenedores"]
description : "Una web de recomendaciones y colecciones \"must hear\" a partir de Last.fm y ListenBrainz, con usuarios secundarios, desplegada en el free tier de AWS con un servidor bien cerrado."
---

# Qué es

Repo: `~/gits/pollo/escuchowsky` (`https://github.com/volteret4/escuchowsky.git`).

- **Recomendaciones** a partir de los scrobbles de Last.fm y **ListenBrainz**, también con **usuarios secundarios**: amigos cuyos gustos se cruzan con los tuyos.
- **Colecciones "must hear"** (listas de RYM, Scaruffi, AOTY…), con lo que ya has escuchado marcado.
- **Árbol de géneros** en mermaid (`app_genre_mermaid.py`) y descubrimiento (`app_discover.py`).
- Traducciones (las claves tipo `um.search.button` tienen que existir en todos los idiomas, o salen sin traducir).

# Despliegue

No corre en casa, sino en una instancia del **free tier de AWS**. Fueron mis primeros pasos con AWS.

`docker-compose.yml`: servicios `escuchowsky`, `musthear` y `tumtumpa` (Flask + Gunicorn), un `nginx` con **ModSecurity** como proxy inverso (puertos 8085 y 8443 dentro, 80 y 443 hacia fuera con iptables), y `nginx-exporter` + `node-exporter` para Prometheus/Grafana en otro servidor.

Endurecimiento, según `CLAUDE.md` del repo:

- Docker **rootless**.
- Solo 80/443 expuestos.
- CrowdSec leyendo los logs del sistema y de las apps.
- `auditd`, rotación de logs y escaneos con **trivy** (`trivy_reports/`).
- Certificados con acme.sh, uno de ellos solo para Prometheus.
- Imágenes fijadas por `sha256`.

```bash
docker compose up -d --build
```

Los secretos van en `.env` cifrado con sops (`sops_env.py`).

# Revertir

Es un proyecto aparte: para quitarlo, `docker compose down -v` en la instancia y terminar la instancia en la consola de AWS (vigila que no queden volúmenes EBS ni IPs elásticas facturando).

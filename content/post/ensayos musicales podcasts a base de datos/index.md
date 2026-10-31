---
title : "ensayos-musicales, de podcasts a una base de datos de relaciones entre artistas"
slug : "ensayos-musicales-podcasts-a-base-de-datos"
date : "2026-10-31"
image : ""
tags : ["whisper", "gemini", "claude", "sqlite", "flask", "d3", "podcasts"]
categories : ["scripts"]
description : "Transcribo podcasts de música con Whisper, extraigo artistas, discos, sellos y relaciones con un LLM y lo reviso en un editor web propio, con un mapa de influencias."
---

# Qué es

Repo: `~/gits/pollo/ensayos-musicales` (`https://github.com/volteret4/ensayos-musicales.git`). Antes se llamaba `podcast_to_text`.

Pipeline diario, `main.sh`:

1. **Transcribir** MP3 → TXT con **Whisper** en local (`scripts/1_audio_to_text.py`).
2. **Resumir y extraer datos** de cada transcripción: artistas, discos, sellos, géneros y relaciones (influencias, colaboraciones…). Al principio con **Gemini** (`scripts/2_gemini_resumen.py`); ahora con **Claude** (`claude -p`, mismo prompt) por defecto. `RESUMEN=gemini ./main.sh` vuelve a Gemini.
3. Si se acaba la cuota (código 1), el flujo se para.
4. Borrar los MP3 ya transcritos.
5. **Incorporar** solo los resúmenes nuevos a `data/` y `pendiente/`, aplicando las decisiones de `correcciones/decisiones_artistas.txt`.
6. Premios, charts y listas para los artistas que aún no los tienen.
7. Preparar `LOTES_DIARIOS` lotes de `LOTE_TAM` artistas para revisar (no llama a Claude; se revisan con `/revisar-lotes` o desde el editor).

```bash
LOTES_DIARIOS=2 LOTE_TAM=25 ./main.sh
```

# Editor y mapa

- **Editor web** (`Dockerfile.editor`, `ensayos_editor.service`): para revisar y corregir artistas y relaciones, con una pestaña 🤖 *Lotes*.
- **Mapa** (`Dockerfile.map`, d3.js): grafo de influencias. Los nodos se pueden arrastrar.
- La base de datos normalizada es `music_facts.db`. También la usa **encarta** (la enciclopedia que cruza ensayos, Music_Books y rym_lastfm).

Los dos se publican en pepecono detrás de SWAG (`ensayos_editor.subdomain.conf`, `ensayos_map.subdomain.conf`).

# Revertir

```bash
ssh pepecono 'sudo systemctl disable --now ensayos_editor'
```

Y borra los dos `proxy-confs` de SWAG. Los datos están en `data/` y `db/` del repo; haz una copia antes de tocar `music_facts.db`.

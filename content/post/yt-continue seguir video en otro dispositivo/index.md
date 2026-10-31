---
title : "yt-continue, seguir un vídeo de YouTube en otro dispositivo"
slug : "yt-continue-seguir-video-en-otro-dispositivo"
date : "2026-10-31"
image : ""
tags : ["youtube", "flask", "docker", "tampermonkey", "tasker", "powershell"]
categories : ["scripts"]
description : "Un servidor mínimo en casa que guarda la última URL de YouTube con su minuto, un botón de Tampermonkey para enviarla y clientes para Linux, Windows y Android."
---

# La idea

YouTube ya sabe generar un enlace con el minuto (`https://youtu.be/ID?t=123`). Solo falta un sitio donde dejarlo y de donde recogerlo. Ese sitio es un servicio Flask en pepecono que guarda **una sola URL**.

| Pieza | Dónde |
|---|---|
| Servidor | `pepecono:~/contenedores/yt-continue/` (Docker, puerto **5050**) |
| Linux | `~/.local/bin/yt-continue enviar\|recibir` |
| Firefox (Arch y Windows) | script de Tampermonkey con un botón "Enviar video" |
| Windows | `C:\Users\hulio\yt-continue\` con `yt-continue.ps1`, `yt-enviar.bat` y `yt-recibir.bat` |
| Android | perfil y tarea de Tasker con *HTTP Request* |

API:

- `POST /continue` con `{"url": "..."}` guarda la URL.
- `GET /continue` devuelve `{"url": "...", "updated_at": "..."}`.

# Servidor

`docker-compose.yml`:

```yaml
services:
  web:
    build: .
    ports:
      - "5050:5050"
    volumes:
      - yt_continue_data:/data
    restart: unless-stopped

volumes:
  yt_continue_data:
```

`Dockerfile`:

```dockerfile
FROM python:3.12-slim
WORKDIR /app
RUN pip install --no-cache-dir flask
COPY app.py .
EXPOSE 5050
CMD ["python", "app.py"]
```

`app.py`:

```python
import json
import os
import time

from flask import Flask, jsonify, request

app = Flask(__name__)

STATE_PATH = "/data/state.json"


def load_state():
    if not os.path.exists(STATE_PATH):
        return {"url": None, "updated_at": None}
    with open(STATE_PATH, encoding="utf-8") as f:
        return json.load(f)


def save_state(url):
    state = {"url": url, "updated_at": time.strftime("%Y-%m-%dT%H:%M:%S")}
    with open(STATE_PATH, "w", encoding="utf-8") as f:
        json.dump(state, f)
    return state


@app.route("/continue", methods=["GET"])
def get_continue():
    return jsonify(load_state())


@app.route("/continue", methods=["POST"])
def post_continue():
    data = request.get_json(force=True, silent=True) or {}
    url = data.get("url", "").strip()
    if not url:
        return jsonify({"error": "missing url"}), 400
    return jsonify(save_state(url))


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5050)
```

```bash
ssh pepecono 'cd ~/contenedores/yt-continue && docker compose up -d --build'
```

Prueba:

```bash
curl -s -X POST http://192.168.1.133:5050/continue -H "Content-Type: application/json" -d '{"url":"https://youtu.be/dQw4w9WgXcQ?t=42"}'; curl -s http://192.168.1.133:5050/continue
```

No tiene autenticación: solo es accesible desde la LAN.

# Linux: `~/.local/bin/yt-continue`

```bash
#!/usr/bin/env bash
set -euo pipefail

SERVER="http://192.168.1.133:5050/continue"

case "${1:-}" in
  enviar|push|send)
    url="$(xclip -selection clipboard -o)"
    if [[ "$url" != *youtu* ]]; then
      echo "El portapapeles no parece una URL de YouTube: $url" >&2
      exit 1
    fi
    curl -s -X POST "$SERVER" -H "Content-Type: application/json" -d "{\"url\":\"$url\"}" > /dev/null
    echo "Enviado: $url"
    ;;
  recibir|pull|receive)
    url="$(curl -s "$SERVER" | python3 -c 'import json,sys; print(json.load(sys.stdin)["url"] or "")')"
    if [[ -z "$url" ]]; then
      echo "No hay ningún vídeo guardado todavía." >&2
      exit 1
    fi
    echo "Abriendo: $url"
    xdg-open "$url"
    ;;
  *)
    echo "Uso: yt-continue enviar|recibir" >&2
    exit 1
    ;;
esac
```

En YouTube: clic derecho en el reproductor → *Copiar URL en el momento actual*, y luego `yt-continue enviar`. En el otro equipo, `yt-continue recibir`.

# Firefox: botón con Tampermonkey

Añade un botón rojo flotante "Enviar video" en `youtube.com/watch*`. Construye `https://youtu.be/ID?t=SEGUNDOS` a partir del `currentTime` del `<video>`, así que no hace falta copiar nada. Se instala igual en Arch y en Windows: Tampermonkey → *Crear un nuevo script* → pegar → Ctrl+S.

```javascript
// ==UserScript==
// @name         YT Continue - Enviar video
// @namespace    yt-continue
// @version      1.0
// @description  Boton flotante en YouTube que envia el video actual (con el minuto exacto) al servidor yt-continue de casa, para retomarlo en otro dispositivo.
// @match        https://www.youtube.com/watch*
// @grant        GM_xmlhttpRequest
// @connect      192.168.1.133
// ==/UserScript==

(function () {
    "use strict";

    const SERVER = "http://192.168.1.133:5050/continue";

    function buildUrl() {
        const params = new URLSearchParams(window.location.search);
        const videoId = params.get("v");
        const video = document.querySelector("video");
        const t = video ? Math.floor(video.currentTime) : 0;
        return `https://youtu.be/${videoId}?t=${t}`;
    }

    function sendVideo(btn) {
        const url = buildUrl();
        GM_xmlhttpRequest({
            method: "POST",
            url: SERVER,
            headers: { "Content-Type": "application/json" },
            data: JSON.stringify({ url }),
            onload: function () {
                btn.textContent = "Enviado";
                setTimeout(() => (btn.textContent = "Enviar video"), 2000);
            },
            onerror: function () {
                btn.textContent = "Error";
                setTimeout(() => (btn.textContent = "Enviar video"), 2000);
            },
        });
    }

    function createButton() {
        if (document.getElementById("yt-continue-btn")) return;
        const btn = document.createElement("button");
        btn.id = "yt-continue-btn";
        btn.textContent = "Enviar video";
        Object.assign(btn.style, {
            position: "fixed",
            bottom: "20px",
            right: "20px",
            zIndex: "9999",
            padding: "10px 16px",
            background: "#cc0000",
            color: "#fff",
            border: "none",
            borderRadius: "20px",
            fontSize: "14px",
            fontWeight: "bold",
            cursor: "pointer",
            boxShadow: "0 2px 8px rgba(0,0,0,0.3)",
        });
        btn.addEventListener("click", () => sendVideo(btn));
        document.body.appendChild(btn);
    }

    createButton();
    new MutationObserver(createButton).observe(document.body, { childList: true, subtree: true });
    document.addEventListener("yt-navigate-finish", createButton);
})();
```

`@connect 192.168.1.133` es obligatorio para que `GM_xmlhttpRequest` pueda hablar con un servidor de otro dominio.

# Windows

`C:\Users\hulio\yt-continue\yt-continue.ps1`:

```powershell
param([Parameter(Mandatory=$true)][ValidateSet("enviar","recibir")][string]$Accion)
$Server = "http://192.168.1.133:5050/continue"
if ($Accion -eq "enviar") {
    $url = Get-Clipboard
    if ($url -notmatch "youtu") { Write-Error "El portapapeles no parece una URL de YouTube: $url"; exit 1 }
    $body = @{ url = $url } | ConvertTo-Json
    Invoke-RestMethod -Uri $Server -Method Post -Body $body -ContentType "application/json" | Out-Null
    Write-Host "Enviado: $url"
} else {
    $resp = Invoke-RestMethod -Uri $Server -Method Get
    if ([string]::IsNullOrEmpty($resp.url)) { Write-Error "No hay ningun video guardado todavia."; exit 1 }
    Write-Host "Abriendo: $($resp.url)"
    Start-Process $resp.url
}
```

`yt-enviar.bat` (`yt-recibir.bat` es igual, con `-Accion recibir`):

```bat
@echo off
powershell -NoProfile -ExecutionPolicy Bypass -File "%~dp0yt-continue.ps1" -Accion enviar
timeout /t 3 >nul
```

## Los `.bat` se abrían con Notepad++

`.bat` apuntaba al ProgID `Notepad++_file` en lugar de al nativo `batfile`. No había `UserChoice` de por medio, así que bastó con:

```bat
assoc .bat=batfile
```

Para comprobarlo, `assoc .bat` debe mostrar `.bat=batfile` y `ftype batfile`, `"%1" %*`. Para revertir: `assoc .bat=Notepad++_file`.

# Android (Tasker, sin root)

**Enviar**: perfil *Evento → Sistema → Intent recibido* con acción `android.intent.action.SEND`. En YouTube: *Compartir* con la casilla *Empezar en…* marcada → Tasker. Tarea asociada:

- *HTTP Request*: `POST` a `http://192.168.1.133:5050/continue`, cabecera `Content-Type: application/json`, cuerpo `{"url":"%estext"}`.

**Recibir**: tarea `YT Recibir` en un widget:

1. *HTTP Request*: `GET http://192.168.1.133:5050/continue` (el resultado queda en `%http_data`).
2. *JSON Read* del campo `url` en `%video_url`.
3. *Browse URL* `%video_url`, que abre la app de YouTube en ese minuto.

# Revertir

```bash
ssh pepecono 'cd ~/contenedores/yt-continue && docker compose down -v && cd .. && rm -r yt-continue'
```

```bash
rm ~/.local/bin/yt-continue
```

Además, borra el script de Tampermonkey en cada Firefox, la carpeta `C:\Users\hulio\yt-continue\` y el script del Escritorio de Windows, y el perfil y la tarea de Tasker.

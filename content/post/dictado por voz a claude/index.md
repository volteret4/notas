---
title : "Dictar preguntas a Claude con nerd-dictation y la cuenta Pro"
slug : "dictado-por-voz-a-claude"
date : "2026-10-31"
image : ""
tags : ["nerd-dictation", "vosk", "claude", "tkinter", "awesomewm", "python"]
categories : ["scripts"]
description : "Una ventana flotante que escribe lo que dictas con nerd-dictation y se lo manda a claude -p con la suscripción, sin API ni gasto por tokens. Muestra la respuesta en markdown."
---

# Qué hace

Pulsas una tecla, hablas, la vuelves a pulsar y la pregunta va a Claude. La respuesta aparece formateada en la misma ventana.

1. **Primera pulsación**: se abre una ventana flotante y centrada y empieza el dictado. El texto aparece en vivo en la caja de abajo, y puedes corregirlo con el teclado antes de enviarlo.
2. **Segunda pulsación**: para el dictado y envía la consulta. La respuesta se va mostrando arriba mientras se genera.
3. **Siguientes pulsaciones**: otra pregunta en la misma conversación.

| Atajo en la ventana | Acción |
|---|---|
| Ctrl+Enter | enviar |
| Ctrl+D | empezar o parar el dictado |
| Ctrl+N | conversación nueva |
| Esc | cerrar |

# Archivos

| Archivo | Qué es |
|---|---|
| `~/Scripts/utilities/aliases/claude-dictado.sh` | lanzador: abre la ventana o, si ya está abierta, le manda `SIGUSR1` para alternar dictar/enviar |
| `~/Scripts/utilities/aliases/claude_dictado.py` | la ventana (Tkinter) |
| `~/.config/awesome/rc.lua` | atajo y regla de ventana flotante |
| `/tmp/claude-dictado.pid` y `/tmp/claude-dictado.log` | PID de la ventana y errores de nerd-dictation |

Lanzador:

```bash
PIDFILE="/tmp/claude-dictado.pid"

if [ -f "$PIDFILE" ] && kill -0 "$(cat "$PIDFILE")" 2>/dev/null; then
    kill -USR1 "$(cat "$PIDFILE")"
else
    exec python3 "$(dirname "$(readlink -f "$0")")/claude_dictado.py"
fi
```

# Dictado

nerd-dictation se lanza con **su propia cookie**, para no chocar con el dictado normal (`nerd-dictation.sh`), y escribe a **stdout** en lugar de teclear con xdotool. Así el texto va a la ventana y no a la aplicación que tenga el foco:

```python
cmd = [VENV_PYTHON, NERD_DICTATION, "begin", "--cookie", COOKIE,
       "--vosk-model-dir", VOSK_MODEL, "--simulate-input-tool", "STDOUT",
       "--full-sentence", "--numbers-as-digits", "--numbers-min-value", "10"]
```

Si pulsas el atajo mientras el modelo todavía carga, la cookie aún no existe y `end` no haría nada. La ventana espera a que aparezca antes de pararlo.

El modelo por defecto es `~/.config/nerd-dictation/model` (el pequeño de español). Para probar el grande:

```bash
unzip ~/gits/nerd-dictation/vosk-model-es-0.42.zip -d ~/.config/nerd-dictation/
```

```bash
CLAUDE_DICTADO_MODEL=~/.config/nerd-dictation/vosk-model-es-0.42 ~/Scripts/utilities/aliases/claude-dictado.sh
```

# Claude con la cuenta Pro, sin API

```python
cmd = [CLAUDE, "-p", prompt, "--output-format", "stream-json", "--verbose",
       "--include-partial-messages", "--no-session-persistence",
       "--append-system-prompt", SYSTEM_PROMPT,
       "--allowedTools", "WebSearch", "WebFetch"]
```

- Antes de lanzarlo se quitan del entorno `ANTHROPIC_API_KEY`, `ANTHROPIC_AUTH_TOKEN`, `CLAUDE_CODE_USE_BEDROCK` y `CLAUDE_CODE_USE_VERTEX`. Así siempre usa la sesión de claude.ai y nunca cobra por la API. Cuenta para el límite de uso del plan.
- `--allowedTools WebSearch WebFetch`: puede buscar en la web, pero no tocar archivos ni ejecutar comandos.
- `--no-session-persistence`: no guarda nada en disco. El contexto se mantiene **solo en memoria**: en cada pregunta se reenvía la conversación anterior. Al cerrar la ventana o pulsar Ctrl+N, se pierde.
- `stream-json` con `--include-partial-messages` permite ir pintando la respuesta según llega.

## Fallo: no respondía

awesome lanza los programas **sin `~/.local/bin` en el `PATH`**, y `claude` está ahí. Por eso se usa `shutil.which("claude") or ~/.local/bin/claude`, y si `claude` no arranca, el error se muestra en rojo en la conversación.

## Markdown

La respuesta se muestra en un panel de solo lectura, con un pequeño renderizador propio: títulos, negrita, cursiva, código en línea y en bloque, listas anidadas, citas, separadores, enlaces que se pueden pulsar y tablas en monoespaciada.

# Atajo en awesome

Primero estaba en `Mod+Shift+#194`. Ahora va en la **tecla cuadrado** (keycode 193), sin modificador:

```lua
    awful.key({}, "#193",
        function() awful.spawn.with_shell("${HOME}/Scripts/utilities/aliases/claude-dictado.sh &") end,
        { description = "Tecla cuadrado: dictar consulta a Claude (ventana flotante)", group = "APPS" }),
```

Regla de ventana:

```lua
    {
        rule = { class = "ClaudeDictado" },
        properties = {
            floating = true,
            ontop = true,
            sticky = true,
            focus = true,
            placement = awful.placement.centered
        }
    },
```

# Revertir

```bash
rm ~/Scripts/utilities/aliases/claude-dictado.sh ~/Scripts/utilities/aliases/claude_dictado.py
```

Quita del `rc.lua` el `awful.key({}, "#193", ...)` y la regla `ClaudeDictado`. Hay copias en `~/.config/awesome/rc.lua.bak.20261007*`. La única carpeta que deja Claude Code es `~/.claude/projects/-home-huan--cache-claude-dictado/`, y se puede borrar.

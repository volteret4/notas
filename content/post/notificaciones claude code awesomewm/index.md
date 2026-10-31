---
title : "Notificaciones de Claude Code en awesomewm, grandes y en los dos monitores"
slug : "notificaciones-claude-code-awesomewm"
date : "2026-10-31"
image : ""
tags : ["claude", "awesomewm", "naughty", "yad", "zellij", "hooks", "notificaciones"]
categories : ["arch"]
description : "Hooks de Claude Code que abren un aviso grande y centrado en cada monitor cuando necesita respuesta, con botón para saltar al panel de zellij, y un aviso discreto con color por proyecto cuando termina."
---

# Qué quería

Con `dunstify` no me enteraba cuando Claude Code necesitaba mi respuesta. Quería:

- Cuando **necesita input** (permiso, una pregunta…): un aviso **grande y centrado en los dos monitores**, con un botón para ir al panel donde está Claude.
- Cuando **termina** o se queda inactivo: un aviso discreto en la esquina, sin molestar.
- Distinguir de un vistazo de qué proyecto viene cada aviso.

# Lo que descubrí: awesome controla el bus

Cualquier llamada a `notify-send` o `dunstify` la estaba atendiendo **naughty**, la librería de notificaciones de awesome, y no dunst. awesome se registra como servidor de `org.freedesktop.Notifications` al arrancar, así que los estilos se configuran en `rc.lua` con naughty.

```bash
busctl --user status org.freedesktop.Notifications | grep -E "^(PID|Comm)"
```

# Archivos

| Archivo | Para qué |
|---|---|
| `~/.claude/settings.json` | sección `hooks.Notification`: qué script se lanza para cada tipo de aviso |
| `~/.claude/scripts/notify-both-monitors.sh` | aviso grande con yad en cada monitor |
| `~/.claude/scripts/notify-complete.sh` | "Tarea completada" en la esquina (`ClaudeCode:<proyecto>`) |
| `~/.claude/scripts/notify-idle.sh` | "Inactivo" en la esquina (`ClaudeIdle:<proyecto>`) |
| `~/.config/awesome/rc.lua` | colores y bordes con naughty, `claude_popup()` y reglas de ventana `ClaudeCodeNotify1/2` |

Hacen falta `yad`, `xdotool`, `jq` y `awesome-client`.

```bash
sudo pacman -S yad xdotool jq
```

# Hooks de Claude Code

En `~/.claude/settings.json`:

```json
"hooks": {
  "Notification": [
    { "matcher": "agent_completed",
      "hooks": [{ "type": "command", "command": "~/.claude/scripts/notify-complete.sh", "async": true }] },
    { "matcher": "idle_prompt",
      "hooks": [{ "type": "command", "command": "~/.claude/scripts/notify-idle.sh", "async": true }] },
    { "matcher": "permission_prompt|auth_success|elicitation_dialog|elicitation_url_dialog|elicitation_complete|elicitation_response|agent_needs_input|quota_auto_resume_fired|quota_auto_resume_stale|quota_auto_resume_disabled",
      "hooks": [{ "type": "command", "command": "~/.claude/scripts/notify-both-monitors.sh", "async": true }] }
  ]
}
```

# Aviso discreto: borde de color por proyecto

`notify-complete.sh` y `notify-idle.sh` solo hacen un `notify-send` con un `--app-name` especial:

```bash
notify-send \
    --app-name="ClaudeIdle:${project}" \
    --icon=dialog-information \
    --expire-time=6000 \
    -- "Inactivo" "$message"
```

En `rc.lua`, un `naughty.config.notify_callback` reconoce ese `appname`, pone el proyecto en el título y elige el borde:

- **Inactivo**: siempre `#d97757`, el naranja de Anthropic.
- **Completado**: un color de la paleta Catppuccin, calculado con un hash del nombre del proyecto, para que cada proyecto tenga siempre el mismo.

```lua
    local claude_project_colors = { "#89B4FA", "#A6E3A1", "#F9E2AF", "#F5C2E7", "#94E2D5", "#FAB387", "#EBA0AC" }
    local claude_idle_color = "#d97757" -- naranja/arcilla de marca de Anthropic
    naughty.config.notify_callback = function(args)
        local appname = args.appname or ""
        local idle_project = appname:match("^ClaudeIdle:(.+)$")
        local completed_project = appname:match("^ClaudeCode:(.+)$")
        local project = idle_project or completed_project
        if project then
            args.title = project .. (args.title and (": " .. args.title) or "")
            if idle_project then
                args.border_color = claude_idle_color
            else
                local hash = 0
                for i = 1, #project do
                    hash = (hash * 31 + project:byte(i)) % 9973
                end
                args.border_color = claude_project_colors[(hash % #claude_project_colors) + 1]
            end
            args.border_width = 3
        end
        return args
    end
```

# Aviso grande: `notify-both-monitors.sh`

Al principio el aviso grande también era de naughty. En octubre lo cambié por ventanas de **yad**, una por monitor. Lo que hace:

1. El hook devuelve el control al momento. El diálogo lo lleva una copia del script separada con `setsid -f "$0" --popup`, para que Claude Code no lo mate al limpiar el hook.
2. Lee los monitores con `xrandr --listmonitors` y abre un `yad` de 560×280 centrado en cada uno, con `--class=ClaudeCodeNotify` y `--name=ClaudeCodeNotify1`, `2`…
3. Las ventanas forman **un grupo**: si cierras una (botón, Esc o a los 12 s), se cierran todas.
4. Si enfocas el terminal donde está esa sesión de Claude, se cierran solas. Un bucle mira `xdotool getactivewindow` cada 0,3 s.
5. El botón **"Ir al panel"** enfoca el terminal (`c:jump_to()` por `awesome-client`) y, dentro de zellij, cambia a la pestaña y el panel de Claude (`zellij action go-to-tab-by-id` y `focus-pane-id`).
6. Mientras está abierto, llama a `claude_popup(key, true)` en awesome, que descarta los avisos de esquina repetidos (el de kitty y los `Claude*`).

Las reglas de `rc.lua` fijan cada ventana a su pantalla, para que la regla genérica de `yad` no las ponga las dos en el mismo monitor:

```lua
    {
        rule = { instance = "ClaudeCodeNotify1" },
        properties = {
            floating = true, ontop = true, above = true, sticky = true,
            skip_taskbar = true, focus = false, screen = 1
        }
    },
    {
        rule = { instance = "ClaudeCodeNotify2" },
        properties = {
            floating = true, ontop = true, above = true, sticky = true,
            skip_taskbar = true, focus = false, screen = 2
        }
    },
```

# Probar sin esperar a Claude

```bash
echo '{"title":"Prueba","message":"Necesito permiso"}' | ~/.claude/scripts/notify-both-monitors.sh
```

```bash
echo '{"cwd":"/tmp/proyecto","message":"Hecho"}' | ~/.claude/scripts/notify-idle.sh
```

# Revertir

Copias de seguridad:

- `~/.claude/scripts/notify-both-monitors.sh.bak.20260826234452` (versión de agosto) y `.bak.20261023022317` (justo antes de pasar a yad)
- `~/.config/awesome/rc.lua.bak.20260826180503` y `rc.lua.bak.20261023022317`

Para volver a la versión anterior del aviso grande:

```bash
cp ~/.claude/scripts/notify-both-monitors.sh.bak.20261023022317 ~/.claude/scripts/notify-both-monitors.sh
```

Para quitarlo todo: borra la sección `"Notification"` de `hooks` en `~/.claude/settings.json`, los tres scripts y los dos bloques `do ... end` de "Claude Code" del principio de `rc.lua`, más las dos reglas `ClaudeCodeNotify`.

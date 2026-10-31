---
title : "Avisos de GitHub en el escritorio con botones que funcionan"
slug : "notificaciones-de-github-en-el-escritorio"
date : "2026-10-31"
image : ""
tags : ["github", "gh", "cron", "dunst", "dbus", "notificaciones"]
categories : ["scripts"]
description : "Un cron que consulta tus notificaciones de GitHub con gh y muestra un aviso con dos botones. Como dunstify no dice qué botón pulsaste, se escucha la señal de D-Bus."
---

# Qué hace

Cada 5 minutos, `~/Scripts/utilities/notificaciones/github_notif.sh`:

1. consulta `gh api /notifications`,
2. si hay alguna sin leer, muestra un aviso con el repositorio, el título, el tipo, el motivo y cuántas quedan sin leer,
3. ofrece dos botones: **Abrir Notificaciones** (abre `github.com/notifications`) y **Ver Repositorio**.

Necesita `gh` con sesión iniciada (`gh auth login`), `jq`, `gawk` y `dbus-monitor`.

# Cron

```
*/5 * * * * /home/huan/Scripts/utilities/notificaciones/github_notif.sh >> /home/huan/Scripts/.content/logs/cron/github_notif.log 2>&1
```

Cron no tiene acceso a tu sesión gráfica, así que el script lo exporta él mismo:

```bash
export DISPLAY=:0
export DBUS_SESSION_BUS_ADDRESS="unix:path=/run/user/$(id -u)/bus"
```

# Por qué los botones no hacían nada

Con `dunstify -A accion,Texto -w`, dunstify espera a que se cierre el aviso, pero **solo te dice cómo se cerró** (expirado, descartado, cerrado), no **qué botón** se pulsó. Por eso los enlaces no se abrían.

La solución es lanzar el aviso sin bloquear (`-p` devuelve su ID) y escuchar en segundo plano la señal `ActionInvoked` de D-Bus, que sí lleva el ID del aviso y el nombre de la acción:

```bash
notif_id=$(dunstify -p -u normal -t 15000 \
    "🐙 GitHub: $repository" \
    "$message" \
    -A "open,Abrir Notificaciones" \
    -A "default,Ver Repositorio" \
    -h string:category:github 2>/dev/null)

(
    timeout 16 dbus-monitor --session \
        "type='signal',interface='org.freedesktop.Notifications',member='ActionInvoked'" 2>/dev/null \
    | gawk -v target="$notif_id" '
        /member=ActionInvoked/ { state = 1; next }
        state == 1 && /uint32/ { gsub(/[^0-9]/, "", $0); id = $0; state = 2; next }
        state == 2 && /string/ {
            if (match($0, /"([^"]*)"/, arr) && id == target) { print arr[1]; fflush(); exit }
            state = 0
        }
    ' \
    | { read -r action
        case "$action" in
            open) xdg-open "https://github.com/notifications" ;;
            default) xdg-open "$repo_url" ;;
        esac
      }
) &
```

El `timeout 16` es un segundo más que la duración del aviso (`-t 15000`), para que no queden procesos colgados.

En `~/.config/dunst/dunstrc` también cambié el clic izquierdo para que ejecute la acción por defecto antes de cerrar:

```diff
-    mouse_left_click = close_current
+    mouse_left_click = do_action, close_current
```

Hoy el bus de notificaciones lo atiende awesome (naughty) y no dunst, pero el script funciona igual porque la señal `ActionInvoked` es estándar.

# Probar

```bash
~/Scripts/utilities/notificaciones/github_notif.sh debug
```

# Revertir

Quita la línea del `crontab -e`. Para dejar el aviso sin botones, sustituye el bloque de `dunstify` y el subshell de `dbus-monitor` por un simple `notify-send`. Lo de dunstrc se deshace con:

```bash
chezmoi apply ~/.config/dunst/dunstrc
```

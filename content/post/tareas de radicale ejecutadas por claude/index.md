---
title : "Tareas de Radicale que ejecuta Claude con el cupo semanal que sobra"
slug : "tareas-radicale-ejecutadas-por-claude"
date : "2026-11-12"
image : ""
tags : ["radicale", "caldav", "vdirsyncer", "awesomewm", "claude code", "cron", "age", "telegram"]
categories : ["scripts"]
description : "Los atajos para añadir tareas a Radicale llevaban meses rotos porque vdirsyncer apuntaba al servidor antiguo. Los arreglé, añadí un calendario «claude» y un runner en pepecono que, al final de cada semana, gasta el cupo de Claude que no he usado ejecutando esas tareas."
---

# Idea

Apunto ideas y tareas en un calendario de Radicale. Muchas son cosas que Claude Code podría hacer solo, y casi todas las semanas me sobra cupo del plan cuando se reinicia el límite semanal. La idea:

- Un calendario `claude` en Radicale, con tareas VTODO. El título es el nombre de la tarea y la descripción es el prompt.
- Un cron en el servidor (`pepecono`) que, en los últimos días antes del reinicio, ejecuta esas tareas con `claude -p` y reparte lo que queda de cupo.
- Al terminar cada ejecución: sube el progreso de la tarea, añade un resumen a la descripción y avisa por Telegram.
- Cada tarea tiene su carpeta con un `CLAUDE.md`, para poder retomarla a mano.

Antes había que arreglar lo básico: los atajos de awesome para añadir tareas.

# Síntoma

`Super+Alt+F5` (`~/Scripts/snippets/tareas/crear_tarea_caldav.py`) y `Super+Ctrl+F5` (`crear_evento_caldav.sh`) ya no subían nada a Radicale. A mano, `vdirsyncer sync` daba esto:

```
critical: Empty value for password.fetch, this most likely indicates an error.
```

# Causa

Eran varias cosas:

1. **IP vieja.** Todos los pares de `~/.config/vdirsyncer/config` apuntaban a `192.168.1.191`, que ya no existe. Radicale está ahora en Docker en pepecono (`192.168.1.133:5232`). La copia local de Tareas no se sincronizaba desde agosto de 2025.
2. **Contraseña de discos.** El par `discos_calendar` sacaba la contraseña con `pass show radicale`, pero en mi `pass` `radicale` es una carpeta (`pw`, `user`, `discos_id`), no una entrada, así que devolvía el árbol. Además usaba `--pinentry-mode loopback`, que sin terminal devuelve vacío.
3. **Rutas del script antiguo.** `crear_tarea_caldav.py` llamaba a `~/Scripts/tareas/...`, que se movió a `~/Scripts/snippets/tareas/`. Además ejecutaba un `.pl` con python y leía el JSON para taskwarrior de un directorio distinto al que lo escribía.

# Cambios

## vdirsyncer

Copia de seguridad antes de tocar nada:

```bash
cp -a ~/.config/vdirsyncer ~/.config/vdirsyncer.bak.$(date +%Y%m%d%H%M%S)
```

Cambio de IP:

```bash
sed -i 's#http://192.168.1.191:5232/#http://192.168.1.133:5232/#g' ~/.config/vdirsyncer/config
```

Todos los pares usan ya la misma forma de sacar la contraseña:

```ini
password.fetch = ["shell", "echo $(pass show radicale/pw)"]
```

En local y en el servidor había tareas distintas: 30 en común, 251 solo en local y 54 solo en el servidor. Decidí que mandaba el servidor. Para eso borré el estado de vdirsyncer y los `.ics` locales de Tareas y discos (siguen en el backup) y volví a sincronizar:

```bash
vdirsyncer discover && vdirsyncer sync
```

## Calendario `claude` en Radicale

Un calendario de Radicale es solo un directorio con un `.Radicale.props`. Lo creé dentro del contenedor con `docker exec -u 2999 radicale`. Hace falta ese uid porque los ficheros de Radicale le pertenecen, y como root daba `Permission denied`. El directorio es `/data/collections/collection-root/pollo/claude/` y su `.Radicale.props` contiene esto:

```json
{"C:supported-calendar-component-set": "VTODO", "D:displayname": "claude", "ICAL:calendar-color": "#d97757ff", "tag": "VCALENDAR"}
```

Y su par en vdirsyncer, sin descubrimiento de colecciones:

```ini
[pair claude_calendar]
a = "claude_calendar_local"
b = "claude_calendar_remote"
collections = null
conflict_resolution = "b wins"
metadata = ["displayname", "color"]

[storage claude_calendar_local]
type = "filesystem"
path = "/home/huan/.config/vdirsyncer/calendars/claude/"
fileext = ".ics"

[storage claude_calendar_remote]
type = "caldav"
url = "http://192.168.1.133:5232/pollo/claude/"
username = "pollo"
password.fetch = ["shell", "echo $(pass show radicale/pw)"]
```

## Atajos: `bin/tarea`

Todo vive en un repo privado, `tareas_claude`. `bin/tarea` es un script en Python con `icalendar` que escribe el `.ics` en la copia local de vdirsyncer y sincroniza solo ese par. La interfaz es rofi para una línea y zenity para el texto largo:

- `tarea nueva --lista claude|Tareas`: título, descripción (o prompt) y, solo en Tareas, fecha límite opcional.
- `tarea editar`: elige tarea y cambia título, descripción, progreso o fecha; también completar, reabrir o borrar.
- `tarea listar`: salida de texto.

Al editar actualiza `LAST-MODIFIED` y `DTSTAMP` y sube `SEQUENCE`, para que los clientes CalDAV vean el cambio.

Los atajos van en `~/.config/awesome/keys_tareas.lua`:

```lua
local awful = require("awful")

local TAREA = os.getenv("HOME") .. "/gits/pollo/tareas_claude/bin/tarea"

local keys = awful.util.table.join(
    awful.key({ "Mod4" }, "plus",
        function() awful.spawn({ TAREA, "nueva", "--lista", "claude" }) end,
        { description = "nueva tarea para Claude", group = "Tareas" }),
    awful.key({ "Mod4", "Mod1" }, "plus",
        function() awful.spawn({ TAREA, "nueva", "--lista", "Tareas" }) end,
        { description = "nueva tarea (Tareas)", group = "Tareas" }),
    awful.key({ "Mod4", "Shift" }, "plus",
        function() awful.spawn({ TAREA, "editar" }) end,
        { description = "editar tarea (Radicale)", group = "Tareas" })
)

return { keys = keys }
```

En `rc.lua`:

```lua
local tareas = require("keys_tareas")
globalkeys = awful.util.table.join(globalkeys, tareas.keys)
```

| Atajo | Qué hace |
|---|---|
| `Super` + `+` | Tarea para Claude: título y prompt, sin fecha |
| `Super` + `Alt` + `+` | Tarea normal: título, descripción y fecha opcional |
| `Super` + `Shift` + `+` | Editar una tarea de cualquiera de las dos listas |

## El runner en pepecono

`bin/runner` corre en pepecono cada hora y habla CalDAV directamente con Radicale (`REPORT` para leer, `PUT` con `If-Match` para escribir). No usa vdirsyncer ni `pass`.

### Cuánto cupo queda

El reinicio semanal no cae siempre a la misma hora. El runner lo lee del mismo endpoint que usa `/usage` en Claude Code, con el token de `~/.claude/.credentials.json`:

```
GET https://api.anthropic.com/api/oauth/usage
Authorization: Bearer <accessToken>
anthropic-beta: oauth-2025-04-20
```

De la respuesta usa `seven_day.utilization`, `seven_day.resets_at` y `five_hour.utilization`. La primera prueba falló con `401 Invalid bearer token` porque la sesión de Claude en pepecono había caducado en agosto. Se arregló iniciando sesión otra vez:

```bash
ssh -t pepecono '~/.local/bin/claude /login'
```

### Cuándo ejecuta

- **Más de 48 h antes del reinicio:** nada.
- **Entre 48 h y 10 h antes:** traza una recta desde el uso semanal que vio la primera vez hasta el 97 % en el último lote. Si el uso va por debajo, ejecuta una o dos tareas según el coste medio por tarea, que va aprendiendo.
- **Último lote, 10 h antes del reinicio:** tareas hasta llegar al 97 %.
- **Siempre:** si la ventana de 5 h pasa del 80 %, espera, para no quitarme cupo si estoy usando Claude.

Para ver qué haría sin ejecutar nada:

```bash
ssh pepecono '~/Scripts/python_venv/bin/python ~/gits/pollo/tareas_claude/bin/runner --estado'
```

```
uso semana 71.0% · 5 h 21.0% · reinicio Thu 12 09:00 · pendientes 1 → 0 (último lote ya pasado (faltan 7.5 h))
```

### Cada tarea

1. Crea `tareas/<nombre>/CLAUDE.md` con el prompt. El nombre de la carpeta se guarda en una propiedad `X-TAREAS-CLAUDE-DIR` del VTODO, así que no cambia si edito el título.
2. Ejecuta `claude -p --permission-mode auto --output-format json` en esa carpeta. El prompt le pide leer y actualizar el `CLAUDE.md` y terminar con dos líneas: `PROGRESO: N` y `RESUMEN: ...`.
3. Actualiza `PERCENT-COMPLETE` y `STATUS` y añade `[fecha] N% — resumen` a la descripción, debajo de una marca `── Ejecuciones (runner) ──`. Lo de encima de la marca sigue siendo el prompt original.
4. Manda un mensaje con el bot RUNNER de Telegram.

Uso `CLAUDE.md` en mayúsculas porque es el nombre que Claude Code carga solo. Para retomar una tarea a mano basta con abrir `claude` en su carpeta.

### Secretos con age

La contraseña de Radicale y el token de Telegram van en `secretos.env.age`, cifrados con la clave pública de `~/.config/age/keys.txt` de pepecono. El token del bot RUNNER sale del fichero que ya usa `tg_run`. La contraseña se pasa desde el escritorio, sin escribirla:

```bash
pass show radicale/pw | head -n1 | ssh pepecono '~/gits/pollo/tareas_claude/bin/secretos crear'
```

`bin/secretos claves` comprueba que están todos sin mostrar los valores.

### Cron

Va envuelto en `tg_run -e`, que solo avisa por Telegram si el runner falla:

```
5 * * * * /home/pepe/Scripts/utilities/notificaciones/tg_run -n tareas-claude -i 🤖 -b RUNNER -e -- /home/pepe/Scripts/python_venv/bin/python /home/pepe/gits/pollo/tareas_claude/bin/runner
```

Lo probé con el entorno mínimo de cron y termina con código 0:

```bash
ssh pepecono 'env -i HOME=/home/pepe PATH=/usr/bin:/bin /home/pepe/Scripts/python_venv/bin/python /home/pepe/gits/pollo/tareas_claude/bin/runner'
```

Todavía no ha ejecutado ninguna tarea de verdad: la primera vez que lo miré ya había pasado la hora del último lote de esa semana.

Posts relacionados: [Notificaciones de Claude Code en awesomewm](/p/notificaciones-claude-code-awesomewm/) y [Una VM para Claude Code](/p/vm-para-claude-code/).

# Deshacer

Quitar el cron en pepecono (la copia del crontab anterior está en `~/crontab.bak.<fecha>`):

```bash
ssh pepecono 'crontab -l | grep -v tareas_claude/bin/runner | crontab -'
```

Volver a la configuración de vdirsyncer anterior (la carpeta entera, con calendarios y estado):

```bash
rm -rf ~/.config/vdirsyncer && cp -a ~/.config/vdirsyncer.bak.20261111235334 ~/.config/vdirsyncer
```

Quitar los atajos: borra `~/.config/awesome/keys_tareas.lua` y las dos líneas de `tareas` en `rc.lua`, o recupera `~/.config/awesome/rc.lua.bak.20261111235621`.

Script antiguo:

```bash
cp ~/Scripts/snippets/tareas/crear_tarea_caldav.py.bak.20261111235428 ~/Scripts/snippets/tareas/crear_tarea_caldav.py
```

Borrar el calendario `claude` de Radicale:

```bash
ssh pepecono 'docker exec -u 2999 radicale rm -r /data/collections/collection-root/pollo/claude'
```

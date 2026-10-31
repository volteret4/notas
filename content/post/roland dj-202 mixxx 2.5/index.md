---
title : "Roland DJ-202 en Mixxx 2.5, botones y perillas de efectos"
slug : "roland-dj-202-mixxx-2.5"
date : "2026-10-31"
image : ""
tags : ["mixxx", "midi", "javascript", "roland", "dj"]
categories : ["apps"]
description : "El script del controlador Roland DJ-202 fallaba en Mixxx 2.4 o superior con \"this.flash is not a function\". La causa es que los temporizadores ya no conservan this."
---

# El error

```
Uncaught exception: file:///home/huan/.mixxx/controllers/Roland_DJ-202-scripts.js:651:
TypeError: Property 'flash' of object [object Object] is not a function
```

Saltaba sobre todo al mantener pulsado un botón de efecto, que es lo que hace parpadear el LED.

# La causa

Desde Mixxx **2.4** (tengo la 2.5.6), `engine.beginTimer` ya no ejecuta el callback con el `this` del componente cuando se le pasa una `function() { ... }`. Dentro del temporizador, `this` dejaba de ser el botón y `this.flash` no existía.

# El arreglo

En `~/.mixxx/controllers/Roland_DJ-202-scripts.js`, las **9** llamadas a `engine.beginTimer` pasan de `function` a **función flecha**, que mantiene el `this` de fuera:

```diff
-        this.timer = engine.beginTimer(250, function() {
+        this.timer = engine.beginTimer(250, () => {
             this.flash();
         });
```

Solo la línea 651 daba error, pero las otras tenían el mismo problema y fallaban sin avisar: pulsación larga en Sync, doble toque, cambio de deck, key +/−, etc.

Comprobación de sintaxis:

```bash
node --check ~/.mixxx/controllers/Roland_DJ-202-scripts.js
```

# Las perillas de efectos no hacían nada

En `~/.mixxx/mixxx.log` salía:

```
Script tried setting (, focused_effect) to NaN
```

La perilla solo controla el efecto **seleccionado**, y para seleccionarlo hay que **mantener pulsado** su botón. Esa pulsación larga estaba rota por el mismo fallo de `this`, así que nunca había un efecto seleccionado.

Además del arreglo anterior, cambié el comportamiento:

| Situación | Antes | Ahora |
|---|---|---|
| Perilla sin efecto seleccionado | nada | mueve la perilla principal de la unidad (`super1`), que actúa sobre los tres efectos |
| Perilla con un efecto seleccionado | ese efecto | igual |
| Shift + perilla | mezcla dry/wet | igual |

Puede que la perilla no responda hasta que pase por la posición que muestra Mixxx en pantalla. Es la "recogida suave" del script, para que el valor no pegue saltos.

Si sigue sin responder, mira si aparece algo nuevo en `~/.mixxx/mixxx.log` al girarla. También puede que el DJ-202 mande la perilla por otro número MIDI distinto de `0xB8/0xB9 0x00/0x03`; se comprueba con el asistente de aprendizaje MIDI de Mixxx.

Para aplicar los cambios: reinicia Mixxx o desactiva y vuelve a activar el controlador en *Preferencias → Controladores*.

# Revertir

| Copia | Estado |
|---|---|
| `Roland_DJ-202-scripts.js.bak` | antes de cambiar los temporizadores |
| `Roland_DJ-202-scripts.js.bak2` | antes de cambiar las perillas |

```bash
cp ~/.mixxx/controllers/Roland_DJ-202-scripts.js.bak ~/.mixxx/controllers/Roland_DJ-202-scripts.js
```

El mapeo `Roland_DJ-202.midi.xml` también está en chezmoi (`dot_mixxx/controllers/`).

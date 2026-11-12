---
title : "Snippet de scripts en CopyQ con búsqueda por nombre"
slug : "snippets-scripts-copyq"
date : "2026-11-12"
image : ""
tags : ["copyq", "snippets", "javascript", "atajos"]
categories : ["apps"]
description : "El comando Snippets de CopyQ (Super+Alt+Q) ahora busca lo que escribes dentro de las opciones de cada placeholder y, si la opción es «nombre:comando», pega solo el comando. Así tengo un snippet Scripts donde escribo «mix» y se pega la ruta del script."
---

# Idea

Uso el comando **Snippets** de CopyQ con el atajo `Super+Alt+Q`: sale un diálogo para elegir un elemento de la pestaña `Snippets` y, si tiene placeholders `${nombre:opción1,opción2}`, un segundo diálogo para rellenarlos con un desplegable.

Quería un snippet **Scripts** con mis comandos largos: escribir `mix` o `vvmm`, pulsar Enter y que se pegue el comando completo.

# Síntoma

1. El elemento estaba escrito como `${vvmm:...},${mix:...}`. Eso son **dos placeholders**, o sea, dos campos en el formulario, no una lista para elegir.
2. El desplegable de CopyQ solo completa por el principio del texto. Si escribo `mix`, no encuentra `~/Scripts/.../mover_mix_playerctl.py`.
3. Si en las opciones pongo una etiqueta (`mix:~/Scripts/...`), se pega también el `mix:`.

# Cambios

Copia de seguridad de los comandos:

```bash
cp ~/.config/copyq/copyq-commands.ini ~/.config/copyq/copyq-commands.ini.bak-$(date +%F)
```

## El snippet

Un solo placeholder `script` con opciones `nombre:comando` separadas por comas:

```
${script:vvmm:/home/huan/Scripts/python_venv/bin/python3 $HOME/Scripts/Musica/blog/vvmm/main.py --no-preview,mix:~/Scripts/Musica/MoverMusica/mover_mix_playerctl.py}
```

En la nota del elemento pone `Scripts`, que es el nombre que sale en el primer diálogo.

## El comando Snippets

En la función `askToAssignPlaceholders`, justo después de `var values = dialog.apply(this, dialogVars) || abort()`, añadí este bloque:

```js
  // Busqueda: si lo escrito no es una opcion exacta, elegir la primera que lo contenga
  for (var pname in placeholders) {
    var opts = placeholders[pname].split(',')
    if (opts.length < 2) continue
    var isObj = (values instanceof Object)
    var v = str(isObj ? values[pname] : values)
    var low = v.trim().toLowerCase()
    if (low && opts.indexOf(v) == -1) {
      for (var k = 0; k < opts.length; ++k) {
        if (opts[k].toLowerCase().indexOf(low) != -1) { v = opts[k].trim(); break }
      }
    }
    // 'nombre:comando' -> pegar solo el comando
    var lab = /^\s*([\w.-]+):(?!\/\/)(.+)$/.exec(v)
    if (lab) v = lab[2].trim()
    if (isObj) values[pname] = v; else values = v
  }
```

- Si lo escrito no coincide exactamente con una opción, se queda con la primera que lo contiene, sin distinguir mayúsculas. Sirve para todos los snippets con listas (mail, dominio, docker…).
- Si la opción empieza por `nombre:`, lo quita antes de pegar. Las URLs (`https://...`) no se tocan, por el `(?!\/\/)`. Comprobado: `mix:~/a.py` da `~/a.py`, y `https://x.com` y `docker ps` se quedan como están.
- Con un solo campo, `dialog()` devuelve el valor directamente y no un objeto. Por eso está lo de `isObj`.

Los cambios los hice desde la terminal con la API de CopyQ (`commands()` / `setCommands()` y `change()` para el elemento), con el script en un fichero:

```bash
copyq eval 'source("/ruta/patch.js")'
```

Ojo: `copyq eval '...'` con el código en línea **interpreta las barras invertidas**. Los `\n` de las cadenas se convierten en saltos de línea reales y las regex pierden sus `\`. Con `source()` el fichero se lee tal cual.

# Uso

`Super+Alt+Q`, escribo `Scripts` y Enter, luego `mix` y Enter: se pega `~/Scripts/Musica/MoverMusica/mover_mix_playerctl.py`.

Para añadir un script nuevo, edito el elemento y pongo `,nombre:comando` antes de la `}` final.

Limitaciones:

- Ningún comando puede contener `,` (separa opciones) ni `}` (cierra el placeholder).
- El nombre solo admite letras, números, `_`, `.` y `-`.
- Si varias opciones contienen lo escrito, gana la primera de la lista.

Otro uso de CopyQ desde scripts: [tabla markdown con enlaces de Odesli](/p/odesli-api-key-y-tabla-markdown/).

# Deshacer

Cerrar CopyQ, restaurar la copia de los comandos y volver a abrirlo:

```bash
copyq exit; cp ~/.config/copyq/copyq-commands.ini.bak-2026-11-11 ~/.config/copyq/copyq-commands.ini; copyq &
```

El elemento `Scripts` de la pestaña Snippets se edita a mano (F2) para dejarlo como estaba.

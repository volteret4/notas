---
title : "ES-DE en Windows, juegos multidisco con .m3u y accesos de Steam y Epic"
slug : "es-de-multidisco-y-accesos-de-steam"
date : "2026-10-31"
image : ""
tags : ["es-de", "emulationstation", "m3u", "psx", "dreamcast", "ps2", "steam", "epic", "windows"]
categories : ["pamplinas"]
description : "Cómo organizar los juegos de varios discos para que ES-DE muestre una sola entrada por juego, y qué hacer con los accesos .url de Steam y Epic."
---

# Multidisco con `.m3u`

Un juego de PS1 o Dreamcast con varios discos aparece en ES-DE como varias entradas, una por `.cue` o `.chd`. La solución es un **`.m3u`** con la lista de discos, que es lo que se lanza. El emulador (RetroArch, Duckstation, Flycast…) cambia de disco desde su menú.

```
Final Fantasy VII (Europe).m3u
```

```
Final Fantasy VII (Europe) (Disc 1).chd
Final Fantasy VII (Europe) (Disc 2).chd
Final Fantasy VII (Europe) (Disc 3).chd
```

Las rutas son **relativas** a la ubicación del `.m3u`. Para generarlos en Linux:

```bash
for g in "$(ls *"(Disc 1)"* | sed 's/ (Disc 1).*//')"; do ls "$g"*"(Disc "* > "$g.m3u"; done
```

# Una sola entrada por juego: carpeta con el nombre del archivo

ES-DE trata como **archivo** una carpeta cuyo nombre termina en una extensión del sistema (*directories interpreted as files*). Si dentro hay un archivo con **el mismo nombre**, lanza ese. Así se agrupan los discos y la carpeta no se ve como tal en la interfaz:

```
psx/
└── Final Fantasy VII (Europe).m3u/          ← carpeta
    ├── Final Fantasy VII (Europe).m3u       ← lo que se lanza
    ├── Final Fantasy VII (Europe) (Disc 1).chd
    ├── Final Fantasy VII (Europe) (Disc 2).chd
    └── Final Fantasy VII (Europe) (Disc 3).chd
```

Apliqué el mismo criterio a **todos** los multidisco (PSX, Dreamcast, PS2…), y en juegos de un disco con varios archivos (`.cue` + `.bin`), la carpeta se llama como el `.cue`.

En ES-DE: *Ajustes de la interfaz → Mostrar archivos ocultos* desactivado y, tras mover cosas, *Opciones → Actualizar lista de juegos*. Los metadatos (`gamelist.xml`) y las imágenes se buscan por la ruta, así que puede que haya que repetir el *scraping* de los juegos que has movido.

Las ROMs pasaron de la unidad Samba de pepecono a **`C:\ROMs`** (*Ajustes → Directorio de ROMs*). Desde disco local, ES-DE arranca y escanea mucho más rápido.

# Accesos `.url` de Steam y Epic

ES-DE puede importar como "juegos" los accesos directos que crean Steam y Epic, que son archivos `.url`:

```ini
[InternetShortcut]
URL=steam://rungameid/1145360
```

```ini
[InternetShortcut]
URL=com.epicgames.launcher://apps/Fortnite?action=launch&silent=true
```

Al lanzarlos, Windows preguntaba con qué programa abrir el `.url`. Que se abran sin preguntar depende de dos cosas:

1. Que `.url` tenga su asociación normal (`InternetShortcut`):

```bat
assoc .url
ftype InternetShortcut
```

2. Que los protocolos `steam://` y `com.epicgames.launcher://` estén registrados. Se registran al reinstalar o reparar los lanzadores, o abriendo una vez un enlace `steam://` desde el navegador y marcando "recordar".

> Nota: la sesión del 22-07 ya no se conserva y hoy no puedo entrar en Windows, así que no he comprobado la asociación ni la configuración de ES-DE.

# Revertir

Mover los archivos de cada carpeta `Nombre.m3u/` al directorio del sistema y borrar los `.m3u`. En ES-DE, volver a apuntar el directorio de ROMs a la unidad de red si hace falta.

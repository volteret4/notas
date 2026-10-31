---
title : "discogs-scrobbler, scrobblear a Last.fm los vinilos de Discogs desde Android"
slug : "android-scrobbler-discogs-lastfm"
date : "2026-10-31"
image : ""
tags : ["android", "kotlin", "discogs", "lastfm", "acoustid", "scrobbling"]
categories : ["apps"]
description : "Una app Android que lee mi colección de Discogs y manda a Last.fm el disco que pongo en el tocadiscos. Y por qué reconocerlo por el micrófono con AcoustID no funciona."
---

# Qué es

Repo: `~/gits/pollo/android_scrobbler` (`https://github.com/volteret4/discogs-scrobbler.git`), en Kotlin con Android Studio.

- Carga la **colección de Discogs** (`CollectionItem`, `Release`, `Track`, `ReleaseContributor`) y la cachea en una base local (Room, `DatabaseModule`).
- En la ficha de un disco, **Scrobble** manda el disco entero a **Last.fm** (`track.scrobble` por cada pista del tracklist).
- Pantallas extra: **escáner** (`ui/screens/scanner`), **búsqueda en MusicBrainz** (`mbzsearch`), **añadir a la colección** de Discogs y **alias** de artista y título, para que el scrobble salga con el nombre que usa Last.fm.
- Muestra los scrobbles recientes (`RecentScrobble`).
- Arquitectura con inyección de dependencias (`di/NetworkModule`, `RepositoryModule`, `DatabaseModule`).

Hace falta un token de Discogs y la API key y el secreto de Last.fm (sesión móvil). Se configuran en la app; no van en el repo.

# AcoustID con el micrófono: no

Intenté reconocer lo que suena en el tocadiscos grabando con el micrófono y consultando **AcoustID** (Chromaprint). **No funciona**: AcoustID compara huellas del **archivo de audio** limpio. El sonido de la sala (ruido, ecualización, reverberación, velocidad del plato) cambia la huella lo bastante para que no coincida casi nunca.

Para reconocer por micrófono hace falta un servicio pensado para eso (Shazam y similares, por ejemplo con `shazamio`, que es lo que uso en tracklist-id y en el atajo `super+º`).

# Compilar

```bash
cd ~/gits/pollo/android_scrobbler && ./gradlew assembleDebug
```

El APK queda en `app/build/outputs/apk/debug/`. `local.properties` apunta al SDK local y no se sube.

# Revertir

Desinstala la app y revoca su acceso en Last.fm (*Ajustes → Aplicaciones*) y el token en Discogs (*Ajustes → Desarrolladores*).

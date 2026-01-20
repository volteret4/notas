---
title : "Casa"
date : "2026-01-20"
image : ""
tags : ["commands", "notificaciones", "oneline", "shell"]
categories : ["oneline"]
description : ""
---


# Wercome

Soy un apasionado de la música con ligeros conocimientos informáticos que me han permitido crear diferentes utilidades para facilitar el descubrimiento de música.

Uso varias apps para consumir música, priorizando funcionalidad y velocidad a la estética

- Servidor [airsonic](https://airsonic.github.io/) para exponer mi biblioteca digital en el formato original del archivo. Junto a koel (probando actualmente) son los dos únicos capaces de manejar una biblioteca de 40.000 canciones
- [Tempo](https://f-droid.org/en/packages/com.cappielloantonio.notquitemy.tempo/): Aplicación android para airsonic
- [deadbeef](https://deadbeef.sourceforge.io/): reproductor local en pc
- 

# Utilidades

## Scripts

### Calendario

![Pasted image 20260120132501.png](/images/Pasted image 20260120132501.png)

Con esta utilidad puedes añadir a muspy:
- Tus artistas seguidos en spotify
- Los artistas de música digital
- Un archivo de texto en el que escribirás un artista por linea

Una vez añadidos, podrás añadir los lanzamientos de dichos artistas a:
- Tu calendario en google
- Añadirlo a un servicio de calendario caldav como radicale
- Crear un archivo `.ics` que importar en tu aplicación de calendario deseada.


### Compartir
Con un hotkey obtengo el enlace del album que esté sonando para compartirlo fácilmente 

Con otro hotkey enlazado a otro script:
1. copio la canción actual a una carpeta de canciones favoritas ordenadas por gusto personal
2. añado dicha canción a las favoritas de lastfm
3. actualizo la playlist de la carpeta destino para luego importar a airsonic o spotify

### Playlists

#### Airsonic
lastfm
m3u

#### Spotify
lastfm
m3u

### Calidad de vida
Tengo varios scripts simples que automatizan o facilitan tareas repetitivas:

#### Hotkey para facilitar búsquedas

 
Tras seleccionar un texto en cualquier app, pulsa uno de los siguientes atajos para buscar el texto seleccionado:


| atajo         | servicio      | comentario                                           |
| ------------- | ------------- | ---------------------------------------------------- |
| super+a       | google        | busca el texto en google                             |
| super+b       | bandcamp      | busca el texto en badncamp                           |
| super+shift+r | rateyourmusic | busca la metadata del disco sonando en rateyourmusic |
| super+shift+y | youtube       | buscala metadata del disco sonando en youtube        |
| super+p       | todo          | Abre una pestaña de cada servicio                    |


Este script abre un pequeña ventana flotante usando el navegador `qutebrowser` mientras que la ventana que tenga el focus no sea un navegador de la lista.


_Necesitas configurar tu los hotkeys, en windows recomendaría encarecidamente [autohotkey](https://autohotkey.com/), en linux hay cientos de maneras, yo uso awesomewm como windows manager con la siguiente configuración_
```sh
awful.key({ modkey }, "y",
	function() awful.spawn.with_shell("$HOME/PATH/TO/SCRIPT/busqueda_en.sh youtube") end,
	{ description = "Buscar en Youtube", group = "Buscar en..." }),
```

## Bot telegram
Conciertos WIP

## Aplicación mfuzz (several wip)
A base de prompts he creado este Frankestein que recopila información (letras, wikipedia, instrumentos usados, articulos en revistas, entrevistas, ...) de mi biblioteca musical y me permite reproducirla rápidamente, manejear playlists en varios servicios, consultar conciertos...

- Repositorio de la app
- Versión dockerizada
- Documentación

## Blogs

### Blog FreshRSS
En FreshRSS tengo varios feeds que aportan música usando youtube, soundcloud o bandcamp y con esta herramienta crea una web en github.io que recopila todos estos embeds y los ofrece organizados por feeds.

### Blog bandcamp
Gracias a los artistas que sigo en bandcamp, recibo en mi correo mails con los nuevos discos que van a sacar. 
Con esta herramienta puedo crear una web similar a la anterior en github.io que recopila los mails de bandcamp y los expone todos reunidos.

### Blog vvmm
En este blog recopilo los discos que me gustan conforme los escucho gracias a un hotkey en mi pc.
Al usar la base de datos creada para `mfuzz` puedo añadir información relacionada con el álbum en cuestión.


# Ingredientes
Uso estas herramientas como base de las mias:

## Apps
- [FreshRss](http://freshrss.github.io/FreshRSS/): Un lector de feeds rss
- [Wallabag](https://wallabag.org/): Servicio para agregar artículos para leer más tarde
- [Airsonic](https://airsonic.github.io/): Mi servidor de música preferido
- [Tempo](https://f-droid.org/en/packages/com.cappielloantonio.notquitemy.tempo/): Aplicación android que conecta a airsonic

## Información
- [last.fm](last.fm): Servicio para guardar un historial de reproducciones
- [listenbrainz](listenbrainz.org): Alternativa de musicbrainz a Last.fm
- [musicbrainz](musicbrainz.org): Mejor base de datos musical existente.
- [discogs](discogs.com): Base de datos de lanzamientos físicos.
- [rateyormusic](rateyormusic.com): Servicio comunitario imprescindible.
- : Provee información de conciertos pasados
- [equipboard](equipboard.com/): Ofrece información sobre el equipo usado por los artistas
- [metacritic](https://www.metacritic.com/): Indice de reviews de discos

## Lanzamientos nuevos
- [muspy](muspy.com): Servicio que detecta nuevos lanzamientos de los artistas que sigues
- [bandcamp](bandcamp.com)
- [spotify](https://open.spotify.com/)

## Conciertos
- [Ticketmaster](https://www.ticketmaster.es/)

## Menciones honorables
- [musicbuttler](https://www.musicbutler.io/): recolector de nuevos lanzamientos
- Servicios \*arr como [lidarr](https://lidarr.audio/) o [jackett](https://github.com/Jackett/Jackett)
- 
- 
---
title : "Construir Koel App Para Android"
date : "2026-01-19"
image : ""
tags : ["android", "música"]
categories : ["APPS"]
description : ""
---


## Requisitos

Exporta estas variables, modifica las rutas a tu gusto

```sh
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/emulator
```

Clona el repositorio

```sh
git clone --recursive https://github.com/koel/player.git
cd player
```

Instala apps necesarias, con Arch sería algo asi

```sh
sudo pacman -S android-tools jdk-openjdk
sudo paru -S flutter
```



Ejecuta android-tools e instala:
- al menos un sdk
- Android SDK Command-line Tools
- NDK (Side by side)

Acepta las licencias de android con `flutter doctor --android-licenses`

Verifica la instalación con `flutter doctor -v` si no tienes instalado chrome es probable que aparezca marcado en rojo, pero no lo necesitas

Instala requisitos de flutter con `flutter pub get`

## Instalación

Ejecuta este commando, tardará un rato

`flutter build apk --debug`

Es possible que te encuentres con errores de versiones, en mi caso NDK, sorteé el mismo modificando el archivo 

`nano PATH/player/android/app/build.gradle`

especificando la versión instalada de NDK

```gradle
android {
    ndkVersion "29.0.14206865"  // Cambia esto para usar tu versión
```

Puedes ver que versiones tienes instaladas con 

```sh
ls -la /home/huan/Android/Sdk/ndk/

# en mi caso buscaba este archivo
ls -la /home/huan/Android/Sdk/ndk/28.2.13676358/source.properties
```

Al finalizar te muestra la ruta donde se ha creado la apk
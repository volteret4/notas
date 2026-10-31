---
title : "Bombillas Tuya y aspirador Xiaomi atados a una cuenta antigua"
slug : "tuya-y-xiaomi-en-home-assistant"
date : "2026-10-31"
image : ""
tags : ["home assistant", "tuya", "xiaomi", "smart life", "domotica"]
categories : ["pamplinas"]
description : "Qué hacer cuando los dispositivos Tuya y Xiaomi siguen vinculados a una cuenta a la que ya no tienes acceso y Home Assistant no los ve."
---

# El problema

Tras reinstalar el móvil, la app de Tuya no reconocía mis bombillas ni el aspirador Xiaomi. Seguramente seguían vinculados a una cuenta antigua a la que ya no tengo acceso, así que Home Assistant (en otro servidor) tampoco podía usarlos.

Un dispositivo Tuya o Xiaomi solo puede estar en **una cuenta a la vez**. Mientras siga en la vieja, la nueva no lo encuentra al buscar.

> Nota: de esta sesión (11-06-2026) solo queda la petición. La transcripción ya no existe, así que esta entrada recoge el procedimiento, no los comandos exactos de ese día.

# Bombillas Tuya

1. **Reset de fábrica** de cada bombilla: encender y apagar 3 veces seguidas (encendido 1–2 s) hasta que parpadee rápido. Eso la saca de la cuenta vieja.
2. Emparejarla en **Smart Life** (o Tuya Smart) con la cuenta nueva. La red tiene que ser de **2,4 GHz**.
3. En Home Assistant: integración oficial **Tuya** → *Añadir* → escanear el **QR** con la app Smart Life (*Yo → icono de escanear*). Ya no hace falta crear un proyecto en la plataforma de desarrolladores de Tuya.

Si no quieres depender de la nube, la alternativa es **LocalTuya** (HACS), que necesita el `local_key` de cada dispositivo (se saca con `tinytuya wizard`).

# Aspirador Xiaomi

1. **Reset del wifi** del robot: mantener pulsados a la vez los botones indicados en el manual (normalmente *Inicio* y *Base*) hasta que avise por voz.
2. Añadirlo en **Xiaomi Home** con la cuenta nueva y la **región** correcta. La región tiene que coincidir con la de Home Assistant.
3. En Home Assistant: integración **Xiaomi Home** (oficial) o **Xiaomi Miot Auto** (HACS), con la cuenta y la región. Para el mapa, *Xiaomi Cloud Map Extractor*.

Si Xiaomi dice que el dispositivo está "vinculado a otra cuenta", hay que pedir la desvinculación al soporte de Xiaomi con una foto de la etiqueta y del ticket de compra.

# Revertir

Borra la integración en *Ajustes → Dispositivos y servicios* y los dispositivos de las apps. No hay archivos que restaurar.

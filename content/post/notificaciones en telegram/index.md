---
title : "Notificaciones En Telegram"
date : "2025-11-12"
image : ""
tags : ["notificaciones", "telegram"]
categories : ["Scripts"]
description : "Crea un alias para recibir notificaciones en telegram para comandos largos"
---


Añade este script como un alias para poder enviar notificaciones a Telegram cuando finalicen commandos de larga duración

```bash
#!/bin/bash

# Configuración de Telegram
BOT_TOKEN="tu_token_de_bot"
CHAT_ID="tu_chat_id"

# Función para enviar mensajes a Telegram
send_telegram_message() {
    MESSAGE="$1"

    curl -s -X POST "https://api.telegram.org/bot$BOT_TOKEN/sendMessage" \
         -d chat_id="$CHAT_ID" \
         -d text="$MESSAGE" \
         -d parse_mode="HTML"

# Verificar si se proporcionó un mensaje
if [ -z "$1" ]; then
    echo "Uso: $0 \"Mensaje a enviar\"" 
    exit 1
fi

# Enviar el mensaje
send_telegram_message "$1"
```

---
title: Sudo Pw Timeout
date: 2025-07-09 16:00
tags:
  - linux
  - sudo
  - contraseña
  - password
  - pw
  - timeout
  - ssh
categories:
  - sudo
description: sudo timeout
---

## ✅ 1. **Configurar el tiempo antes de volver a pedir la contraseña**

Esto lo controla `sudo` mediante la opción `timestamp_timeout`.

### 🔧 ¿Cómo cambiarlo?

Edita o crea un archivo en `/etc/sudoers.d/`, por ejemplo:

`sudo visudo -f /etc/sudoers.d/timeout`

Y agrega esta línea:

`Defaults timestamp_timeout=5`

🕐 Esto significa que **sudo recordará la contraseña durante 5 minutos**. Puedes poner:

- `0`: pide siempre la contraseña.

- `-1`: no la vuelve a pedir durante la sesión.

---

## ✅ 2. **Hacer que sudo se "olvide" cuando cerrás SSH**

Esto se puede hacer de varias formas. La más directa:

### 🔧 Método: usar `pam_exec` para limpiar sudo al cerrar sesión

Edita el archivo PAM de tu shell, por ejemplo para `bash` (esto depende de tu distribución):

`sudo nano /etc/pam.d/sshd`

Agrega esta línea al **final** del archivo:

`session optional pam_exec.so type=close_session /usr/bin/sudo -K`

Esto ejecutará `sudo -K` cuando se cierre una sesión SSH, que **borra el timestamp de sudo**.

---

## 🧪 Alternativa simple: alias en `.bash_logout`

Si no quieres meterte con `pam`, puedes hacer algo más simple:

En tu Raspberry o servidor remoto, edita:

`nano ~/.bash_logout`

Agregá esta línea:

`sudo -K`

Eso hace que, cuando cierras una sesión interactiva (`exit`, cerrar shell), se borre el caché de `sudo`.

> 📌 Esto **no funciona** si usas otro shell que no lee `.bash_logout`, como `zsh`. Para `zsh`, puedes usar `~/.zlogout`.

---

## 🧠 Verificación rápida

Podés comprobar si el caché está activo con:

`sudo -v`

Y ver si está activo con:

`sudo -n true && echo "Tiene caché sudo" || echo "No tiene caché sudo"`

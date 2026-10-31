---
title : "Supersonic pide la contraseña cada vez (keyring y autologin)"
slug : "supersonic-pide-la-contrasena"
date : "2026-10-31"
image : ""
tags : ["supersonic", "gnome-keyring", "keyring", "autologin", "airsonic"]
categories : ["apps"]
description : "Con autologin, PAM no puede desbloquear el llavero y las apps que guardan contraseñas en el Secret Service las piden cada vez. Solución: llavero Default sin contraseña."
---

# El problema

Supersonic (cliente de Airsonic/Subsonic) pedía la contraseña del servidor **cada vez** que lo abría. Si cerraba el diálogo, no conectaba. Parecía que pedía desbloquear gpg.

# La causa

Supersonic no guarda la contraseña en `config.toml`, sino en el **Secret Service** (gnome-keyring), aunque tenga `EnablePasswordStorage = true`.

Lo que había en `~/.local/share/keyrings/`:

- El alias `default` apuntaba al llavero **`Default`**, que estaba cifrado.
- `pam_gnome_keyring` (en SDDM) solo desbloquea al entrar el llavero llamado **`login`**, y solo si su contraseña es la de tu usuario.
- Además había un `login_1` bloqueado que sobraba.
- **Uso autologin**: al no escribir contraseña al entrar, PAM no tiene con qué desbloquear ningún llavero cifrado.

Así que, tras cada reinicio, `Default` arrancaba bloqueado. Supersonic no podía leer ni guardar y volvía a pedir la contraseña.

Para ver qué tiene guardado Supersonic:

```bash
secret-tool search --all service supersonic 2>&1 | grep username
```

# La solución (opción A: llavero sin contraseña)

Con autologin es lo más práctico:

```bash
seahorse
```

En Seahorse: clic derecho en el llavero `Default` → *Cambiar contraseña*, escribe la actual y deja la nueva **en blanco**. Acepta el aviso de que se guardará sin cifrar.

Después, en Supersonic, conéctate una vez a cada servidor marcando la opción de guardar la contraseña.

Puedes comprobar que el llavero ha quedado sin cifrar: el archivo empieza por `[keyring]` en texto plano.

```bash
head -2 ~/.local/share/keyrings/Default.keyring
```

**Contras:** las contraseñas del llavero quedan en texto plano en `~/.local/share/keyrings/` (solo tu usuario puede leerlas). Con autologin, quien entra en tu sesión ya tiene acceso a todo, así que en la práctica se pierde poca seguridad. Con el disco cifrado, aún menos.

La opción B es dejarlo cifrado y escribir la contraseña del llavero una vez por sesión, que es justo lo que quería evitar.

# Revertir

En Seahorse, *Cambiar contraseña* del llavero `Default` y pon una contraseña. O, para que se desbloquee solo, quita el autologin y haz que el llavero `login` tenga la misma contraseña que tu usuario y sea el predeterminado.

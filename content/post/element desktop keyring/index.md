---
title : "Element desktop pide un keyring al arrancar"
slug : "element-desktop-keyring"
date : "2026-10-31"
image : ""
tags : ["element", "keyring", "gnome-keyring", "keepassxc", "electron"]
categories : ["apps"]
description : "Element (Electron) se queja de que no hay keyring y ofrece cancelar o usar cifrado débil. Hay que decirle que use libsecret y tener un Secret Service activo."
---

# El problema

Al abrir `element-desktop` salía un diálogo diciendo que no encontraba un llavero, con dos opciones: *Cancelar* o usar **weak security encryption**.

Hace tiempo intenté dejar gnome-keyring para usar KeePassXC como gestor de contraseñas, y Element se quedó sin un *Secret Service* claro.

# La causa

Element guarda su clave de cifrado local (la de la base de datos de mensajes y la de Seshat, el buscador) con `safeStorage` de Electron. Electron busca un backend: `gnome_libsecret`, `kwallet` o `basic`. Si no encuentra ninguno desbloqueado, ofrece `basic`, que es la *weak encryption*: una clave fija en el código.

# La solución

1. Que haya **un solo** proveedor de `org.freedesktop.secrets`. En mi caso es gnome-keyring. KeePassXC sigue siendo mi gestor de contraseñas, pero con la integración de Secret Service **desactivada** para que no se peleen. Para comprobar quién da el servicio:

```bash
busctl --user list | grep org.freedesktop.secrets
```

2. Decirle a Element qué backend usar. En `~/.config/Element/electron-config.json`:

```json
	"safeStorageBackend": "gnome_libsecret",
	"safeStorageBackendOverride": true
```

Se puede hacer lo mismo lanzándolo con:

```bash
element-desktop --password-store="gnome-libsecret"
```

3. Que el llavero esté desbloqueado al iniciar sesión. Con autologin, ver [Supersonic pide la contraseña cada vez](../supersonic-pide-la-contrasena).

Ese día también reinstalé el paquete (`pacman -S element-desktop`, versión 1.12.23).

# Revertir

Quita las dos líneas `safeStorageBackend*` de `electron-config.json`. Ojo: si cambias de backend, Element no podrá descifrar lo guardado y te pedirá iniciar sesión otra vez y restaurar la clave de seguridad.

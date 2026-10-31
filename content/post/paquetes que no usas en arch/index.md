---
title : "Encontrar los paquetes que no usas en Arch"
slug : "paquetes-que-no-usas-en-arch"
date : "2026-10-31"
image : ""
tags : ["pacman", "arch", "limpieza"]
categories : ["arch"]
description : "Cómo listar lo que instalaste a mano, ver qué no se ha ejecutado en meses y qué quité yo, con el comando para volver a ponerlo."
---

# Listar lo que instalaste tú

`pacman -Qe` lista los paquetes instalados explícitamente, no como dependencias. `-m` muestra solo los que no vienen de los repositorios (AUR):

```bash
pacman -Qeq | wc -l
```

```bash
pacman -Qmq
```

Huérfanos (dependencias que ya nadie necesita):

```bash
pacman -Qdtq
```

# Ver cuáles no usas

No hay un registro de uso, pero se puede mirar **cuándo se ejecutó por última vez** el binario principal de cada paquete (`atime`). Con `relatime`, que es lo normal en ext4, el `atime` se actualiza como mínimo una vez al día.

```bash
for p in $(pacman -Qeq); do b=$(pacman -Qlq $p | grep -m1 '^/usr/bin/.'); [ -n "$b" ] && printf '%s %s\n' "$(stat -c %x "$b" | cut -d' ' -f1)" "$p"; done | sort | head -60
```

Los de arriba son los que llevan más tiempo sin abrirse. Ojo con los que no tienen binario en `/usr/bin` (fuentes, temas, librerías) y con los que se ejecutan como servicio.

Para ver qué instalaste y cuándo:

```bash
expac --timefmt='%F' '%l %n' $(pacman -Qeq) | sort | head -40
```

# Lo que quité (22-07-2026)

Según `/var/log/pacman.log`:

| Tipo | Paquetes |
|---|---|
| Terminales | `terminator`, `xfce4-terminal`, `termite` |
| Navegador | `brave-origin-beta-bin` |
| Mensajería | `pidgin`, `discord`, `fluffychat`, `matterbridge` |
| YouTube | `youtube-dl` (ya está `yt-dlp`), `ytfzf`, `ytui-bin` |
| Backups | `borg`, `restic` (uso Kopia) |
| PDF | `pdfarranger`, `masterpdfeditor-free` |
| Otros | `lf`, `xfce4-artwork` |

Para ver lo que borraste en una fecha concreta:

```bash
grep -E '^\[2026-07-22.*\] removed' /var/log/pacman.log
```

Ojo: si un script o atajo lanzaba algún programa de la lista, deja de funcionar. Me pasó con `termite` en `rc.lua` de awesome, que cambié por alacritty.

```bash
grep -rn "termite\|terminator\|xfce4-terminal" ~/.config/awesome ~/Scripts 2>/dev/null | grep -v "\.bak"
```

# Revertir

Los del repositorio oficial se reinstalan con pacman (o desde `/var/cache/pacman/pkg` si quieres la versión exacta):

```bash
sudo pacman -S terminator xfce4-terminal pidgin discord pdfarranger restic borg lf youtube-dl ytfzf
```

Los de AUR (`termite`, `brave-origin-beta-bin`, `ytui-bin`, `fluffychat`, `masterpdfeditor-free`, `matterbridge`), con tu helper:

```bash
yay -S brave-origin-beta-bin ytui-bin fluffychat masterpdfeditor-free matterbridge
```

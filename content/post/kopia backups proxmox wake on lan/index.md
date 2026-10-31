---
title : "Backups con Kopia hacia un Proxmox que se enciende de noche por Wake-on-LAN"
slug : "kopia-backups-proxmox-wake-on-lan"
date : "2026-10-31"
image : ""
tags : ["kopia", "backups", "proxmox", "wakeonlan", "systemd", "telegram", "flask"]
categories : ["proxmox"]
description : "Este PC y pepecono hacen snapshot cada noche a un servidor Kopia en Proxmox, que se despierta por WOL y se apaga solo. Con resumen semanal por Telegram y un dashboard web."
---

# Esquema

```
02:00  pepecono: cron → wake_proxmox.sh (WOL con reintentos)
02:45  arcolinux y pepecono: kopia-snapshot.timer → snapshot_<host>.sh
       └─ servidor Kopia en el LXC lxc-docker (192.168.1.89:51515) dentro de Proxmox
+15 min  Proxmox: kopia-idle-shutdown-check.sh → se apaga si nadie lo usa
lunes 09:00  arcolinux: kopia-weekly-report.timer → resumen por Telegram
```

Proxmox está en 192.168.1.222. Comparte máquina con Windows: SSH en el puerto **2288** (ver [SSH a Windows](../ssh-a-windows-10-con-claves) para el `HostKeyAlias`).

# Archivos

En arcolinux, `~/Scripts/kopia/`. En pepecono, `/home/pepe/Scripts/kopia/`.

| Archivo | Qué hace |
|---|---|
| `wake_proxmox.sh` | si el puerto 2288 no responde, manda WOL a `74:56:3C:5D:E6:1D` cada 20 s, hasta 45 intentos |
| `snapshot_arcolinux.sh` / `snapshot_pepecono.sh` | despierta Proxmox, **espera a que responda el servidor Kopia** del LXC (el host responde antes que el LXC), prepara lo no trackeado y crea los snapshots |
| `stage_untracked_pollo.sh` | copia a `~/.cache/kopia-untracked-staging/pollo` lo que no está en git dentro de `~/gits/pollo` (bases de datos, `.env`…) |
| `kopia-notify.sh` | envoltorio que avisa por Telegram del resultado |
| `kopia_weekly_report.sh` | por cada fuente, mira si hubo snapshot cada uno de los últimos 7 días y manda el resumen ✅/❌ |
| `dashboard_agent.sh` | punto de entrada forzado (`command=` en `authorized_keys`) para que el dashboard pida acciones sin ejecutar comandos arbitrarios |
| `dump_dbs_pepecono.sh` | volcados de las bases de datos de los contenedores antes del snapshot |
| `dashboard/app.py` (pepecono) | dashboard web |

## Qué se copia

| arcolinux | pepecono |
|---|---|
| `~/.local/share/chezmoi` | `~/.local/share/chezmoi` |
| `~/Scripts` | `~/Scripts` |
| staging de `~/gits/pollo` | staging de `~/gits/pollo` |
| `/etc/fstab`, `/etc/ssh/sshd_config` | `/etc/fstab`, `/etc/ssh/sshd_config`, `/etc/exports` |
| — | `~/hugo/web` (los blogs), `~/contenedores` |

# Timers

`~/.config/systemd/user/kopia-snapshot.timer` (arcolinux):

```ini
[Timer]
OnCalendar=*-*-* 02:45:00
Persistent=false
RandomizedDelaySec=120
```

**`Persistent=false` en arcolinux** es la clave del problema "este PC intenta arrancar Proxmox cada vez que lo enciendo". Con `true`, systemd lanza al arrancar el snapshot que se perdió por la noche, y eso despierta Proxmox. En pepecono, que está siempre encendido, se deja en `true`.

```ini
# kopia-snapshot.service
[Service]
Type=oneshot
ExecStart=%h/Scripts/kopia/kopia-notify.sh "Snapshot arcolinux" -- %h/Scripts/kopia/snapshot_arcolinux.sh
```

`kopia-weekly-report.timer`: `OnCalendar=Mon *-*-* 09:00:00`, `Persistent=true`.

```bash
systemctl --user enable --now kopia-snapshot.timer kopia-weekly-report.timer
```

## Wake-on-LAN

Lo que funcionaba de verdad era el **crontab** de pepecono, no un servicio. Había dos líneas a las 2:30 (la de ansible y la del script). El 31-10-2026 se dejó solo la del script, a las **2:00**, para que Proxmox tenga 45 minutos para arrancar antes de los snapshots de las 2:45. La de ansible queda comentada:

```
# WOL at night
# 30 2 * * * ansible-playbook --vault-password-file=~/.ansible/.vault_pass ~/.ansible/playbooks/tools/wakeonlan.yml
0 2 * * * /home/pepe/Scripts/kopia/kopia-notify.sh "Despertar proxmox" -- /home/pepe/Scripts/kopia/wake_proxmox.sh
```

El playbook de ansible también despertaba al Debian de Kodi (`192.168.1.135`), así que eso ya no pasa por la noche. El crontab anterior está en `~/crontab.bak.20261031` de pepecono. Para restaurarlo:

```bash
ssh pepecono 'crontab ~/crontab.bak.20261031'
```

## Apagado de Proxmox

Al terminar, cada snapshot programa en Proxmox una comprobación a los 15 minutos. Si nadie lo está usando (Jellyfin, etc.), se apaga:

```bash
ssh proxmox "sudo systemd-run --on-active=900 --unit=kopia-idle-shutdown-\$(date +%s) /root/kopia-idle-shutdown-check.sh"
```

# Dashboard

`pepecono:~/Scripts/kopia/dashboard/app.py`, servicio de usuario `kopia-dashboard.service`, en el puerto **8790** con **autenticación básica** (`DASH_USER` / `DASH_PASSWORD`). Muestra cada fuente con su último snapshot, el espacio que ocupa y enlaces a cada instancia de Kopia. Puede lanzar un snapshot remoto a través de `dashboard_agent.sh`.

```bash
ssh pepecono 'systemctl --user status kopia-dashboard --no-pager'
```

Las configuraciones antiguas (servidor Kopia local, Raspberry, dietpi) están en `~/Scripts/kopia/_archived_legacy_configs/`.

# Comprobar

```bash
systemctl --user list-timers | grep kopia
```

```bash
journalctl --user -u kopia-snapshot -n 30 --no-pager
```

```bash
kopia snapshot list --all | tail
```

# Revertir

```bash
systemctl --user disable --now kopia-snapshot.timer kopia-weekly-report.timer
```

```bash
ssh pepecono 'systemctl --user disable --now kopia-snapshot.timer kopia-dashboard.service'
```

Quita las líneas de WOL del `crontab -e` de pepecono si ya no quieres que Proxmox se encienda por la noche, y la línea `command=` de `dashboard_agent.sh` de `~/.ssh/authorized_keys`.

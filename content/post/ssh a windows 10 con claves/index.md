---
title : "SSH a Windows 10 con claves (y Windows en español)"
slug : "ssh-a-windows-10-con-claves"
date : "2026-10-31"
image : ""
tags : ["ssh", "windows", "openssh", "icacls", "powershell", "proxmox"]
categories : ["network"]
description : "OpenSSH Server en Windows 10, acceso con clave para un administrador, el grupo que se llama Administradores y no Administrators, y la huella que cambia porque Proxmox y Windows usan la misma IP."
---

# 1. Instalar OpenSSH Server

En PowerShell como administrador (lo hice desde una sesión VNC):

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
Get-NetFirewallRule -Name *ssh*
```

La instalación crea la regla `OpenSSH-Server-In-TCP` en el firewall. Si no existe:

```powershell
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
```

Si la cuenta no tiene contraseña, SSH no deja entrar con contraseña: pónsela desde la sesión gráfica antes de configurar las claves.

# 2. Claves para un usuario administrador

Si el usuario es **administrador**, Windows **no** lee `C:\Users\<usuario>\.ssh\authorized_keys`, sino `C:\ProgramData\ssh\administrators_authorized_keys`. Ese archivo solo puede tener permisos para SYSTEM y Administradores, o sshd lo ignora.

```powershell
notepad C:\ProgramData\ssh\administrators_authorized_keys
```

Pega la clave pública (una por línea) y deja solo los permisos buenos:

```powershell
icacls C:\ProgramData\ssh\administrators_authorized_keys /inheritance:r /grant "Administradores:F" /grant "SYSTEM:F"
```

## El error de Windows en español

```
icacls ... /grant BUILTIN\Administrators:F
No se efectuó ninguna asignación entre los nombres de cuenta y los identificadores de seguridad
```

En un Windows en español el grupo se llama **`Administradores`**, no `Administrators`. Para no depender del idioma, usa el SID:

```powershell
icacls C:\ProgramData\ssh\administrators_authorized_keys /inheritance:r /grant "*S-1-5-32-544:F" /grant "*S-1-5-18:F"
```

`S-1-5-32-544` es Administradores y `S-1-5-18` es SYSTEM. Comprueba:

```powershell
icacls C:\ProgramData\ssh\administrators_authorized_keys
```

Para cerrar el acceso por contraseña, en `C:\ProgramData\ssh\sshd_config`:

```
PasswordAuthentication no
PubkeyAuthentication yes
```

```powershell
Restart-Service sshd
```

# 3. La huella cambia: Proxmox y Windows en la misma IP

En el mismo PC (192.168.1.222) arranco **o** Proxmox (SSH en el puerto 2288) **o** Windows (puerto 22). Las dos claves de host quedan guardadas para la misma IP en `known_hosts`, y con `StrictHostKeyChecking` salta el aviso de huella cambiada. La solución es `HostKeyAlias`, que guarda cada clave con un nombre distinto:

```
Host = proxmox
	User = huanitox
	Hostname = 192.168.1.222
	Port = 2288
	IdentityFile = ${HOME}/.ssh/keys/proxmox
	IdentitiesOnly=yes
	HostKeyAlias proxmox-192.168.1.222

Host = windows
	Hostname 192.168.1.222
	User hulio
	HostKeyAlias windows-192.168.1.222
	IdentityFile ${HOME}/.ssh/keys/windows
	IdentitiesOnly yes
```

Si ya entraste por la IP sin alias, borra la entrada vieja:

```bash
ssh-keygen -R 192.168.1.222
```

# 4. Desde pepecono no entraba

Desde arcolinux funcionaba, pero desde pepecono salía:

```
kex_exchange_identification: read: Connection reset by peer
```

Y en el log de Windows, `Timeout before authentication`. `nc -v 192.168.1.222 22` sí conectaba. Para depurar:

```powershell
Get-WinEvent -LogName "OpenSSH/Operational" -MaxEvents 20 | Format-List TimeCreated, Message
```

```powershell
Get-NetFirewallRule -DisplayName "*ssh*" -Direction Inbound | Select-Object Name, DisplayName, Enabled, Action, Profile
```

```bash
ssh -vvv windows
```

Se probaron una clave nueva generada en pepecono (añadida a `administrators_authorized_keys` sin quitar la otra), `-o IPQoS=none` y las reglas del firewall.

> Nota: quedó funcionando el 17-07, pero no se conserva la transcripción, así que no sé qué cambio fue el definitivo.

# Revertir

En Windows (PowerShell como administrador):

```powershell
Stop-Service sshd; Set-Service sshd -StartupType Disabled; Remove-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

Borra `C:\ProgramData\ssh\administrators_authorized_keys` y el bloque `Host = windows` de `~/.ssh/config`.

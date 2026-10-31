---
title : "Una VM para Claude Code con libvirt, cloud-init y sshfs"
slug : "vm-para-claude-code"
date : "2026-10-31"
image : ""
tags : ["kvm", "libvirt", "qemu", "cloud-init", "sshfs", "ssh", "zellij"]
categories : ["arch"]
description : "Claude Code corre dentro de una VM Arch sin acceso al resto del sistema. El proyecto se monta por sshfs y se lanza desde un panel de zellij."
---

# Objetivo

Que el panel de Claude de mis layouts de zellij ([layouts por proyecto](../zellij-layouts-por-proyecto)) abra Claude Code **dentro de una VM**, con acceso solo a la carpeta del proyecto y a `~/.claude`, en lugar de a todo mi `$HOME`.

```
host (arcolinux)                         VM claude-vm (Arch cloud image)
─────────────────                        ───────────────────────────────
zellij → vm_claude_pane.sh  ──ssh:2222──▶ ~/bin/mount_and_claude.sh
                                            sshfs host:~/.claude   → ~/.claude
                                            sshfs host:<proyecto>  → ~/projects/<nombre>
         sshd (solo internal-sftp) ◀──────  claude --continue
```

# 1. KVM

```
$ sudo modprobe kvm_amd
modprobe: FATAL: Module kvm_amd not found in directory /lib/modules/7.0.13-arch1-1
```

No es que falte el módulo: ese mismo día había hecho `pacman -Syu` y se había actualizado el kernel. El kernel en marcha ya no tenía su carpeta en `/lib/modules`. **Basta con reiniciar.** Luego se carga solo, sin necesidad de `/etc/modules-load.d/`.

```bash
lsmod | grep kvm
```

# 2. La VM

Es una VM de **sesión de usuario** (`qemu:///session`), sin root ni red puenteada.

| Archivo | Qué es |
|---|---|
| `~/.local/share/libvirt/images/base/Arch-Linux-x86_64-cloudimg.qcow2` | imagen cloud oficial de Arch |
| `~/.local/share/libvirt/images/claude-vm.qcow2` | overlay sobre la imagen base |
| `~/.local/share/libvirt/images/claude-vm-seed.iso` | ISO de cloud-init (user-data y meta-data) |
| `~/Scripts/menus/vm/claude-vm.xml` | definición del dominio: 4 GB de RAM y 4 vCPU |
| `~/Scripts/menus/vm/build_seed.sh` | genera el user-data de cloud-init |
| `~/Scripts/menus/vm/vm_claude_ensure.sh` | define y arranca la VM si hace falta y espera a SSH y cloud-init |
| `~/Scripts/menus/vm/vm_claude_pane.sh` | comando del panel de zellij |

La red es `type='user'` con backend **passt** (`pacman -S passt`) y una redirección del puerto 2222 del host al 22 de la VM:

```xml
    <interface type='user'>
      <model type='virtio'/>
      <backend type='passt'/>
      <portForward proto='tcp'>
        <range start='2222' to='22'/>
      </portForward>
    </interface>
```

## cloud-init

`build_seed.sh` genera un `user-data` que:

- crea el usuario `huan` con la clave pública `vm_claude_host_to_guest.pub`,
- instala `openssh` y `sshfs`,
- deja la clave privada `vm_claude_guest_to_host` y el script `mount_and_claude.sh` en `/root/claude-vm-payload/` y los instala con `runcmd`, porque `write_files` se ejecuta antes de crear el usuario y `/home/huan` todavía no existe,
- instala Claude Code con `curl -fsSL https://claude.ai/install.sh | bash`.

Dentro de la VM, `~/bin/mount_and_claude.sh` monta por sshfs, usando la puerta de enlace de la red user, que es el host:

```bash
HOST_GW=$(ip route show default | awk '{print $3; exit}')
sshfs -o IdentityFile="$IDENTITY",StrictHostKeyChecking=accept-new,reconnect,ServerAliveInterval=15 \
    "$HOST_USER@$HOST_GW:/home/$HOST_USER/.claude" "$HOME/.claude"
sshfs -o IdentityFile="$IDENTITY",StrictHostKeyChecking=accept-new,reconnect,ServerAliveInterval=15 \
    "$HOST_USER@$HOST_GW:$HOST_PROJECT_PATH" "$HOME/projects/$PROJECT_NAME"
cd "$HOME/projects/$PROJECT_NAME"
exec "$HOME/.local/bin/claude" --continue
```

# 3. SSH

## Del host a la VM

En `~/.ssh/config`:

```
Host = claude-vm
	User = huan
	Hostname = 127.0.0.1
	Port = 2222
	IdentityFile = ${HOME}/.ssh/keys/vm_claude_host_to_guest
	IdentitiesOnly = yes
	UserKnownHostsFile = ${HOME}/.ssh/keys/vm_claude_known_hosts
```

## De la VM al host (solo SFTP)

La clave de la VM está en `~/.ssh/authorized_keys` del host, limitada a SFTP: no puede abrir una shell ni redirigir puertos.

```
command="internal-sftp",no-pty,no-X11-forwarding,no-port-forwarding,no-agent-forwarding ssh-ed25519 AAAA... claude-vm-to-host
```

Al principio intenté hacerlo con un drop-in de sshd, `/etc/ssh/sshd_config.d/99-claude-vm-loopback.conf`, con un bloque `Match` para las conexiones por loopback. Falló así:

```
/etc/ssh/sshd_config.d/99-claude-vm-loopback.conf line 2: Directive 'StrictModes' is not allowed within a Match block
```

`StrictModes` es una opción global y no se puede poner dentro de `Match`. Hoy ese drop-in **ya no existe**, y la restricción va solo en `authorized_keys`. Después de tocar sshd, compruébalo siempre antes de recargar:

```bash
sudo sshd -t && sudo systemctl reload sshd
```

# 4. Zellij

`~/.config/zellij/templates/claude_vm_proyecto.kdl` es igual que la plantilla normal, pero el panel de Claude ejecuta:

```kdl
        pane size="40%" {
            command "/home/huan/Scripts/menus/vm/vm_claude_pane.sh"
            args "{{PROJECT_DIR}}"
        }
```

`~/Scripts/menus/rofi/generar_layouts_claude_vm.sh` genera los layouts en `~/.config/zellij/layouts/claude-vm/`, y `zellij_claude_vm_tab.sh` es su rofi.

Error que salió: `vm_claude_pane.sh: línea 7: 1: uso: vm_claude_pane.sh <ruta_del_proyecto>`. El layout no le pasaba `args`.

# Comprobar

```bash
virsh -c qemu:///session list --all
```

```bash
ssh claude-vm 'mount | grep sshfs'
```

# Revertir

```bash
virsh -c qemu:///session destroy claude-vm; virsh -c qemu:///session undefine claude-vm
```

```bash
rm ~/.local/share/libvirt/images/claude-vm.qcow2 ~/.local/share/libvirt/images/claude-vm-seed.iso ~/.ssh/keys/vm_claude_* && rm -r ~/.config/zellij/layouts/claude-vm ~/.config/zellij/templates/claude_vm_proyecto.kdl
```

Y además:

- borra la línea `claude-vm-to-host` de `~/.ssh/authorized_keys`,
- borra el bloque `Host = claude-vm` de `~/.ssh/config`,
- y, si no la usas para otra cosa, `sudo pacman -Rns passt`.

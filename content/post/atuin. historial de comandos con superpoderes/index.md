---
title : "Atuin. Historial De Comandos Con Superpoderes"
date : "2025-12-17"
image : ""
tags : ["commands", "historial", "shell"]
categories : ["APPS"]
description : "Instala y configura atuin para controlar el historial de la consola"
---


## Instala atuin con
```bash
curl --proto '=https' --tlsv1.2 -LsSf https://setup.atuin.sh | sh
```

## Inicialo con tu shell

bash:
```sh
echo 'eval "$(atuin init bash)"' >> ~/.bashrc
```

zsh:
```sh
echo 'eval "$(atuin init zsh)"' >> ~/.zshrc
```

fish:
```sh
atuin init fish | source
```

nushell:
```sh
mkdir ~/.local/share/atuin/
atuin init nu | save ~/.local/share/atuin/init.nu

# Añade esto a config.nu
source ~/.local/share/atuin/init.nu
```

## Carga tu historial anterior

`atuin import auto` Importará el shell actual.
También se puede especificar que shell importar:
```sh
atuin import bash
atuin import zsh # etc
```

## Sincronización
Añade esto a `~/.config/atuin/config.toml`
```toml
sync_address = "http://192.168.1.133:8778"
```
---
title : "Instala Lazyvim En Raspberry"
date : "2025-12-18"
image : ""
tags : ["vim"]
categories : ["APPS"]
description : "Guia para compilar nvim e instalar lazyvim"
---


## Nvim
Es necesario compilarlo asi que primero instala
```bash
sudo apt-get install git cmake
```

Luego clona el repositorio, compila e instala. Tardará un rato.
```bash
git clone https://github.com/neovim/neovim.git
cd neovim
make CMAKE_BUILD_TYPE=RelWithDebInfo
cd build
cpack -G DEB
sudo dpkg -i nvim-linux64.deb
```

Comprueba la versión con 
```bash
nvim --version
```

## Lazyvim
Haz una copia de seguridad si es necesario
```sh
# required
mv ~/.config/nvim{,.bak}

# optional but recommended
mv ~/.local/share/nvim{,.bak}
mv ~/.local/state/nvim{,.bak}
mv ~/.cache/nvim{,.bak}
```

Clona el repo
```sh
git clone https://github.com/LazyVim/starter ~/.config/nvim
```

Elimina la carpeta `.git` para poder usarlo como repositorio luego
```sh
rm -rf ~/.config/nvim/.git
```

### Instala los siguientes paquetes opcionales
```sh
sudo apt install lazygit ripgrep fd-find fzf lua5.4:armhf luarocks # las versiones pueden variar con el tiempo
```

Para manejar imagenes (creo) puedes usar una de estas tres terminales
- kitty
- wezterm
- ghostty



Usa `neovim` y ejecuta el commando `:LazyHealth`para comprobar errores
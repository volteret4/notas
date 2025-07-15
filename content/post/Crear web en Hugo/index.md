---
title: "Crear Web En Hugo"
date: "2025-07-10"
image: ""
tags: ["--baseURL"]
categories: [blog]
description: "Crea un blog estático en github con hugo"
---

## 1.Crear El blog

```sh
hugo new site <nombre del sitio>
cd <nombre del sitio>
git init
```

## 3.Elegir Un tema e instalarlo

<https://themes.gohugo.io/themes/lightbi-hugo/>

```bash
git submodule add https://github.com/binokochumolvarghese/lightbi-hugo themes/lightbi-hugo
echo "theme = 'lightbi-hugo'" >> hugo.toml
hugo server
cp -r "themes/lightbi-hugo/example site/*" $web_dir/content/
```

## 4.Comprueba Que funciona localmente

```bash
hugo serve --bind "0.0.0.0" #--baseURL 192.168.1.14 # ip of server
```

## 5.Crea Repositorio en github terminado en `.github.io`

## 6.Sube El directorio de la web recien creada

```bash
git add . # subira todo el contendio de la carpeta en la que se está
git commit -m "comentario"
git push
```

## 7.Accede A github y ve a `Settings` > `Pages` hasta ver esto

[1;33mImagen no encontrada: /mnt/windows/FTP/wiki/Obsidian/Dibujos/img/Pasted image 20240723144203.png[0m

<!-- Imagen no encontrada: Pasted image 20240723144203.png -->

## 8.Cambia `source` A `Github Actions` (se guarda solo)

[1;33mImagen no encontrada: /mnt/windows/FTP/wiki/Obsidian/Dibujos/img/Pasted image 20240723144218.png[0m

<!-- Imagen no encontrada: Pasted image 20240723144218.png -->

## 9.Crea Un archivo en `.github/workflows/hugo.yaml`

### ¡Importante! Cambiar la versión de hugo a la que esté instalada

```yaml
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches:
      - main

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.128.0
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
          TZ: America/Los_Angeles
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

## 10.Crear Un commit en el repositorio y subirlo

```bash
git add . # subira todo el contendio de la carpeta en la que se está
git commit -m "comentario"
git push
```

## 11.En Github, elije `Actions`

[1;33mImagen no encontrada: /mnt/windows/FTP/wiki/Obsidian/Dibujos/img/Pasted image 20240723144801.png[0m

<!-- Imagen no encontrada: Pasted image 20240723144801.png -->

Una vez terminado el push y la creación del sitio debes ver algo así
[1;33mImagen no encontrada: /mnt/windows/FTP/wiki/Obsidian/Dibujos/img/Pasted image 20240723144903.png[0m

<!-- Imagen no encontrada: Pasted image 20240723144903.png -->

Se veria asi en el commit

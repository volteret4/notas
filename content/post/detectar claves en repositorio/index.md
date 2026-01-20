---
title : "Detectar Claves En Repositorio"
date : "2026-01-20"
image : ""
tags : ["git", "secrets", "shell"]
categories : ["Seguridad"]
description : ""
---


Con la aplicación gitleaks puedes detectar si un repositorio, o directorio expone claves secretas.

Puedes hacerlo de modo manual

```sh
gitleaks -git /path/to/repo
gitleaks -dir /path/to/dir
```

O puedes añadir este archivo `.pre-commit-config.yaml` a la raiz de tu repositorio github, lo que lanzará este commando antes de realizar `git push` lo que evitará que envies credenciales.

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.24.0
    hooks:
      - id: gitleaks
```
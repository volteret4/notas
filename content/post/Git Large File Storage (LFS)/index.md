---
title : "Git Large File Storage (Lfs)"
date : "2026-03-14"
image : ""
tags : ["archivos", "git"]
categories : ["git"]
description : "Como subir archivos de más de 150 mb a github."
---


Git Large File Storage


1. Descargue e instale la extensión de línea de commandos de Git. Una vez descargado e instalado, configure Git LFS para su cuenta de usuario ejecutando:
```sh
	git lfs install
```
	Sólo necesita ejecutar esto una vez por cuenta de usuario.
2. En cada repositorio de Git donde desee utilizar Git LFS, seleccione los tipos de archivos que desea que Git LFS administre (o edite directamente sus atributos .gitat). Puede configurar extensions de archivos adicionales en cualquier memento.
```sh
	git lfs track "\*.psd"
	Now make sure .gitattributes is tracked:
	git add .gitattributes
```
	
	Tenga en cuenta que definir los tipos de archivos que Git LFS debe rastrear no convertirá, por sí solo, ningún archivo preexistente a Git LFS, como archivos en otras ramas o en su historial de confirmaciones anterior. Para hacer eso, use el commando [git lfs migrate(1)](https://github.com/git-lfs/git-lfs/blob/main/docs/man/git-lfs-migrate.adoc?utm_source=gitlfs_site&utm_medium=doc_man_migrate_link&utm_campaign=gitlfs), que tiene una variedad de opciones diseñadas para adaptarse a varios casos de uso potenciales.
	
3. No existe el paso tres. Simplemente comprométete y presiona como lo harías normalmente; por ejemplo, si su sucursal actual se llama `main`:
```sh
	git add file.psd
	git commit -m "Add design file"
	git push origin main
```


---
title : "Megatools"
date : "2026-01-17"
image : ""
tags : ["archivos", "cloud", "mega"]
categories : ["APPS"]
description : ""
---


Con [megatools](https://xff.cz/megatools/man/megatools.html) puedes registrar y subir archivos a mega.nz

```bash
# Registra la cuenta
megatools reg --register --username mail@domain.com --password QWERTY --name NAME
# Continua con el comando propuesto por megatools para validar el registro
```

```bash
# Sube un archivo
megatools put --path /Root/remote/path file.zip file2.csv --username mail@domain.com

# Crea una carpeta
megatools mkdir /Root/Path --username mail@domain.com

# Sube un directorio
megatools copy --source /path/to/dir --remote /Root/path --username mail@domain.com
```


---
title: "Copia De Particiones Con Dd"
date: "2025-07-15"
image: ""
tags: []
categories: [básico]
description: "Crea una copia de seguridad de la estructura de las particiones de un disco"
---

### Usar `sfdisk` para copiar la estructura de particiones

`Sfdisk` es una herramienta de línea de comandos que permite gestionar las particiones de un disco y copiar su tabla de particiones a un archivo. Esta herramienta solo maneja la **estructura de particiones**, no los datos contenidos en ellas.

#### Copiar la estructura de particiones:

1. **Exportar la tabla de particiones a un archivo**: Usando `sfdisk`, puedes copiar la tabla de particiones de un dispositivo (por ejemplo, `/dev/sda`) a un archivo de texto:

```bash
    sudo sfdisk --dump /dev/sda > particiones.txt
```

Esto creará un archivo de texto (`particiones.txt`) que contiene la estructura de particiones del dispositivo. Este archivo incluye la información sobre los tamaños y las ubicaciones de las particiones, pero no los datos dentro de ellas.

#### Restaurar la estructura de particiones en otro dispositivo:

2. **Restaurar la tabla de particiones en otro dispositivo**: Si quieres crear la misma estructura de particiones en otro dispositivo (por ejemplo, `/dev/sdb`), puedes usar el archivo exportado para restaurar la tabla de particiones:

```bash
    sudo sfdisk /dev/sdb < particiones.txt
```
